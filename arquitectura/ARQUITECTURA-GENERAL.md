# Arquitectura general del sistema

## 1. Visión de alto nivel

Ice Cream App implementa una arquitectura **Cliente-Servidor distribuida** con **3 componentes principales**:

```
┌─────────────────────┐     HTTPS     ┌──────────────────────┐    TCP    ┌──────────────────┐
│                     │  (JSON/REST)  │                      │ (5432)    │                  │
│  App Móvil Expo     │ ─────────────►│   Backend NestJS     │ ────────► │   PostgreSQL     │
│  (fly-app)          │               │   (fly-api)          │           │   en Docker      │
│                     │ ◄─────────────│                      │ ◄──────── │                  │
└─────────────────────┘               └──────────────────────┘           └──────────────────┘
    Cliente                              Servidor                           Persistencia
```

En desarrollo, entre la app móvil y el backend se interpone **ngrok** como tunnel HTTPS público.

## 2. Estilo arquitectónico

### 2.1 Cliente-Servidor

El frontend (app móvil) es el **cliente** y el backend REST es el **servidor**. Se comunican por HTTP con mensajes JSON.

### 2.2 Arquitectura distribuida

- Cada componente (frontend, backend, BD) se ejecuta en un proceso/host **diferente**
- Los componentes se comunican por **red** (HTTP, TCP)
- **Dos repositorios Git independientes**: `fly-app` y `fly-api`
- Cada repo puede evolucionar, testearse y desplegarse por separado

### 2.3 Arquitectura por capas (Layered) — dentro del backend

```
┌──────────────────────────────────────┐
│  CAPA DE PRESENTACIÓN (Controllers)  │   ← rutas HTTP, extracción de params
├──────────────────────────────────────┤
│    CAPA DE NEGOCIO (Services)        │   ← lógica, validaciones, reglas
├──────────────────────────────────────┤
│   CAPA DE ACCESO A DATOS (Repos)     │   ← queries, CRUD
├──────────────────────────────────────┤
│      CAPA DE DATOS (Prisma + PG)     │   ← persistencia
└──────────────────────────────────────┘
```

### 2.4 Patrón MVC adaptado en el frontend

React Native con Expo Router y Context sigue un patrón parecido a MVVM:
- **Vista:** JSX de los archivos `(tabs)/*.tsx`
- **ViewModel:** hooks `useState`, `useEffect`, `useCart`
- **Modelo:** axios hacia el backend + Context (estado compartido)

## 3. Principios arquitectónicos aplicados

### 3.1 Separación de responsabilidades

- **Frontend** solo se encarga de UI, estado local y llamadas al API
- **Backend** solo se encarga de negocio, validación y acceso a datos
- **Base de datos** solo almacena

### 3.2 Comunicación stateless

Cada petición HTTP al backend contiene toda la info necesaria. No hay sesiones en memoria del servidor (solo el login retorna un objeto admin, pero no se mantiene estado en memoria).

### 3.3 Fuente única de verdad

- **Catálogo**: la BD es la fuente única. La app siempre lo pide por API.
- **Carrito**: vive únicamente en el `CartContext` del frontend (memoria).
- **Pedidos confirmados**: se persisten en la BD al enviar.

### 3.4 Open/Closed

La arquitectura permite agregar nuevos endpoints / entidades sin modificar las existentes. Por ejemplo, agregar un módulo de `Reseñas` no requiere tocar `Productos` ni `Toppings`.

## 4. Componentes del sistema

### 4.1 Frontend (fly-app / ice-cream-app)

```
ice-cream-app/
├── app/                      ← rutas (Expo Router)
│   ├── _layout.tsx
│   ├── index.tsx             ← Welcome
│   ├── admin.tsx             ← Panel admin (modal, oculto)
│   └── (tabs)/
│       ├── _layout.tsx       ← Tab bar inferior
│       ├── home.tsx          ← Catálogo (3 tabs internos)
│       └── Carrito.tsx       ← Carrito + formulario
├── context/
│   └── CartContext.tsx       ← Estado global del carrito
├── services/
│   └── api.ts                ← instancia axios configurada
├── constants/
│   ├── Config.ts             ← URL del backend
│   └── api.ts                ← (duplicado histórico)
└── assets/images/            ← portada, etc.
```

### 4.2 Backend (fly-api / ice-cream)

```
ice-cream/
├── src/
│   ├── main.ts               ← bootstrap
│   ├── app.module.ts         ← registro de módulos
│   ├── prisma/               ← PrismaService
│   ├── auth/                 ← login admin
│   ├── productos/            ← CRUD helados y tamaños
│   ├── topping/              ← CRUD toppings
│   ├── pedidos/              ← creación y listado de pedidos
│   └── admin/                ← dashboard y reportes (protegido)
├── prisma/
│   ├── schema.prisma
│   ├── seed.ts
│   └── migrations/
└── docker-compose.yml
```

### 4.3 Base de datos

PostgreSQL 16 en Docker, con 5 tablas:
- `Admin`
- `Producto`
- `Topping`
- `Pedido`
- `PedidoDetalle`

Ver diagrama ERD en [../diagramas/06-base-de-datos.md](../diagramas/06-base-de-datos.md).

## 5. Flujo de datos principales

### 5.1 Cliente arma y envía un pedido

```
1. App GET /productos  → recibe lista
2. App GET /toppings   → recibe lista
3. Usuario agrega items al CartContext (memoria)
4. Usuario llena formulario
5. App POST /pedidos { clienteNombre, telefono, direccion, items }
6. Backend:
   a. Valida DTO
   b. Calcula total consultando precios actuales
   c. Crea Pedido + PedidoDetalle en BD (transacción)
   d. Devuelve pedido creado
7. App abre WhatsApp con url pre-formateada
8. App vacía el CartContext
```

### 5.2 Admin gestiona el catálogo

```
1. POST /auth/login { admin, admin123 }
2. Backend: bcrypt.compare → éxito
3. App: guarda estado "logueado" en memoria
4. App GET /productos, /toppings, /pedidos (refresh)
5. Admin crea producto: POST /productos { nombre, tipo, precio }
6. App refresca la lista
```

### 5.3 Admin cambia estado de pedido

```
1. PATCH /pedidos/:id/toggle
2. Backend: lee pedido, invierte `completado`, actualiza
3. Devuelve pedido con detalles + producto + topping
4. App refresca
```

## 6. Consideraciones de seguridad

| Aspecto | Estado actual | Recomendación producción |
|---|---|---|
| Contraseñas | bcrypt ✅ | Mantener, aumentar costo a 12 |
| Login | Retorna objeto admin | Retornar JWT firmado |
| CRUD productos | Sin auth | Agregar Guard JWT |
| CRUD pedidos | Sin auth | Pedidos públicos OK, lectura admin con Guard |
| CORS | `enableCors()` abierto | Restringir a dominios específicos |
| Rate limiting | No hay | Agregar `@nestjs/throttler` |
| HTTPS | Solo por ngrok | Desplegar detrás de TLS (Nginx/Caddy) |
| Variables sensibles | `.env` local | Gestor secretos (AWS Secrets, Doppler) |
| Admin API Key | Placeholder `tu_clave_secreta_aqui` | Reemplazar con valor env |

## 7. Consideraciones de escalabilidad

Diseño preparado para escalar horizontal y verticalmente:

- **Backend stateless** → se puede replicar detrás de un LB
- **Prisma + pool PG** soporta cientos de conexiones concurrentes
- **Frontend móvil** no tiene límites de escala
- **Docker** facilita migrar PG a un cluster (RDS, Neon, etc.)

## 8. Deployment topology (conceptual)

### Desarrollo actual
```
Celular (iOS) ── ngrok ── Backend local ── Docker PG
```

### Producción propuesta
```
App publicada en Play Store
       │
       ▼
   CDN / WAF
       │
       ▼
Load balancer HTTPS (Nginx)
       │
       ▼
   Backend NestJS (varios pods)
       │
       ▼
PostgreSQL administrado (Neon / Supabase / RDS)
```
