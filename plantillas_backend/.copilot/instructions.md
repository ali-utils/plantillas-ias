# Instrucciones para GitHub Copilot - Backend NestJS

> **Configuración para GitHub Copilot en bMOI Backend API Multi-Tenant**

---

## 🏢 Proyecto

```
Nombre: bMOI Backend API Multi-Tenant
Tipo: API REST Multi-Tenant con NestJS
Framework: NestJS 10 + TypeScript 5
Database: PostgreSQL 15 con Row-Level Security
Entorno: Docker (OBLIGATORIO)
```

## 🐳 Desarrollo con Docker (OBLIGATORIO)

**⚠️ TODO el desarrollo se hace en Docker**

```bash
# ✅ Iniciar backend dockerizado
docker compose up -d

# ✅ Ver logs
docker compose logs -f backend

# ✅ Ejecutar comandos dentro del container
docker compose exec backend npm install [package]
docker compose exec backend npm run test
docker compose exec backend npx prisma migrate dev
docker compose exec backend npx prisma studio

# ✅ Detener
docker compose down

# ❌ NUNCA ejecutar directamente:
npm install  # NO
npm run start:dev  # NO
npx prisma migrate dev  # NO
```

**Razón:** Consistencia, aislamiento, CI/CD ready

---

## 🔤 Convenciones de Nomenclatura

```typescript
// ✅ Archivos: kebab-case
users.controller.ts
create-user.dto.ts
jwt-auth.guard.ts
tenant.interceptor.ts

// ✅ Clases: PascalCase
export class UsersController { }
export class CreateUserDto { }
export class JwtAuthGuard { }
export class TenantInterceptor { }

// ✅ Métodos y variables: camelCase
async findAll() { }
const userList = []
const currentTenant = request.tenant

// ✅ Constantes: UPPER_SNAKE_CASE
const JWT_SECRET = process.env.JWT_SECRET
const DATABASE_URL = process.env.DATABASE_URL

// ✅ Enums: PascalCase + UPPER_CASE values
export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user'
}

// ❌ PROHIBIDO: snake_case
const user_name = "John"  // NUNCA
function get_user() { }   // NUNCA
```

---

## 🏢 Multi-Tenant Patterns (OBLIGATORIO)

### Template de Controller

```typescript
import { Controller, Get, Post, Body, UseGuards, UseInterceptors, Logger } from '@nestjs/common'
import { ApiTags, ApiOperation, ApiBearerAuth } from '@nestjs/swagger'

import { JwtAuthGuard } from '@/common/guards/jwt-auth.guard'
import { TenantGuard } from '@/common/guards/tenant.guard'
import { TenantInterceptor } from '@/common/interceptors/tenant.interceptor'
import { Tenant } from '@/common/decorators/tenant.decorator'
import { RequirePermission } from '@/common/decorators/permissions.decorator'
import { CheckPlanLimit } from '@/common/decorators/plan-limit.decorator'
import { ITenantContext } from '@/common/interfaces/tenant-context.interface'

@Controller('users')
@ApiTags('users')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {
  private readonly logger = new Logger(UsersController.name)

  constructor(private readonly usersService: UsersService) {}

  @Get()
  @RequirePermission('users.read')
  @ApiOperation({ summary: 'Get all users' })
  async findAll(@Tenant() tenant: ITenantContext) {
    return this.usersService.findAll(tenant.id)
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
```

### Template de Service

```typescript
import { Injectable, Logger, NotFoundException } from '@nestjs/common'
import { PrismaService } from '@/database/prisma.service'
import * as bcrypt from 'bcrypt'

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name)

  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string) {
    // ✅ SIEMPRE set tenant context para RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    return this.prisma.user.findMany({
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        // NO incluir password
      },
    })
  }

  async create(tenantId: string, dto: CreateUserDto) {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    const hashedPassword = await bcrypt.hash(dto.password, 10)

    return this.prisma.user.create({
      data: {
        tenant_id: tenantId,  // ← EXPLÍCITO
        ...dto,
        password: hashedPassword,
      },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
      },
    })
  }
}
```

### Template de DTO

```typescript
import { IsEmail, IsString, MinLength, IsEnum } from 'class-validator'
import { ApiProperty } from '@nestjs/swagger'
import { UserRole } from '@/shared/enums/user-role.enum'

export class CreateUserDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail()
  email: string

  @ApiProperty({ example: 'John Doe', minLength: 3 })
  @IsString()
  @MinLength(3)
  name: string

  @ApiProperty({ example: 'SecurePass123!', minLength: 8 })
  @IsString()
  @MinLength(8)
  password: string

  @ApiProperty({ enum: UserRole, example: UserRole.USER })
  @IsEnum(UserRole)
  role: UserRole
}
```

### Template de Test

```typescript
import { Test, TestingModule } from '@nestjs/testing'
import { UsersService } from './users.service'
import { PrismaService } from '@/database/prisma.service'

describe('UsersService', () => {
  let service: UsersService
  let prisma: PrismaService

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
              create: jest.fn(),
            },
          },
        },
      ],
    }).compile()

    service = module.get<UsersService>(UsersService)
    prisma = module.get<PrismaService>(PrismaService)
  })

  it('should be defined', () => {
    expect(service).toBeDefined()
  })

  it('should set tenant context before query', async () => {
    const tenantId = 'tenant-123'
    await service.findAll(tenantId)

    expect(prisma.$executeRaw).toHaveBeenCalledWith(
      expect.arrayContaining([expect.stringContaining('set_config')])
    )
  })
})
```

---

## 📋 Orden de Imports

```typescript
// 1. NestJS core
import { Injectable, Logger } from '@nestjs/common'
import { ConfigService } from '@nestjs/config'

// 2. Librerías externas
import { PrismaService } from 'nestjs-prisma'
import * as bcrypt from 'bcrypt'

// 3. Módulos internos (con @)
import { UsersService } from '@/modules/users/users.service'
import { TenantsService } from '@/modules/tenants/tenants.service'

// 4. DTOs y Entities
import { CreateUserDto } from './dto/create-user.dto'
import { User } from './entities/user.entity'

// 5. Enums y Types
import { UserRole } from '@/shared/enums/user-role.enum'
import { ITenantContext } from '@/common/interfaces/tenant-context.interface'
```

---

## 🚫 PROHIBIDO

```typescript
// ❌ snake_case
const user_name = "John"
function get_user_data() { }

// ❌ any type
const data: any = await fetchData()

// ❌ console.log
console.log('User:', user)  // Usar Logger

// ❌ Queries sin tenant
this.prisma.user.findMany()  // Retorna TODOS

// ❌ Passwords en response
return { id, email, password }  // ❌ NUNCA

// ❌ Hardcodear secrets
const JWT_SECRET = "my-secret"

// ❌ Controllers sin guards
@Get()
async findAll() {
  // ❌ Falta @UseGuards(TenantGuard)
}

// ❌ DTOs sin validación
export class CreateUserDto {
  email: string  // ❌ Falta @IsEmail()
}
```

---

## ✅ SIEMPRE

```typescript
// ✅ Guards en controllers
@UseGuards(JwtAuthGuard, TenantGuard)

// ✅ Interceptors para tenant
@UseInterceptors(TenantInterceptor)

// ✅ Set tenant context en services
await this.prisma.$executeRaw`
  SELECT set_config('app.current_tenant_id', ${tenantId}, true)
`

// ✅ Validación en DTOs
@IsEmail()
@IsString()
@MinLength(8)

// ✅ Logger en lugar de console.log
this.logger.log('User created')
this.logger.error('Error:', error)

// ✅ Hash passwords
const hashed = await bcrypt.hash(password, 10)

// ✅ NO retornar passwords
select: {
  id: true,
  email: true,
  // password: false  ← NO incluir
}

// ✅ Tests con coverage >80%
describe('UsersService', () => {
  it('should...', () => { })
})
```

---

## 📚 Referencia Rápida

### Decorators Comunes

```typescript
// Controllers
@Controller('path')
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
@ApiTags('resource')
@ApiBearerAuth()

// Endpoints
@Get()
@Post()
@Put(':id')
@Patch(':id')
@Delete(':id')

// Params
@Param('id') id: string
@Query() filters: FilterDto
@Body() dto: CreateDto
@Tenant() tenant: ITenantContext

// Custom
@RequirePermission('users.read')
@CheckPlanLimit('users')
@Roles('admin', 'manager')

// Swagger
@ApiOperation({ summary: '...' })
@ApiResponse({ status: 200, type: User })
@ApiProperty({ example: '...' })
```

### Guards y Interceptors

```typescript
// Guards disponibles
JwtAuthGuard         // Valida JWT token
TenantGuard          // Valida tenant access
PermissionGuard      // Valida permisos
PlanLimitGuard       // Valida límites del plan
RolesGuard           // Valida roles

// Interceptors disponibles
TenantInterceptor    // Inyecta tenant context
AuditInterceptor     // Logging de auditoría
LoggingInterceptor   // Logging general
TransformInterceptor // Transforma response
```

### Services Disponibles

```typescript
PrismaService        // Database ORM
ConfigService        // Environment vars
Logger               // Winston logger
```

---

<div align="center">

**Sigue estas instrucciones SIEMPRE**

NestJS Multi-Tenant Backend

</div>
