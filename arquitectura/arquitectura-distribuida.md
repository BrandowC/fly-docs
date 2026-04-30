# Arquitectura distribuida — dos repositorios

## 1. Decisión central

El sistema se divide en **dos repositorios Git independientes**:

| Repositorio | Ruta | Contenido |
|---|---|---|
| `fly-api` | `PROYECTO/fly-api/` | Backend NestJS + Prisma + Docker |
| `fly-app` | `PROYECTO/fly-app/` | App móvil React Native + Expo |

Esta decisión está formalizada en [ADR-003](adrs/ADR-003-repositorios-separados.md).

## 2. ¿Por qué distribuida?

Una arquitectura distribuida significa que **diferentes procesos corren en diferentes hosts** y se comunican por red. En nuestro caso:

- **Proceso 1:** App móvil (en el celular del cliente)
- **Proceso 2:** Backend NestJS (en nuestro PC / servidor)
- **Proceso 3:** PostgreSQL (en Docker)
- **Proceso 4:** ngrok (tunnel intermediario, solo dev)

Ventajas frente a una arquitectura monolítica:

| Ventaja | Explicación |
|---|---|
| **Independencia de despliegue** | Se actualiza el backend sin tocar la app y viceversa |
| **Independencia tecnológica** | TypeScript en ambos hoy, mañana podría ser Go en el backend |
| **Escalabilidad selectiva** | Si el backend se sobrecarga, solo escalamos el backend |
| **Tolerancia a fallos** | Si el backend cae, la app sigue pudiendo mostrar cache (no implementado aún) |
| **Separación de responsabilidades** | Frontend ⇒ UI/UX · Backend ⇒ lógica/datos |

## 3. Topología física

```
┌─────────────────────────┐          ┌─────────────────────────┐
│                         │  HTTPS   │                         │
│   CELULAR (cliente)     │ ◄──────► │    NGROK TUNNEL         │
│                         │          │  (diagram-dreary-...)   │
└─────────────────────────┘          └───────────┬─────────────┘
                                                 │
                                                 │ reenvía a localhost:3000
                                                 ▼
                                   ┌─────────────────────────────┐
                                   │                             │
                                   │     PC DEL DESARROLLADOR    │
                                   │                             │
                                   │  ┌────────────────────────┐ │
                                   │  │   BACKEND NESTJS       │ │
                                   │  │   (puerto 3000)        │ │
                                   │  └──────────┬─────────────┘ │
                                   │             │ TCP 5432      │
                                   │  ┌──────────▼─────────────┐ │
                                   │  │   POSTGRESQL DOCKER    │ │
                                   │  │  (icecream_postgres)   │ │
                                   │  └────────────────────────┘ │
                                   │                             │
                                   └─────────────────────────────┘
```

## 4. Protocolos de comunicación

| Par de procesos | Protocolo | Formato |
|---|---|---|
| App ↔ ngrok | HTTPS | JSON |
| ngrok ↔ Backend | HTTP (local) | JSON |
| Backend ↔ PostgreSQL | TCP puerto 5432 | Protocolo nativo PG |
| App → WhatsApp | Deep-link `wa.me` | URL-encoded text |

## 5. Retos de una arquitectura distribuida

### 5.1 Latencia de red
Cada llamada API tarda 50-500ms según la red. El frontend mitiga esto con:
- Spinners `ActivityIndicator` durante la carga
- Estados optimistas (no implementados aún)
- Cache en memoria del carrito

### 5.2 Errores parciales
El backend puede estar caído o el tunnel puede morir. El frontend maneja:
```typescript
try {
  await api.post("/pedidos", payload);
} catch (error) {
  Alert.alert("Error al enviar", "Revisa tu conexión...");
  return;  // ← no abre WhatsApp, evita pedidos fantasma
}
```

### 5.3 Consistencia
Al crear un pedido con detalles, se usa una **transacción implícita de Prisma** (nested create). Si falla algún detalle, no se crea el pedido:
```typescript
await prisma.pedido.create({
  data: {
    ...data,
    detalles: { create: detalles },   // transacción atómica
  },
});
```

### 5.4 Configuración entre entornos
Cada ambiente necesita URLs diferentes. En desarrollo usamos:
- `DATABASE_URL` en `.env` del backend
- `API_URL` en `constants/Config.ts` del frontend

### 5.5 CORS
El backend habilita CORS para cualquier origen (desarrollo):
```typescript
app.enableCors();
```
En producción se restringiría a dominios específicos.

## 6. Ventajas de dos repositorios (vs monorepo)

| Aspecto | Dos repos (actual) | Monorepo |
|---|---|---|
| **Commits** | Aislados por dominio | Mezclados |
| **CI/CD** | Uno independiente por repo | Complejo setup |
| **Permisos** | Distintos por equipo | Unificados |
| **Versionado** | Libre | Coordinado |
| **Dependencias** | Duplicadas (`axios` en ambos) | Compartidas |

Desventaja principal: los cambios que tocan contrato API (ej: agregar un campo en un DTO) requieren **sincronización manual** entre repos. Mitigación: commits con mensaje `[breaking]` y tag conjunto.

## 7. Flujo de trabajo entre repos

Ejemplo: agregar un nuevo endpoint `/pedidos/:id/cancel`.

### Paso 1 (repo `fly-api`)
```bash
cd fly-api/ice-cream
# Editar pedidos.controller.ts + service + repository
git add src/pedidos/
git commit -m "feat(pedidos): agregar endpoint /:id/cancel"
# Deploy del backend
```

### Paso 2 (repo `fly-app`)
```bash
cd fly-app/ice-cream-app
# Llamar al nuevo endpoint desde admin.tsx
git add app/admin.tsx
git commit -m "feat(admin): usar endpoint /pedidos/:id/cancel"
# Deploy de la app
```

**Importante:** el paso 1 debe desplegarse **antes** del paso 2 para evitar 404.

## 8. Deployment topology para producción

### Opción recomendada

```
                   GOOGLE PLAY STORE
                         │
                         │ descarga
                         ▼
                    APP en celular
                         │
                         │ HTTPS
                         ▼
            api.icecream.com (Nginx/Caddy)
                         │
                         ▼
          ┌─────────────────────────────┐
          │   Backend NestJS (Docker)   │
          │   docker-compose.yml        │
          │  ┌───────────┐              │
          │  │ app:3000  │              │
          │  └─────┬─────┘              │
          │        │                    │
          │  ┌─────▼─────┐              │
          │  │ postgres  │              │
          │  └───────────┘              │
          └─────────────────────────────┘
                   VPS (DigitalOcean / Hetzner)
```

**Cambios para producción:**

- `fly-app`:
  - URL en `Config.ts` → `https://api.icecream.com`
  - Remover header `ngrok-skip-browser-warning`
  - Correr `eas build --platform android --profile production`

- `fly-api`:
  - Dockerizar también el backend (agregar `Dockerfile`)
  - Usar `.env.production` con `DATABASE_URL` de BD administrada
  - Activar HTTPS via reverse proxy

## 9. Observabilidad (no implementado, recomendado)

En un sistema distribuido es crítico tener:

- **Logs centralizados** → actualmente solo en consola local
- **Métricas** → Prometheus + Grafana
- **Tracing distribuido** → OpenTelemetry
- **Health checks** → endpoint `/health` en el backend
