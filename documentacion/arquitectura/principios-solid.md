# Aplicación de los principios SOLID

SOLID es un acrónimo de cinco principios de diseño orientado a objetos propuestos por Robert C. Martin ("Uncle Bob"). A continuación se documenta cómo cada uno se aplicó en Ice Cream App.

---

## S — Single Responsibility Principle (SRP)

> **"Una clase debe tener una única razón para cambiar."**

### Aplicación en el proyecto

El backend sigue estrictamente SRP con la separación **Controller → Service → Repository**.

#### Ejemplo: módulo `productos`

| Clase | Única responsabilidad |
|---|---|
| `ProductosController` | Traducir HTTP → llamada a service |
| `ProductosService` | Lógica de negocio (validar IDs, orquestar) |
| `ProductosRepository` | Queries Prisma |
| `CreateProductoDto` | Validar el cuerpo del POST |
| `UpdateProductoDto` | Validar el cuerpo del PATCH |

```typescript
// productos.controller.ts — SOLO traduce HTTP
@Controller("productos")
export class ProductosController {
  constructor(private readonly service: ProductosService) {}

  @Post()
  create(@Body() dto: CreateProductoDto) {
    return this.service.create(dto);   // ← delega
  }
}
```

```typescript
// productos.service.ts — SOLO lógica de negocio
@Injectable()
export class ProductosService {
  constructor(private readonly repository: ProductosRepository) {}

  async findOne(id: number) {
    const producto = await this.repository.findById(id);
    if (!producto) throw new NotFoundException(`Producto ${id} no encontrado`);
    return producto;
  }
}
```

```typescript
// productos.repository.ts — SOLO acceso a datos
@Injectable()
export class ProductosRepository {
  constructor(private readonly prisma: PrismaService) {}

  async findById(id: number) {
    return this.prisma.producto.findUnique({ where: { id } });
  }
}
```

Si mañana se cambia de Prisma a TypeORM, **solo cambia el Repository**. Si se agrega una regla de negocio (ej: no permitir precios negativos), **solo cambia el Service**. Si se migra a GraphQL, **solo cambia el Controller**.

### En el frontend

- `services/api.ts` → solo configuración de axios
- `context/CartContext.tsx` → solo estado del carrito
- `app/(tabs)/home.tsx` → solo UI del catálogo
- `app/admin.tsx` → solo UI del panel admin

---

## O — Open/Closed Principle (OCP)

> **"Las entidades deben estar abiertas a extensión pero cerradas a modificación."**

### Aplicación en el proyecto

#### Backend: agregar un módulo nuevo

Agregar un módulo (ej: `Reseñas`) **no requiere modificar** módulos existentes:

```typescript
// app.module.ts
@Module({
  imports: [
    ProductosModule,
    PedidosModule,
    AdminModule,
    ToppingsModule,
    AuthModule,
    // ResenasModule,  ← solo se agrega esta línea
  ],
})
export class AppModule {}
```

Los demás módulos no saben de `ResenasModule`. Abierto a extensión, cerrado a modificación.

#### Frontend: agregar una nueva categoría en el catálogo

El arreglo `CATEGORIAS` en [`home.tsx`](../../fly-app/ice-cream-app/app/(tabs)/home.tsx) está diseñado para extenderse:

```typescript
const CATEGORIAS = [
  { key: "tamano", label: "Tamaño", icon: "beaker", color: "#00BCD4" },
  { key: "helado", label: "Helados", icon: "ice-cream", color: "#FF4D94" },
  { key: "topping", label: "Toppings", icon: "sparkles", color: "#FF9800" },
  // { key: "bebida", label: "Bebidas", icon: "water", color: "#4CAF50" },  ← extensión
];
```

El componente mapea `CATEGORIAS` y se adapta automáticamente. **No necesita modificarse** para aceptar más categorías.

#### Frontend: agregar un nuevo tipo de emoji

La función `getEmoji()` se puede extender con más condicionales sin romper el contrato:

```typescript
const getEmoji = (nombre: string): string => {
  if (n.includes("fresa")) return "🍓";
  // ... agregar más líneas sin tocar las existentes
};
```

---

## L — Liskov Substitution Principle (LSP)

> **"Los objetos de una superclase deben poder sustituirse por objetos de sus subclases sin alterar el comportamiento correcto del programa."**

### Aplicación en el proyecto

#### Uso de interfaces/tipos como contrato

El frontend define `CartItem` como contrato. Cualquier "cosa" que cumpla ese contrato puede ir al carrito:

```typescript
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image?: any;
}
```

Un **helado**, un **topping** y un **tamaño** (grupos totalmente diferentes en el backend) son indistinguibles para el carrito mientras cumplan esta interfaz:

```typescript
addToCart({
  id: `helado-${prod.id}`,
  name: prod.nombre,
  price: prod.precio,
  quantity: 1,
});
```

El `CartContext` no necesita saber si es helado, topping o tamaño. Sustituible.

#### PrismaService extiende PrismaClient

```typescript
@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  // ...
}
```

En cualquier lugar donde se espere un `PrismaClient`, un `PrismaService` funciona igual. Se puede sustituir sin romper nada.

---

## I — Interface Segregation Principle (ISP)

> **"Los clientes no deben verse forzados a depender de interfaces que no usan."**

### Aplicación en el proyecto

#### DTOs específicos por operación

Cada operación CRUD tiene su propio DTO. No se usa un gigantesco `ProductoDto`:

```typescript
// Crear requiere todos los campos
export class CreateProductoDto {
  @IsString() nombre!: string;
  @IsString() tipo!: string;
  @IsNumber() precio!: number;
  @IsOptional() @IsBoolean() disponible?: boolean;
}

// Actualizar permite campos parciales (mapped-types)
export class UpdateProductoDto extends PartialType(CreateProductoDto) {}
```

El consumidor del endpoint PATCH **no está obligado** a mandar todos los campos.

#### CartContext expone solo lo necesario

```typescript
interface CartContextType {
  cart: CartItem[];
  addToCart: (product: CartItem) => void;
  removeFromCart: (id: string) => void;
  increaseQty: (id: string) => void;
  decreaseQty: (id: string) => void;
  clearCart: () => void;
  totalPrice: number;
}
```

Un componente que solo necesita `totalPrice` lo toma con destructuring:
```typescript
const { totalPrice } = useCart();
```

No tiene que conocer ni manipular el resto de la API del contexto.

---

## D — Dependency Inversion Principle (DIP)

> **"Depender de abstracciones, no de implementaciones concretas."**

### Aplicación en el proyecto

#### NestJS con Inyección de Dependencias

Todas las clases del backend declaran sus dependencias por constructor, y Nest las inyecta automáticamente:

```typescript
@Injectable()
export class AuthService {
  constructor(private readonly prisma: PrismaService) {}  // ← abstracción inyectada
  // No sabe ni le importa cómo se crea el PrismaService
}
```

Si mañana se reemplaza `PrismaService` con `MockPrismaService` para tests, **el código no cambia**. Solo cambia el registro en el módulo:

```typescript
@Module({
  providers: [
    AuthService,
    {
      provide: PrismaService,
      useClass: MockPrismaService,  // ← swap en tests
    },
  ],
})
```

#### Frontend: axios como abstracción

Todos los componentes usan el mismo `api` centralizado:

```typescript
import api from "../../services/api";

await api.get("/productos");
```

Si mañana se reemplaza axios por fetch nativo, **solo se cambia `services/api.ts`**, no los 10 componentes que lo usan.

#### Ejemplo concreto de DIP

```
                    ProductosController
                            │
                            │ depende de (abstracción)
                            ▼
                    ProductosService ◄───┐
                            │            │ es inyectable
                            │ depende de │
                            ▼            │
                    ProductosRepository  │
                            │            │
                            │            │
                            ▼            │
                    PrismaService ───────┘
```

Ninguna capa depende de una clase concreta por nombre — todas dependen de **abstracciones inyectadas**.

---

## Resumen tabla

| Principio | Cómo se aplica | Archivos clave |
|---|---|---|
| **S** · SRP | Controller/Service/Repository separados | Módulo `productos/`, `pedidos/`, etc. |
| **O** · OCP | Añadir módulos/categorías sin modificar código existente | `app.module.ts`, `CATEGORIAS` en `home.tsx` |
| **L** · LSP | Tipos/interfaces como contrato de sustitución | `CartItem`, `PrismaService extends PrismaClient` |
| **I** · ISP | DTOs específicos (`Create`/`Update`), context expone solo lo necesario | `dto/*.ts`, `CartContext.tsx` |
| **D** · DIP | Inyección de dependencias de NestJS, axios singleton | Todos los `*.service.ts`, `services/api.ts` |
