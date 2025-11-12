# PROJECT_INSTRUCTIONS.md - Backend

> **Documento Maestro para Asistentes de IA**
>
> Este archivo contiene instrucciones completas para que Claude Code, GitHub Copilot, Cursor AI y otros asistentes trabajen efectivamente en el backend de bMOI.

---

## 📋 Tabla de Contenidos

- [Introducción](#introducción)
- [Contexto del Proyecto](#contexto-del-proyecto)
- [Stack Tecnológico](#stack-tecnológico)
- [Arquitectura Multi-Tenant](#arquitectura-multi-tenant)
- [Convenciones Obligatorias](#convenciones-obligatorias)
- [Desarrollo con Docker](#desarrollo-con-docker)
- [Patrones de Código](#patrones-de-código)
- [Seguridad](#seguridad)
- [Testing](#testing)
- [Flujos Críticos](#flujos-críticos)
- [Troubleshooting](#troubleshooting)
- [Sistema de Memoria](#sistema-de-memoria)

---

## 🎯 Introducción

### ¿Qué es este documento?

Este documento es la **guía maestra** para cualquier IA que trabaje en el backend de bo. Si eres Claude Code, GitHub Copilot, Cursor AI u otro asistente, **DEBES leer este documento completo** antes de empezar a programar.

### Orden de Lectura Obligatorio

Cuando empieces a trabajar en este proyecto, lee en este orden:

1. **Este archivo (`PROJECT_INSTRUCTIONS.md`)** - Guía completa
2. **`.claude/context.md`** - Contexto detallado del backend
3. **`.claude/conventions.md`** - Convenciones obligatorias
4. **`CODING_STANDARDS.md`** - Estándares de código con ejemplos
5. **`ARQUITECTURA.md`** - Decisiones arquitectónicas
6. **`.claude/session-state.md`** - Estado actual del desarrollo
7. **`CLAUDE.md`** - Referencia rápida

### ¿Por Qué Este Documento?

Las IAs pueden "olvidar" contexto en conversaciones largas. Este documento asegura que:

- ✅ Siempre recuerdes que es un backend **multi-tenant con NestJS**
- ✅ Siempre apliques **Row-Level Security** en queries
- ✅ Siempre uses las **convenciones correctas** (kebab-case, PascalCase)
- ✅ Nunca uses **anti-patrones** (snake_case, `any` type, queries sin tenant)
- ✅ Mantengas **consistencia** entre sesiones

---

## 🏢 Contexto del Proyecto

### ¿Qué Estamos Construyendo?

**Nombre:** bMOI Backend API Multi-Tenant

**Descripción:** API REST que sirve como backend para un panel de control multi-tenant. Permite que múltiples organizaciones (tenants) usen la misma aplicación con **aislamiento total de datos**.

**Ejemplo de uso:**
- Tenant A (Empresa X): 50 usuarios, 200 productos
- Tenant B (Empresa Y): 30 usuarios, 150 productos
- Tenant C (Empresa Z): 100 usuarios, 500 productos

Cada tenant **NUNCA** puede ver los datos de otro tenant.

### Características Principales

1. **Multi-Tenant con Row-Level Security**
   - Aislamiento garantizado a nivel de base de datos
   - Imposible mezclar datos entre tenants

2. **Autenticación JWT con RS256**
   - Algoritmo asimétrico más seguro
   - Refresh tokens con rotación

3. **RBAC (Role-Based Access Control)**
   - Roles: admin, manager, user
   - Permisos granulares: `users.read`, `products.create`, etc.

4. **Plan-Based Limits**
   - Free: 10 users, 50 products, 1GB storage
   - Plan1: 50 users, 500 products, 10GB storage
   - Plan2: 200 users, 2000 products, 50GB storage
   - Plan3: Unlimited users/products, 500GB storage

5. **Arquitectura Modular**
   - Fácil evolución a microservicios
   - Cada módulo es independiente

---

## 🛠️ Stack Tecnológico

### Framework y Runtime

```yaml
Framework: NestJS 10
Runtime: Node.js 20 LTS
Lenguaje: TypeScript 5
Package Manager: npm
```

**¿Por qué NestJS?**
- ✅ TypeScript end-to-end con frontend
- ✅ Decorators perfectos para multi-tenant (@UseGuards, @Tenant)
- ✅ Dependency Injection automático
- ✅ Swagger auto-generado
- ✅ Arquitectura modular de fábrica

**Alternativas descartadas:**
- ❌ Express: Demasiado boilerplate
- ❌ Fastify: Menos features out-of-the-box
- ❌ Go: Lenguaje diferente al frontend

### Database y Storage

```yaml
Database: PostgreSQL 15
ORM: Prisma
Cache: Redis 7
Queue: BullMQ (sobre Redis)
File Storage: MinIO (S3-compatible)
```

**¿Por qué PostgreSQL con RLS?**
- ✅ Row-Level Security garantiza aislamiento a nivel DB
- ✅ Una sola database (cost-effective hasta 10K tenants)
- ✅ Políticas RLS se aplican automáticamente
- ✅ Backups simples (una sola DB)

**Alternativas descartadas:**
- ❌ Schema per tenant: Muy complejo para 1-10K tenants
- ❌ Database per tenant: Muy costoso y difícil de mantener
- ❌ MongoDB: No soporta RLS nativo

**¿Por qué Prisma sobre TypeORM?**
- ✅ Type-safe a nivel de compilación
- ✅ Auto-completion excelente
- ✅ Migrations más simples
- ✅ Mejor Developer Experience

### Security

```yaml
Authentication: JWT con RS256
Password Hashing: bcrypt (10 rounds)
Validation: class-validator + class-transformer
Rate Limiting: @nestjs/throttler
CORS: Configurado por dominio
```

### Development

```yaml
Containerization: Docker + Docker Compose (OBLIGATORIO)
Testing: Jest + Supertest
Linting: ESLint + Prettier
Documentation: Swagger/OpenAPI
```

---

## 🏗️ Arquitectura Multi-Tenant

### Estrategia: PostgreSQL Row-Level Security

**Decisión arquitectónica más importante del proyecto.**

#### ¿Cómo Funciona?

1. **Todas las tablas tienen `tenant_id`**

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  password VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Índice compuesto para queries rápidas
CREATE INDEX idx_users_tenant_id ON users(tenant_id, created_at DESC);
```

2. **PostgreSQL RLS Policy se aplica automáticamente**

```sql
-- Habilitar RLS en la tabla
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Política: solo retorna filas del tenant actual
CREATE POLICY tenant_isolation ON users
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Política para inserts
CREATE POLICY tenant_isolation_insert ON users
  FOR INSERT
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

3. **Backend configura el contexto antes de cada query**

```typescript
// En cada service method
async findAll(tenantId: string) {
  // ✅ CRÍTICO: Set PostgreSQL session variable
  await this.prisma.$executeRaw`
    SELECT set_config('app.current_tenant_id', ${tenantId}, true)
  `;

  // Ahora este query SOLO retorna datos del tenant actual
  return this.prisma.user.findMany();
}
```

#### Flujo Completo de Request

```
1. Client envía request
   ├── Header: Authorization: Bearer <JWT>
   └── Header: X-Tenant-ID: <tenant-uuid>

2. JwtAuthGuard
   ├── Valida JWT token
   ├── Extrae user del payload
   └── Inyecta user en request

3. TenantGuard
   ├── Valida header X-Tenant-ID
   ├── Verifica que user pertenece al tenant
   └── Pasa si válido, lanza 403 si no

4. PermissionGuard (opcional, según endpoint)
   ├── Verifica permisos del usuario
   └── Ejemplo: @RequirePermission('users.read')

5. PlanLimitGuard (opcional, según endpoint)
   ├── Verifica límites del plan del tenant
   └── Ejemplo: @CheckPlanLimit('users')

6. TenantInterceptor
   ├── Inyecta tenant context en request
   └── request.tenant = { id, name, plan, ... }

7. Controller
   └── Extrae tenant con @Tenant() decorator

8. Service
   ├── Recibe tenantId como parámetro
   ├── Ejecuta set_config() para RLS
   └── Ejecuta query (automáticamente filtrado)

9. PostgreSQL RLS
   ├── Aplica policy automáticamente
   └── Solo retorna filas donde tenant_id = current_setting(...)

10. Response
    └── Cliente recibe solo datos de su tenant
```

### Ejemplo Completo

**Controller:**

```typescript
import { Controller, Get, Post, Body, Param, UseGuards, UseInterceptors, Logger } from '@nestjs/common';
import { ApiTags, ApiOperation, ApiBearerAuth, ApiResponse } from '@nestjs/swagger';

import { JwtAuthGuard } from '@/common/guards/jwt-auth.guard';
import { TenantGuard } from '@/common/guards/tenant.guard';
import { TenantInterceptor } from '@/common/interceptors/tenant.interceptor';
import { Tenant } from '@/common/decorators/tenant.decorator';
import { RequirePermission } from '@/common/decorators/permissions.decorator';
import { CheckPlanLimit } from '@/common/decorators/plan-limit.decorator';
import { ITenantContext } from '@/common/interfaces/tenant-context.interface';

import { UsersService } from './users.service';
import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { User } from './entities/user.entity';

@Controller('users')
@ApiTags('users')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {
  private readonly logger = new Logger(UsersController.name);

  constructor(private readonly usersService: UsersService) {}

  @Get()
  @RequirePermission('users.read')
  @ApiOperation({ summary: 'Get all users for current tenant' })
  @ApiResponse({ status: 200, type: [User] })
  async findAll(@Tenant() tenant: ITenantContext): Promise<User[]> {
    this.logger.log(`Finding all users for tenant: ${tenant.id}`);
    return this.usersService.findAll(tenant.id);
  }

  @Get(':id')
  @RequirePermission('users.read')
  @ApiOperation({ summary: 'Get user by ID' })
  async findOne(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ): Promise<User> {
    return this.usersService.findOne(tenant.id, id);
  }

  @Post()
  @RequirePermission('users.create')
  @CheckPlanLimit('users')
  @ApiOperation({ summary: 'Create new user' })
  async create(
    @Tenant() tenant: ITenantContext,
    @Body() createUserDto: CreateUserDto,
  ): Promise<User> {
    return this.usersService.create(tenant.id, createUserDto);
  }

  @Patch(':id')
  @RequirePermission('users.update')
  @ApiOperation({ summary: 'Update user' })
  async update(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<User> {
    return this.usersService.update(tenant.id, id, updateUserDto);
  }

  @Delete(':id')
  @RequirePermission('users.delete')
  @ApiOperation({ summary: 'Delete user' })
  async remove(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ): Promise<void> {
    return this.usersService.remove(tenant.id, id);
  }
}
```

**Service:**

```typescript
import { Injectable, Logger, NotFoundException, ConflictException } from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';
import * as bcrypt from 'bcrypt';

import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name);

  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string): Promise<User[]> {
    // ✅ CRÍTICO: Set tenant context para RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    const users = await this.prisma.user.findMany({
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
        updatedAt: true,
        // ❌ NUNCA incluir password
      },
      orderBy: {
        createdAt: 'desc',
      },
    });

    return users;
  }

  async findOne(tenantId: string, id: string): Promise<User> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    const user = await this.prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
        updatedAt: true,
      },
    });

    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }

    return user;
  }

  async create(tenantId: string, dto: CreateUserDto): Promise<User> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Verificar email único dentro del tenant
    const existing = await this.prisma.user.findFirst({
      where: { email: dto.email },
    });

    if (existing) {
      throw new ConflictException('Email already exists');
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(dto.password, 10);

    const user = await this.prisma.user.create({
      data: {
        tenant_id: tenantId, // ✅ EXPLÍCITO
        email: dto.email,
        name: dto.name,
        password: hashedPassword,
        role: dto.role,
      },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
        updatedAt: true,
      },
    });

    this.logger.log(`User created: ${user.id} for tenant: ${tenantId}`);

    return user;
  }

  async update(tenantId: string, id: string, dto: UpdateUserDto): Promise<User> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Verify user exists
    const existing = await this.findOne(tenantId, id);

    // Si actualiza password, hashear
    let dataToUpdate: any = { ...dto };
    if (dto.password) {
      dataToUpdate.password = await bcrypt.hash(dto.password, 10);
    }

    const user = await this.prisma.user.update({
      where: { id },
      data: dataToUpdate,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
        updatedAt: true,
      },
    });

    return user;
  }

  async remove(tenantId: string, id: string): Promise<void> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    await this.findOne(tenantId, id); // Verify exists

    await this.prisma.user.delete({
      where: { id },
    });

    this.logger.log(`User deleted: ${id} for tenant: ${tenantId}`);
  }
}
```

**DTO:**

```typescript
import { IsEmail, IsString, MinLength, IsEnum, IsOptional } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { UserRole } from '@/shared/enums/user-role.enum';

export class CreateUserDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 'John Doe', minLength: 3 })
  @IsString()
  @MinLength(3)
  name: string;

  @ApiProperty({ example: 'SecurePass123!', minLength: 8 })
  @IsString()
  @MinLength(8)
  password: string;

  @ApiProperty({ enum: UserRole, example: UserRole.USER })
  @IsEnum(UserRole)
  role: UserRole;
}

export class UpdateUserDto {
  @ApiPropertyOptional({ example: 'user@example.com' })
  @IsOptional()
  @IsEmail()
  email?: string;

  @ApiPropertyOptional({ example: 'John Doe' })
  @IsOptional()
  @IsString()
  @MinLength(3)
  name?: string;

  @ApiPropertyOptional({ example: 'NewPass123!' })
  @IsOptional()
  @IsString()
  @MinLength(8)
  password?: string;

  @ApiPropertyOptional({ enum: UserRole })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;
}
```

---

## 📏 Convenciones Obligatorias

### Nomenclatura (ESTRICTAMENTE ENFORCEADO)

```typescript
// ✅ CORRECTO: Archivos - kebab-case
users.controller.ts
create-user.dto.ts
jwt-auth.guard.ts
tenant.interceptor.ts
user-role.enum.ts

// ✅ CORRECTO: Clases - PascalCase
export class UsersController { }
export class CreateUserDto { }
export class JwtAuthGuard { }
export class TenantInterceptor { }
export enum UserRole { }

// ✅ CORRECTO: Métodos y variables - camelCase
async findAll() { }
const userList = []
const currentTenant = request.tenant

// ✅ CORRECTO: Constantes - UPPER_SNAKE_CASE
const JWT_SECRET = process.env.JWT_SECRET
const DATABASE_URL = process.env.DATABASE_URL
const MAX_RETRY_ATTEMPTS = 3

// ❌ PROHIBIDO: snake_case en TypeScript
const user_name = "John"  // ❌ NUNCA
function get_user_data() { }  // ❌ NUNCA
class user_controller { }  // ❌ NUNCA
```

### Imports (Orden Obligatorio)

```typescript
// 1. NestJS core
import { Injectable, Logger, NotFoundException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

// 2. Librerías externas
import { PrismaService } from 'nestjs-prisma';
import * as bcrypt from 'bcrypt';

// 3. Módulos internos (con @)
import { UsersService } from '@/modules/users/users.service';
import { TenantsService } from '@/modules/tenants/tenants.service';

// 4. DTOs y Entities
import { CreateUserDto } from './dto/create-user.dto';
import { User } from './entities/user.entity';

// 5. Enums y Types
import { UserRole } from '@/shared/enums/user-role.enum';
import { ITenantContext } from '@/common/interfaces/tenant-context.interface';
```

### Estructura de Módulos

```
src/modules/resource/
├── resource.module.ts              # Module definition
├── resource.controller.ts          # HTTP endpoints
├── resource.service.ts             # Business logic
├── dto/
│   ├── create-resource.dto.ts      # DTO para POST
│   ├── update-resource.dto.ts      # DTO para PATCH
│   └── resource-filters.dto.ts     # DTO para query params
├── entities/
│   └── resource.entity.ts          # Entity (Swagger)
└── tests/
    ├── resource.controller.spec.ts # Unit tests para controller
    ├── resource.service.spec.ts    # Unit tests para service
    └── resource.e2e-spec.ts        # E2E tests
```

---

## 🐳 Desarrollo con Docker

### ⚠️ OBLIGATORIO: TODO se Desarrolla en Docker

```bash
# ❌ NUNCA ejecutar en tu máquina local:
npm install
npm run start:dev
npx prisma migrate dev

# ✅ SIEMPRE usar Docker:
docker compose up -d
docker compose exec backend npm install [package]
docker compose exec backend npx prisma migrate dev
```

### ¿Por Qué Docker es Obligatorio?

1. **Consistencia:** Mismo entorno en dev, staging, prod
2. **Row-Level Security:** Requiere configuración específica de PostgreSQL
3. **Onboarding:** Nuevo dev productivo en 5 minutos
4. **CI/CD Ready:** Mismo container en todo el pipeline
5. **No contaminación:** Dependencias aisladas de tu sistema

### Comandos Docker Esenciales

```bash
# Iniciar entorno completo
docker compose up -d

# Ver logs
docker compose logs -f backend
docker compose logs -f postgres

# Ejecutar comandos dentro del backend container
docker compose exec backend npm install [package]
docker compose exec backend npm run test
docker compose exec backend npx prisma migrate dev
docker compose exec backend npx prisma studio

# Detener entorno
docker compose down

# Detener y eliminar volúmenes (⚠️ borra datos)
docker compose down -v

# Rebuild backend image
docker compose build backend
```

---

## 🔐 Seguridad

### 7 Capas de Seguridad

1. **Network**
   - HTTPS obligatorio en producción
   - Rate Limiting: 10 req/min por IP
   - CORS configurado por dominio

2. **Authentication**
   - JWT con RS256 (asymmetric)
   - Access tokens: 15min TTL
   - Refresh tokens: 7 días TTL con rotación

3. **Authorization**
   - RBAC: admin > manager > user
   - Permisos granulares: `users.read`, `products.create`

4. **Tenant Isolation**
   - Header X-Tenant-ID validado
   - PostgreSQL RLS enforceado

5. **Input Validation**
   - class-validator en TODOS los DTOs
   - Pipes de validación globales

6. **Plan Limits**
   - PlanLimitGuard verifica antes de crear recursos
   - Límites enforceados a nivel de aplicación Y DB

7. **Audit**
   - Winston logging (info, warn, error)
   - Tabla `audit` para eventos críticos

### Buenas Prácticas de Seguridad

```typescript
// ✅ SIEMPRE hashear passwords
const hashedPassword = await bcrypt.hash(password, 10);

// ✅ NUNCA retornar passwords
return this.prisma.user.findMany({
  select: {
    id: true,
    email: true,
    // password: false  ← NO incluir
  },
});

// ✅ SIEMPRE validar DTOs
export class CreateUserDto {
  @IsEmail()
  email: string;

  @IsString()
  @MinLength(8)
  password: string;
}

// ✅ SIEMPRE set tenant context
await this.prisma.$executeRaw`
  SELECT set_config('app.current_tenant_id', ${tenantId}, true)
`;

// ❌ NUNCA hardcodear secrets
const JWT_SECRET = "my-secret-key";  // ❌ NUNCA

// ✅ SIEMPRE usar variables de entorno
const JWT_SECRET = this.configService.get<string>('JWT_SECRET');
```

---

## 🧪 Testing

### Objetivo de Coverage

- **Mínimo general:** 80%
- **Archivos críticos:** 95%+
  - Auth module
  - Tenant isolation
  - Plan limits
  - RLS enforcement

### Unit Tests

**Template para Services:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { UsersService } from './users.service';
import { PrismaService } from '@/database/prisma.service';

describe('UsersService', () => {
  let service: UsersService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UsersService,
        {
          provide: PrismaService,
          useValue: {
            $executeRaw: jest.fn(),
            user: {
              findMany: jest.fn(),
              findUnique: jest.fn(),
              create: jest.fn(),
              update: jest.fn(),
              delete: jest.fn(),
            },
          },
        },
      ],
    }).compile();

    service = module.get<UsersService>(UsersService);
    prisma = module.get<PrismaService>(PrismaService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('findAll', () => {
    it('should set tenant context before query', async () => {
      const tenantId = 'tenant-123';
      await service.findAll(tenantId);

      expect(prisma.$executeRaw).toHaveBeenCalledWith(
        expect.arrayContaining([expect.stringContaining('set_config')])
      );
    });

    it('should return array of users', async () => {
      const tenantId = 'tenant-123';
      const mockUsers = [
        { id: '1', email: 'user1@test.com', name: 'User 1' },
        { id: '2', email: 'user2@test.com', name: 'User 2' },
      ];

      (prisma.user.findMany as jest.Mock).mockResolvedValue(mockUsers);

      const result = await service.findAll(tenantId);

      expect(result).toEqual(mockUsers);
      expect(prisma.user.findMany).toHaveBeenCalled();
    });
  });

  describe('create', () => {
    it('should hash password before saving', async () => {
      const tenantId = 'tenant-123';
      const dto = {
        email: 'new@test.com',
        name: 'New User',
        password: 'Password123!',
        role: 'user' as UserRole,
      };

      const bcrypt = require('bcrypt');
      jest.spyOn(bcrypt, 'hash').mockResolvedValue('hashed_password');

      (prisma.user.findFirst as jest.Mock).mockResolvedValue(null);
      (prisma.user.create as jest.Mock).mockResolvedValue({
        id: '1',
        ...dto,
        password: 'hashed_password',
      });

      await service.create(tenantId, dto);

      expect(bcrypt.hash).toHaveBeenCalledWith(dto.password, 10);
    });

    it('should include tenant_id in create', async () => {
      const tenantId = 'tenant-123';
      const dto = {
        email: 'new@test.com',
        name: 'New User',
        password: 'Password123!',
        role: 'user' as UserRole,
      };

      (prisma.user.findFirst as jest.Mock).mockResolvedValue(null);
      (prisma.user.create as jest.Mock).mockResolvedValue({ id: '1' });

      await service.create(tenantId, dto);

      expect(prisma.user.create).toHaveBeenCalledWith(
        expect.objectContaining({
          data: expect.objectContaining({
            tenant_id: tenantId,
          }),
        })
      );
    });
  });
});
```

### E2E Tests

```bash
# Ejecutar E2E tests
docker compose exec backend npm run test:e2e
```

**Template:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { INestApplication, ValidationPipe } from '@nestjs/common';
import * as request from 'supertest';
import { AppModule } from '@/app.module';

describe('UsersController (e2e)', () => {
  let app: INestApplication;
  let authToken: string;
  let tenantId: string;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleFixture.createNestApplication();
    app.useGlobalPipes(new ValidationPipe());
    await app.init();

    // Login to get token
    const loginResponse = await request(app.getHttpServer())
      .post('/auth/login')
      .send({ email: 'admin@test.com', password: 'Password123!' });

    authToken = loginResponse.body.access_token;
    tenantId = loginResponse.body.tenants[0].id;
  });

  afterAll(async () => {
    await app.close();
  });

  describe('/users (GET)', () => {
    it('should return 401 without auth token', () => {
      return request(app.getHttpServer())
        .get('/users')
        .expect(401);
    });

    it('should return 403 without tenant header', () => {
      return request(app.getHttpServer())
        .get('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(403);
    });

    it('should return users array with valid auth', () => {
      return request(app.getHttpServer())
        .get('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .set('X-Tenant-ID', tenantId)
        .expect(200)
        .expect((res) => {
          expect(Array.isArray(res.body)).toBe(true);
        });
    });
  });

  describe('/users (POST)', () => {
    it('should create user with valid data', () => {
      const newUser = {
        email: 'newuser@test.com',
        name: 'New User',
        password: 'Password123!',
        role: 'user',
      };

      return request(app.getHttpServer())
        .post('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .set('X-Tenant-ID', tenantId)
        .send(newUser)
        .expect(201)
        .expect((res) => {
          expect(res.body).toHaveProperty('id');
          expect(res.body.email).toBe(newUser.email);
          expect(res.body).not.toHaveProperty('password');
        });
    });

    it('should validate DTO and return 400', () => {
      const invalidUser = {
        email: 'invalid-email',  // Invalid email
        name: 'A',  // Too short
        password: '123',  // Too short
      };

      return request(app.getHttpServer())
        .post('/users')
        .set('Authorization', `Bearer ${authToken}`)
        .set('X-Tenant-ID', tenantId)
        .send(invalidUser)
        .expect(400);
    });
  });
});
```

---

## 🚨 Prohibiciones Absolutas

### ❌ NUNCA Hagas Esto

```typescript
// ❌ 1. snake_case en TypeScript
const user_name = "John"
function get_user_data() { }
class user_controller { }

// ❌ 2. any type
const data: any = await fetchData()
function processData(input: any) { }

// ❌ 3. console.log en producción
console.log('User:', user)
console.error('Error:', error)

// ❌ 4. Queries sin tenant context
const users = await this.prisma.user.findMany()  // Retorna TODOS

// ❌ 5. Passwords en responses
return { id, email, password }

// ❌ 6. Hardcodear secrets
const JWT_SECRET = "my-secret-key"

// ❌ 7. Controllers sin guards
@Get()
async findAll() {
  // Falta @UseGuards(TenantGuard)
}

// ❌ 8. DTOs sin validación
export class CreateUserDto {
  email: string  // Falta @IsEmail()
  password: string  // Falta @MinLength()
}

// ❌ 9. Inserts sin tenant_id
await this.prisma.user.create({
  data: {
    email: dto.email,
    // Falta tenant_id
  },
});

// ❌ 10. Passwords sin hashear
await this.prisma.user.create({
  data: {
    password: dto.password,  // SIN HASHEAR
  },
});
```

---

## 🧠 Sistema de Memoria

### Checkpoint Cada 5 Mensajes

Cada ~5 mensajes en una conversación, verifica mentalmente:

```markdown
✅ Proyecto: BO Backend API Multi-Tenant
✅ Framework: NestJS 10 + TypeScript 5
✅ Database: PostgreSQL 15 con RLS
✅ Multi-tenant: X-Tenant-ID header + RLS automático
✅ Convenciones: kebab-case (archivos), PascalCase (clases), camelCase (métodos)
✅ NUNCA: snake_case, any type, queries sin tenant
✅ SIEMPRE: set_config para RLS, hashear passwords, validar DTOs
```

### Actualizar Estado al Final de Sesión

Antes de terminar, actualizar `.claude/session-state.md`:

```markdown
## Última Actualización
- Fecha: YYYY-MM-DD HH:mm
- Módulos creados: Auth, Tenants, Users
- Endpoints implementados: 15
- Tests escritos: 42
- Coverage: 85%
- Próximo: Dashboard Module
```

### Recuperación de Contexto

Si pierdes contexto en medio de una conversación:

1. Lee `.claude/context.md` inmediatamente
2. Lee `.claude/session-state.md` para saber dónde quedaste
3. Confirma que recuerdas las convenciones
4. Pregunta: "¿En qué módulo/endpoint estábamos trabajando?"

---

## 📚 Recursos Adicionales

### Documentación

- **NestJS:** https://docs.nestjs.com
- **Prisma:** https://www.prisma.io/docs
- **PostgreSQL RLS:** https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- **class-validator:** https://github.com/typestack/class-validator

### Archivos Críticos del Proyecto

- `CODING_STANDARDS.md` - Estándares detallados
- `ARQUITECTURA.md` - Decisiones arquitectónicas
- `QUICKSTART.md` - Setup en 5 minutos
- `.claude/conventions.md` - Convenciones obligatorias

---

<div align="center">

**Sigue estas instrucciones SIEMPRE**

Backend Multi-Tenant Profesional con NestJS + PostgreSQL RLS

Type-safe | Isolated | Scalable | Docker-first

</div>
