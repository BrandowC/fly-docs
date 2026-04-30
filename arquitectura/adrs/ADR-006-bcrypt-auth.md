# ADR-006 · bcrypt para hashing de contraseñas

- **Fecha:** 2026-02-20
- **Estado:** Aceptado
- **Decisor:** Brandow Stivent

## Contexto

El admin debe autenticarse con usuario y contraseña. Nunca se deben almacenar contraseñas en texto plano: si la BD se filtra, todas las credenciales quedarían expuestas.

Opciones de hashing:
- MD5 / SHA1 (obsoletos, inseguros)
- SHA256 / SHA512 (rápidos, vulnerables a brute force)
- PBKDF2
- bcrypt
- scrypt
- Argon2

## Decisión

Se usa **bcryptjs** (implementación JS pura de bcrypt) con un cost factor de **10**.

## Alternativas consideradas

| Alternativa | Pros | Contras | Veredicto |
|---|---|---|---|
| **MD5** | Rápido | **Roto criptográficamente** | ❌ NUNCA |
| **SHA-256** | Rápido, estándar | Muy rápido → vulnerable a brute force con GPUs | ❌ |
| **PBKDF2** | Estándar NIST | Más débil que bcrypt contra ASICs | ❌ |
| **Argon2** | Estado del arte, ganador de PHC | Requiere binding nativo, más complejo en JS | ❌ |
| **bcrypt (node-bcrypt)** | Sólido, clásico | Requiere compilación nativa (node-gyp) | ❌ |
| **bcryptjs** | JS puro, sin deps nativas, compatible API bcrypt | Un poco más lento que bcrypt nativo | ✅ Elegido |

## Justificación

1. **Resistente a brute force:** bcrypt está diseñado para ser lento (costoso), haciendo inviable probar millones de contraseñas por segundo.
2. **Cost factor ajustable:** podemos subir el costo a medida que el hardware mejora.
3. **Salt automático:** bcrypt incluye salt en el hash, sin gestión manual.
4. **Sin binding nativo (bcryptjs):** evita problemas con `node-gyp` en Windows.
5. **API simple:** solo dos funciones: `hash()` y `compare()`.

## Implementación en el proyecto

### Seed (almacenar hash)

```typescript
// prisma/seed.ts
import * as bcrypt from "bcryptjs";

const hashedPassword = await bcrypt.hash("admin123", 10);

await prisma.admin.upsert({
  where: { username: "admin" },
  update: {},
  create: {
    username: "admin",
    password: hashedPassword,
  },
});
```

### Login (verificar)

```typescript
// src/auth/auth.service.ts
async login(username: string, password: string) {
  const admin = await this.prisma.admin.findUnique({ where: { username } });
  if (!admin) {
    throw new UnauthorizedException("Usuario o contraseña incorrectos");
  }

  const isPasswordValid = await bcrypt.compare(password, admin.password);
  if (!isPasswordValid) {
    throw new UnauthorizedException("Usuario o contraseña incorrectos");
  }

  return { success: true, ... };
}
```

### Lo que queda almacenado en BD

```
$2a$10$jzpwrzYxbSVpGPWT0rM5zOSr0ogfbxS3LQHyDtRthHcXwmtxwHwei
│  │ │                                                      
│  │ │
│  │ └── Salt + hash derivado (base64)
│  └──── Cost factor (10 = 2^10 = 1024 iteraciones)
└─────── Versión del algoritmo bcrypt
```

## Consecuencias

### Positivas

- Las contraseñas **nunca** están en texto plano en la BD
- Mismo input produce hash diferente (por el salt aleatorio)
- Cost 10 tarda ~100ms en hardware moderno (aceptable para login manual)
- Sin dependencias nativas → instala en Windows/Linux/Mac por igual

### Negativas

- **Login tarda ~100ms** (un usuario no lo nota, pero un bot que prueba millones sí)
- El cost factor 10 puede ser bajo en 2030+; habría que migrar a costo 12-14
- No permite "recuperar" contraseñas (es por diseño) → hay que hacer reset

## Implicaciones técnicas

### Por qué el mensaje de error es genérico

```typescript
if (!admin) throw new UnauthorizedException("Usuario o contraseña incorrectos");
if (!isPasswordValid) throw new UnauthorizedException("Usuario o contraseña incorrectos");
```

Ambos errores dicen lo mismo. Esto previene **username enumeration attacks** (distinguir si existe el usuario o no).

### Migración futura

Si se requiere mayor seguridad en el futuro:
1. Aumentar cost factor: `bcrypt.hash(password, 12)` (4x más lento = 4x más seguro)
2. Migrar a Argon2id con una estrategia lazy (al login, si el hash es bcrypt, rehash a argon2)
3. Agregar 2FA (autenticador de Google, SMS, etc.)

## Referencias

- [OWASP Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [bcryptjs en npm](https://www.npmjs.com/package/bcryptjs)
