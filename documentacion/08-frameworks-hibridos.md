# Tema 8 · Frameworks móviles híbridos

## 1. Qué es un framework híbrido

Un framework móvil **híbrido** permite escribir código una sola vez que corre en **iOS y Android** (y a veces web) como si fuera nativo. Se distingue de los nativos (Swift, Kotlin) y de las webapps puras (HTML/CSS/JS en un navegador).

### Tipos de frameworks multiplataforma

| Tipo | Cómo funciona | Ejemplos |
|---|---|---|
| **WebView-based** | Empaqueta un navegador dentro de una app | Cordova, Ionic (modo clásico) |
| **Cross-compiled** | Compila a código nativo de cada plataforma | Flutter, React Native (nuevo motor) |
| **Bridge-based** | JavaScript corre en un motor, se comunica con UI nativa vía puente | React Native (motor antiguo) |

## 2. React Native + Expo (elegido)

### Arquitectura alto nivel

```
┌─────────────────────────────────────────────────────┐
│         TU CÓDIGO (JavaScript / TypeScript)         │
│     Componentes, lógica, estado, hooks, etc.        │
└─────────────────────────────────────────────────────┘
                         ↕
┌─────────────────────────────────────────────────────┐
│         MOTOR JS (Hermes o JSC)                     │
│     Ejecuta el código en el dispositivo             │
└─────────────────────────────────────────────────────┘
                         ↕
┌─────────────────────────────────────────────────────┐
│         PUENTE NATIVO / JSI / Fabric                │
│     Comunica JS con vistas e APIs nativas           │
└─────────────────────────────────────────────────────┘
                         ↕
┌───────────────────────┬─────────────────────────────┐
│    iOS (UIView)       │    Android (View)           │
│    APIs nativas       │    APIs nativas             │
└───────────────────────┴─────────────────────────────┘
```

**Lo importante:** `<View>` en RN se traduce a `UIView` en iOS y a `android.view.View` en Android. **No es una webview** — son widgets nativos reales.

### ¿Qué aporta Expo?

Expo envuelve React Native para eliminar el dolor de la configuración nativa (Xcode, Android Studio). Incluye:

- **Expo SDK:** APIs unificadas para cámara, ubicación, push, fonts, etc.
- **Expo Go:** app que ejecuta tu bundle sin tener que compilar a `.ipa/.apk`
- **Metro:** bundler con hot reload
- **Expo Router:** navegación basada en archivos
- **EAS Build:** servicio cloud para generar `.apk/.ipa` reales
- **EAS Submit:** subir a Play Store / App Store

## 3. Componentes principales usados en el proyecto

### Componentes base de React Native

| Componente | Equivalente web | Uso en el proyecto |
|---|---|---|
| `<View>` | `<div>` | Contenedores |
| `<Text>` | `<p>` / `<span>` | Texto (RN no renderiza strings sueltos) |
| `<ScrollView>` | `<div style="overflow:auto">` | Listas de productos |
| `<FlatList>` | — | Listas con virtualización (eficiente) |
| `<TouchableOpacity>` | `<button>` | Botones con feedback opacidad |
| `<TextInput>` | `<input>` | Formulario de carrito |
| `<Modal>` | `<dialog>` | Popup de crear/editar producto |
| `<Image>` / `<ImageBackground>` | `<img>` | Portada de bienvenida |
| `<StatusBar>` | — | Control del status bar del OS |
| `<ActivityIndicator>` | — | Spinner de carga |
| `<Animated.View>` | — | Animaciones |

### Componentes de Expo

| Paquete | Uso |
|---|---|
| `expo-router` | Navegación (tabs, stack, grupos) |
| `@expo/vector-icons` | Iconos (Ionicons en nuestro caso) |
| `expo-status-bar` | Configurar la barra de estado |

## 4. Hooks usados en el proyecto

```typescript
// State
const [categoriaActiva, setCategoriaActiva] = useState<Categoria>("tamano");

// Side effects
useEffect(() => { fetchDatos(); }, []);

// Refs (para animaciones)
const contentFade = useRef(new Animated.Value(1)).current;

// Custom hook (del CartContext)
const { cart, addToCart, removeFromCart, totalPrice } = useCart();

// Router de Expo
const router = useRouter();
```

## 5. Estado global con Context API

En vez de Redux o Zustand, el proyecto usa Context API (más simple para este tamaño). Implementado en [`context/CartContext.tsx`](../fly-app/ice-cream-app/context/CartContext.tsx):

```typescript
interface CartContextType {
  cart: CartItem[];
  addToCart: (product: CartItem) => void;
  removeFromCart: (id: string) => void;
  increaseQty: (id: string) => void;
  decreaseQty: (id: string) => void;
  clearCart: () => void;
  totalPrice: number;
}

const CartContext = createContext<CartContextType | undefined>(undefined);

export const CartProvider = ({ children }) => {
  const [cart, setCart] = useState<CartItem[]>([]);
  // ... funciones
  return (
    <CartContext.Provider value={{ cart, addToCart, ... }}>
      {children}
    </CartContext.Provider>
  );
};

export const useCart = () => useContext(CartContext);
```

El `CartProvider` envuelve toda la app en [`app/_layout.tsx`](../fly-app/ice-cream-app/app/_layout.tsx):
```tsx
<CartProvider>
  <Stack screenOptions={{ headerShown: false }}>
    ...
  </Stack>
</CartProvider>
```

## 6. Navegación con Expo Router

Estructura de archivos → rutas:

```
app/
├── _layout.tsx           ← layout root (con CartProvider)
├── index.tsx             ← ruta "/"  (pantalla de bienvenida)
├── admin.tsx             ← ruta "/admin" (modal)
└── (tabs)/               ← grupo con tab bar
    ├── _layout.tsx       ← tab bar (home + carrito)
    ├── home.tsx          ← ruta "/home"
    └── Carrito.tsx       ← ruta "/Carrito"
```

Navegación programática:
```typescript
const router = useRouter();
router.push("/admin");        // ir a pantalla
router.replace("/home");      // ir y reemplazar en history
router.back();                // volver
```

## 7. Animaciones nativas

Uso `Animated` de React Native (ejecuta animaciones en el hilo nativo con `useNativeDriver: true`):

### Fade-in escalonado del Welcome

```typescript
Animated.sequence([
  Animated.parallel([
    Animated.timing(titleFade, { toValue: 1, duration: 700, useNativeDriver: true }),
    Animated.timing(titleTranslate, { toValue: 0, duration: 700, useNativeDriver: true }),
  ]),
  Animated.timing(subtitleFade, { toValue: 1, duration: 500, useNativeDriver: true }),
  Animated.spring(buttonScale, { toValue: 1, friction: 6, tension: 60, useNativeDriver: true }),
]).start();
```

### Transición entre tabs del home

```typescript
useEffect(() => {
  contentFade.setValue(0);
  contentTranslate.setValue(20);
  Animated.parallel([
    Animated.timing(contentFade, { toValue: 1, duration: 350, useNativeDriver: true }),
    Animated.timing(contentTranslate, { toValue: 0, duration: 350, useNativeDriver: true }),
  ]).start();
}, [categoriaActiva]);
```

## 8. Estilos con StyleSheet

React Native usa un sistema similar a CSS-in-JS:

```typescript
const styles = StyleSheet.create({
  container: { flex: 1, backgroundColor: "#E0F7FA" },
  card: {
    padding: 16,
    borderRadius: 20,
    shadowColor: "#000",
    shadowOpacity: 0.1,
    elevation: 4,          // ← solo Android
  },
});
```

### Diferencias con CSS web

| CSS Web | React Native |
|---|---|
| `display: flex` | Por defecto (todas las Views) |
| `padding-top` | `paddingTop` (camelCase) |
| `color: blue` | `color: "#0000FF"` (o nombre exacto) |
| `%` / `rem` / `em` | `number` (density-independent pixels) |
| selectores | inline / stylesheet objects |
| `box-shadow` | `shadowColor` + `shadowOffset` + `shadowOpacity` (iOS) + `elevation` (Android) |

## 9. Comunicación con el backend

Axios centralizado en [`services/api.ts`](../fly-app/ice-cream-app/services/api.ts):

```typescript
import axios from "axios";

const api = axios.create({
  baseURL: "https://diagram-dreary-recount.ngrok-free.dev",
  timeout: 15000,
  headers: {
    "Content-Type": "application/json",
    Accept: "application/json",
    "ngrok-skip-browser-warning": "true",
  },
});

export default api;
```

Uso en componentes:
```typescript
// GET
const response = await api.get("/productos");
setProductos(response.data);

// POST
await api.post("/pedidos", { clienteNombre, telefono, direccion, items });

// PATCH
await api.patch(`/pedidos/${id}/toggle`);

// DELETE
await api.delete(`/toppings/${id}`);
```

## 10. Comparación: Hybrid vs Nativo

### Pros de React Native + Expo

- **Un solo código base** para iOS y Android
- **Desarrolladores JS/TS** pueden hacer móvil
- **Hot reload** acelera la iteración
- **Librerías JS reutilizables** (axios, date-fns, etc.)
- **Publicación a Play Store / App Store** con EAS Build sin tener Mac

### Contras

- **Performance** ligeramente inferior a nativo puro en UI muy demandante
- **Problemas raros** al integrar módulos nativos complejos
- **Actualizaciones** de RN / Expo requieren cuidado con breaking changes
- **Debug** del puente puede ser opaco

### Para qué tipo de apps funciona bien

- ✅ Apps CRUD (como la nuestra): catálogo + carrito + formularios
- ✅ Apps sociales con feeds
- ✅ Apps de contenido
- ❌ Apps gaming de alta performance
- ❌ Apps con procesamiento pesado de imagen/video sin librerías específicas

## 11. Publicación

Para publicar a Google Play Store se usa **EAS Build + EAS Submit**. Ver guía específica en [../play-store/guia-publicacion.md](play-store/guia-publicacion.md).

Flujo resumido:
1. `npm install -g eas-cli`
2. `eas login`
3. `eas build:configure`
4. `eas build --platform android --profile production`
5. Descargar el `.aab` resultante
6. Subirlo a Google Play Console
