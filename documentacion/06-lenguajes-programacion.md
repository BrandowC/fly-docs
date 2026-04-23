# Tema 6 · Lenguajes de programación móviles

## 1. Stack de lenguajes del proyecto

| Capa | Lenguaje | Versión | Uso |
|---|---|---|---|
| Frontend móvil | TypeScript + JSX | TS 5.7 | Lógica y componentes React Native |
| Backend | TypeScript | TS 5.7 | Controladores, servicios, repositorios NestJS |
| Base de datos | SQL (PostgreSQL) | 16 | Queries generadas por Prisma |
| Schema ORM | Prisma Schema Language (PSL) | 7.6 | Definición declarativa de modelos |
| Configuración | YAML / JSON | — | `docker-compose.yml`, `package.json`, `tsconfig.json` |

## 2. TypeScript (lenguaje principal)

### ¿Por qué TypeScript y no JavaScript?

**TypeScript** es JavaScript con tipos estáticos. En este proyecto se eligió porque:

| Ventaja | Ejemplo práctico |
|---|---|
| Detección de errores en tiempo de compilación | Evitó que `precio: string` se sumara como número |
| Autocompletado inteligente en VS Code | Al escribir `cliente.` sugiere `nombre`, `telefono`, etc. |
| Refactors seguros | Renombrar un campo propaga el cambio a todos los archivos |
| Documentación integrada | Las interfaces son contrato ejecutable |

### Ejemplo de tipado del proyecto

```typescript
// Interface usada en el CartContext
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image?: any;
}

// Type literal usado en el admin
type AdminTab = "helados" | "toppings" | "pedidos";
```

### DTOs (Data Transfer Objects)

El backend usa DTOs con validadores de `class-validator`:

```typescript
// fly-api/ice-cream/src/productos/dto/create-producto.dto.ts
export class CreateProductoDto {
  @IsString()
  nombre!: string;

  @IsString()
  tipo!: string;

  @IsNumber()
  @Min(0)
  precio!: number;

  @IsOptional()
  @IsBoolean()
  disponible?: boolean;
}
```

El decorador `@IsNumber()` actúa en runtime (vía `ValidationPipe` global) y rechaza un POST con `precio: "hola"` con HTTP 400 automáticamente.

## 3. JSX / TSX

JSX (JavaScript XML) permite escribir UI declarativa dentro del código JS/TS. En React Native se usa TSX (TypeScript + JSX).

### Ejemplo del proyecto

```tsx
<TouchableOpacity
  style={[styles.tab, isActive && { backgroundColor: cat.color }]}
  onPress={() => setCategoriaActiva(cat.key)}
>
  <Ionicons name={cat.icon} size={22} color={isActive ? "white" : cat.color} />
  <Text style={[styles.tabText, { color: isActive ? "white" : cat.color }]}>
    {cat.label}
  </Text>
</TouchableOpacity>
```

Observaciones:
- `style={[arr1, arr2]}` permite combinar estilos
- `isActive && { ... }` aplica un objeto condicionalmente
- Los eventos llevan prefijo `on` (camelCase)
- Los componentes empiezan con mayúscula (`<TouchableOpacity>`)

## 4. SQL (vía Prisma)

Aunque nunca escribimos SQL manualmente, Prisma genera consultas SQL bajo el capó. Ejemplos de lo que pasa detrás:

### Prisma query → SQL

```typescript
// Código
await prisma.pedido.findMany({
  include: { detalles: { include: { producto: true, topping: true } } },
  orderBy: { createdAt: "desc" },
});
```

Se traduce a SQL similar a:

```sql
SELECT p.*,
       d.id AS d_id, d.cantidad, d.subtotal, d.producto_id, d.topping_id,
       prod.nombre AS prod_nombre, prod.tipo, prod.precio AS prod_precio,
       top.nombre AS top_nombre, top.precio AS top_precio
FROM "Pedido" p
LEFT JOIN "PedidoDetalle" d ON d.pedidoId = p.id
LEFT JOIN "Producto" prod ON prod.id = d.productoId
LEFT JOIN "Topping" top ON top.id = d.toppingId
ORDER BY p."createdAt" DESC;
```

### Migraciones SQL generadas

Prisma genera automáticamente archivos SQL al crear migraciones. Ejemplo (`prisma/migrations/XXX_init_with_admin/migration.sql`):

```sql
CREATE TABLE "Admin" (
    "id" SERIAL NOT NULL,
    "username" TEXT NOT NULL,
    "password" TEXT NOT NULL,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT "Admin_pkey" PRIMARY KEY ("id")
);

CREATE UNIQUE INDEX "Admin_username_key" ON "Admin"("username");
```

## 5. PSL (Prisma Schema Language)

Lenguaje declarativo propio de Prisma. No es SQL, no es TypeScript: es un DSL propio.

Ejemplo:

```prisma
model Pedido {
  id            Int             @id @default(autoincrement())
  clienteNombre String
  telefono      String
  direccion     String
  total         Decimal         @db.Decimal(10,2)
  completado    Boolean         @default(false)
  createdAt     DateTime        @default(now())
  detalles      PedidoDetalle[]
}
```

Sintaxis clave:
- `@id` → llave primaria
- `@default(autoincrement())` → autoincremento
- `@db.Decimal(10,2)` → tipo nativo de la BD
- `PedidoDetalle[]` → relación 1:N

## 6. Comparación con otros lenguajes móviles

| Lenguaje | Pros | Contras | ¿Por qué no en este proyecto? |
|---|---|---|---|
| **Swift** | Nativo iOS, alto rendimiento | Solo iOS | Queríamos soporte Android + iOS |
| **Kotlin** | Nativo Android, moderno | Solo Android | Idem |
| **Java** | Ecosistema Android maduro | Verboso, sin null-safety moderna | Idem |
| **Dart (Flutter)** | Un solo lenguaje, UI propia | Curva de aprendizaje, menos JS share | El equipo ya sabía JS/TS |
| **TypeScript + RN** | ✅ Un solo lenguaje para front+back+móvil | Puente JS ↔ nativo | **La que elegimos** |

## 7. Decoradores (característica clave de TS en este proyecto)

NestJS usa decoradores extensivamente. Son funciones que modifican clases/métodos en tiempo de compilación.

### Ejemplo en el proyecto

```typescript
@Controller("pedidos")               // ← decorador de clase
export class PedidosController {
  constructor(private readonly pedidosService: PedidosService) {}

  @Get()                             // ← decorador de método
  async findAll() {
    return await this.pedidosService.findAll();
  }

  @Patch(":id/toggle")               // ← registra PATCH /pedidos/:id/toggle
  async toggleCompletado(
    @Param("id", ParseIntPipe) id: number,  // ← extrae y castea param
  ) {
    return await this.pedidosService.toggleCompletado(id);
  }
}
```

## 8. Lenguajes auxiliares

### YAML
Configuración legible. Ejemplo: `docker-compose.yml`.

### JSON
Formato de intercambio entre frontend y backend. Ejemplo:
```json
{
  "id": 1,
  "nombre": "Helado de Fresa",
  "tipo": "CREMA",
  "precio": "4500",
  "disponible": true
}
```

### Markdown
Esta documentación está escrita en Markdown. Se combina bien con Mermaid para diagramas embebidos.

## 9. Tipado estricto (configuración de TS)

El proyecto usa `tsconfig.json` con `"strict": true`, lo que habilita:
- `noImplicitAny`
- `strictNullChecks`
- `strictFunctionTypes`
- `alwaysStrict`

Esto previene errores como acceder a una propiedad de `undefined`:

```typescript
// Sin strict: compila pero crashea en runtime
const nombre = producto.nombre;  // si producto es undefined → 💥

// Con strict: NO compila, obliga a verificar
if (producto) {
  const nombre = producto.nombre;  // OK
}
```

## 10. Tablas de equivalencias

### Tipos TS ↔ BD (Prisma)

| TypeScript | Prisma | SQL PostgreSQL |
|---|---|---|
| `number` | `Int` | INTEGER / SERIAL |
| `number` | `Decimal` | DECIMAL / NUMERIC |
| `string` | `String` | TEXT / VARCHAR |
| `boolean` | `Boolean` | BOOLEAN |
| `Date` | `DateTime` | TIMESTAMP |
| `string \| null` | `String?` | TEXT (nullable) |

### Tipos TS ↔ JSON

| TypeScript | JSON |
|---|---|
| `number` | `5`, `4.5` |
| `string` | `"texto"` |
| `boolean` | `true`, `false` |
| `object` | `{ "key": "value" }` |
| `Array<T>` | `[ ... ]` |
| `null` | `null` |
| `undefined` | (no serializa) |
