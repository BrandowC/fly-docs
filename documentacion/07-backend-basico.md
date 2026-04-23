# Tema 7 · Backend básico para aplicaciones móviles

## 1. Stack elegido

- **Framework:** NestJS 11
- **Runtime:** Node.js 20+
- **ORM:** Prisma 7.6
- **Base de datos:** PostgreSQL 16 en Docker
- **Hash de contraseñas:** bcryptjs
- **Transpilación:** TypeScript 5.7
- **Validación:** class-validator + class-transformer

## 2. ¿Por qué NestJS?

| Razón | Detalle |
|---|---|
| Arquitectura modular | Cada dominio (auth, productos, pedidos) en su propio módulo |
| Inversión de dependencias | Inyección automática via decoradores `@Injectable()` y constructor |
| Patrón en capas impuesto | Controller → Service → Repository |
| Validación declarativa | DTOs con `class-validator` + `ValidationPipe` global |
| Ecosistema rico | Integración natural con Prisma, JWT, TypeORM, etc. |
| TypeScript first | Sin hacks para tipos, todo viene tipado |

## 3. Estructura del backend

```
fly-api/ice-cream/
├── prisma/
│   ├── schema.prisma           ← modelos de datos
│   ├── seed.ts                 ← datos iniciales
│   └── migrations/             ← migraciones automáticas
├── src/
│   ├── main.ts                 ← bootstrap (dotenv, CORS, ValidationPipe)
│   ├── app.module.ts           ← módulo raíz que registra otros módulos
│   ├── prisma/
│   │   └── prisma.service.ts   ← singleton que extiende PrismaClient
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts  ← POST /auth/login
│   │   └── auth.service.ts     ← validación bcrypt
│   ├── productos/
│   │   ├── productos.module.ts
│   │   ├── productos.controller.ts
│   │   ├── productos.service.ts
│   │   ├── productos.repository.ts
│   │   └── dto/
│   │       ├── create-producto.dto.ts
│   │       └── update-producto.dto.ts
│   ├── topping/              ← misma estructura
│   ├── pedidos/              ← misma estructura
│   └── admin/
│       ├── admin.module.ts
│       ├── admin.controller.ts
│       ├── admin.service.ts
│       ├── admin.repository.ts
│       └── guards/
│           └── api-key.guard.ts
├── docker-compose.yml
├── .env
├── package.json
├── prisma.config.ts
└── tsconfig.json
```

## 4. Arquitectura en capas

```
                   ┌──────────────────────┐
                   │  HTTP Request (JSON) │
                   └──────────┬───────────┘
                              ▼
              ┌───────────────────────────────┐
              │       VALIDATION PIPE         │  ← valida DTO con class-validator
              └───────────────┬───────────────┘
                              ▼
              ┌───────────────────────────────┐
              │         CONTROLLER            │  ← define rutas, extrae params
              └───────────────┬───────────────┘
                              ▼
              ┌───────────────────────────────┐
              │          SERVICE              │  ← lógica de negocio
              └───────────────┬───────────────┘
                              ▼
              ┌───────────────────────────────┐
              │        REPOSITORY             │  ← operaciones de datos
              └───────────────┬───────────────┘
                              ▼
              ┌───────────────────────────────┐
              │      PRISMA CLIENT            │  ← traduce a SQL
              └───────────────┬───────────────┘
                              ▼
              ┌───────────────────────────────┐
              │       POSTGRESQL              │
              └───────────────────────────────┘
```

## 5. Endpoints REST implementados

### Auth
| Método | Ruta | Cuerpo | Respuesta |
|---|---|---|---|
| POST | `/auth/login` | `{username, password}` | `{success, message, admin}` |

### Productos
| Método | Ruta | Propósito |
|---|---|---|
| GET | `/productos` | Listar todos |
| GET | `/productos?tipo=CREMA` | Filtrar por tipo |
| GET | `/productos/:id` | Uno por id |
| POST | `/productos` | Crear |
| PATCH | `/productos/:id` | Actualizar |
| DELETE | `/productos/:id` | Eliminar |

### Toppings
| Método | Ruta | Propósito |
|---|---|---|
| GET | `/toppings` | Listar todos |
| GET | `/toppings/disponibles` | Solo disponibles |
| GET | `/toppings/:id` | Uno por id |
| POST | `/toppings` | Crear |
| PATCH | `/toppings/:id` | Actualizar |
| DELETE | `/toppings/:id` | Eliminar |

### Pedidos
| Método | Ruta | Propósito |
|---|---|---|
| GET | `/pedidos` | Listar (con detalles + producto + topping) |
| GET | `/pedidos/:id` | Uno por id |
| POST | `/pedidos` | Crear (calcula total desde precios actuales) |
| PATCH | `/pedidos/:id/toggle` | Alternar completado/pendiente |

### Admin (protegido con ApiKeyGuard)
| Método | Ruta | Propósito |
|---|---|---|
| GET | `/admin/dashboard` | Estadísticas |
| GET | `/admin/reporte-pedidos?inicio=&fin=` | Pedidos por fecha |

## 6. Bootstrap de la aplicación

Archivo [`src/main.ts`](../fly-api/ice-cream/src/main.ts):

```typescript
import "dotenv/config";
import { ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { PrismaService } from "./prisma/prisma.service";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableCors();

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );

  const prismaService = app.get(PrismaService);
  await prismaService.enableShutdownHooks(app);

  await app.listen(3000, "0.0.0.0");
}
bootstrap();
```

**Observaciones clave:**
- `dotenv/config` en la primera línea carga variables `.env` antes de instanciar Prisma (bug crítico del que aprendimos)
- `enableCors()` permite que la app Expo/web acceda desde otro origen
- `ValidationPipe` aplica los decoradores de los DTO automáticamente a todas las rutas
- `listen(3000, "0.0.0.0")` expone el servidor a toda la interfaz (no solo localhost), necesario para que el celular pueda conectarse

## 7. Ejemplo completo: POST /auth/login

### Controlador
```typescript
@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post("login")
  async login(@Body() body: { username: string; password: string }) {
    return this.authService.login(body.username, body.password);
  }
}
```

### Servicio
```typescript
@Injectable()
export class AuthService {
  constructor(private readonly prisma: PrismaService) {}

  async login(username: string, password: string) {
    const admin = await this.prisma.admin.findUnique({ where: { username } });

    if (!admin) {
      throw new UnauthorizedException("Usuario o contraseña incorrectos");
    }

    const isPasswordValid = await bcrypt.compare(password, admin.password);
    if (!isPasswordValid) {
      throw new UnauthorizedException("Usuario o contraseña incorrectos");
    }

    return {
      success: true,
      message: "Login exitoso",
      admin: { id: admin.id, username: admin.username },
    };
  }
}
```

### Flujo
1. Cliente envía `POST /auth/login` con `{username, password}`
2. `ValidationPipe` valida el body (omitido aquí por simplicidad)
3. Nest llama `AuthController.login()`
4. Controller delega a `AuthService.login()`
5. Service busca el admin en BD (vía Prisma)
6. Si existe, compara hash con `bcrypt.compare()`
7. Si match, devuelve el objeto de éxito
8. Si no, lanza `UnauthorizedException` → Nest responde 401 automáticamente

## 8. Prisma como ORM

### PrismaService (singleton)

```typescript
@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  constructor() {
    const adapter = new PrismaPg({
      connectionString: process.env.DATABASE_URL!,
    });
    super({ adapter });
  }

  async onModuleInit() {
    await this.$connect();
  }

  async enableShutdownHooks(app: INestApplication) {
    process.on("beforeExit", async () => {
      await app.close();
    });
  }
}
```

Todos los repositorios inyectan este servicio y usan su API:
```typescript
constructor(private readonly prisma: PrismaService) {}
async findAll() { return this.prisma.producto.findMany(); }
```

### Ventajas de Prisma vs escribir SQL

- **Type-safe:** `prisma.producto.findMany()` devuelve `Producto[]` tipado
- **Migraciones automáticas:** cambiar schema.prisma → `migrate dev` genera SQL
- **Autocompletado:** IDE sugiere campos existentes
- **Relaciones:** `include: { detalles: true }` hace el JOIN automáticamente

## 9. Seed de la base de datos

Archivo [`prisma/seed.ts`](../fly-api/ice-cream/prisma/seed.ts) crea:
- 1 admin
- 11 productos (3 tamaños gratis + 8 helados)
- 12 toppings
- 3 pedidos de ejemplo con detalles

Ejecutado con `npx prisma db seed`.

## 10. Errores y excepciones

NestJS convierte excepciones HTTP a respuestas automáticamente:

| Excepción | Status HTTP |
|---|---|
| `UnauthorizedException` | 401 |
| `NotFoundException` | 404 |
| `BadRequestException` | 400 |
| `ForbiddenException` | 403 |
| `ConflictException` | 409 |
| `InternalServerErrorException` | 500 |

## 11. CORS y seguridad mínima

- `app.enableCors()` permite cualquier origen (OK para desarrollo)
- En producción debería restringirse a dominios específicos:
  ```typescript
  app.enableCors({ origin: ["https://miapp.com"] });
  ```
- El backend **no tiene autenticación** en los endpoints CRUD de productos/toppings/pedidos (para mantener el proyecto simple). En producción se debe agregar un Guard JWT.

## 12. Próximos pasos de backend (fuera del alcance actual)

- Agregar JWT + Guards para proteger endpoints CRUD
- Agregar rate-limiting (`@nestjs/throttler`)
- Agregar logs estructurados (pino / winston)
- Tests unitarios con Jest + supertest
- Dockerizar el backend también
- CI/CD con GitHub Actions
