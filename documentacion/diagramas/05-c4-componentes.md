# C4 · Nivel 3 · Diagrama de Componentes (Backend)

El nivel 3 del modelo C4 muestra los **componentes internos** de un contenedor. Aquí se expande el contenedor "Backend NestJS" en sus módulos y servicios.

## Diagrama (Mermaid)

```mermaid
flowchart TB
    App["📱 App Móvil<br/>(contenedor externo)"]

    subgraph Backend["🖥️ Backend NestJS"]
        direction TB

        VP["<b>Validation Pipe (global)</b><br/>class-validator<br/>Valida DTOs"]

        subgraph Auth["🔐 AuthModule"]
            AuthC["AuthController<br/>POST /auth/login"]
            AuthS["AuthService<br/>bcrypt.compare"]
        end

        subgraph Prod["🍦 ProductosModule"]
            ProdC["ProductosController<br/>CRUD /productos"]
            ProdS["ProductosService<br/>Lógica negocio"]
            ProdR["ProductosRepository<br/>prisma.producto"]
        end

        subgraph Top["✨ ToppingsModule"]
            TopC["ToppingsController<br/>CRUD /toppings"]
            TopS["ToppingsService"]
            TopR["ToppingsRepository<br/>prisma.topping"]
        end

        subgraph Ped["📋 PedidosModule"]
            PedC["PedidosController<br/>CRUD /pedidos<br/>PATCH /:id/toggle"]
            PedS["PedidosService<br/>calcula total<br/>orquesta detalles"]
            PedR["PedidosRepository<br/>prisma.pedido + detalle"]
        end

        subgraph AdminM["📊 AdminModule"]
            AdminC["AdminController<br/>/admin/dashboard<br/>/admin/reporte-pedidos"]
            AdminG["ApiKeyGuard<br/>header x-api-key"]
            AdminS["AdminService"]
            AdminR["AdminRepository"]
        end

        Prisma["🗄️ PrismaService<br/>singleton<br/>PrismaClient + PrismaPg"]
    end

    DB[("🐘 PostgreSQL<br/>(contenedor externo)")]

    App -->|HTTPS| VP

    VP --> AuthC
    VP --> ProdC
    VP --> TopC
    VP --> PedC
    VP --> AdminC

    AuthC --> AuthS
    AuthS --> Prisma

    ProdC --> ProdS
    ProdS --> ProdR
    ProdR --> Prisma

    TopC --> TopS
    TopS --> TopR
    TopR --> Prisma

    PedC --> PedS
    PedS --> PedR
    PedS -. "usa para validar IDs" .-> ProdS
    PedS -. "usa para validar IDs" .-> TopS
    PedR --> Prisma

    AdminG -->|protege| AdminC
    AdminC --> AdminS
    AdminS --> AdminR
    AdminR --> Prisma

    Prisma -->|SQL sobre TCP| DB

    classDef module fill:#85C1E9,color:#000,stroke:#3498DB
    classDef component fill:#D6EAF8,color:#000,stroke:#3498DB
    classDef external fill:#999,color:#fff,stroke:#666
    classDef infra fill:#F39C12,color:#fff,stroke:#B9770E

    class App,DB external
    class Prisma infra
    class VP component
```

## Componentes

### Validation Pipe (global)

- **Tecnología:** `@nestjs/common` + `class-validator` + `class-transformer`
- **Responsabilidad:** Validar automáticamente los DTOs en TODAS las rutas (whitelist + forbidNonWhitelisted + transform). Rechaza requests mal formados con HTTP 400.
- **Registro:** `main.ts` via `app.useGlobalPipes(new ValidationPipe({...}))`

### AuthModule

| Componente | Función |
|---|---|
| `AuthController` | Expone POST `/auth/login`, recibe `{username, password}` |
| `AuthService` | Busca admin en BD, compara con bcrypt |

### ProductosModule

| Componente | Función |
|---|---|
| `ProductosController` | Expone CRUD de productos (`GET`, `POST`, `PATCH`, `DELETE`) |
| `ProductosService` | Lógica de negocio, lanza `NotFoundException` si no existe |
| `ProductosRepository` | Queries a `prisma.producto` |

### ToppingsModule

Misma estructura que ProductosModule pero para toppings. También expone GET `/toppings/disponibles` (solo los activos).

### PedidosModule

| Componente | Función |
|---|---|
| `PedidosController` | Expone `POST /pedidos` (crear), `GET /pedidos` (listar), `PATCH /pedidos/:id/toggle` |
| `PedidosService` | Calcula total desde precios actuales, orquesta la creación de detalles. **Depende de** `ProductosService` y `ToppingsService` |
| `PedidosRepository` | Inserta pedido + detalles como transacción anidada |

### AdminModule

| Componente | Función |
|---|---|
| `AdminController` | Expone `/admin/dashboard` y `/admin/reporte-pedidos`. **Protegido por** `ApiKeyGuard` |
| `ApiKeyGuard` | Valida header `x-api-key` |
| `AdminService` | Agregaciones y reportes |
| `AdminRepository` | Queries complejas de estadísticas |

### PrismaService (infraestructura)

- Singleton que extiende `PrismaClient`
- Configura el adapter `PrismaPg` para PostgreSQL
- Gestiona ciclo de vida (`onModuleInit` conecta, `enableShutdownHooks` cierra limpiamente)
- Inyectado en todos los Repositories y algunos Services (Auth)

## Dependencias entre componentes

- **Sentido:** Controllers → Services → Repositories → PrismaService → BD
- **Nunca al revés:** un Repository NO llama a un Service (evita ciclos)
- **Cross-module:** `PedidosService` usa `ProductosService` y `ToppingsService` para validar IDs antes de crear detalles (acoplamiento aceptable porque todo es parte del mismo bounded context "pedido")

## Flujo completo: crear un pedido

```
App ──POST /pedidos──► VP (valida DTO)
                        │
                        ▼
                   PedidosController.create(dto)
                        │
                        ▼
                   PedidosService.crearPedido(dto)
                        │
                        ├──► ProductosService.findOne(productoId)  ─┐
                        │                                            │ por cada item
                        ├──► ToppingsService.findOne(toppingId?)    ─┘
                        │
                        │   (calcula precioProducto + precioTopping = subtotal)
                        │   (suma todos los subtotales = totalGeneral)
                        │
                        ▼
                   PedidosRepository.createPedido(data, detalles)
                        │
                        ▼
                   prisma.pedido.create({
                     data: { ..., detalles: { create: detalles } },
                     include: { detalles: { include: { producto, topping } } }
                   })
                        │
                        ▼
                   PostgreSQL (INSERT Pedido, INSERT PedidoDetalle x N)
                        │
                        ▼
                   Devuelve pedido completo → JSON 201
```
