# ADR-003 · Dos repositorios Git separados (front y back)

- **Fecha:** 2026-02-10
- **Estado:** Aceptado
- **Decisor:** Brandow Stivent
- **Relacionados:** [arquitectura-distribuida.md](../arquitectura-distribuida.md)

## Contexto

El sistema tiene dos piezas claramente diferenciadas:

- **Backend REST** (NestJS + Prisma)
- **App móvil** (React Native + Expo)

Hay que decidir si alojar ambos en:

1. **Un solo repositorio** (monorepo)
2. **Repositorios separados** (polirepositorio)

## Decisión

Se crean **dos repositorios Git independientes**:

- `fly-api/` → contiene únicamente el backend NestJS (`ice-cream/`)
- `fly-app/` → contiene únicamente la app Expo (`ice-cream-app/`)

Ambos están bajo la carpeta `PROYECTO/` en el disco local, pero cada uno tiene su propio `.git`.

## Alternativas consideradas

| Alternativa | Pros | Contras | Veredicto |
|---|---|---|---|
| **Monorepo con Lerna/Turborepo** | Tooling compartido, deps unificadas, commits atómicos cross-stack | Configuración extra, tooling a mantener | ❌ Overkill para un proyecto de 2 personas |
| **Monorepo simple con 2 carpetas y un `.git` root** | Un solo historial, fácil sincronizar | Mezcla commits, triggers de CI difíciles | ❌ |
| **Dos repositorios separados** | Independencia de despliegue, historia limpia, CI/CD por repo | Sincronización manual de contratos API | ✅ Elegido |

## Justificación

### Por qué separado

1. **Dominios distintos:** el backend y el frontend evolucionan a ritmos diferentes. Un cambio de color en un botón no debería entrar en el historial del backend.

2. **Ciclo de vida distinto:** el backend se despliega (p.ej. a un VPS); la app se publica (a Play Store). Los pipelines son completamente diferentes.

3. **Permisos:** si mañana se suma otro desarrollador que solo trabaja en la app, se le puede dar acceso solo a `fly-app`.

4. **Tooling independiente:** el backend usa `nest build`, el front usa `npx expo start`. No hay beneficio de compartir configuración.

5. **Educativo (contexto de proyecto académico):** Demuestra comprensión de arquitecturas distribuidas. El docente puede revisar cada repo por separado.

### Por qué NO monorepo

- **Complejidad adicional** (Turborepo, workspaces) que no aporta valor en este tamaño
- **Scripts compartidos innecesarios:** no hay código compartido entre front y back
- **CI/CD más complejo:** ¿cómo decidir qué desplegar cuando un commit toca ambos?

## Consecuencias

### Positivas

- Historial de commits limpio por dominio
- Cada repo tiene su propio `package.json`, `.gitignore`, `README`
- Cada uno puede tener su propio pipeline de CI (GitHub Actions, etc.)
- Facilidad para open-source por separado (hipotéticamente)
- Documentación por dominio

### Negativas

- **Sincronización manual** del contrato API: si el backend cambia un DTO, el frontend debe actualizarse manualmente
- **Duplicación de deps** pequeña (axios en ambos, typescript, etc.)
- No se puede hacer un commit atómico que toque ambos
- Dos pasos de deploy en orden (primero back, luego front)

### Mitigación

- Convención de commit `feat(api): agregar campo X` + commit correspondiente `feat(app): consumir campo X`
- Documentar cambios breaking en CHANGELOG
- En producción: versionar el API (`/v1/`, `/v2/`) para evitar romper clients desplegados

## Implicaciones técnicas

- Cada repo tiene su propio `.gitignore` específico (node_modules, dist, .env, etc.)
- Cada repo tiene su propio README con instrucciones
- La **documentación transversal** (esta) vive en una tercera carpeta `documentacion/` (sin git propio, o como repo aparte)
