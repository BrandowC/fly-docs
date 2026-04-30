# Tema 4 · Diseño de base de datos y mockups

## 1. Motor de base de datos

**PostgreSQL 16** (imagen oficial `postgres:16` en Docker).

Justificación:
- Relacional robusto, ideal para datos transaccionales (pedidos)
- Soporta JSON nativo si se requiere en el futuro
- Free y con amplio soporte en Prisma
- Fácil de levantar con Docker Compose para desarrollo

## 2. Modelo entidad-relación (ERD)

### Entidades y atributos

#### 1. Admin
| Atributo | Tipo | Restricción |
|---|---|---|
| id | Int | PK, auto-incremental |
| username | String | UNIQUE, NOT NULL |
| password | String | NOT NULL (hash bcrypt) |
| createdAt | DateTime | DEFAULT now() |

#### 2. Producto
| Atributo | Tipo | Restricción |
|---|---|---|
| id | Int | PK, auto-incremental |
| nombre | String | NOT NULL |
| tipo | String | NOT NULL (valores: "CREMA", "AGUA", "TAMAÑO") |
| precio | Decimal(10,2) | NOT NULL |
| disponible | Boolean | DEFAULT true |

#### 3. Topping
| Atributo | Tipo | Restricción |
|---|---|---|
| id | Int | PK, auto-incremental |
| nombre | String | NOT NULL |
| descripcion | String | NULL |
| precio | Decimal(10,2) | NOT NULL |
| imagenUrl | String | NULL |
| disponible | Boolean | DEFAULT true |
| createdAt | DateTime | DEFAULT now() |
| updatedAt | DateTime | AUTO |

#### 4. Pedido
| Atributo | Tipo | Restricción |
|---|---|---|
| id | Int | PK, auto-incremental |
| clienteNombre | String | NOT NULL |
| telefono | String | NOT NULL |
| direccion | String | NOT NULL |
| total | Decimal(10,2) | NOT NULL |
| completado | Boolean | DEFAULT false |
| createdAt | DateTime | DEFAULT now() |

#### 5. PedidoDetalle
| Atributo | Tipo | Restricción |
|---|---|---|
| id | Int | PK, auto-incremental |
| pedidoId | Int | FK → Pedido.id |
| productoId | Int | FK → Producto.id |
| toppingId | Int | FK → Topping.id (nullable) |
| cantidad | Int | DEFAULT 1 |
| subtotal | Decimal(10,2) | NOT NULL |

### Relaciones

| Relación | Cardinalidad | Descripción |
|---|---|---|
| Pedido ↔ PedidoDetalle | 1 : N | Un pedido tiene muchos detalles |
| Producto ↔ PedidoDetalle | 1 : N | Un producto puede estar en muchos detalles |
| Topping ↔ PedidoDetalle | 0..1 : N | Un detalle puede o no tener un topping |

### Schema Prisma (`fly-api/ice-cream/prisma/schema.prisma`)

```prisma
model Admin {
  id        Int      @id @default(autoincrement())
  username  String   @unique
  password  String
  createdAt DateTime @default(now())
}

model Topping {
  id          Int      @id @default(autoincrement())
  nombre      String
  descripcion String?
  precio      Decimal  @db.Decimal(10,2)
  imagenUrl   String?
  disponible  Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  pedidoDetalles PedidoDetalle[]
}

model Producto {
  id          Int      @id @default(autoincrement())
  nombre      String
  tipo        String
  precio      Decimal  @db.Decimal(10,2)
  disponible  Boolean  @default(true)
  pedidos     PedidoDetalle[]
}

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

model PedidoDetalle {
  id         Int      @id @default(autoincrement())
  pedidoId   Int
  productoId Int
  toppingId  Int?
  cantidad   Int      @default(1)
  subtotal   Decimal  @db.Decimal(10,2)

  pedido     Pedido   @relation(fields: [pedidoId], references: [id])
  producto   Producto @relation(fields: [productoId], references: [id])
  topping    Topping? @relation(fields: [toppingId], references: [id])
}
```

### Diagrama ERD

Ver archivo: [diagramas/06-base-de-datos.md](diagramas/06-base-de-datos.md)

## 3. Normalización

El modelo está en **3FN (Tercera Forma Normal)**:

- **1FN**: Todos los atributos son atómicos (no hay listas dentro de una columna)
- **2FN**: Todas las entidades tienen PK simple (id autoincremental) y no hay dependencias parciales
- **3FN**: No hay atributos dependientes transitivamente de la PK. `total` en Pedido podría parecer derivable, pero se almacena por auditoría: los precios pueden cambiar después y el pedido debe preservar el total en el momento de la venta.

## 4. Índices y restricciones

| Entidad | Índice/Restricción | Propósito |
|---|---|---|
| Admin | `UNIQUE(username)` | Evitar duplicados de usuarios admin |
| PedidoDetalle | FK con ON UPDATE CASCADE | Integridad referencial |
| Pedido | `ORDER BY createdAt DESC` (consulta) | Listado del admin por fecha |

## 5. Datos de seed (poblado inicial)

El script `prisma/seed.ts` crea:

- **1 admin** (`admin` / `admin123`)
- **11 productos**:
  - 3 tamaños (Pequeño, Mediano, Grande - gratis, tipo TAMAÑO)
  - 5 helados CREMA (Fresa, Chocolate, Vainilla, Mora, Napolitano)
  - 3 helados AGUA (Limón, Maracuyá, Coco)
- **12 toppings** (chocolate, caramelo, arequipe, leche condensada, chispas, maní, almendras, Oreo, fresas, banano, chantilly, gomitas)
- **3 pedidos de ejemplo** con detalles completos

## 6. Mockups de pantallas

Los mockups descritos a continuación corresponden a la implementación final de la app.

### Mockup 1 — Pantalla de bienvenida (`app/index.tsx`)

```
┌─────────────────────────────┐
│                             │
│  [ imagen de helados ]      │
│                             │
│         ICECREAM            │   ← título (long-press = admin)
│   Sabor y frescura en...    │
│                             │
│   ┌─────────────────────┐   │
│   │      INGRESAR       │   │
│   └─────────────────────┘   │
│                             │
└─────────────────────────────┘
```

Funcionalidades:
- Animación de fade-in escalonada en la carga
- Long-press en "ICECREAM" abre panel admin
- Click en "INGRESAR" anima salida y navega a Home

### Mockup 2 — Home: Tamaños (`app/(tabs)/home.tsx`)

```
┌─────────────────────────────────┐
│                                 │
│          Ice Cream 🍦           │   ← título centrado (tappable → welcome)
│    — PERSONALIZA TU HELADO —    │
│                                 │
│  ┌───────┬────────┬─────────┐   │
│  │Tamaño │Helados │Toppings │   │   ← tabs (activo = azul relleno)
│  └───────┴────────┴─────────┘   │
│                                 │
│  Elige tu tamaño                │
│  Selecciona cómo lo quieres     │
│                                 │
│  ┌─────┐ ┌─────┐ ┌─────┐        │
│  │ 🥤  │ │ 🥤  │ │ 🥤  │        │
│  │Gran.│ │Med. │ │Peq. │        │
│  └─────┘ └─────┘ └─────┘        │
│                                 │
│  [ 🏠 ]        [ 🛒 ]           │   ← tab bar inferior
└─────────────────────────────────┘
```

### Mockup 3 — Home: Helados

```
┌─────────────────────────────────┐
│    — PERSONALIZA TU HELADO —    │
│                                 │
│  Tamaño  [Helados] Toppings     │   ← tab "Helados" activa (rosa)
│                                 │
│  Elige tu sabor                 │
│  Sabores deliciosos 🍦          │
│                                 │
│  ┌─────────────┐ ┌─────────────┐│
│  │     🍓       │ │     🍫       ││
│  │   Fresa      │ │  Chocolate   ││
│  │   CREMA      │ │    CREMA     ││
│  │   $4500      │ │    $4500     ││
│  └─────────────┘ └─────────────┘│
│  ┌─────────────┐ ┌─────────────┐│
│  │     🍦       │ │     🫐       ││
│  │  Vainilla    │ │    Mora      ││
│  └─────────────┘ └─────────────┘│
└─────────────────────────────────┘
```

### Mockup 4 — Carrito (`app/(tabs)/Carrito.tsx`)

```
┌─────────────────────────────────┐
│ ═════ Mi Pedido (header cyan) ══│
│                                 │
│  3 productos      ┌──────────┐  │
│                   │Total $9k │  │
│                   └──────────┘  │
│                                 │
│  ┌────────────────────────────┐ │
│  │ 🍓  Fresa          [🗑️]  │ │
│  │   $4500 c/u               │ │
│  │   [−] 2 [+]         $9000 │ │
│  └────────────────────────────┘ │
│                                 │
│  ┌──────────────────────────┐   │
│  │  🏍️  Datos de Entrega   │   │
│  │                          │   │
│  │  Nombre: [___________]   │   │
│  │  Celular: [__________]   │   │
│  │  Dirección: [________]   │   │
│  │  Barrio: [___________]   │   │
│  │  Notas: [____________]   │   │
│  │                          │   │
│  │  Total: $9000            │   │
│  │                          │   │
│  │  ┌──────────────────┐    │   │
│  │  │ 📱 Enviar WhatsApp│    │   │
│  │  └──────────────────┘    │   │
│  └──────────────────────────┘   │
└─────────────────────────────────┘
```

### Mockup 5 — Panel Admin (`app/admin.tsx`)

```
┌─────────────────────────────────┐
│ ═════ Panel Admin (teal) ═══ 🔄 │
│                                 │
│  [Helados] Toppings  Pedidos    │   ← tabs admin
│                                 │
│  ┌─────────────────────────┐    │
│  │ ➕ Agregar Helado       │    │
│  └─────────────────────────┘    │
│                                 │
│  ┌──────────────────────────┐   │
│  │ 🍓 Helado de Fresa       │   │
│  │    CREMA · $4500  [✏️][🗑️]│   │
│  └──────────────────────────┘   │
│                                 │
│  ┌──────────────────────────┐   │
│  │ 🍫 Helado de Chocolate   │   │
│  │    CREMA · $4500  [✏️][🗑️]│   │
│  └──────────────────────────┘   │
│                                 │
│  ┌──────────────────────────┐   │
│  │ 🚪  Cerrar Sesión        │   │
│  └──────────────────────────┘   │
└─────────────────────────────────┘
```

### Mockup 6 — Detalle de Pedido (admin tab "Pedidos")

```
┌─────────────────────────────────┐
│  ┌───────────────────────────┐  │
│  │ 📋 Pedido #3   [✓ Entregado]│ │
│  │                            │  │
│  │ 📅 17/04/2026 · 12:49      │  │
│  │ ┌────────────────────────┐ │  │
│  │ │ 👤 María Gómez         │ │  │
│  │ │ 📞 3156789012          │ │  │
│  │ │ 📍 Carrera 8 # 15-30   │ │  │
│  │ └────────────────────────┘ │  │
│  │                            │  │
│  │ 🛒 Productos pedidos       │  │
│  │  • 1x Grande TAMAÑO   $0   │  │
│  │  • 2x Chocolate CREMA $9000│  │
│  │  • 1x Coco+Gomitas    $5500│  │
│  │                            │  │
│  │ TOTAL            $14500    │  │
│  │                            │  │
│  │ [↻ Marcar como pendiente] │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

## 7. Flujo de datos (caso de uso de compra)

1. Usuario elige items en Home → se agregan al **CartContext** (memoria)
2. Usuario va a Carrito → diligencia datos → pulsa "Enviar"
3. App envía `POST /pedidos` con `{ clienteNombre, telefono, direccion, items[] }`
4. Backend persiste en 2 tablas (`Pedido` + `PedidoDetalle`)
5. App abre WhatsApp con URL pre-formateada
6. App limpia el CartContext
7. Admin al abrir el panel ve el nuevo pedido listado con `createdAt` reciente
