# ADR-004 · Prisma ORM

- **Fecha:** 2026-02-15
- **Estado:** Aceptado
- **Decisor:** Brandow Stivent

## Contexto

El backend necesita persistir datos en PostgreSQL. Las opciones son:

- Escribir SQL directamente (pg driver)
- Usar un Query Builder (Knex, Kysely)
- Usar un ORM (Prisma, TypeORM, Sequelize, MikroORM)

## Decisión

Se adopta **Prisma 7.6** como ORM y herramienta de migraciones.

## Alternativas consideradas

| Alternativa | Pros | Contras | Veredicto |
|---|---|---|---|
| **SQL raw (pg)** | Control total, sin abstracción | Boilerplate enorme, sin type safety | ❌ |
| **Knex / Kysely** | SQL flexible con typesafety parcial | Sin migraciones automáticas ni modelo declarativo | ❌ |
| **TypeORM** | Maduro, usado en Nest oficialmente | Decoradores verbosos, docs inconsistentes | ❌ |
| **Sequelize** | Popular | No es TS first, API vieja | ❌ |
| **Prisma** | Schema declarativo, type-safe 100%, migraciones automáticas, Prisma Studio | Menos flexible para queries muy custom | ✅ Elegido |

## Justificación

1. **Type safety completo:** `prisma.producto.findMany()` retorna `Producto[]` tipado automáticamente.
2. **Schema declarativo:** un solo archivo `schema.prisma` define modelos, relaciones y tipos DB.
3. **Migraciones automáticas:** cambiar el schema + `prisma migrate dev` genera SQL.
4. **Prisma Studio:** GUI web para explorar y editar datos.
5. **Integración con NestJS:** hay docs oficiales y patrones establecidos.
6. **Comunidad fuerte:** actualizaciones regulares, Discord activo.

## Consecuencias

### Positivas

- Un solo archivo (`schema.prisma`) como fuente de verdad del modelo
- `prisma generate` produce un cliente tipado tras cada cambio
- Validaciones de tipos en queries (autocompletado, errores en compilación)
- Relaciones automáticas con `include` (hace JOINs por ti)
- Migraciones versionadas en `prisma/migrations/`
- `prisma/seed.ts` para datos iniciales
- `prisma studio` sirve como herramienta de admin visual para dev

### Negativas

- **Abstrae SQL** → si necesitas una query muy compleja (CTEs, window functions raras) hay que usar `$queryRaw`
- **Migraciones `reset`** borran datos (peligroso en prod)
- **Dependencia del binario de Prisma** (queryEngine)
- **Performance:** overhead pequeño pero existente vs SQL raw

## Implicaciones técnicas

### Uso de Prisma Adapter para pg

El proyecto usa `PrismaPg` adapter (no el driver por defecto) por compatibilidad con la v7:

```typescript
const adapter = new PrismaPg({
  connectionString: process.env.DATABASE_URL!,
});
const prisma = new PrismaClient({ adapter });
```

### Schema con relaciones

```prisma
model Pedido {
  id       Int             @id @default(autoincrement())
  detalles PedidoDetalle[]
}

model PedidoDetalle {
  id       Int      @id @default(autoincrement())
  pedidoId Int
  pedido   Pedido   @relation(fields: [pedidoId], references: [id])
}
```

`prisma.pedido.findMany({ include: { detalles: true } })` trae todo en un solo JOIN.

### Configuración del seed

Archivo `prisma.config.ts` declara cómo correr el seed:

```typescript
export default defineConfig({
  schema: "prisma/schema.prisma",
  datasource: { url: env("DATABASE_URL") },
  migrations: { seed: "npx ts-node prisma/seed.ts" },
});
```
