# C4 · Nivel 1 · Diagrama de Contexto

El nivel de contexto del modelo C4 muestra el **sistema como una caja negra** y sus relaciones con **personas y sistemas externos**.

## Diagrama (Mermaid)

```mermaid
flowchart TB
    Cliente(["👤 Cliente<br/>(consumidor de helados)"])
    Admin(["👔 Administrador<br/>(dueño heladería)"])

    subgraph Sistema["🍦 Ice Cream App (Sistema)"]
        Core[["App móvil + Backend + BD"]]
    end

    WhatsApp[["💬 WhatsApp<br/>(sistema externo)"]]
    Ngrok[["🌐 ngrok<br/>(tunneling dev)"]]

    Cliente -->|Arma y envía pedido<br/>usando la app móvil| Sistema
    Admin -->|Gestiona catálogo y pedidos<br/>desde el panel oculto| Sistema

    Sistema -->|Envía resumen de pedido| WhatsApp
    Sistema -.->|Expone el backend al Internet<br/>(solo en desarrollo)| Ngrok

    classDef user fill:#08427B,stroke:#052D5A,color:#fff
    classDef system fill:#1168BD,stroke:#0B4884,color:#fff
    classDef external fill:#999,stroke:#666,color:#fff

    class Cliente,Admin user
    class Sistema system
    class WhatsApp,Ngrok external
```

## Elementos del diagrama

### Personas (actores humanos)

| Persona | Descripción |
|---|---|
| **Cliente** | Usuario final que arma su helado personalizado y hace un pedido |
| **Administrador** | Dueño de la heladería que gestiona menú y ve pedidos |

### Sistema

| Sistema | Descripción |
|---|---|
| **Ice Cream App** | Sistema completo (app móvil, backend, base de datos) que permite hacer pedidos |

### Sistemas externos

| Sistema | Descripción |
|---|---|
| **WhatsApp** | Canal de confirmación del pedido. El sistema abre WhatsApp con un deep-link que contiene el resumen del pedido. |
| **ngrok** | Tunneling HTTPS que expone el backend local al Internet, solo usado en desarrollo para que el celular del cliente pueda alcanzar el backend cuando no está en la misma WiFi. |

## Interacciones

1. El **Cliente** abre la app móvil y arma su pedido eligiendo tamaño, sabor y toppings.
2. El **Cliente** envía el pedido → el sistema lo guarda y abre WhatsApp con el resumen para enviar a la heladería.
3. El **Administrador** (la misma persona/dueño) accede al panel oculto mediante un long-press en el logo.
4. El **Administrador** gestiona el catálogo y revisa pedidos entrantes.
5. El sistema usa **ngrok** (solo en desarrollo) para ser alcanzable desde Internet.
