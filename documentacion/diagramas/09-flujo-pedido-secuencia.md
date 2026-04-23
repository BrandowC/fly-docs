# Diagrama de secuencia — Flujo completo de un pedido

Muestra la secuencia temporal de interacciones entre el cliente, la app, el backend, la base de datos y WhatsApp cuando un usuario realiza un pedido.

## Secuencia (Mermaid)

```mermaid
sequenceDiagram
    autonumber
    actor C as 👤 Cliente
    participant A as 📱 App Expo
    participant Ctx as CartContext
    participant N as 🌐 ngrok
    participant B as 🖥️ Backend NestJS
    participant P as Prisma
    participant DB as 🐘 PostgreSQL
    participant W as 💬 WhatsApp

    Note over C,A: FASE 1 — Carga inicial

    C->>A: Abre la app
    A->>N: GET /productos
    N->>B: GET /productos
    B->>P: prisma.producto.findMany()
    P->>DB: SELECT * FROM "Producto"
    DB-->>P: rows
    P-->>B: Producto[]
    B-->>N: JSON
    N-->>A: JSON
    A->>A: setProductos(res.data)

    A->>N: GET /toppings
    N->>B: GET /toppings
    B->>P: prisma.topping.findMany()
    P->>DB: SELECT * FROM "Topping"
    DB-->>P: rows
    P-->>B: Topping[]
    B-->>N: JSON
    N-->>A: JSON
    A->>A: setToppings(res.data)

    Note over C,A: FASE 2 — Cliente arma el pedido

    C->>A: Toca "Pequeño" (tab Tamaño)
    A->>Ctx: addToCart({ id: "tamano-13", name: "Pequeño", price: 0 })
    Ctx->>Ctx: cart = [tamano-13]

    C->>A: Toca "Helado de Fresa" (tab Helados)
    A->>Ctx: addToCart({ id: "helado-4", name: "Fresa", price: 4500 })
    Ctx->>Ctx: cart = [tamano-13, helado-4]

    C->>A: Toca "Arequipe" (tab Toppings)
    A->>Ctx: addToCart({ id: "topping-3", name: "Arequipe", price: 2500 })
    Ctx->>Ctx: cart = [tamano-13, helado-4, topping-3]

    Note over C,A: FASE 3 — Cliente va al carrito

    C->>A: Tap en tab Carrito
    A->>A: Muestra lista con emojis + +/- +/- botones
    A->>A: Calcula totalPrice = 0+4500+2500 = 7000
    A->>A: Muestra total en pill rosa

    Note over C,A: FASE 4 — Cliente diligencia datos y envía

    C->>A: Llena nombre, celular, dirección, barrio
    C->>A: Toca "Enviar pedido a WhatsApp"

    A->>A: Valida campos obligatorios
    A->>A: buildItemsForBackend() → [{productoId: 13}, {productoId: 4}, {productoId: 4, toppingId: 3}]

    A->>N: POST /pedidos<br/>{ clienteNombre, telefono, direccion, items }
    N->>B: POST /pedidos

    B->>B: ValidationPipe (verifica CreatePedidoDto)
    B->>B: PedidosService.crearPedido(dto)

    loop por cada item
        B->>P: prisma.producto.findUnique({ id })
        P->>DB: SELECT
        DB-->>P: producto
        P-->>B: producto

        alt item tiene toppingId
            B->>P: prisma.topping.findUnique({ id })
            P->>DB: SELECT
            DB-->>P: topping
            P-->>B: topping
        end

        B->>B: subtotal = producto.precio + (topping?.precio || 0)
    end

    B->>B: totalGeneral = suma de subtotales

    B->>P: prisma.pedido.create({<br/> data: { ..., detalles: { create: detalles } },<br/> include: { detalles: { include: producto, topping } }<br/>})
    P->>DB: BEGIN
    P->>DB: INSERT INTO "Pedido" (...)
    DB-->>P: id = 7
    P->>DB: INSERT INTO "PedidoDetalle" (pedidoId=7, ...) x3
    P->>DB: COMMIT
    DB-->>P: pedido con detalles anidados
    P-->>B: Pedido con relaciones cargadas

    B-->>N: 201 Created<br/>{ pedido, whatsappUrl }
    N-->>A: 201

    A->>W: Linking.openURL("wa.me/...?text=...")
    W-->>C: Abre WhatsApp con mensaje pre-escrito

    A->>Ctx: clearCart()
    Ctx->>Ctx: cart = []
    A->>A: Navega a home

    Note over C,A: FASE 5 — Admin revisa el pedido (opcional)

    actor Adm as 👔 Admin
    Adm->>A: Long-press en logo "ICECREAM"
    A->>A: Navega a /admin

    Adm->>A: Digita credenciales
    A->>N: POST /auth/login
    N->>B: POST /auth/login
    B->>P: prisma.admin.findUnique({ username })
    P->>DB: SELECT
    DB-->>P: admin
    B->>B: bcrypt.compare(password, admin.password)
    B-->>N: 200 { success, admin }
    N-->>A: 200

    Adm->>A: Tab "Pedidos"
    A->>N: GET /pedidos
    N->>B: GET /pedidos
    B->>P: prisma.pedido.findMany({ include: { detalles: { include: producto, topping } } })
    P->>DB: SELECT + JOINs
    DB-->>P: pedidos con detalles
    P-->>B: Pedido[] con relaciones
    B-->>N: JSON
    N-->>A: JSON

    A->>A: Muestra el pedido #7 con fecha, cliente, detalle y estado "Pendiente"
    Adm->>A: Tap "Marcar como entregado"
    A->>N: PATCH /pedidos/7/toggle
    N->>B: PATCH /pedidos/7/toggle
    B->>P: pedido.update({ completado: !pedido.completado })
    P->>DB: UPDATE
    DB-->>P: pedido actualizado
    P-->>B: Pedido
    B-->>N: JSON
    N-->>A: JSON
    A->>A: Refresca lista con badge "Entregado" verde
```

## Observaciones

### Puntos clave del flujo

- **Paso 27**: el carrito vive en memoria (CartContext). Si el usuario cierra la app antes de enviar, pierde todo.
- **Paso 37**: el backend calcula el total **consultando precios actuales**, no confía en el precio que envía el cliente (seguridad).
- **Paso 49**: la creación usa **nested writes** de Prisma → transacción atómica (si falla un detalle, nada se crea).
- **Paso 54**: WhatsApp se abre **solo si el POST al backend fue exitoso** (evita pedidos fantasma).
- **Paso 57**: `clearCart()` se llama al final para que el usuario no vuelva a enviar por error el mismo carrito.

### Manejo de errores

Si cualquier paso falla:

- **Timeout de axios (15s)** → `AxiosError`, se muestra alerta y no se abre WhatsApp
- **400 Bad Request** (DTO inválido) → backend rechaza, se muestra error
- **500** (error interno) → backend responde error, la app muestra "No se pudo registrar tu pedido"
- **BD caída** → backend devuelve 500, mismo mensaje

## Duración esperada

| Fase | Tiempo |
|---|---|
| Carga inicial (GET productos + toppings) | 300-800ms |
| Interacción del usuario (seleccionar items) | N/A (sin backend) |
| Envío del pedido (POST /pedidos + abrir WhatsApp) | 500-1500ms |
| **Total desde apertura hasta WhatsApp** | ~1-2 minutos con usuario humano |
