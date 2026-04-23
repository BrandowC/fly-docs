# Arquitectura del Backend

Diagrama que muestra la organización interna del backend NestJS: capas, módulos y flujo de datos.

## Diagrama de capas (Mermaid)

```mermaid
flowchart TB
    Client["📱 Cliente HTTP<br/>(App móvil Expo)"]

    subgraph Backend["Backend NestJS :3000"]
        direction TB

        CORS["<b>Middleware CORS</b><br/>enableCors()"]

        Pipes["<b>Global Validation Pipe</b><br/>class-validator<br/>whitelist + transform"]

        subgraph Presentation["🎨 Capa de Presentación - Controllers"]
            direction LR
            C1["AuthController"]
            C2["ProductosController"]
            C3["ToppingsController"]
            C4["PedidosController"]
            C5["AdminController"]
        end

        subgraph Business["⚙️ Capa de Negocio - Services"]
            direction LR
            S1["AuthService"]
            S2["ProductosService"]
            S3["ToppingsService"]
            S4["PedidosService"]
            S5["AdminService"]
        end

        subgraph DataAccess["💾 Capa de Datos - Repositories"]
            direction LR
            R2["ProductosRepository"]
            R3["ToppingsRepository"]
            R4["PedidosRepository"]
            R5["AdminRepository"]
        end

        subgraph Infrastructure["🔧 Infraestructura"]
            Prisma["PrismaService<br/>(singleton)"]
            DTOs["DTOs<br/>+ class-validator"]
            Guards["Guards<br/>(ApiKeyGuard)"]
        end
    end

    DB[("🐘 PostgreSQL<br/>Docker :5432")]

    Client -->|HTTP Request| CORS
    CORS --> Pipes
    Pipes --> Presentation
    DTOs -.valida.-> Pipes
    Guards -.protege.-> C5

    C1 --> S1
    C2 --> S2
    C3 --> S3
    C4 --> S4
    C5 --> S5

    S1 --> Prisma
    S2 --> R2
    S3 --> R3
    S4 --> R4
    S4 -.->|usa| S2
    S4 -.->|usa| S3
    S5 --> R5

    R2 --> Prisma
    R3 --> Prisma
    R4 --> Prisma
    R5 --> Prisma

    Prisma --> DB

    classDef client fill:#E8F8F5,stroke:#16A085
    classDef layer fill:#D6EAF8,stroke:#2874A6
    classDef infra fill:#FDEBD0,stroke:#D68910
    classDef db fill:#F8D7DA,stroke:#C0392B

    class Client client
    class CORS,Pipes infra
    class Prisma,DTOs,Guards infra
    class DB db
```

## Estructura de directorios

```
fly-api/ice-cream/
├── prisma/
│   ├── schema.prisma              ← Modelo declarativo
│   ├── seed.ts                    ← Datos iniciales
│   └── migrations/                ← Historial SQL
├── src/
│   ├── main.ts                    ← Bootstrap (dotenv, CORS, Pipes)
│   ├── app.module.ts              ← Registra todos los módulos
│   │
│   ├── prisma/
│   │   └── prisma.service.ts      ← Singleton extendido
│   │
│   ├── auth/
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   └── auth.service.ts        ← sin repository (es simple)
│   │
│   ├── productos/
│   │   ├── productos.module.ts
│   │   ├── productos.controller.ts
│   │   ├── productos.service.ts
│   │   ├── productos.repository.ts
│   │   └── dto/
│   │       ├── create-producto.dto.ts
│   │       └── update-producto.dto.ts
│   │
│   ├── topping/                   ← misma estructura
│   ├── pedidos/                   ← misma estructura
│   │
│   └── admin/
│       ├── admin.module.ts
│       ├── admin.controller.ts    ← protegido con guard
│       ├── admin.service.ts
│       ├── admin.repository.ts
│       └── guards/
│           └── api-key.guard.ts
├── docker-compose.yml
├── .env
├── prisma.config.ts
└── package.json
```

## Módulos y su composición

```mermaid
flowchart TB
    App[AppModule]

    App --> Prod[ProductosModule]
    App --> Top[ToppingsModule]
    App --> Ped[PedidosModule]
    App --> Adm[AdminModule]
    App --> Auth[AuthModule]

    Prod --> PC[ProductosController]
    Prod --> PS[ProductosService]
    Prod --> PR[ProductosRepository]
    Prod --> PP[PrismaService]

    Top --> TC[ToppingsController]
    Top --> TS[ToppingsService]
    Top --> TR[ToppingsRepository]
    Top --> TP[PrismaService]

    Ped --> PedC[PedidosController]
    Ped --> PedS[PedidosService]
    Ped --> PedR[PedidosRepository]
    Ped --> PedP[PrismaService]
    Ped -.imports.-> Prod
    Ped -.imports.-> Top

    Adm --> AdmC[AdminController]
    Adm --> AdmS[AdminService]
    Adm --> AdmR[AdminRepository]
    Adm --> AdmG[ApiKeyGuard]
    Adm --> AdmP[PrismaService]
    Adm -.imports.-> Prod
    Adm -.imports.-> Top

    Auth --> AC[AuthController]
    Auth --> AS[AuthService]
    Auth --> AP[PrismaService]
```

## Inyección de dependencias (DI)

NestJS usa DI por constructor. Ejemplo completo:

```typescript
// productos.repository.ts
@Injectable()
export class ProductosRepository {
  constructor(private readonly prisma: PrismaService) {}  // ← DI
  // ...
}

// productos.service.ts
@Injectable()
export class ProductosService {
  constructor(private readonly repository: ProductosRepository) {}  // ← DI
  // ...
}

// productos.controller.ts
@Controller("productos")
export class ProductosController {
  constructor(private readonly productosService: ProductosService) {}  // ← DI
  // ...
}

// productos.module.ts
@Module({
  controllers: [ProductosController],
  providers: [ProductosService, ProductosRepository, PrismaService],
  exports: [ProductosService],  // exportado para que PedidosModule lo use
})
export class ProductosModule {}
```

## Ciclo de vida de una petición

```mermaid
sequenceDiagram
    autonumber
    actor App as App Móvil
    participant Cors as CORS Middleware
    participant Pipe as ValidationPipe
    participant Guard as Guard (si aplica)
    participant C as Controller
    participant S as Service
    participant R as Repository
    participant P as PrismaService
    participant DB as PostgreSQL

    App->>Cors: HTTP Request
    Cors->>Pipe: Headers OK
    Pipe->>Pipe: Valida body con DTO
    alt DTO inválido
        Pipe-->>App: 400 Bad Request
    else DTO válido
        Pipe->>Guard: Petición pasa
        alt Guard bloquea
            Guard-->>App: 401 Unauthorized
        else Guard permite
            Guard->>C: Invoca método
            C->>S: Delega lógica
            S->>R: Query
            R->>P: prisma.entity.xxx()
            P->>DB: SQL
            DB-->>P: filas
            P-->>R: objetos tipados
            R-->>S: retorna
            S->>S: aplica reglas de negocio
            S-->>C: retorna resultado
            C-->>App: JSON 200/201
        end
    end
```

## Configuración clave (main.ts)

```typescript
import "dotenv/config";                              // ← carga .env
import { ValidationPipe } from "@nestjs/common";
import { NestFactory } from "@nestjs/core";

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.enableCors();                                  // ← middleware CORS global

  app.useGlobalPipes(new ValidationPipe({            // ← validación global
    whitelist: true,
    forbidNonWhitelisted: true,
    transform: true,
  }));

  const prismaService = app.get(PrismaService);
  await prismaService.enableShutdownHooks(app);      // ← graceful shutdown

  await app.listen(3000, "0.0.0.0");                 // ← bind a todas las IFs
}
bootstrap();
```
