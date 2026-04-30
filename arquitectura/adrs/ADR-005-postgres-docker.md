# ADR-005 · PostgreSQL en Docker

- **Fecha:** 2026-02-15
- **Estado:** Aceptado
- **Decisor:** Brandow Stivent

## Contexto

Se necesita una base de datos relacional para persistir admins, productos, toppings, pedidos y sus detalles. Consideraciones:

- Debe ser fácil de levantar en la máquina del desarrollador
- Debe soportar relaciones 1:N y decimales (precios)
- No debe acoplar el desarrollador a una instalación específica del OS
- Ideal si es la misma tecnología que usaríamos en producción

## Decisión

Se usa **PostgreSQL 16** corriendo en un contenedor **Docker**, orquestado con **Docker Compose**.

## Alternativas consideradas

| Alternativa | Pros | Contras | Veredicto |
|---|---|---|---|
| **SQLite** | Archivo único, cero config | No apto para producción multi-cliente | ❌ |
| **MySQL** | Popular, gratis | Menos funcionalidades avanzadas que PG | ❌ |
| **MongoDB** | Flexible | No es relacional, overkill aquí | ❌ |
| **PostgreSQL instalado localmente** | Performance ligeramente mejor | Difícil de desinstalar, contamina el sistema | ❌ |
| **PostgreSQL en Docker** | Aislado, reproducible, fácil reset | Requiere Docker Desktop instalado | ✅ Elegido |

## Justificación

1. **Reproducibilidad:** `docker compose up -d` y ya está corriendo igual en cualquier máquina.
2. **Aislamiento:** la BD vive en un contenedor, no afecta el OS host.
3. **Fácil reset:** `docker compose down -v` borra todo, permite empezar de cero.
4. **Paridad producción:** usar la misma versión de PG que usaríamos en producción (ej. RDS PG 16).
5. **Volumen persistente:** los datos sobreviven reinicios del contenedor gracias al volumen `postgres_data`.

## Configuración

Archivo [`fly-api/ice-cream/docker-compose.yml`](../../fly-api/ice-cream/docker-compose.yml):

```yaml
services:
  postgres:
    image: postgres:16
    container_name: icecream_postgres
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: 123456
      POSTGRES_DB: icecream_db
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Conexión desde Prisma (en `.env`):
```
DATABASE_URL="postgresql://postgres:123456@localhost:5432/icecream_db?schema=public"
```

## Consecuencias

### Positivas

- Setup en 1 comando (`docker compose up -d`)
- Fácil crear múltiples BDs para pruebas independientes
- No hay servicio de Windows que arranque solo
- Se puede versionar el `docker-compose.yml`
- Fácil migrar a producción: mismo PG en RDS/Neon/Supabase

### Negativas

- **Docker Desktop** es un requisito (pesado en recursos)
- **Docker credential helper** puede fallar (arreglado en el proyecto cambiando `credsStore` → `credStore`)
- **Puerto 5432** ocupado localmente (conflicto si ya tienes PG instalado)

## Implicaciones técnicas

- El desarrollador necesita Docker Desktop corriendo antes de arrancar el backend
- Los datos persisten en el volumen `postgres_data` de Docker (no en el filesystem del proyecto)
- Para reset completo: `docker compose down -v && docker compose up -d && npx prisma migrate dev && npx prisma db seed`
- En producción se usaría PG administrado (no Docker local)
