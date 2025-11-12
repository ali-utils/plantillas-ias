# Contexto del Proyecto - bMOI Backend API Multi-Tenant

> **Contexto completo para Claude Code y otros asistentes de IA**
> Última actualización: Noviembre 2025

---

## 🏢 Información del Proyecto

```yaml
Nombre: BO Backend API Multi-Tenant
Tipo: API REST Multi-Tenant para Panel de Control Empresarial
Versión: 1.0.0
Fecha Inicio: Noviembre 2025
Estado: En Desarrollo - Fase 0
Frontend: Separado (repositorio cpanel-multitenant-bo)
```

---

## 🎯 Objetivo Principal

Crear una **API REST multi-tenant profesional** que:

- 🔐 Gestione autenticación JWT con refresh tokens
- 🏢 Aísle completamente datos entre tenants (Row-Level Security)
- 👥 Implemente RBAC granular (admin, manager, user)
- 📊 Enforce límites por plan (Free, Plan1, Plan2, Plan3)
- 📝 Registre auditoría de todas las acciones críticas
- 🚀 Escale horizontalmente de 0 a 10K+ tenants
- 🐳 Todo dockerizado (dev, staging, prod)

---

## 🎨 Identidad del Proyecto

### Nombre del Sistema
```
BO Backend API
```

### Headers Críticos Multi-Tenant
```typescript
{
  'X-Tenant-ID': '<tenant-uuid>',
  'Authorization': 'Bearer <jwt-token>',
  'Content-Type': 'application/json',
  'Accept-Language': 'es-ES' // para i18n
}
```

---

## 🐳 Entorno de Desarrollo: DOCKER (OBLIGATORIO)

**⚠️ CRÍTICO: Este proyecto SE DESARROLLA COMPLETAMENTE EN DOCKER.**

```bash
# ✅ SIEMPRE usa Docker para desarrollo
docker compose up -d

# ❌ NUNCA ejecutes comandos directamente en tu máquina
npm install  # ❌ NO
npm run start:dev  # ❌ NO

# ✅ Ejecuta comandos DENTRO del container
docker compose exec backend npm install [package]
docker compose exec backend npm run test
docker compose exec backend npx prisma migrate dev
```

**Razones:**
- Consistencia: Mismo entorno para todos
- Onboarding: 5 minutos para ser productivo
- No contamina: Dependencias aisladas
- CI/CD: Mismo container en dev/staging/prod
- PostgreSQL + Redis incluidos

Ver `DOCKER.md` para configuración completa.

---

## 💻 Stack Tecnológico

```typescript
// Backend Core
Framework:       NestJS 10
Language:        TypeScript 5
Node:            20 LTS
Runtime:         Node.js
Docker:          ✅ OBLIGATORIO

// Database
Database:        PostgreSQL 15
ORM:             Prisma
Migration:       Prisma Migrate
Seeding:         Prisma Seed

// Cache & Queue
Cache:           Redis 7
Queue:           BullMQ (Redis-based)
Session:         Redis

// Storage
File Storage:    MinIO (S3-compatible)
                 O AWS S3 en producción

// Authentication
JWT:             @nestjs/jwt
Strategy:        Passport JWT + Refresh Token
Hashing:         bcrypt (password)
Signing:         RS256 (asymmetric keys)

// Validation
Validation:      class-validator
Transformation:  class-transformer
Pipes:           ValidationPipe (global)

// Testing
Unit Tests:      Jest
E2E Tests:       Supertest
Coverage:        >80% target
Mocking:         @nestjs/testing

// DevOps
Container:       Docker + Docker Compose
Orchestration:   Kubernetes (Fase 4+)
CI/CD:           GitHub Actions
Monitoring:      Prometheus + Grafana
Logging:         Winston
Error Tracking:  Sentry

// Documentation
API Docs:        Swagger/OpenAPI (auto-generado)
Architecture:    C4 Model
```

---

## 🏗️ Arquitectura Multi-Tenant

### Estrategia: Row-Level Security (PostgreSQL)

```
Frontend → API Gateway (Nginx) → NestJS Backend
                                      ↓
                            Tenant Interceptor
                                      ↓
                            Set PostgreSQL session
                            (app.current_tenant_id)
                                      ↓
                            PostgreSQL Row-Level Security
                            (filtra automáticamente)
                                      ↓
                            Retorna solo datos del tenant
```

### Flujo de Autenticación Multi-Tenant

```typescript
/**
 * FLUJO COMPLETO
 *
 * 1. POST /auth/login
 *    Body: { email, password }
 *
 * 2. Backend valida credenciales
 *
 * 3. Backend retorna:
 *    {
 *      accessToken: "eyJhbGc...",
 *      refreshToken: "eyJhbGc...",
 *      user: { id, email, name, role },
 *      tenants: [
 *        { id: "uuid", name: "Empresa A", plan: "plan1" },
 *        { id: "uuid2", name: "Empresa B", plan: "plan2" }
 *      ]
 *    }
 *
 * 4. Frontend guarda tokens y muestra selector de tenant
 *
 * 5. Usuario selecciona tenant
 *
 * 6. Todas las requests subsecuentes incluyen:
 *    Header: X-Tenant-ID: <tenant-uuid>
 *    Header: Authorization: Bearer <access-token>
 *
 * 7. Backend Tenant Guard valida:
 *    - Token es válido
 *    - User pertenece al tenant del header
 *    - Tenant está activo
 *
 * 8. Backend Tenant Interceptor:
 *    - Inyecta tenant_id en request.tenant
 *    - Set PostgreSQL session variable
 *    - RLS filtra automáticamente
 *
 * 9. Service Layer ejecuta query
 *    - PostgreSQL RLS policy se aplica automáticamente
 *    - Solo retorna datos de ese tenant
 */
```

### Límites por Plan

```typescript
export const PLAN_LIMITS = {
  free: {
    users: 10,
    products: 50,
    orders: 100,
    storageGB: 1,
    apiCallsPerDay: 1000,
    features: ['dashboard.basic', 'users.read', 'products.read']
  },

  plan1: {
    users: 50,
    products: 500,
    orders: 1000,
    storageGB: 10,
    apiCallsPerDay: 10000,
    features: ['dashboard.advanced', 'users.full', 'products.full', 'reports.basic']
  },

  plan2: {
    users: 200,
    products: 2000,
    orders: 10000,
    storageGB: 50,
    apiCallsPerDay: 100000,
    features: ['dashboard.pro', 'analytics.full', 'api.access']
  },

  plan3: {
    users: -1, // Ilimitado
    products: -1,
    orders: -1,
    storageGB: 500,
    apiCallsPerDay: -1,
    features: ['all', 'white-label', 'priority-support']
  }
} as const
```

---

## 📋 Convenciones OBLIGATORIAS

### Nomenclatura NestJS

```typescript
// ✅ Archivos: kebab-case
users.controller.ts
users.service.ts
create-user.dto.ts
user.entity.ts

// ✅ Clases: PascalCase
export class UsersController { }
export class UsersService { }
export class CreateUserDto { }
export class User { }

// ✅ Métodos y variables: camelCase
async findAll() { }
const userList = []
const currentTenant = request.tenant

// ✅ Constantes: UPPER_SNAKE_CASE
export const JWT_SECRET = process.env.JWT_SECRET
export const DATABASE_URL = process.env.DATABASE_URL

// ✅ Interfaces: I + PascalCase (solo si no es entity)
interface ITenantContext { }
interface IAuthPayload { }

// ✅ Enums: PascalCase
export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user'
}

// ❌ PROHIBIDO: snake_case en TypeScript
const user_name = "John"  // NUNCA
function get_user_data() { }  // NUNCA
```

### Reglas Multi-Tenant OBLIGATORIAS

```typescript
// ✅ SIEMPRE validar tenant en controllers
@Controller('users')
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {

  @Get()
  @RequirePermission('users.read')
  async findAll(@Tenant() tenant: ITenantContext) {
    return this.usersService.findAll(tenant.id)
  }
}

// ✅ SIEMPRE usar tenant_id en queries
async findAll(tenantId: string) {
  // Set PostgreSQL session para RLS
  await this.prisma.$executeRaw`
    SELECT set_config('app.current_tenant_id', ${tenantId}, true)
  `

  return this.prisma.user.findMany({
    // RLS filtra automáticamente por tenant_id
  })
}

// ✅ SIEMPRE validar permisos
@RequirePermission('users.create')
@CheckPlanLimit('users')  // Verificar límites del plan
async create() { }

// ❌ PROHIBIDO: Queries sin tenant_id
async findAll() {
  // ❌ NUNCA hagas esto
  return this.prisma.user.findMany()  // Retorna TODOS los users de TODOS los tenants
}
```

### Estructura de Módulos

```typescript
// Cada módulo debe tener esta estructura
src/modules/users/
├── users.module.ts          // @Module decorator
├── users.controller.ts      // Endpoints REST
├── users.service.ts         // Lógica de negocio
├── users.repository.ts      // Opcional: Repository pattern
├── entities/
│   └── user.entity.ts       // Prisma model / TypeORM entity
├── dto/
│   ├── create-user.dto.ts   // DTOs con class-validator
│   ├── update-user.dto.ts
│   └── user-filters.dto.ts
├── guards/                   // Opcional: Guards específicos
│   └── user-ownership.guard.ts
└── tests/
    ├── users.controller.spec.ts
    ├── users.service.spec.ts
    └── users.e2e-spec.ts
```

---

## 📂 Estructura del Proyecto

```
backend/
├── src/
│   ├── main.ts                    # Bootstrap
│   ├── app.module.ts              # Root module
│   │
│   ├── common/                    # Shared
│   │   ├── decorators/            # @Tenant(), @RequirePermission()
│   │   ├── guards/                # JwtAuthGuard, TenantGuard, etc.
│   │   ├── interceptors/          # TenantInterceptor, LoggingInterceptor
│   │   ├── pipes/                 # ValidationPipe, TransformPipe
│   │   ├── filters/               # ExceptionFilter
│   │   └── interfaces/            # Shared interfaces
│   │
│   ├── config/                    # Configuration
│   │   ├── database.config.ts
│   │   ├── redis.config.ts
│   │   ├── jwt.config.ts
│   │   └── app.config.ts
│   │
│   ├── modules/
│   │   ├── auth/                  # JWT authentication
│   │   ├── tenants/               # Tenant management
│   │   ├── users/                 # User CRUD
│   │   ├── products/              # Product catalog
│   │   ├── orders/                # Order management
│   │   ├── dashboard/             # Metrics
│   │   ├── audit/                 # Audit logging
│   │   └── plans/                 # Plan limits
│   │
│   ├── database/                  # Prisma/TypeORM
│   │   ├── migrations/
│   │   ├── seeds/
│   │   └── schema.prisma
│   │
│   └── shared/                    # Shared types
│       ├── enums/
│       └── types/
│
├── test/                          # E2E tests
├── docker/
├── prisma/
├── .env.example
├── docker-compose.yml
├── Dockerfile
└── nest-cli.json
```

---

## 🔐 Seguridad - 7 Capas

```
1. Network Layer
   └─ HTTPS, Rate Limiting, CORS

2. Authentication Layer
   └─ JWT RS256, Refresh Tokens, Token Blacklist

3. Authorization Layer
   └─ RBAC, Permission Guards

4. Tenant Isolation Layer
   └─ X-Tenant-ID validation, PostgreSQL RLS

5. Input Validation Layer
   └─ class-validator, DTO validation

6. Plan Limits Layer
   └─ PlanLimitGuard, Background monitoring

7. Audit Layer
   └─ Winston logging, Audit table
```

---

## 🚫 Prohibiciones ABSOLUTAS

```typescript
// ❌ NO usar snake_case en TypeScript
const user_name = "John"

// ❌ NO usar any type
const data: any = await fetchData()

// ❌ NO queries sin tenant_id
this.prisma.user.findMany()  // Retorna TODOS los users

// ❌ NO hardcodear secrets
const JWT_SECRET = "my-secret-key"

// ❌ NO exponer información sensible
return {
  id: user.id,
  password: user.password  // ❌ NUNCA retornar passwords
}

// ❌ NO console.log en producción
console.log('User data:', user)  // Usar Winston logger

// ❌ NO ignorar validación de tenant
@Get()
async findAll() {
  // ❌ Falta @UseGuards(TenantGuard)
}
```

---

## ✅ Patrones Correctos

```typescript
// ✅ Controller completo y seguro
@Controller('users')
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor, AuditInterceptor)
@ApiTags('users')
export class UsersController {
  constructor(
    private readonly usersService: UsersService,
    private readonly logger: Logger,
  ) {}

  @Get()
  @RequirePermission('users.read')
  @ApiOperation({ summary: 'Get all users' })
  @ApiResponse({ status: 200, type: [User] })
  async findAll(
    @Tenant() tenant: ITenantContext,
    @Query() filters: UserFiltersDto,
  ) {
    this.logger.log(`Getting users for tenant ${tenant.id}`)
    return this.usersService.findAll(tenant.id, filters)
  }

  @Post()
  @RequirePermission('users.create')
  @CheckPlanLimit('users')
  @ApiOperation({ summary: 'Create user' })
  async create(
    @Tenant() tenant: ITenantContext,
    @Body() dto: CreateUserDto,
  ) {
    return this.usersService.create(tenant.id, dto)
  }
}

// ✅ Service con tenant isolation
@Injectable()
export class UsersService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly logger: Logger,
  ) {}

  async findAll(tenantId: string, filters: UserFiltersDto) {
    // Set tenant context para RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    return this.prisma.user.findMany({
      where: {
        // tenant_id automáticamente filtrado por RLS
        email: filters.email ? { contains: filters.email } : undefined,
        role: filters.role,
      },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        // NO incluir password
      },
    })
  }
}
```

---

## 🧠 Sistema Anti-Olvido

### Cada 5 Mensajes

```markdown
🧠 Checkpoint:
- Proyecto: bMOI Backend API Multi-Tenant
- Framework: NestJS 10 + TypeScript 5
- Database: PostgreSQL 15 con RLS
- Multi-tenant: Header X-Tenant-ID + RLS automático
- NO usar: snake_case, any type, queries sin tenant
- Docker: TODO en containers
```

### Al Final de Sesión

Actualizar `.claude/session-state.md` con:
- Endpoints implementados hoy
- Decisiones técnicas tomadas
- Próximos pasos
- Progreso de fase

---

## 📚 Documentos Críticos

**LEER ANTES DE PROGRAMAR:**

1. [PROJECT_INSTRUCTIONS.md](../.docs/PROJECT_INSTRUCTIONS.md) - Guía completa
2. [CODING_STANDARDS.md](../.docs/CODING_STANDARDS.md) - Estándares NestJS
3. [ARQUITECTURA.md](../.docs/ARQUITECTURA.md) - Decisiones técnicas
4. [BACKEND_ARCHITECTURE.md](../../BACKEND_ARCHITECTURE.md) - Arquitectura detallada

---

## 🎯 Estado Actual

```yaml
Fase Actual: Fase 0 - Setup Backend
Última Actualización: 2025-11-03
Módulos Creados: 0
Tests Coverage: 0%
Endpoints: 0
```

---

<div align="center">

**Este contexto es tu fuente de verdad. Consúltalo frecuentemente.**

NestJS + PostgreSQL RLS + Docker = Backend Multi-Tenant Profesional

</div>
