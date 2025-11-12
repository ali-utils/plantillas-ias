# Convenciones de Código - Backend NestJS - OBLIGATORIAS

> **Reglas que NUNCA se violan**

---

## 🔤 Nomenclatura

### Archivos: `kebab-case`

```typescript
// ✅ CORRECTO
users.controller.ts
users.service.ts
create-user.dto.ts
user.entity.ts
tenant.guard.ts
jwt-auth.strategy.ts

// ❌ INCORRECTO
UsersController.ts           // PascalCase
users_controller.ts          // snake_case
usersController.ts           // camelCase
```

### Clases: `PascalCase`

```typescript
// ✅ CORRECTO
export class UsersController { }
export class UsersService { }
export class CreateUserDto { }
export class User { }
export class TenantGuard { }
export class JwtAuthStrategy { }

// ❌ INCORRECTO
export class usersController { }  // camelCase
export class users_service { }    // snake_case
```

### Métodos y Variables: `camelCase`

```typescript
// ✅ CORRECTO
async findAll() { }
async createUser() { }
const userList = []
const currentTenant = request.tenant
const hasPermission = true

// ❌ INCORRECTO
async FindAll() { }          // PascalCase
async create_user() { }      // snake_case
const UserList = []          // PascalCase
const current_tenant = {}    // snake_case
```

### Constantes y Enums: `UPPER_SNAKE_CASE` o `PascalCase`

```typescript
// ✅ CORRECTO - Constantes
export const JWT_SECRET = process.env.JWT_SECRET
export const DATABASE_URL = process.env.DATABASE_URL
export const MAX_RETRIES = 3

// ✅ CORRECTO - Enums
export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user'
}

export enum PlanType {
  FREE = 'free',
  PLAN1 = 'plan1',
  PLAN2 = 'plan2',
  PLAN3 = 'plan3'
}

// ❌ INCORRECTO
export const jwtSecret = "..."      // camelCase
export const jwt_secret = "..."     // snake_case en const
```

### Interfaces: `I` + `PascalCase` (opcional en NestJS)

```typescript
// ✅ CORRECTO - Con prefijo I (cuando no es entity)
interface ITenantContext {
  id: string
  name: string
  plan: PlanType
}

interface IAuthPayload {
  userId: string
  email: string
  tenantIds: string[]
}

// ✅ TAMBIÉN CORRECTO - Sin prefijo (para entities/DTOs)
export class User {
  id: string
  email: string
  name: string
}

// ❌ INCORRECTO
interface tenant_context { }      // snake_case
interface tenantContext { }       // camelCase en interface
```

---

## 🏢 Multi-Tenant OBLIGATORIO

### Siempre Usar Guards y Decorators

```typescript
// ✅ CORRECTO
@Controller('users')
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {

  @Get()
  @RequirePermission('users.read')
  async findAll(@Tenant() tenant: ITenantContext) {
    return this.usersService.findAll(tenant.id)
  }

  @Post()
  @RequirePermission('users.create')
  @CheckPlanLimit('users')
  async create(
    @Tenant() tenant: ITenantContext,
    @Body() dto: CreateUserDto,
  ) {
    return this.usersService.create(tenant.id, dto)
  }
}

// ❌ INCORRECTO - Falta tenant validation
@Controller('users')
export class UsersController {
  @Get()
  async findAll() {
    // ❌ No valida tenant
    return this.usersService.findAll()
  }
}
```

### Siempre Set PostgreSQL Session (RLS)

```typescript
// ✅ CORRECTO
@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string, filters: UserFiltersDto) {
    // Set tenant context para RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    return this.prisma.user.findMany({
      where: filters,
    })
  }
}

// ❌ INCORRECTO - Sin set tenant
async findAll() {
  // ❌ RLS no se aplica, retorna TODOS los datos
  return this.prisma.user.findMany()
}
```

### Siempre Incluir tenant_id en Inserts

```typescript
// ✅ CORRECTO
async create(tenantId: string, dto: CreateUserDto) {
  await this.prisma.$executeRaw`
    SELECT set_config('app.current_tenant_id', ${tenantId}, true)
  `

  return this.prisma.user.create({
    data: {
      tenant_id: tenantId,  // ← EXPLÍCITO
      ...dto,
    },
  })
}

// ❌ INCORRECTO - Falta tenant_id
async create(dto: CreateUserDto) {
  // ❌ tenant_id es NULL o undefined
  return this.prisma.user.create({
    data: dto,
  })
}
```

---

## 📦 DTOs con Validación

### Siempre Usar class-validator

```typescript
// ✅ CORRECTO
import { IsEmail, IsString, MinLength, IsEnum, IsOptional } from 'class-validator'
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger'

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

  @ApiPropertyOptional({ example: '+1234567890' })
  @IsOptional()
  @IsString()
  phone?: string
}

// ❌ INCORRECTO - Sin validación
export class CreateUserDto {
  email: string        // ❌ Sin @IsEmail()
  name: string         // ❌ Sin @IsString()
  password: string     // ❌ Sin @MinLength()
  role: UserRole       // ❌ Sin @IsEnum()
}
```

### Usar PartialType para Updates

```typescript
// ✅ CORRECTO
import { PartialType } from '@nestjs/swagger'

export class UpdateUserDto extends PartialType(CreateUserDto) {
  // Todos los campos de CreateUserDto ahora son opcionales
}

// ❌ INCORRECTO - Duplicar código
export class UpdateUserDto {
  @IsOptional()
  @IsEmail()
  email?: string

  @IsOptional()
  @IsString()
  name?: string
  // ... repetir todos los campos
}
```

---

## 🧪 Testing OBLIGATORIO

### Coverage Mínimo: 80%

```typescript
// users.service.spec.ts
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

  describe('findAll', () => {
    it('should set tenant context', async () => {
      const tenantId = 'tenant-123'
      await service.findAll(tenantId, {})

      expect(prisma.$executeRaw).toHaveBeenCalledWith(
        expect.arrayContaining([expect.stringContaining('set_config')])
      )
    })

    it('should return users', async () => {
      const mockUsers = [
        { id: '1', email: 'user1@test.com', name: 'User 1' },
      ]
      jest.spyOn(prisma.user, 'findMany').mockResolvedValue(mockUsers)

      const result = await service.findAll('tenant-123', {})
      expect(result).toEqual(mockUsers)
    })
  })
})
```

---

## 📋 Imports Ordenados

```typescript
// 1. NestJS core
import { Injectable, Logger, BadRequestException } from '@nestjs/common'
import { ConfigService } from '@nestjs/config'

// 2. Librerías externas
import { PrismaService } from 'nestjs-prisma'
import * as bcrypt from 'bcrypt'

// 3. Módulos internos (absolutos con @)
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
// ❌ snake_case en TypeScript
const user_name = "John"
function get_user_data() { }

// ❌ any type
const data: any = await fetchData()
function processData(input: any) { }

// ❌ console.log en producción
console.log('User data:', user)  // Usar Logger

// ❌ Queries sin tenant_id
this.prisma.user.findMany()  // Retorna TODOS

// ❌ Passwords en response
return {
  id: user.id,
  password: user.password  // ❌ NUNCA
}

// ❌ Hardcodear secrets
const JWT_SECRET = "my-secret"

// ❌ Endpoints sin guards
@Get()
async findAll() {
  // ❌ Falta @UseGuards(JwtAuthGuard, TenantGuard)
}

// ❌ Controllers > 300 líneas
// Dividir en múltiples controllers

// ❌ Services con lógica de negocio en controllers
@Post()
async create(@Body() dto: CreateUserDto) {
  // ❌ Lógica aquí, debería estar en service
  const hashedPassword = await bcrypt.hash(dto.password, 10)
  return this.prisma.user.create({ data: { ...dto, password: hashedPassword } })
}
```

---

## ✅ TEMPLATE de Controller

```typescript
import { Controller, Get, Post, Body, Param, Query, UseGuards, UseInterceptors, Logger } from '@nestjs/common'
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger'

import { UsersService } from './users.service'
import { CreateUserDto } from './dto/create-user.dto'
import { UpdateUserDto } from './dto/update-user.dto'
import { UserFiltersDto } from './dto/user-filters.dto'
import { User } from './entities/user.entity'

import { JwtAuthGuard } from '@/common/guards/jwt-auth.guard'
import { TenantGuard } from '@/common/guards/tenant.guard'
import { TenantInterceptor } from '@/common/interceptors/tenant.interceptor'
import { AuditInterceptor } from '@/common/interceptors/audit.interceptor'
import { Tenant } from '@/common/decorators/tenant.decorator'
import { RequirePermission } from '@/common/decorators/permissions.decorator'
import { CheckPlanLimit } from '@/common/decorators/plan-limit.decorator'
import { ITenantContext } from '@/common/interfaces/tenant-context.interface'

@Controller('users')
@ApiTags('users')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor, AuditInterceptor)
export class UsersController {
  private readonly logger = new Logger(UsersController.name)

  constructor(private readonly usersService: UsersService) {}

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

  @Get(':id')
  @RequirePermission('users.read')
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiResponse({ status: 200, type: User })
  async findOne(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ) {
    return this.usersService.findOne(tenant.id, id)
  }

  @Post()
  @RequirePermission('users.create')
  @CheckPlanLimit('users')
  @ApiOperation({ summary: 'Create user' })
  @ApiResponse({ status: 201, type: User })
  async create(
    @Tenant() tenant: ITenantContext,
    @Body() dto: CreateUserDto,
  ) {
    return this.usersService.create(tenant.id, dto)
  }

  @Patch(':id')
  @RequirePermission('users.update')
  @ApiOperation({ summary: 'Update user' })
  @ApiResponse({ status: 200, type: User })
  async update(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
    @Body() dto: UpdateUserDto,
  ) {
    return this.usersService.update(tenant.id, id, dto)
  }

  @Delete(':id')
  @RequirePermission('users.delete')
  @ApiOperation({ summary: 'Delete user' })
  @ApiResponse({ status: 200 })
  async remove(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ) {
    return this.usersService.remove(tenant.id, id)
  }
}
```

---

## ✅ TEMPLATE de Service

```typescript
import { Injectable, Logger, NotFoundException } from '@nestjs/common'
import { PrismaService } from '@/database/prisma.service'
import * as bcrypt from 'bcrypt'

import { CreateUserDto } from './dto/create-user.dto'
import { UpdateUserDto } from './dto/update-user.dto'
import { UserFiltersDto } from './dto/user-filters.dto'

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name)

  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string, filters: UserFiltersDto) {
    // Set tenant context para RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    return this.prisma.user.findMany({
      where: {
        email: filters.email ? { contains: filters.email } : undefined,
        role: filters.role,
      },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
        updatedAt: true,
        // NO incluir password
      },
    })
  }

  async findOne(tenantId: string, id: string) {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

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
    })

    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`)
    }

    return user
  }

  async create(tenantId: string, dto: CreateUserDto) {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    // Hash password
    const hashedPassword = await bcrypt.hash(dto.password, 10)

    return this.prisma.user.create({
      data: {
        tenant_id: tenantId,
        ...dto,
        password: hashedPassword,
      },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        // NO retornar password
      },
    })
  }

  async update(tenantId: string, id: string, dto: UpdateUserDto) {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    // Si actualiza password, hashear
    if (dto.password) {
      dto.password = await bcrypt.hash(dto.password, 10)
    }

    return this.prisma.user.update({
      where: { id },
      data: dto,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
      },
    })
  }

  async remove(tenantId: string, id: string) {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `

    await this.prisma.user.delete({
      where: { id },
    })

    return { message: 'User deleted successfully' }
  }
}
```

---

<div align="center">

**Estas convenciones NO son negociables**

NestJS + TypeScript + Multi-Tenant = Código Profesional

</div>
