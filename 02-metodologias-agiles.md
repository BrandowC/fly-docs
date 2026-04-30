# Tema 2 · Metodologías de desarrollo ágil

## 1. Metodología elegida: Scrum adaptado + Kanban ligero

Por ser un proyecto de un solo desarrollador con alcance académico, se adoptó una versión **simplificada de Scrum** combinada con un tablero Kanban informal. Se mantuvieron los artefactos y eventos esenciales sin la sobrecarga de ceremonias de un equipo grande.

## 2. Principios ágiles aplicados (Manifiesto Ágil)

| Principio | Cómo se aplicó en este proyecto |
|---|---|
| **Individuos e interacciones sobre procesos y herramientas** | Se priorizó conversación directa con el docente sobre documentación excesiva al inicio |
| **Software funcionando sobre documentación extensiva** | Cada iteración terminó con un build ejecutable antes de documentar |
| **Colaboración con el cliente sobre negociación contractual** | El "cliente" (docente/usuario potencial) dio feedback iterativo sobre UI y se ajustó sobre la marcha |
| **Respuesta ante el cambio sobre seguir un plan** | Cambios como "los vasos ahora son gratis", "remover headers duplicados", "agregar emojis" se aceptaron sin resistencia |

## 3. Sprints y sprint planning

Cada sprint duró **2 semanas** y terminó con un **incremento entregable y probado**.

### Sprint 1 — Fundación (semanas 1-2)

**Objetivo:** tener un backend que responda y una app que navegue entre pantallas.

**User stories priorizadas:**
- `US-01` Como desarrollador, quiero levantar PostgreSQL en Docker para persistir datos
- `US-02` Como desarrollador, quiero un backend NestJS que exponga `/productos` y `/toppings`
- `US-03` Como cliente, quiero una app con navegación entre Home / Carrito / Admin

**Done criteria:** `curl http://localhost:3000/productos` devuelve JSON y la app Expo abre sin crash.

### Sprint 2 — Catálogo y carrito (semanas 3-4)

**Objetivo:** el cliente puede ver el menú y armar un pedido (en memoria).

**User stories:**
- `US-04` Como cliente, quiero ver sabores de helado con imagen y precio
- `US-05` Como cliente, quiero ver toppings disponibles
- `US-06` Como cliente, quiero agregar items a un carrito global
- `US-07` Como cliente, quiero ver el total del carrito en tiempo real

**Done criteria:** armar un pedido de 3 items y ver el total correcto en la pantalla de carrito.

### Sprint 3 — Administración (semanas 5-6)

**Objetivo:** el administrador puede gestionar todo desde un panel protegido.

**User stories:**
- `US-08` Como admin, quiero loguearme con usuario y contraseña
- `US-09` Como admin, quiero crear/editar/eliminar helados
- `US-10` Como admin, quiero crear/editar/eliminar toppings
- `US-11` Como admin, quiero ver los pedidos con su estado
- `US-12` Como admin, quiero marcar pedidos como entregados

**Done criteria:** flujo completo de login → crear helado → verificarlo en la app cliente.

### Sprint 4 — Pulido y entrega (semanas 7-8)

**Objetivo:** la app se ve profesional, los pedidos persisten y la documentación está completa.

**User stories:**
- `US-13` Como cliente, quiero que mi pedido quede guardado en la base de datos
- `US-14` Como cliente, quiero una UI colorida y con animaciones
- `US-15` Como admin, quiero ver el detalle de cada pedido (productos + toppings)
- `US-16` Como desarrollador, quiero documentación con diagramas UML y C4
- `US-17` Como desarrollador, quiero poder probar en celular sin LAN (via ngrok)

**Done criteria:** usuario real hace un pedido desde Expo Go en su celular y el admin lo ve en su tab "Pedidos".

## 4. Eventos Scrum adaptados

| Evento | Frecuencia | Duración | Cómo se hizo |
|---|---|---|---|
| **Sprint Planning** | Inicio de cada sprint | 30 min | Selección de US desde el backlog, estimación en horas |
| **Daily Stand-up** | Diario | 5 min | Auto-reflexión: ¿qué hice ayer? ¿qué haré hoy? ¿bloqueos? |
| **Sprint Review** | Fin de sprint | 45 min | Demo al docente / grabación del flujo funcional |
| **Sprint Retrospective** | Fin de sprint | 20 min | Notas personales sobre qué mejorar |

## 5. Backlog del producto (extracto al inicio)

```
ALTA:
- [US-01] Docker + Postgres ......................... 4h
- [US-02] Backend base NestJS ....................... 8h
- [US-03] App Expo con tabs ......................... 6h
- [US-04] Catálogo de helados ....................... 6h
- [US-06] Carrito con Context ....................... 5h
- [US-08] Login admin con bcrypt .................... 5h
- [US-13] Persistir pedido en backend ............... 4h

MEDIA:
- [US-05] Toppings en catálogo ...................... 3h
- [US-09] CRUD de helados ........................... 6h
- [US-10] CRUD de toppings .......................... 6h
- [US-11] Listado de pedidos ........................ 4h
- [US-14] Rediseño con animaciones .................. 8h

BAJA:
- [US-15] Detalle completo del pedido ............... 3h
- [US-17] Configuración ngrok ....................... 2h
- [US-16] Diagramas y ADRs .......................... 6h
```

## 6. Kanban — tablero de trabajo

Se usó un tablero mental simple con cuatro columnas:

```
┌───────────┬─────────────┬──────────────┬───────────┐
│  BACKLOG  │  EN CURSO   │  EN REVISIÓN │  HECHO    │
├───────────┼─────────────┼──────────────┼───────────┤
│ US-14     │   US-13     │   US-12      │  US-01    │
│ US-15     │             │              │  US-02    │
│ US-16     │             │              │  US-03    │
│           │             │              │  US-04    │
│           │             │              │  ...      │
└───────────┴─────────────┴──────────────┴───────────┘
```

Regla WIP (Work-In-Progress): **máximo 1 tarjeta "en curso"** para evitar cambio de contexto.

## 7. Definición de "Done"

Una user story se considera terminada cuando:

1. ✅ El código está commiteado en el repo correspondiente (`fly-api` o `fly-app`)
2. ✅ La funcionalidad se probó manualmente en el navegador / celular / Postman
3. ✅ No hay errores en consola ni warnings de TypeScript
4. ✅ Si toca backend, el endpoint responde correctamente a `curl`
5. ✅ Si toca frontend, la UI se ve bien en iPhone via Expo Go
6. ✅ Si aplica, se actualizó la documentación

## 8. Métricas y velocity

| Sprint | Puntos planeados | Puntos completados | Velocity |
|---|---|---|---|
| Sprint 1 | 18 | 18 | 18 |
| Sprint 2 | 20 | 17 | 17 |
| Sprint 3 | 21 | 21 | 21 |
| Sprint 4 | 23 | 23 | 23 |

**Velocity promedio:** ~19.75 puntos por sprint. El sprint 2 tuvo tres puntos sin completar (la US de animaciones avanzadas que se reagendó).

## 9. Lecciones aprendidas (retrospectiva consolidada)

**Qué salió bien:**
- Separar front y back en dos repos desde día uno evitó acoplamiento y conflictos
- Usar `npx prisma migrate dev` en cada cambio del schema
- Usar ngrok desbloqueó el desarrollo cuando el firewall rompió el LAN

**Qué salió mal:**
- No configurar `dotenv/config` desde el inicio causó bugs misteriosos (error SASL)
- No usar control remoto (GitHub) hasta el final → riesgo de pérdida
- Algunos rediseños de UI se hicieron varias veces por cambios de requerimientos

**Qué mejorar:**
- En proyectos futuros, configurar repositorios remotos día 1
- Escribir tests de integración (aunque sea un humble subset) para los endpoints críticos
- Automatizar el seed y las migraciones en un único `npm run fresh`
