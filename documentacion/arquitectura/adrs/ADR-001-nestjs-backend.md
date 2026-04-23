# ADR-001 · Usar NestJS como framework backend

- **Fecha:** 2026-02-09
- **Estado:** Aceptado
- **Decisor:** Brandow Stivent
- **Relacionados:** ADR-004 (Prisma), ADR-005 (PostgreSQL Docker)

## Contexto

Se necesita un backend REST que exponga endpoints para catálogo, pedidos y autenticación. Los requisitos son:

- Estructura mantenible (proyecto académico pero con código real)
- Integración natural con TypeScript (el equipo ya conoce TS)
- Validación de inputs sin escribir boilerplate manual
- Inyección de dependencias para testabilidad
- Ecosistema maduro (documentación, ejemplos, Stack Overflow)

## Decisión

Se adopta **NestJS 11** como framework backend.

## Alternativas consideradas

| Alternativa | Pros | Contras | Veredicto |
|---|---|---|---|
| **Express.js** | Minimalista, enorme ecosistema | Sin estructura, sin DI, mucho boilerplate | ❌ Se descartan por falta de estructura |
| **Fastify** | Muy rápido, plugins | Menos docs en español, sin opinión arquitectónica | ❌ |
| **Koa** | Middleware moderno | Poca comunidad comparada con Express | ❌ |
| **NestJS** | Arquitectura modular, DI, TS first-class, decoradores, ecosistema rico | Curva de aprendizaje inicial, puede sentirse "sobredimensionado" | ✅ Elegido |
| **AdonisJS** | Estructura tipo Rails | Ecosistema más pequeño | ❌ |

## Consecuencias

### Positivas

- Arquitectura en capas impuesta por el framework (Controller/Service/Repository)
- Inyección de dependencias automática vía constructor
- `ValidationPipe` global valida DTOs sin código repetitivo
- Soporte first-class de TypeScript y decoradores
- Integración directa con Prisma vía `PrismaService` extendido
- Ecosistema maduro: `@nestjs/jwt`, `@nestjs/swagger`, `@nestjs/throttler`, etc.
- Comunidad grande, docs extensas en https://docs.nestjs.com

### Negativas

- Curva de aprendizaje: decoradores, módulos, providers
- Overhead percibido para proyectos muy simples (pero este ya tiene suficiente complejidad)
- Peso del bundle mayor que Express

## Implicaciones técnicas

- Dependencia del patrón DI → hay que registrar providers correctamente
- Testing requiere `@nestjs/testing` (no hecho en este proyecto, pero posible)
- Migración futura a otro framework sería costosa
