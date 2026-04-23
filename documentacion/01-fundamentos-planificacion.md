# Tema 1 · Fundamentos y planificación de proyectos móviles

## 1. Descripción del proyecto

**Ice Cream App** es una aplicación móvil para una heladería de Neiva, Huila, que permite al cliente armar su helado personalizado (tamaño + sabor + toppings) desde su celular y enviar el pedido por WhatsApp, quedando también registrado en la base de datos de la heladería.

El proyecto incluye un **panel administrativo oculto** (accesible mediante un gesto secreto) para que el dueño pueda gestionar el menú y ver los pedidos entrantes.

## 2. Motivación y problema que resuelve

Las heladerías pequeñas de Neiva no cuentan con catálogos digitales propios. Los pedidos se hacen por WhatsApp en texto plano, donde el cliente describe de memoria los sabores disponibles, toppings y precios. Esto causa:

- **Pérdida de pedidos** por descripciones incompletas o ambiguas
- **Tiempos de atención largos** por ida y vuelta aclarando productos
- **Difícil seguimiento** para el dueño (no hay histórico digital)
- **Imposibilidad de análisis** de productos más pedidos

La solución propuesta le ofrece al cliente un catálogo visual interactivo, y al dueño un registro centralizado de pedidos con estado (pendiente/entregado).

## 3. Stakeholders

| Stakeholder | Rol | Interés principal |
|---|---|---|
| Cliente final | Consumidor de helados | Armar su helado de forma visual y rápida |
| Administrador | Dueño de la heladería | Gestionar menú y ver pedidos organizados |
| Desarrollador | Estudiante (Brandow Stivent) | Completar el proyecto académico y aprender |
| Docente | Evaluador | Verificar cumplimiento de requisitos académicos |

## 4. Objetivos

### Objetivo general

Desarrollar una aplicación móvil híbrida (iOS/Android) para la venta personalizada de helados, conectada a un backend REST y base de datos relacional.

### Objetivos específicos

1. Implementar un flujo de cliente para elegir tamaño, sabor y toppings con UI atractiva.
2. Construir un panel administrativo con autenticación y CRUD para gestionar productos, toppings y ver pedidos.
3. Exponer un API REST que persista todo en PostgreSQL.
4. Integrar WhatsApp como canal de confirmación de pedidos.
5. Documentar la arquitectura con diagramas UML y C4.

## 5. Alcance (Scope)

### Funcionalidades **dentro** del alcance

- Catálogo con tamaños, helados y toppings
- Carrito con controles +/– y eliminación
- Formulario de datos de cliente (nombre, teléfono, dirección, barrio, notas)
- Envío de pedido al backend + WhatsApp
- Login admin con bcrypt
- CRUD de productos y toppings desde el admin
- Listado de pedidos con detalle y cambio de estado (pendiente ↔ entregado)

### Funcionalidades **fuera** del alcance (v1)

- Pagos en línea (PSE, tarjetas)
- Notificaciones push
- Registro de múltiples usuarios administradores
- Sistema de fidelización / cupones
- Tracking GPS del domiciliario
- Internacionalización (solo español)

## 6. Planificación temporal

El proyecto se planificó en **4 entregables iterativos** de 2 semanas cada uno:

| Iteración | Semanas | Entregable principal |
|---|---|---|
| Iteración 1 | 1–2 | Esqueleto del backend (NestJS + Prisma + modelo BD) y de la app (Expo + navegación tabs) |
| Iteración 2 | 3–4 | Catálogo conectado al backend, carrito funcional con estado global |
| Iteración 3 | 5–6 | Autenticación admin + CRUD completo + panel de pedidos |
| Iteración 4 | 7–8 | Rediseño visual (colores, animaciones), envío de pedidos al backend, ngrok, documentación |

## 7. Recursos necesarios

### Humanos
- 1 desarrollador full-stack (yo)

### Técnicos
- PC Windows 11 con Node.js 20+, Docker Desktop, Git
- Celular iPhone / Android con Expo Go instalado
- Conexión a Internet (para ngrok y descargas)

### Herramientas
- **IDE:** VS Code (+ Claude Code)
- **Control de versiones:** Git (dos repositorios locales)
- **Testing manual:** Postman / `curl` / navegador
- **Diagramas:** Mermaid (texto → imagen)

## 8. Riesgos identificados

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| Prisma no carga `.env` en runtime | Media | Alto | Agregar `import 'dotenv/config'` en `main.ts` |
| Celular no alcanza backend en LAN por firewall | Alta | Alto | Reglas de firewall + ngrok como fallback |
| IP local cambia entre sesiones | Alta | Medio | Usar variables en `Config.ts`, centralizar |
| Docker credential helper falla | Baja | Medio | Corregir `credsStore` → `credStore` |
| Drift entre schema Prisma y BD | Media | Medio | `prisma migrate reset` con consentimiento |

## 9. Criterios de aceptación

1. El cliente puede hacer un pedido completo (tamaño + sabor + topping) en menos de 1 minuto desde la app.
2. El administrador ve el pedido en el panel con fecha/hora, cliente, detalle y total.
3. El admin puede crear un nuevo helado o topping que aparece inmediatamente en la app del cliente.
4. Todas las contraseñas están hasheadas con bcrypt (no en texto plano).
5. La documentación incluye diagramas UML, C4, ERD y ADRs.
