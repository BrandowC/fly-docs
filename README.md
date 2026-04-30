# Documentación del Proyecto - Ice Cream App

Aplicación móvil de pedidos de helados personalizados (tamaño + sabor + toppings) con panel administrativo oculto.

**Autor:** Brandow Stivent (manueldev@grupohayplan.com)
**Fecha:** Abril 2026
**Asignatura:** Programación Móvil - Séptimo Semestre

---

## Arquitectura resumida

- **Frontend móvil**: React Native + Expo (repo `fly-app`)
- **Backend REST**: NestJS + Prisma (repo `fly-api`)
- **Base de datos**: PostgreSQL 16 en Docker
- **Autenticación admin**: bcrypt + credenciales locales
- **Tunneling dev**: ngrok (frontend alcanza backend desde Internet)
- **Estilo arquitectónico**: Cliente-Servidor distribuido, dos repositorios independientes

---

## Índice de documentación

### Temas académicos

| # | Tema | Archivo |
|---|---|---|
| 1 | Fundamentos y planificación de proyectos móviles | [01-fundamentos-planificacion.md](01-fundamentos-planificacion.md) |
| 2 | Metodologías de desarrollo ágil | [02-metodologias-agiles.md](02-metodologias-agiles.md) |
| 3 | Requerimientos y casos de uso | [03-requerimientos-casos-uso.md](03-requerimientos-casos-uso.md) |
| 4 | Diseño de base de datos y mockups | [04-diseno-bd-mockups.md](04-diseno-bd-mockups.md) |
| 5 | Herramientas de desarrollo móvil | [05-herramientas-desarrollo.md](05-herramientas-desarrollo.md) |
| 6 | Lenguajes de programación móviles | [06-lenguajes-programacion.md](06-lenguajes-programacion.md) |
| 7 | Backend básico para aplicaciones móviles | [07-backend-basico.md](07-backend-basico.md) |
| 8 | Frameworks móviles híbridos | [08-frameworks-hibridos.md](08-frameworks-hibridos.md) |

### Arquitectura de software

| Documento | Archivo |
|---|---|
| Arquitectura general del sistema | [arquitectura/ARQUITECTURA-GENERAL.md](arquitectura/ARQUITECTURA-GENERAL.md) |
| Arquitectura distribuida (dos repositorios) | [arquitectura/arquitectura-distribuida.md](arquitectura/arquitectura-distribuida.md) |
| Aplicación de principios SOLID | [arquitectura/principios-solid.md](arquitectura/principios-solid.md) |

### ADRs (Architecture Decision Records)

| Nº | Decisión | Archivo |
|---|---|---|
| 001 | NestJS como framework backend | [arquitectura/adrs/ADR-001-nestjs-backend.md](arquitectura/adrs/ADR-001-nestjs-backend.md) |
| 002 | React Native + Expo para móvil | [arquitectura/adrs/ADR-002-expo-react-native.md](arquitectura/adrs/ADR-002-expo-react-native.md) |
| 003 | Repositorios separados (front/back) | [arquitectura/adrs/ADR-003-repositorios-separados.md](arquitectura/adrs/ADR-003-repositorios-separados.md) |
| 004 | Prisma ORM | [arquitectura/adrs/ADR-004-prisma-orm.md](arquitectura/adrs/ADR-004-prisma-orm.md) |
| 005 | PostgreSQL en Docker | [arquitectura/adrs/ADR-005-postgres-docker.md](arquitectura/adrs/ADR-005-postgres-docker.md) |
| 006 | bcrypt para hashing de contraseñas | [arquitectura/adrs/ADR-006-bcrypt-auth.md](arquitectura/adrs/ADR-006-bcrypt-auth.md) |
| 007 | React Context para estado global | [arquitectura/adrs/ADR-007-context-estado-global.md](arquitectura/adrs/ADR-007-context-estado-global.md) |

### Diagramas (en formato Mermaid)

| Diagrama | Archivo |
|---|---|
| Casos de uso | [diagramas/01-casos-de-uso.md](diagramas/01-casos-de-uso.md) |
| Clases | [diagramas/02-clases.md](diagramas/02-clases.md) |
| C4 - Contexto | [diagramas/03-c4-contexto.md](diagramas/03-c4-contexto.md) |
| C4 - Contenedores | [diagramas/04-c4-contenedores.md](diagramas/04-c4-contenedores.md) |
| C4 - Componentes | [diagramas/05-c4-componentes.md](diagramas/05-c4-componentes.md) |
| Entidad-Relación (BD) | [diagramas/06-base-de-datos.md](diagramas/06-base-de-datos.md) |
| Arquitectura del backend | [diagramas/07-arquitectura-backend.md](diagramas/07-arquitectura-backend.md) |
| Arquitectura del frontend | [diagramas/08-arquitectura-frontend.md](diagramas/08-arquitectura-frontend.md) |
| Flujo de pedidos (secuencia) | [diagramas/09-flujo-pedido-secuencia.md](diagramas/09-flujo-pedido-secuencia.md) |

**Importante:** Los diagramas están en formato Mermaid dentro de archivos `.md`. Para convertirlos a `.jpg` sigue la guía: [diagramas/COMO-EXPORTAR-A-JPG.md](diagramas/COMO-EXPORTAR-A-JPG.md)

### Guías operativas

| Guía | Archivo |
|---|---|
| Cómo exportar diagramas Mermaid a JPG | [diagramas/COMO-EXPORTAR-A-JPG.md](diagramas/COMO-EXPORTAR-A-JPG.md) |
| Publicación en Google Play Store | [play-store/guia-publicacion.md](play-store/guia-publicacion.md) |

---

## Credenciales de desarrollo

| Recurso | Valor |
|---|---|
| Usuario admin | `admin` |
| Contraseña admin | `admin123` |
| PostgreSQL | `postgres` / `123456` (Docker local) |
| Base de datos | `icecream_db` |

---

## Cómo correr el proyecto

1. **Docker Desktop** abierto
2. **Terminal 1** — levantar PostgreSQL:
   ```bash
   cd fly-api/ice-cream
   docker compose up -d
   ```
3. **Terminal 2** — backend:
   ```bash
   cd fly-api/ice-cream
   npm run start:dev
   ```
4. **Terminal 3** — tunnel (solo si el celular no está en la misma WiFi):
   ```bash
   ngrok http 3000
   ```
5. **Terminal 4** — app móvil:
   ```bash
   cd fly-app/ice-cream-app
   npx expo start --tunnel
   ```
6. Escanear el QR con Expo Go en el celular.
