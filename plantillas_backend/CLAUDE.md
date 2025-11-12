# CLAUDE.md - Backend

This file provides guidance to Claude Code (claude.ai/code) when working with the **backend** code in this repository.

## Project Overview

**bMOI Backend API Multi-Tenant** is a RESTful API built with NestJS that serves the frontend control panel. It provides complete tenant isolation, JWT authentication, RBAC, and plan-based resource limits.

**Framework:** NestJS 10 + TypeScript 5
**Database:** PostgreSQL 15 with Row-Level Security (RLS)
**Cache:** Redis 7
**ORM:** Prisma
**Docker:** MANDATORY for all development

**Current Status:** Phase 0 - Backend Setup (0% complete)

## 📚 Critical Documentation Ecosystem

This backend project has a comprehensive documentation system. **Read these files in this order when starting:**

1. **`.claude/context.md`** - Complete backend context (MUST READ FIRST)
2. **`.docs/PROJECT_INSTRUCTIONS.md`** - Master document for AI assistants
3. **`.docs/CODING_STANDARDS.md`** - Detailed NestJS coding standards
4. **`.claude/conventions.md`** - Mandatory naming conventions
5. **`.claude/session-state.md`** - Current development status
6. **This file (CLAUDE.md)** - Quick reference guide

**Other AI Configurations:**
- `.copilot/instructions.md` - Instructions for GitHub Copilot
- `.cursorrules` - Rules for Cursor AI
- `.docs/START.md` - Step-by-step guide for starting development with AI
- `.docs/ARQUITECTURA.md` - Architectural decisions
- `.docs/QUICKSTART.md` - 5-minute quick start

**Frontend (Separate Repository):**
- The frontend is developed separately in repository `cpanel-multitenant-bo`
- Communication via REST API with `X-Tenant-ID` header

## 🐳 Development Environment: DOCKER (MANDATORY)

**⚠️ CRITICAL: This backend MUST be developed using Docker.**

### Why Docker is Mandatory

1. **Consistency**: PostgreSQL 15 + Redis + MinIO included
2. **Row-Level Security**: Requires specific PostgreSQL configuration
3. **Onboarding**: New dev productive in 5 minutes
4. **CI/CD Ready**: Same container in dev, staging, prod
5. **No pollution**: Dependencies isolated from your system

### Docker Commands (Use These)

```bash
# Start backend environment
docker compose up -d

# View logs
docker compose logs -f backend

# Execute commands inside container
docker compose exec backend npm install [package]
docker compose exec backend npm run test
docker compose exec backend npx prisma migrate dev
docker compose exec backend npx prisma studio

# Stop environment
docker compose down

# Full stack (backend + postgres + redis + minio)
docker compose up -d
```

### ❌ DON'T Run Commands Directly

```bash
# ❌ NEVER do this (runs on your machine):
npm install
npm run start:dev
npx prisma migrate dev

# ✅ ALWAYS do this (runs in Docker):
docker compose up -d
docker compose exec backend npm install
docker compose exec backend npx prisma migrate dev
```

See `DOCKER.md` for complete Docker documentation.

## 📊 Key Technical Decisions

### Backend Framework: NestJS 10 ✅

**Decision:** Use **NestJS** (not Express, not Fastify, not Go)

**Why:**
- ✅ **TypeScript end-to-end** with frontend (share types)
- ✅ **Decorators**: Perfect for multi-tenant (@UseGuards, @Tenant)
- ✅ **Dependency Injection**: Excellent testability
- ✅ **Swagger auto-generated**: API documentation automatic
- ✅ **Modular from day 1**: Easy to scale to microservices

**Trade-off accepted:** Slightly more opinionated than Express, but worth it for the structure

### Database Strategy: PostgreSQL Row-Level Security ✅

**Decision:** PostgreSQL 15 with **Row-Level Security (RLS)**

**Why:**
- ✅ **Isolation guaranteed at DB level**: Impossible to mix tenant data
- ✅ **Single database**: Cost-effective up to 10K tenants
- ✅ **Automatic filtering**: RLS policies apply automatically
- ✅ **Simple backups**: One database to backup
- ✅ **Easy migrations**: Single schema

**How it works:**
```sql
-- All tables have tenant_id
CREATE TABLE users (
  id UUID PRIMARY KEY,
  tenant_id UUID NOT NULL,
  email VARCHAR(255),
  ...
);

-- PostgreSQL RLS policy
CREATE POLICY tenant_isolation ON users
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- In your service, set tenant context
await prisma.$executeRaw`
  SELECT set_config('app.current_tenant_id', ${tenantId}, true)
`;

-- Now all queries automatically filter by tenant
const users = await prisma.user.findMany();  // Only returns current tenant's users
```

**Alternatives discarded:**
- ❌ Schema per tenant: Too complex for 1-10K tenants
- ❌ Database per tenant: Very expensive and hard to maintain

### ORM: Prisma ✅

**Decision:** Use **Prisma** (not TypeORM)

**Why:**
- ✅ **Type-safe** at compile time
- ✅ **Excellent auto-completion**
- ✅ **Simpler migrations**
- ✅ **Better DX**
- ✅ **Declarative schema**

**Installation:**
```bash
docker compose exec backend npm install prisma @prisma/client
docker compose exec backend npx prisma init
```

### Authentication: JWT with RS256 ✅

**Decision:** JWT with **RS256** (asymmetric)

**Why:**
- ✅ More secure than HS256
- ✅ Public key for validation (can share with other services)
- ✅ Private key only in backend
- ✅ Refresh tokens with rotation

**Trade-off accepted:** Slightly slower than HS256, but more secure

## Development Commands (Inside Docker Container)

When you exec into the container, these commands are available:

### Core Development
```bash
npm run start:dev        # Start dev server with hot-reload
npm run build            # Build for production
npm run start:prod       # Start production server
npm run lint             # Run ESLint
npm run format           # Format with Prettier
npm run test             # Run unit tests
npm run test:e2e         # Run E2E tests
npm run test:cov         # Generate coverage report
```

### Prisma Commands
```bash
npx prisma generate      # Generate Prisma Client
npx prisma migrate dev   # Create and apply migration
npx prisma migrate deploy  # Apply migrations (prod)
npx prisma studio        # Open Prisma Studio GUI
npx prisma db seed       # Run seed scripts
```

### Database Commands
```bash
npx prisma db push       # Push schema without migrations (dev only)
npx prisma db pull       # Pull schema from database
npx prisma format        # Format schema.prisma
```

## 🧠 AI Memory System (Anti-Forgetting)

This project implements a memory checkpoint system to prevent context loss in long conversations:

### Every 5 Messages - Memory Checkpoint
When working with Claude Code, every ~5 messages you should mentally verify:
- [ ] Project name: **bMOI Backend API Multi-Tenant**
- [ ] Framework: **NestJS 10 + TypeScript 5**
- [ ] Database: **PostgreSQL 15 with RLS**
- [ ] Multi-tenant: **X-Tenant-ID header + RLS automatic**
- [ ] Conventions: **kebab-case** (files), **PascalCase** (classes), **camelCase** (methods)
- [ ] **NEVER** use snake_case in TypeScript
- [ ] **NEVER** use `any` type
- [ ] **ALWAYS** set PostgreSQL session for RLS
- [ ] **ALWAYS** include tenant_id in inserts
- [ ] **ALWAYS** use Guards (@UseGuards(TenantGuard))
- [ ] **ALWAYS** use Interceptors (@UseInterceptors(TenantInterceptor))

### End of Session - Update State
Before ending a development session, update `.claude/session-state.md` with:
- Modules created today
- Endpoints implemented
- Technical decisions made
- Problems encountered and solutions
- Next steps
- Current phase progress

### Context Recovery
If you lose context mid-conversation:
1. Read `.claude/context.md` immediately
2. Read `.claude/session-state.md` to know where you left off
3. Confirm you remember the mandatory conventions
4. Ask the user: "What module/endpoint were we working on?"

## Architecture & Key Patterns

### Multi-Tenant Architecture

**Critical**: Every API request MUST include tenant context and RLS must be applied.

The system uses:
1. **Tenant Guards**: Validate `X-Tenant-ID` header and user access
2. **Tenant Interceptors**: Inject tenant context into request
3. **PostgreSQL RLS**: Automatic data filtering at database level

**Flow:**
```
1. Request arrives with X-Tenant-ID header
2. JwtAuthGuard validates JWT token
3. TenantGuard validates tenant access
4. TenantInterceptor injects tenant into request
5. Service sets PostgreSQL session variable
6. RLS policy automatically filters data
7. Response only contains tenant's data
```

**Example implementation:**
```typescript
// Controller
@Controller('users')
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {
  @Get()
  @RequirePermission('users.read')
  async findAll(@Tenant() tenant: ITenantContext) {
    return this.usersService.findAll(tenant.id);
  }
}

// Service
@Injectable()
export class UsersService {
  async findAll(tenantId: string) {
    // Set tenant context for RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Query automatically filtered by RLS
    return this.prisma.user.findMany();
  }
}
```

### Naming Conventions (MANDATORY)

From `.claude/conventions.md` - these are strictly enforced:
- **Files**: kebab-case (`users.controller.ts`, `create-user.dto.ts`)
- **Classes**: PascalCase (`UsersController`, `CreateUserDto`)
- **Methods/Variables**: camelCase (`findAll`, `currentTenant`)
- **Constants**: UPPER_SNAKE_CASE (`JWT_SECRET`, `DATABASE_URL`)
- **Enums**: PascalCase (`UserRole`, `PlanType`)
- **NEVER** use snake_case in TypeScript

### Module Structure

Every NestJS module should follow this structure:
```
src/modules/users/
├── users.module.ts
├── users.controller.ts
├── users.service.ts
├── dto/
│   ├── create-user.dto.ts
│   ├── update-user.dto.ts
│   └── user-filters.dto.ts
├── entities/
│   └── user.entity.ts
└── tests/
    ├── users.controller.spec.ts
    ├── users.service.spec.ts
    └── users.e2e-spec.ts
```

### Security Layers

The backend implements 7 security layers:
1. **Network**: HTTPS, Rate Limiting, CORS
2. **Authentication**: JWT RS256, Refresh Tokens
3. **Authorization**: RBAC, Permission Guards
4. **Tenant Isolation**: X-Tenant-ID validation, RLS
5. **Input Validation**: class-validator, DTO validation
6. **Plan Limits**: PlanLimitGuard
7. **Audit**: Winston logging, Audit table

### Plan Limits System

Four plan tiers with enforced limits:
- **Free**: 10 users, 50 products, 1GB storage
- **Plan1**: 50 users, 500 products, 10GB storage
- **Plan2**: 200 users, 2000 products, 50GB storage
- **Plan3**: Unlimited users/products, 500GB storage

Limits are checked via `@CheckPlanLimit()` decorator before creating resources.

## Common Patterns

### Creating a New Module

```bash
# Inside Docker container
docker compose exec backend nest g module modules/products
docker compose exec backend nest g controller modules/products
docker compose exec backend nest g service modules/products
```

### Creating a Controller with Multi-Tenant

```typescript
import { Controller, Get, Post, Body, UseGuards, UseInterceptors } from '@nestjs/common';
import { ApiTags, ApiBearerAuth } from '@nestjs/swagger';

import { JwtAuthGuard } from '@/common/guards/jwt-auth.guard';
import { TenantGuard } from '@/common/guards/tenant.guard';
import { TenantInterceptor } from '@/common/interceptors/tenant.interceptor';
import { Tenant } from '@/common/decorators/tenant.decorator';
import { RequirePermission } from '@/common/decorators/permissions.decorator';
import { ITenantContext } from '@/common/interfaces/tenant-context.interface';

@Controller('products')
@ApiTags('products')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class ProductsController {
  constructor(private readonly productsService: ProductsService) {}

  @Get()
  @RequirePermission('products.read')
  async findAll(@Tenant() tenant: ITenantContext) {
    return this.productsService.findAll(tenant.id);
  }

  @Post()
  @RequirePermission('products.create')
  @CheckPlanLimit('products')
  async create(
    @Tenant() tenant: ITenantContext,
    @Body() dto: CreateProductDto,
  ) {
    return this.productsService.create(tenant.id, dto);
  }
}
```

### Creating a Service with RLS

```typescript
import { Injectable, Logger } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';

@Injectable()
export class ProductsService {
  private readonly logger = new Logger(ProductsService.name);

  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string) {
    // Set tenant context for RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    return this.prisma.product.findMany({
      select: {
        id: true,
        name: true,
        price: true,
        // ...
      },
    });
  }

  async create(tenantId: string, dto: CreateProductDto) {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    return this.prisma.product.create({
      data: {
        tenant_id: tenantId,  // EXPLICIT
        ...dto,
      },
    });
  }
}
```

### Creating a DTO with Validation

```typescript
import { IsString, IsNumber, Min, IsOptional } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateProductDto {
  @ApiProperty({ example: 'Product Name' })
  @IsString()
  name: string;

  @ApiProperty({ example: 99.99, minimum: 0 })
  @IsNumber()
  @Min(0)
  price: number;

  @ApiPropertyOptional({ example: 'Product description' })
  @IsOptional()
  @IsString()
  description?: string;
}
```

## Development Phases

The project is organized in phases. Current status from `.claude/session-state.md`:

```
Phase 0 (Setup Backend):    ░░░░░░░░░░  0%  - NestJS + Docker + Prisma
Phase 1 (Auth & Tenants):   ░░░░░░░░░░  0%  - JWT, Tenants, Guards
Phase 2 (Core Modules):     ░░░░░░░░░░  0%  - Users, Dashboard, Audit
Phase 3 (Business Logic):   ░░░░░░░░░░  0%  - Products, Orders, Reports
Phase 4 (Optimization):     ░░░░░░░░░░  0%  - Cache, Queue, Monitoring
```

**Next Immediate Steps (Phase 0 completion):**
1. Initialize NestJS project
2. Create docker-compose.yml (backend + postgres + redis)
3. Configure Prisma with PostgreSQL
4. Setup Row-Level Security policies
5. Create modular folder structure

**Next Phase (Phase 1):**
1. Implement Auth Module (JWT + Refresh)
2. Implement Tenants Module
3. Create Guards (JWT, Tenant, Permission, PlanLimit)
4. Create Interceptors (Tenant, Audit)
5. Implement basic Users Module

## Important Notes

- **Never bypass tenant isolation**: All services must call `set_config` for RLS
- **Permission checks are in both places**: Frontend (UX) + Backend (security)
- **Plan limits are hard limits**: Backend enforces them, frontend warns
- **Development logs**: Use Logger, never `console.log`
- **Test coverage**: Minimum 80%

## Troubleshooting AI Behavior

### If AI forgets conventions:
```
🧠 CONVENTION REMINDER - BACKEND:

This project uses STRICTLY:
- Files: kebab-case (users.controller.ts)
- Classes: PascalCase (UsersController)
- Methods: camelCase (findAll)
- Constants: UPPER_SNAKE_CASE (JWT_SECRET)
- NEVER snake_case in TypeScript
- NEVER any type

Multi-tenant MANDATORY:
- @UseGuards(JwtAuthGuard, TenantGuard)
- @UseInterceptors(TenantInterceptor)
- await prisma.$executeRaw` SELECT set_config(...) `

Please confirm you'll remember this.
```

### If AI creates code without tenant context:
```
⚠️ ERROR: Missing tenant isolation

ALL services MUST:
1. Receive tenantId parameter
2. Set PostgreSQL session:
   await this.prisma.$executeRaw`
     SELECT set_config('app.current_tenant_id', ${tenantId}, true)
   `
3. Include tenant_id in inserts

Please correct the code above.
```

### If AI uses snake_case:
```
❌ STOP: snake_case detected

This project PROHIBITS snake_case in TypeScript.

CORRECT examples:
- findAll (camelCase for methods)
- UsersController (PascalCase for classes)
- users.controller.ts (kebab-case for files)
- JWT_SECRET (UPPER_SNAKE_CASE for constants)

Please correct and DON'T use snake_case again.
```

## Additional Resources

- Backend architecture in `BACKEND_ARCHITECTURE.md` (root of frontend repo)
- Docker quickstart available in `DOCKER.md`
- `.cursorrules` contains mandatory coding standards for Cursor AI
- Prisma schema in `prisma/schema.prisma` is the source of truth
- Complete step-by-step guide in `.docs/START.md`
- Architectural decisions documented in `.docs/ARQUITECTURA.md`
- Memory checkpoint system in `.claude/memory-checkpoints.md`

---

<div align="center">

**Backend Multi-Tenant with NestJS + PostgreSQL RLS**

Type-safe | Isolated | Scalable | Docker-first

</div>
