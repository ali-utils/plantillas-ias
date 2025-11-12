# bMOI Backend API Multi-Tenant

> **API REST Multi-Tenant con NestJS + PostgreSQL RLS + Redis**

![NestJS](https://img.shields.io/badge/NestJS-10-red?logo=nestjs)
![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?logo=postgresql)
![Redis](https://img.shields.io/badge/Redis-7-DC382D?logo=redis)
![Docker](https://img.shields.io/badge/Docker-Required-2496ED?logo=docker)

---

## 📋 Tabla de Contenidos

- [Descripción](#descripción)
- [Stack Tecnológico](#stack-tecnológico)
- [Características Principales](#características-principales)
- [Inicio Rápido](#inicio-rápido)
- [Arquitectura Multi-Tenant](#arquitectura-multi-tenant)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Documentación](#documentación)
- [Scripts Disponibles](#scripts-disponibles)
- [Testing](#testing)
- [Contribución](#contribución)

---

## 📝 Descripción

**bMOI Backend API** es una API REST multi-tenant construida con NestJS que sirve como backend para el panel de control bMOI. Proporciona:

- ✅ **Aislamiento total de datos** por tenant usando Row-Level Security de PostgreSQL
- ✅ **Autenticación JWT con RS256** (algoritmo asimétrico)
- ✅ **RBAC (Role-Based Access Control)** con permisos granulares
- ✅ **Límites por plan** (Free, Plan1, Plan2, Plan3)
- ✅ **Cache distribuido** con Redis
- ✅ **Arquitectura modular** lista para escalar a microservicios
- ✅ **Documentación automática** con Swagger/OpenAPI
- ✅ **Testing completo** (Unit + E2E) con >80% coverage

---

## 🛠️ Stack Tecnológico

### Core
- **Framework:** NestJS 10
- **Runtime:** Node.js 20 LTS
- **Lenguaje:** TypeScript 5
- **Package Manager:** npm

### Database & Storage
- **Database:** PostgreSQL 15 con Row-Level Security
- **ORM:** Prisma
- **Cache:** Redis 7
- **Queue:** BullMQ
- **File Storage:** MinIO (S3-compatible)

### Security & Auth
- **Authentication:** JWT con RS256 (asymmetric)
- **Password Hashing:** bcrypt (10 rounds)
- **Validation:** class-validator + class-transformer
- **Security:** Helmet, CORS, Rate Limiting

### Development
- **Containerization:** Docker + Docker Compose (OBLIGATORIO)
- **Testing:** Jest + Supertest
- **Linting:** ESLint + Prettier
- **Documentation:** Swagger/OpenAPI

---

## ✨ Características Principales

### 🏢 Multi-Tenant con Row-Level Security

Aislamiento garantizado a nivel de base de datos:

```sql
-- Todas las tablas tienen tenant_id
CREATE TABLE users (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL REFERENCES tenants(id),
  email VARCHAR(255) NOT NULL,
  ...
);

-- PostgreSQL RLS policy
CREATE POLICY tenant_isolation ON users
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

Cada request incluye el header `X-Tenant-ID` y el backend configura automáticamente el contexto:

```typescript
await prisma.$executeRaw`
  SELECT set_config('app.current_tenant_id', ${tenantId}, true)
`;
```

### 🔐 Autenticación y Autorización

- **JWT con RS256:** Tokens firmados con clave privada, validados con clave pública
- **Refresh Tokens:** Rotación automática para seguridad
- **RBAC:** Roles (admin, manager, user) con permisos granulares
- **Guards personalizados:** JwtAuthGuard, TenantGuard, PermissionGuard, PlanLimitGuard

### 📊 Plan-Based Limits

Cuatro planes con límites enforceados:

| Plan | Usuarios | Productos | Storage | Precio |
|------|----------|-----------|---------|--------|
| Free | 10 | 50 | 1GB | $0 |
| Plan1 | 50 | 500 | 10GB | $9.99 |
| Plan2 | 200 | 2000 | 50GB | $29.99 |
| Plan3 | ∞ | ∞ | 500GB | $99.99 |

```typescript
@Post()
@CheckPlanLimit('users')
async create(@Tenant() tenant, @Body() dto: CreateUserDto) {
  // Plan limit verified before creating
  return this.usersService.create(tenant.id, dto);
}
```

### 📈 Performance y Escalabilidad

- **Cache con Redis:** Respuestas cacheadas por tenant
- **Queue con BullMQ:** Procesamiento asíncrono de tareas pesadas
- **Indexación optimizada:** Índices compuestos con tenant_id
- **Paginación:** Cursor-based pagination para listados grandes

---

## 🚀 Inicio Rápido

### Prerrequisitos

- **Docker** (20.10+)
- **Docker Compose** (2.0+)
- **Git**

### Instalación en 3 Pasos

#### 1. Clonar el repositorio

```bash
git clone https://github.com/your-org/bmoi-backend-api.git
cd bmoi-backend-api
```

#### 2. Configurar variables de entorno

```bash
cp .env.example .env
# Editar .env con tus valores
```

#### 3. Iniciar con Docker

```bash
# Iniciar todos los servicios (backend + postgres + redis + minio)
docker compose up -d

# Ver logs
docker compose logs -f backend

# Ejecutar migraciones
docker compose exec backend npx prisma migrate dev

# Seed data inicial
docker compose exec backend npm run seed
```

**¡Listo!** La API estará disponible en http://localhost:5000

- **API:** http://localhost:5000/api
- **Swagger Docs:** http://localhost:5000/api/docs
- **Prisma Studio:** Ejecuta `docker compose exec backend npx prisma studio`

---

## 🏗️ Arquitectura Multi-Tenant

### Flujo de Request

```
┌─────────────┐
│   Client    │
│  (Frontend) │
└──────┬──────┘
       │ Headers:
       │ - Authorization: Bearer <JWT>
       │ - X-Tenant-ID: <tenant-uuid>
       ▼
┌─────────────────────────────────────────┐
│          NestJS Backend                  │
│                                          │
│  1. JwtAuthGuard                         │
│     ✓ Validates JWT token                │
│     ✓ Extracts user + tenants            │
│                                          │
│  2. TenantGuard                          │
│     ✓ Validates X-Tenant-ID header       │
│     ✓ Checks user has access to tenant   │
│                                          │
│  3. PermissionGuard                      │
│     ✓ Validates user has permission      │
│                                          │
│  4. PlanLimitGuard                       │
│     ✓ Checks plan limits not exceeded    │
│                                          │
│  5. TenantInterceptor                    │
│     ✓ Injects tenant context to request  │
│                                          │
│  6. Service Layer                        │
│     ✓ Sets PostgreSQL session variable:  │
│       SET app.current_tenant_id = <uuid> │
│                                          │
│  7. Prisma (ORM)                         │
│     ✓ Executes query                     │
│                                          │
└──────────────┬──────────────────────────┘
               ▼
    ┌─────────────────────┐
    │   PostgreSQL 15     │
    │   with RLS enabled  │
    │                     │
    │   RLS Policy:       │
    │   WHERE tenant_id = │
    │   current_setting() │
    └─────────────────────┘
               ▼
         Only tenant's data returned
```

### Ejemplo de Implementación

**Controller:**
```typescript
import { Controller, Get, UseGuards, UseInterceptors } from '@nestjs/common';
import { JwtAuthGuard } from '@/common/guards/jwt-auth.guard';
import { TenantGuard } from '@/common/guards/tenant.guard';
import { TenantInterceptor } from '@/common/interceptors/tenant.interceptor';
import { Tenant } from '@/common/decorators/tenant.decorator';
import { RequirePermission } from '@/common/decorators/permissions.decorator';

@Controller('users')
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  @RequirePermission('users.read')
  async findAll(@Tenant() tenant: ITenantContext) {
    return this.usersService.findAll(tenant.id);
  }
}
```

**Service:**
```typescript
import { Injectable } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';

@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string) {
    // Set tenant context for Row-Level Security
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Query automatically filtered by RLS
    return this.prisma.user.findMany({
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        // NEVER include password
      },
    });
  }
}
```

---

## 📂 Estructura del Proyecto

```
backend/
├── .claude/                    # Claude Code context & conventions
│   ├── context.md
│   ├── conventions.md
│   ├── memory-checkpoints.md
│   └── session-state.md
├── .copilot/                   # GitHub Copilot instructions
│   └── instructions.md
├── .docs/                      # Project documentation
│   ├── PROJECT_INSTRUCTIONS.md
│   ├── CODING_STANDARDS.md
│   ├── QUICKSTART.md
│   ├── ARQUITECTURA.md
│   └── START.md
├── prisma/                     # Prisma ORM
│   ├── schema.prisma
│   ├── migrations/
│   └── seed.ts
├── src/
│   ├── common/                 # Shared utilities
│   │   ├── decorators/         # @Tenant(), @RequirePermission()
│   │   ├── guards/             # JwtAuthGuard, TenantGuard
│   │   ├── interceptors/       # TenantInterceptor, AuditInterceptor
│   │   ├── pipes/              # Validation pipes
│   │   ├── filters/            # Exception filters
│   │   └── interfaces/         # Shared interfaces
│   ├── config/                 # Configuration
│   │   ├── database.config.ts
│   │   ├── jwt.config.ts
│   │   └── redis.config.ts
│   ├── database/               # Database module
│   │   └── prisma.service.ts
│   ├── modules/                # Feature modules
│   │   ├── auth/               # Authentication
│   │   ├── tenants/            # Tenant management
│   │   ├── users/              # User management
│   │   ├── dashboard/          # Dashboard metrics
│   │   ├── audit/              # Audit logging
│   │   └── ...
│   ├── app.module.ts
│   └── main.ts
├── test/                       # E2E tests
│   ├── auth.e2e-spec.ts
│   └── users.e2e-spec.ts
├── .cursorrules                # Cursor AI rules
├── .env.example                # Environment variables template
├── .eslintrc.js
├── .prettierrc
├── docker-compose.yml          # Docker services
├── Dockerfile                  # Backend container
├── nest-cli.json
├── package.json
├── tsconfig.json
├── CLAUDE.md                   # Quick reference for Claude Code
└── README.md                   # This file
```

---

## 📚 Documentación

### Para Desarrolladores

- **[QUICKSTART.md](.docs/QUICKSTART.md)** - Inicio rápido en 5 minutos
- **[CODING_STANDARDS.md](.docs/CODING_STANDARDS.md)** - Estándares de código NestJS
- **[ARQUITECTURA.md](.docs/ARQUITECTURA.md)** - Decisiones arquitectónicas
- **[START.md](.docs/START.md)** - Guía paso a paso para empezar

### Para IAs (Claude Code, Copilot, Cursor)

- **[CLAUDE.md](CLAUDE.md)** - Referencia rápida para Claude Code
- **[.claude/context.md](.claude/context.md)** - Contexto completo del proyecto
- **[.claude/conventions.md](.claude/conventions.md)** - Convenciones obligatorias
- **[.copilot/instructions.md](.copilot/instructions.md)** - Instrucciones para Copilot
- **[.cursorrules](.cursorrules)** - Reglas para Cursor AI

### API Documentation

- **Swagger UI:** http://localhost:5000/api/docs (cuando el backend está corriendo)
- **OpenAPI JSON:** http://localhost:5000/api/docs-json

---

## 🔧 Scripts Disponibles

### Dentro del Container Docker

```bash
# Development
docker compose exec backend npm run start:dev    # Dev server con hot-reload
docker compose exec backend npm run build        # Build para producción
docker compose exec backend npm run start:prod   # Start producción

# Testing
docker compose exec backend npm run test         # Unit tests
docker compose exec backend npm run test:watch   # Unit tests en watch mode
docker compose exec backend npm run test:cov     # Coverage report
docker compose exec backend npm run test:e2e     # E2E tests

# Code Quality
docker compose exec backend npm run lint         # Run ESLint
docker compose exec backend npm run format       # Format con Prettier

# Database
docker compose exec backend npx prisma generate      # Generar Prisma Client
docker compose exec backend npx prisma migrate dev   # Crear y aplicar migration
docker compose exec backend npx prisma migrate deploy  # Aplicar migrations (prod)
docker compose exec backend npx prisma studio        # Prisma Studio GUI
docker compose exec backend npm run seed             # Seed database
```

### Docker Compose

```bash
# Start all services
docker compose up -d

# Start specific service
docker compose up -d postgres

# View logs
docker compose logs -f backend
docker compose logs -f postgres

# Stop all services
docker compose down

# Stop and remove volumes (⚠️ deletes data)
docker compose down -v

# Rebuild backend image
docker compose build backend

# Execute command in container
docker compose exec backend <command>
```

---

## 🧪 Testing

### Unit Tests

```bash
docker compose exec backend npm run test

# Con coverage
docker compose exec backend npm run test:cov

# Modo watch
docker compose exec backend npm run test:watch
```

### E2E Tests

```bash
docker compose exec backend npm run test:e2e
```

### Coverage Goal

- **Objetivo mínimo:** 80%
- **Archivos críticos:** 95%+ (auth, tenant isolation, plan limits)

---

## 🛡️ Seguridad

### 7 Capas de Seguridad

1. **Network:** HTTPS, Rate Limiting (10 req/min), CORS configurado
2. **Authentication:** JWT RS256, Refresh Token rotation
3. **Authorization:** RBAC con permisos granulares
4. **Tenant Isolation:** X-Tenant-ID validation + PostgreSQL RLS
5. **Input Validation:** class-validator en todos los DTOs
6. **Plan Limits:** PlanLimitGuard enforcea límites
7. **Audit:** Winston logging + tabla audit con eventos críticos

### Buenas Prácticas

- ✅ **Passwords:** SIEMPRE hashear con bcrypt (10 rounds)
- ✅ **JWT:** Tokens de corta duración (15min) + refresh tokens
- ✅ **RLS:** SIEMPRE ejecutar `set_config` antes de queries
- ✅ **Validación:** Todos los DTOs con class-validator
- ❌ **NUNCA:** Retornar passwords en responses
- ❌ **NUNCA:** Queries sin tenant context
- ❌ **NUNCA:** Hardcodear secrets

---

## 🤝 Contribución

### Convenciones de Código

- **Archivos:** kebab-case (`users.controller.ts`)
- **Clases:** PascalCase (`UsersController`)
- **Métodos/Variables:** camelCase (`findAll`)
- **Constantes:** UPPER_SNAKE_CASE (`JWT_SECRET`)
- **PROHIBIDO:** snake_case en TypeScript
- **PROHIBIDO:** `any` type (usar `unknown` o tipos específicos)
- **PROHIBIDO:** `console.log` (usar Logger de NestJS)

### Workflow

1. Fork el proyecto
2. Crea una branch: `git checkout -b feature/nueva-funcionalidad`
3. Commit con convenciones: `git commit -m "feat: agregar endpoint de productos"`
4. Push: `git push origin feature/nueva-funcionalidad`
5. Abre un Pull Request

### Commit Messages

Seguimos [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` Nueva funcionalidad
- `fix:` Bug fix
- `docs:` Cambios en documentación
- `refactor:` Refactoring sin cambios funcionales
- `test:` Agregar o actualizar tests
- `chore:` Mantenimiento (deps, config)

---

## 📊 Roadmap

### ✅ Fase 0: Setup (Completado)
- [x] Proyecto NestJS inicializado
- [x] Docker Compose configurado
- [x] Prisma + PostgreSQL RLS
- [x] Estructura modular

### ✅ Fase 1: Auth & Tenants (Completado)
- [x] Auth Module (JWT + Refresh)
- [x] Tenants Module
- [x] Guards (JWT, Tenant, Permission, PlanLimit)
- [x] Interceptors (Tenant, Audit)

### 🔄 Fase 2: Core Modules (En Progreso)
- [x] Users Module
- [ ] Dashboard Module
- [ ] Audit Module
- [ ] Notifications Module

### 🔜 Fase 3: Business Logic
- [ ] Products Module
- [ ] Orders Module
- [ ] Reports Module
- [ ] Analytics Module

### 🔜 Fase 4: Optimization
- [ ] Redis cache avanzado
- [ ] BullMQ queues
- [ ] MinIO storage
- [ ] Monitoring con Prometheus

---

## 📄 Licencia

Este proyecto es privado y confidencial. Todos los derechos reservados.

---

## 👥 Equipo

- **Backend Team:** bMOI Development Team
- **Arquitectura:** Lead Developer
- **DevOps:** DevOps Team

---

## 📞 Soporte

- **Issues:** [GitHub Issues](https://github.com/your-org/bmoi-backend-api/issues)
- **Email:** dev@bmoi.com
- **Slack:** #bmoi-backend

---

<div align="center">

**bMOI Backend API Multi-Tenant**

Construido con ❤️ usando NestJS + PostgreSQL RLS + Redis

[Documentación](docs/) • [API Docs](http://localhost:5000/api/docs) • [Contribuir](#contribución)

</div>
