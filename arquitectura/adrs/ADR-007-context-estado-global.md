# ADR-007 · React Context API para estado global del carrito

- **Fecha:** 2026-02-22
- **Estado:** Aceptado
- **Decisor:** Brandow Stivent

## Contexto

El carrito debe ser accesible desde múltiples pantallas:

- `home.tsx` agrega items
- `Carrito.tsx` lista, modifica y envía items
- Cualquier pantalla futura podría leerlo (p.ej. un badge en el tab bar con total de items)

Necesitamos un mecanismo de **estado compartido** (estado global).

## Decisión

Se usa **React Context API** + **`useState`** (Context Provider nativo de React, sin librerías adicionales).

Implementación: [`context/CartContext.tsx`](../../fly-app/ice-cream-app/context/CartContext.tsx).

## Alternativas consideradas

| Alternativa | Pros | Contras | Veredicto |
|---|---|---|---|
| **Props drilling** | Explícito | Pesadilla con 3+ niveles de componentes | ❌ |
| **Redux** | Estándar histórico, devtools | Boilerplate enorme para un carrito simple | ❌ Overkill |
| **Zustand** | Simple, moderno | Una dep extra | ⚠️ Considerado |
| **Jotai / Recoil** | Atómico, reactivo | Menos popular, curva | ❌ |
| **MobX** | Reactivo declarativo | Paradigma distinto, aprender extra | ❌ |
| **React Context + useState** | Nativo React, sin deps | Re-renders si no se cuida (no crítico aquí) | ✅ Elegido |

## Justificación

1. **Nativo de React:** no agrega dependencias.
2. **Suficiente para este caso:** el carrito tiene <10 items típicamente, los re-renders no son problema.
3. **API simple:** solo `createContext`, `Provider`, `useContext`.
4. **Curva cero:** cualquier dev que sepa React lo entiende.
5. **Más que suficiente** para un proyecto académico.

## Implementación

```typescript
// context/CartContext.tsx

interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

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

  const addToCart = (product: CartItem) => {
    setCart(prev => {
      const existing = prev.find(item => item.id === product.id);
      if (existing) {
        return prev.map(item =>
          item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item
        );
      }
      return [...prev, { ...product, quantity: 1 }];
    });
  };

  // ... otras funciones

  const totalPrice = cart.reduce((acc, item) => acc + item.price * item.quantity, 0);

  return (
    <CartContext.Provider value={{ cart, addToCart, ..., totalPrice }}>
      {children}
    </CartContext.Provider>
  );
};

export const useCart = () => {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error("useCart debe usarse dentro de un CartProvider");
  }
  return context;
};
```

### Registro en el layout raíz

```tsx
// app/_layout.tsx
<CartProvider>
  <Stack screenOptions={{ headerShown: false }}>
    ...
  </Stack>
</CartProvider>
```

### Uso en componentes

```tsx
// Cualquier componente
const { cart, addToCart, totalPrice } = useCart();
```

## Consecuencias

### Positivas

- **Sin librerías externas**
- **Tipado fuerte** con TypeScript
- **Hooks limpios** (`useCart()` hace todo)
- **Testeable:** fácil inyectar un Provider mock en tests
- **Educativo:** el docente ve que entendemos Context API

### Negativas

- **Re-renders:** cualquier cambio en `cart` re-renderiza a todos los consumidores del contexto. Para este proyecto no es problema porque el árbol es pequeño y los items son pocos.
- **No persiste:** si la app se cierra, el carrito se pierde. Solución futura: guardar en `AsyncStorage` o expo-secure-store.
- **No hay middleware** (logging, analytics de acciones). Para eso Redux sería mejor, pero aquí no hace falta.

## Alternativas si el proyecto crece

Si en el futuro hay muchas más "slices" de estado global (autenticación, tema, idioma, etc.), migrar a **Zustand** sería el siguiente paso natural:

```typescript
import create from "zustand";

export const useCart = create((set) => ({
  cart: [],
  addToCart: (product) => set((state) => ({ cart: [...state.cart, product] })),
}));
```

Zustand elimina el Provider y mejora rendimiento con selectores.
