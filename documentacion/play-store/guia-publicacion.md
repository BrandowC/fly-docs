# Guía de publicación en Google Play Store

## 1. Requisitos previos

- Cuenta de Google Play Developer (pago único de **USD $25**, una vez en la vida): https://play.google.com/console
- Node.js 20+ y cuenta de Expo (gratis): https://expo.dev/signup
- Icono y splash de la app (1024×1024 PNG)
- Al menos 2 capturas de pantalla del celular (1080×1920 o superior)
- Descripción corta (80 chars) y larga (4000 chars)
- Política de privacidad pública (puede ser un Gist en GitHub)

## 2. Preparar el proyecto Expo

### 2.1 Instalar EAS CLI

```bash
npm install -g eas-cli
eas login
```

### 2.2 Configurar `app.json`

Editar `fly-app/ice-cream-app/app.json`:

```json
{
  "expo": {
    "name": "Ice Cream App",
    "slug": "ice-cream-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/images/icon.png",
    "splash": {
      "image": "./assets/images/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#FFE4EE"
    },
    "android": {
      "package": "com.brandowstivent.icecream",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/images/adaptive-icon.png",
        "backgroundColor": "#FF4D94"
      }
    }
  }
}
```

⚠️ **Importante:** el `package` debe ser único en Play Store (sintaxis Java inversa).

### 2.3 Inicializar EAS

```bash
cd fly-app/ice-cream-app
eas build:configure
```

Esto crea `eas.json` con perfiles:

```json
{
  "build": {
    "development": { "developmentClient": true },
    "preview": { "distribution": "internal", "android": { "buildType": "apk" } },
    "production": { "android": { "buildType": "app-bundle" } }
  },
  "submit": { "production": {} }
}
```

## 3. Generar el AAB (Android App Bundle)

```bash
eas build --platform android --profile production
```

- Tarda 10-20 minutos en los servidores de Expo
- Al terminar, te da un link para descargar el `.aab`
- **Guarda ese archivo** (será lo que subes a Play Store)

## 4. Crear la app en Google Play Console

1. Ir a https://play.google.com/console
2. Click en **"Crear aplicación"**
3. Llenar:
   - Nombre: **Ice Cream App**
   - Idioma predeterminado: Español (Colombia)
   - Tipo: **Aplicación**
   - Gratis / De pago: **Gratis**
4. Aceptar los contratos de desarrollador

## 5. Llenar el formulario de la ficha

### 5.1 Detalles principales

- **Título corto:** Ice Cream App
- **Descripción breve (80 chars):** Arma tu helado personalizado: tamaño, sabor y toppings. Pedido por WhatsApp.
- **Descripción completa (usa este texto):**

```
Ice Cream App es la forma más deliciosa y fácil de pedir helados personalizados
en Neiva, Huila.

✨ CARACTERÍSTICAS ✨

🍦 Elige tu tamaño: Pequeño, Mediano o Grande (¡los vasos son GRATIS!)
🍓 Más de 8 sabores entre helados de crema y agua
✨ 12 toppings para personalizar a tu gusto
🛒 Carrito intuitivo con suma y resta de cantidades
🏍️ Envío directo a tu dirección
💬 Confirmación por WhatsApp

Sabores: Fresa, Chocolate, Vainilla, Mora, Napolitano, Limón, Maracuyá y Coco.
Toppings: Arequipe, Oreo, Gomitas, Maní, Chocolate derretido y muchos más.

Descarga ya y crea tu helado perfecto.
```

### 5.2 Gráficos de la ficha

| Elemento | Requisito |
|---|---|
| Icono | 512×512 PNG |
| Gráfico de funciones | 1024×500 PNG |
| Capturas del teléfono (mín. 2) | 1080×1920 o similar |
| Video promocional (opcional) | YouTube URL |

### 5.3 Categoría y etiquetas

- **Categoría:** Comida y bebida
- **Clasificación de contenido:** Para todos
- **Etiquetas:** helados, pedidos, domicilio, Neiva

## 6. Cuestionarios obligatorios

### 6.1 Clasificación de contenido

- Responder: no hay violencia, no gambling, no sexo, no drogas
- Obtendrás: "PEGI 3 / USK 0 / Para todos"

### 6.2 Público objetivo

- Público: adultos (13+)
- Dirigido a niños: **No**

### 6.3 Anuncios

- ¿Contiene anuncios?: **No**

### 6.4 Política de privacidad

Necesitas una URL pública. Puedes crearla rápido en un GitHub Gist:

```
Política de privacidad - Ice Cream App

Esta app recolecta:
- Nombre, teléfono y dirección ingresados voluntariamente para el pedido
- No se comparten con terceros
- No usa analytics ni cookies
- Datos guardados únicamente en el servidor del negocio

Contacto: manueldev@grupohayplan.com
```

Sube esto a https://gist.github.com y copia el link **raw** al campo de la Console.

### 6.5 Seguridad de datos

Declara:
- **Datos recolectados:** Nombre, número de teléfono, dirección postal
- **Uso:** Funcionalidad de la app (pedidos)
- **Cifrado en tránsito:** Sí (HTTPS)
- **El usuario puede solicitar eliminación:** Sí (contactando al email)

## 7. Subir el AAB

1. En la Console: **Versiones → Producción → Crear una nueva versión**
2. Subir el archivo `.aab` generado por EAS
3. Poner notas de la versión:
   ```
   Primera versión pública de Ice Cream App.
   - Catálogo de tamaños, helados y toppings
   - Carrito con personalización
   - Envío de pedidos por WhatsApp
   - Panel administrativo para gestión
   ```
4. Revisar → Iniciar lanzamiento

## 8. Tiempos de revisión

- **Publicación abierta (producción):** 1-7 días la primera vez
- **Actualizaciones posteriores:** horas a 2 días
- **Cuentas nuevas:** Google revisa manualmente la app (más estricto)

## 9. Alternativa: solo pruebas internas (más rápido)

Si solo necesitas demostrar la app (no tenerla pública):

1. Crear versión en **"Pruebas internas"** (no Producción)
2. Añadir una lista de testers (hasta 100 emails)
3. Subir el AAB
4. Los testers reciben un link para instalarla
5. **Disponible en minutos** (no requiere revisión extensa)

## 10. Comandos resumen

```bash
# Setup (una vez)
npm install -g eas-cli
eas login
eas build:configure

# Build de producción
eas build --platform android --profile production

# Submit automático (opcional, si ya tienes cuenta Play conectada)
eas submit --platform android --latest
```

## 11. Checklist antes de subir

- [ ] `app.json` con nombre, versión, package y iconos correctos
- [ ] Icono 1024×1024 en `assets/images/icon.png`
- [ ] Splash en `assets/images/splash.png`
- [ ] Adaptive icon en `assets/images/adaptive-icon.png`
- [ ] Backend en producción (no localhost ni ngrok-free.dev que expira) — migrar a un VPS o Railway
- [ ] `constants/Config.ts` apunta a la URL de producción
- [ ] 2+ screenshots en el celular
- [ ] Descripción corta y larga redactadas
- [ ] Política de privacidad publicada
- [ ] Cuenta Play Developer activada (pago de USD $25 hecho)

## 12. Costos

| Concepto | Costo |
|---|---|
| Cuenta Play Developer | USD $25 (pago único) |
| EAS Build en Expo | Gratis (plan free: 30 builds/mes) |
| Hosting backend producción | USD $5-10/mes (Railway, Render, Hetzner) |
| Dominio propio (opcional) | USD $10/año |
