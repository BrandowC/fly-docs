# Tema 3 · Requerimientos y casos de uso

## 1. Requerimientos funcionales (RF)

### RF cliente (app móvil)

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-01 | El cliente puede ver un catálogo dividido en tamaños, helados y toppings | Alta |
| RF-02 | El cliente puede seleccionar un tamaño de vaso (pequeño, mediano, grande). Los tamaños son gratis | Alta |
| RF-03 | El cliente puede seleccionar uno o más sabores de helado con su precio | Alta |
| RF-04 | El cliente puede seleccionar uno o más toppings con su precio | Alta |
| RF-05 | El cliente puede ver un carrito con los items seleccionados, el subtotal por item y el total general | Alta |
| RF-06 | El cliente puede aumentar, disminuir o eliminar items del carrito | Alta |
| RF-07 | El cliente puede ingresar sus datos (nombre, celular, dirección, barrio, notas) | Alta |
| RF-08 | El sistema valida que los campos obligatorios estén completos antes de enviar el pedido | Alta |
| RF-09 | El pedido se envía al backend (se persiste en BD) y luego se abre WhatsApp con el resumen | Alta |
| RF-10 | Tras enviar el pedido, el carrito se vacía automáticamente | Media |

### RF administrador (panel oculto)

| ID | Requerimiento | Prioridad |
|---|---|---|
| RF-11 | Existe un acceso al panel admin desde la pantalla de bienvenida mediante long-press en el logo | Alta |
| RF-12 | El admin debe autenticarse con usuario y contraseña antes de acceder al panel | Alta |
| RF-13 | El admin puede ver la lista de helados (productos) con nombre, tipo y precio | Alta |
| RF-14 | El admin puede crear, editar o eliminar helados | Alta |
| RF-15 | El admin puede ver la lista de toppings con nombre, descripción y precio | Alta |
| RF-16 | El admin puede crear, editar o eliminar toppings | Alta |
| RF-17 | El admin puede ver todos los pedidos ordenados por fecha descendente | Alta |
| RF-18 | Cada pedido muestra: número, fecha, cliente, teléfono, dirección, detalle de productos, total y estado | Alta |
| RF-19 | El admin puede alternar el estado de un pedido entre "Pendiente" y "Entregado" | Alta |
| RF-20 | El admin puede cerrar sesión | Media |

## 2. Requerimientos no funcionales (RNF)

| ID | Categoría | Requerimiento |
|---|---|---|
| RNF-01 | **Rendimiento** | Las pantallas de catálogo deben cargar en menos de 2 segundos con 50 items |
| RNF-02 | **Rendimiento** | El POST de pedidos debe responder en menos de 1 segundo |
| RNF-03 | **Seguridad** | Las contraseñas se almacenan hasheadas con bcrypt (costo 10+) |
| RNF-04 | **Seguridad** | El backend rechaza credenciales inválidas con HTTP 401 |
| RNF-05 | **Seguridad** | No exponer detalles internos en errores al cliente (evitar stack traces) |
| RNF-06 | **Usabilidad** | La app debe ser usable en pantallas de 5" a 7" (iOS / Android) |
| RNF-07 | **Usabilidad** | Todos los botones de acción principales tienen feedback visual (color, animación) |
| RNF-08 | **Portabilidad** | Debe correr en iOS 13+ y Android 8+ |
| RNF-09 | **Mantenibilidad** | El backend sigue arquitectura en capas (controller → service → repository) |
| RNF-10 | **Mantenibilidad** | Separación de componentes reutilizables en frontend (context, api, constants) |
| RNF-11 | **Escalabilidad** | El backend y frontend son independientes, cada uno puede escalar por separado |
| RNF-12 | **Disponibilidad** | Desarrollo local: backend accesible vía ngrok desde cualquier red |
| RNF-13 | **Compatibilidad** | El backend expone API REST estándar (JSON, códigos HTTP) |
| RNF-14 | **Idioma** | UI íntegramente en español |

## 3. Actores del sistema

```
┌───────────────────┐         ┌───────────────────┐
│                   │         │                   │
│     CLIENTE       │         │    ADMINISTRADOR  │
│  (consumidor)     │         │  (dueño tienda)   │
│                   │         │                   │
└────────┬──────────┘         └────────┬──────────┘
         │                             │
         │ usa                         │ opera
         ▼                             ▼
┌──────────────────────────────────────────────┐
│             ICE CREAM APP                    │
│  (sistema móvil + backend + base de datos)   │
└──────────────────────────────────────────────┘
```

| Actor | Responsabilidad | Acceso |
|---|---|---|
| **Cliente** | Hacer pedidos de helado personalizado | App móvil (todas las pantallas excepto admin) |
| **Administrador** | Gestionar catálogo y pedidos | App móvil + panel admin protegido |
| **Sistema externo: WhatsApp** | Canal de confirmación de pedido | Recibe mensajes con el detalle del pedido |
| **Sistema externo: Base de datos** | Persistencia | Accedida solo por el backend |

## 4. Casos de uso principales

### CU-01: Cliente arma y envía un pedido

**Actor principal:** Cliente
**Precondiciones:** La app está abierta, hay conexión a Internet
**Postcondiciones:** El pedido queda guardado en la base de datos y WhatsApp abre con el resumen

**Flujo principal:**
1. El cliente abre la app y pulsa "INGRESAR"
2. El sistema muestra la pantalla Home con tab "Tamaño" activa
3. El cliente elige un tamaño (el item se agrega al carrito)
4. El cliente cambia a tab "Helados" y elige uno o más sabores
5. El cliente cambia a tab "Toppings" y elige uno o más toppings
6. El cliente abre la tab "Carrito"
7. El cliente ajusta cantidades con botones +/– o elimina items
8. El cliente diligencia el formulario (nombre, celular, dirección, barrio, notas)
9. El cliente pulsa "Enviar pedido a WhatsApp"
10. El sistema valida los campos obligatorios
11. El sistema hace `POST /pedidos` al backend con los items del carrito
12. El sistema abre WhatsApp con el resumen del pedido
13. El sistema vacía el carrito

**Flujos alternativos:**
- **10a** Si falta un campo obligatorio → el sistema muestra alerta y no envía
- **11a** Si el backend falla → el sistema muestra error y no abre WhatsApp
- **3a/4a/5a** El cliente puede reordenar la selección (elegir primero toppings y luego tamaño)

### CU-02: Admin inicia sesión

**Actor principal:** Administrador
**Precondiciones:** El admin conoce las credenciales
**Postcondiciones:** El admin accede al panel con las 3 tabs (Helados, Toppings, Pedidos)

**Flujo:**
1. El admin abre la app en la pantalla de bienvenida
2. El admin **mantiene presionado** el logo "ICECREAM"
3. El sistema abre la pantalla de acceso administrativo
4. El admin digita usuario y contraseña
5. El sistema envía `POST /auth/login`
6. El backend valida con bcrypt
7. Si es correcto → muestra el panel admin
8. Si no → muestra alerta "Usuario o contraseña incorrectos"

### CU-03: Admin crea un nuevo helado

**Actor principal:** Administrador autenticado
**Precondiciones:** El admin está en la tab "Helados"

**Flujo:**
1. El admin pulsa "Agregar Helado"
2. El sistema abre un modal con formulario
3. El admin ingresa: nombre, tipo (CREMA/AGUA/TAMAÑO), precio, disponible
4. El admin pulsa "Crear"
5. El sistema valida datos mínimos (nombre, precio)
6. El sistema hace `POST /productos`
7. El backend persiste y devuelve 201 + entidad
8. El sistema cierra el modal y refresca la lista
9. El nuevo helado aparece en la app del cliente inmediatamente (al recargar)

### CU-04: Admin cambia estado de un pedido

**Actor principal:** Administrador
**Precondiciones:** El admin está autenticado en la tab "Pedidos"

**Flujo:**
1. El admin ve la lista de pedidos
2. El admin pulsa el botón "Marcar como entregado" en un pedido pendiente
3. El sistema pide confirmación
4. El admin confirma
5. El sistema hace `PATCH /pedidos/:id/toggle`
6. El backend cambia el booleano `completado` y retorna el pedido actualizado
7. El sistema refresca la lista, el pedido cambia su badge a "Entregado" (verde)

### CU-05: Admin edita un topping

**Actor principal:** Administrador
**Flujo:**
1. Admin va a tab "Toppings", toca el icono de editar en un topping
2. Modal se abre con los datos precargados
3. Admin modifica cualquier campo
4. Pulsa "Actualizar" → `PATCH /toppings/:id`
5. Lista se refresca

### CU-06: Admin elimina un topping

**Actor principal:** Administrador
**Flujo:**
1. Admin toca icono de basura en un topping
2. Sistema pide confirmación
3. Confirmar → `DELETE /toppings/:id`
4. Topping desaparece de la lista y del catálogo del cliente

## 5. Diagrama de casos de uso

Ver archivo: [diagramas/01-casos-de-uso.md](diagramas/01-casos-de-uso.md)

## 6. Matriz de trazabilidad (requerimientos ↔ casos de uso)

| Requerimiento | CU-01 | CU-02 | CU-03 | CU-04 | CU-05 | CU-06 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| RF-01, RF-02, RF-03, RF-04 | ✅ | | | | | |
| RF-05, RF-06 | ✅ | | | | | |
| RF-07, RF-08 | ✅ | | | | | |
| RF-09, RF-10 | ✅ | | | | | |
| RF-11, RF-12 | | ✅ | | | | |
| RF-13, RF-14 | | | ✅ | | | |
| RF-15, RF-16 | | | | | ✅ | ✅ |
| RF-17, RF-18 | | | | ✅ | | |
| RF-19 | | | | ✅ | | |
