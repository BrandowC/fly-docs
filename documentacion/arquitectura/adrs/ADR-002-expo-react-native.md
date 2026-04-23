# ADR-002 · React Native + Expo para la app móvil

- **Fecha:** 2026-02-09
- **Estado:** Aceptado
- **Decisor:** Brandow Stivent

## Contexto

Se necesita una app móvil que:

- Corra en **iOS y Android** (no solo una plataforma)
- Permita probar en el celular del desarrollador sin tener Mac ni publicar
- Use un lenguaje que el desarrollador ya conoce (JS/TS)
- Permita publicar en Play Store sin complicaciones adicionales
- Tenga acceso a APIs de plataforma (cámara, notificaciones) si se necesita en el futuro

## Decisión

Se adopta **React Native** como motor móvil con **Expo SDK** como plataforma envolvente.

## Alternativas consideradas

| Alternativa | Pros | Contras | Veredicto |
|---|---|---|---|
| **Swift nativo** | Máximo rendimiento iOS | Solo iOS, requiere Mac | ❌ |
| **Kotlin nativo** | Máximo rendimiento Android | Solo Android, dos bases de código | ❌ |
| **Flutter** | UI bella, un solo lenguaje (Dart) | Dart no conocido, widgets distintos a RN | ❌ |
| **Ionic / Capacitor** | Usa HTML/CSS/JS estándar | WebView → performance inferior | ❌ |
| **React Native "bare"** | Máximo control | Config nativa (Xcode/Android Studio) compleja | ❌ |
| **React Native + Expo** | RN nativo + herramientas listas (Expo Go, EAS Build) | Algunas libs nativas avanzadas no soportadas | ✅ Elegido |

## Decisión específica sobre Expo

Se usa **Expo Managed Workflow** (no Bare Workflow) porque:

- No se necesitan librerías nativas custom
- Expo Go permite probar en celular físico sin build nativo
- EAS Build genera `.apk/.aab` y `.ipa` en la nube (no requiere Mac)
- Los módulos de Expo cubren todos los casos de uso del proyecto (Router, iconos, fonts, etc.)

## Consecuencias

### Positivas

- **Tiempo de desarrollo:** Expo Go ahorra horas de configuración inicial
- **Cross-platform gratis:** mismo código en iOS y Android
- **Hot reload estable** → iteración muy rápida sobre UI
- **Publicación simplificada:** `eas build --platform android` produce `.aab` listo para Play Store
- **Ecosistema de Expo SDK** (expo-router, expo-status-bar, @expo/vector-icons) cubre necesidades comunes
- **TypeScript nativo** sin configuración adicional

### Negativas

- **Puente JS-nativo:** performance inferior a código 100% nativo en UIs muy complejas
- **Dependencia de Expo** → si alguna vez hay que salir del Managed Workflow (eject), toma tiempo
- **Tamaño del bundle inicial:** incluye runtime de Expo
- **Módulos nativos custom:** requieren Bare Workflow (no aplicable aquí)

## Implicaciones técnicas

- Estructura basada en `app/` con Expo Router
- Estilos con `StyleSheet.create` (sin CSS)
- Navegación con `useRouter()` y `Link`
- Publicación vía EAS: `eas build && eas submit`
- Desarrollo requiere Node, npm, Expo CLI (vía npx) y celular con Expo Go
