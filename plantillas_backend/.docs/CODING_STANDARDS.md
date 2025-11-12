# CODING_STANDARDS.md - Backend NestJS

> **Estándares de Código Detallados con Ejemplos**
>
> Este documento contiene estándares de código específicos para el backend NestJS multi-tenant.

---

## 📋 Tabla de Contenidos

- [Nomenclatura](#nomenclatura)
- [Estructura de Archivos](#estructura-de-archivos)
- [Controllers](#controllers)
- [Services](#services)
- [DTOs](#dtos)
- [Entities](#entities)
- [Guards](#guards)
- [Interceptors](#interceptors)
- [Decorators](#decorators)
- [Error Handling](#error-handling)
- [Logging](#logging)
- [Testing](#testing)
- [Performance](#performance)

---

## 📝 Nomenclatura

### Archivos

```typescript
// ✅ CORRECTO: kebab-case
users.controller.ts
users.service.ts
create-user.dto.ts
update-user.dto.ts
user.entity.ts
jwt-auth.guard.ts
tenant.guard.ts
tenant.interceptor.ts
audit.interceptor.ts
user-role.enum.ts
tenant-context.interface.ts

// ❌ INCORRECTO
UsersController.ts  // PascalCase
users_controller.ts  // snake_case
usersController.ts  // camelCase
```

### Clases

```typescript
// ✅ CORRECTO: PascalCase
export class UsersController { }
export class UsersService { }
export class CreateUserDto { }
export class UpdateUserDto { }
export class User { }
export class JwtAuthGuard { }
export class TenantGuard { }
export class TenantInterceptor { }
export class AuditInterceptor { }

// ❌ INCORRECTO
export class usersController { }  // camelCase
export class users_service { }  // snake_case
export class USERS_CONTROLLER { }  // UPPER_CASE
```

### Métodos y Variables

```typescript
// ✅ CORRECTO: camelCase
async findAll() { }
async findOne(id: string) { }
async create(dto: CreateUserDto) { }
async update(id: string, dto: UpdateUserDto) { }
async remove(id: string) { }

const userList = []
const currentTenant = request.tenant
const isAuthenticated = true
const hasPermission = false

// ❌ INCORRECTO
async FindAll() { }  // PascalCase
async find_one() { }  // snake_case
const UserList = []  // PascalCase
const current_tenant = {}  // snake_case
```

### Constantes

```typescript
// ✅ CORRECTO: UPPER_SNAKE_CASE
const JWT_SECRET = process.env.JWT_SECRET
const DATABASE_URL = process.env.DATABASE_URL
const MAX_RETRY_ATTEMPTS = 3
const DEFAULT_PAGE_SIZE = 20
const BCRYPT_ROUNDS = 10

// ❌ INCORRECTO
const jwtSecret = process.env.JWT_SECRET  // camelCase
const databaseUrl = process.env.DATABASE_URL  // camelCase
```

### Enums

```typescript
// ✅ CORRECTO: PascalCase para nombre, UPPER_CASE para valores
export enum UserRole {
  ADMIN = 'admin',
  MANAGER = 'manager',
  USER = 'user',
}

export enum PlanType {
  FREE = 'free',
  PLAN1 = 'plan1',
  PLAN2 = 'plan2',
  PLAN3 = 'plan3',
}

export enum AuditAction {
  CREATE = 'create',
  UPDATE = 'update',
  DELETE = 'delete',
  LOGIN = 'login',
  LOGOUT = 'logout',
}

// ❌ INCORRECTO
export enum userRole { }  // camelCase
export enum USER_ROLE { }  // UPPER_CASE
export enum user_role { }  // snake_case

export enum UserRole {
  admin = 'admin',  // lowercase
  Manager = 'manager',  // PascalCase
}
```

### Interfaces

```typescript
// ✅ CORRECTO: PascalCase con prefijo I
export interface ITenantContext {
  id: string;
  name: string;
  plan: PlanType;
}

export interface IAuthPayload {
  userId: string;
  tenantId: string;
  role: UserRole;
}

export interface IPaginationOptions {
  page: number;
  limit: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
}

// ❌ INCORRECTO
export interface TenantContext { }  // Sin prefijo I
export interface iTenantContext { }  // i minúscula
export interface tenant_context { }  // snake_case
```

---

## 📂 Estructura de Archivos

### Módulo Completo

```
src/modules/users/
├── users.module.ts
├── users.controller.ts
├── users.service.ts
├── dto/
│   ├── create-user.dto.ts
│   ├── update-user.dto.ts
│   ├── user-filters.dto.ts
│   └── user-pagination.dto.ts
├── entities/
│   └── user.entity.ts
├── enums/
│   └── user-status.enum.ts
├── interfaces/
│   └── user-response.interface.ts
└── tests/
    ├── users.controller.spec.ts
    ├── users.service.spec.ts
    └── users.e2e-spec.ts
```

### Organización de src/

```
src/
├── common/
│   ├── decorators/
│   │   ├── tenant.decorator.ts
│   │   ├── permissions.decorator.ts
│   │   ├── plan-limit.decorator.ts
│   │   └── current-user.decorator.ts
│   ├── guards/
│   │   ├── jwt-auth.guard.ts
│   │   ├── tenant.guard.ts
│   │   ├── permission.guard.ts
│   │   └── plan-limit.guard.ts
│   ├── interceptors/
│   │   ├── tenant.interceptor.ts
│   │   ├── audit.interceptor.ts
│   │   ├── logging.interceptor.ts
│   │   └── transform.interceptor.ts
│   ├── pipes/
│   │   ├── validation.pipe.ts
│   │   └── parse-uuid.pipe.ts
│   ├── filters/
│   │   ├── http-exception.filter.ts
│   │   └── prisma-exception.filter.ts
│   └── interfaces/
│       ├── tenant-context.interface.ts
│       ├── auth-payload.interface.ts
│       └── pagination.interface.ts
├── config/
│   ├── database.config.ts
│   ├── jwt.config.ts
│   ├── redis.config.ts
│   └── app.config.ts
├── database/
│   ├── prisma.service.ts
│   └── prisma.module.ts
├── modules/
│   ├── auth/
│   ├── tenants/
│   ├── users/
│   └── ...
├── shared/
│   ├── enums/
│   │   ├── user-role.enum.ts
│   │   └── plan-type.enum.ts
│   └── constants/
│       └── app.constants.ts
├── app.module.ts
└── main.ts
```

---

## 🎮 Controllers

### Template Completo

```typescript
import {
  Controller,
  Get,
  Post,
  Put,
  Patch,
  Delete,
  Body,
  Param,
  Query,
  UseGuards,
  UseInterceptors,
  Logger,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';
import {
  ApiTags,
  ApiOperation,
  ApiResponse,
  ApiBearerAuth,
  ApiParam,
  ApiQuery,
} from '@nestjs/swagger';

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
import { UserFiltersDto } from './dto/user-filters.dto';
import { User } from './entities/user.entity';

@Controller('users')
@ApiTags('users')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {
  private readonly logger = new Logger(UsersController.name);

  constructor(private readonly usersService: UsersService) {}

  // GET /users
  @Get()
  @RequirePermission('users.read')
  @ApiOperation({ summary: 'Get all users for current tenant' })
  @ApiResponse({ status: 200, description: 'Users retrieved successfully', type: [User] })
  @ApiResponse({ status: 403, description: 'Forbidden' })
  async findAll(
    @Tenant() tenant: ITenantContext,
    @Query() filters: UserFiltersDto,
  ): Promise<User[]> {
    this.logger.log(`Finding all users for tenant: ${tenant.id}`);
    return this.usersService.findAll(tenant.id, filters);
  }

  // GET /users/:id
  @Get(':id')
  @RequirePermission('users.read')
  @ApiOperation({ summary: 'Get user by ID' })
  @ApiParam({ name: 'id', type: 'string', format: 'uuid' })
  @ApiResponse({ status: 200, description: 'User found', type: User })
  @ApiResponse({ status: 404, description: 'User not found' })
  async findOne(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ): Promise<User> {
    this.logger.log(`Finding user ${id} for tenant: ${tenant.id}`);
    return this.usersService.findOne(tenant.id, id);
  }

  // POST /users
  @Post()
  @RequirePermission('users.create')
  @CheckPlanLimit('users')
  @ApiOperation({ summary: 'Create new user' })
  @ApiResponse({ status: 201, description: 'User created successfully', type: User })
  @ApiResponse({ status: 400, description: 'Validation error' })
  @ApiResponse({ status: 409, description: 'Email already exists' })
  async create(
    @Tenant() tenant: ITenantContext,
    @Body() createUserDto: CreateUserDto,
  ): Promise<User> {
    this.logger.log(`Creating user for tenant: ${tenant.id}`);
    return this.usersService.create(tenant.id, createUserDto);
  }

  // PATCH /users/:id
  @Patch(':id')
  @RequirePermission('users.update')
  @ApiOperation({ summary: 'Update user' })
  @ApiResponse({ status: 200, description: 'User updated successfully', type: User })
  @ApiResponse({ status: 404, description: 'User not found' })
  async update(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ): Promise<User> {
    this.logger.log(`Updating user ${id} for tenant: ${tenant.id}`);
    return this.usersService.update(tenant.id, id, updateUserDto);
  }

  // DELETE /users/:id
  @Delete(':id')
  @RequirePermission('users.delete')
  @HttpCode(HttpStatus.NO_CONTENT)
  @ApiOperation({ summary: 'Delete user' })
  @ApiResponse({ status: 204, description: 'User deleted successfully' })
  @ApiResponse({ status: 404, description: 'User not found' })
  async remove(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ): Promise<void> {
    this.logger.log(`Deleting user ${id} for tenant: ${tenant.id}`);
    return this.usersService.remove(tenant.id, id);
  }
}
```

### Reglas para Controllers

1. **SIEMPRE** usar Guards: `@UseGuards(JwtAuthGuard, TenantGuard)`
2. **SIEMPRE** usar Interceptors: `@UseInterceptors(TenantInterceptor)`
3. **SIEMPRE** documentar con Swagger: `@ApiTags`, `@ApiOperation`, `@ApiResponse`
4. **SIEMPRE** usar Logger de NestJS (NO `console.log`)
5. **SIEMPRE** extraer tenant con `@Tenant()` decorator
6. **SIEMPRE** validar permisos con `@RequirePermission()`
7. **SIEMPRE** verificar plan limits con `@CheckPlanLimit()` en POST

---

## 🔧 Services

### Template Completo

```typescript
import {
  Injectable,
  Logger,
  NotFoundException,
  ConflictException,
  BadRequestException,
} from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';
import * as bcrypt from 'bcrypt';

import { CreateUserDto } from './dto/create-user.dto';
import { UpdateUserDto } from './dto/update-user.dto';
import { UserFiltersDto } from './dto/user-filters.dto';
import { User } from './entities/user.entity';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name);

  constructor(private readonly prisma: PrismaService) {}

  /**
   * Find all users for a tenant with optional filters
   * @param tenantId - Tenant UUID
   * @param filters - Optional filters (role, status, search)
   * @returns Array of users
   */
  async findAll(tenantId: string, filters?: UserFiltersDto): Promise<User[]> {
    // ✅ CRÍTICO: Set tenant context para RLS
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    const where: any = {};

    if (filters?.role) {
      where.role = filters.role;
    }

    if (filters?.status) {
      where.status = filters.status;
    }

    if (filters?.search) {
      where.OR = [
        { email: { contains: filters.search, mode: 'insensitive' } },
        { name: { contains: filters.search, mode: 'insensitive' } },
      ];
    }

    const users = await this.prisma.user.findMany({
      where,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        status: true,
        createdAt: true,
        updatedAt: true,
        // ❌ NUNCA incluir password
      },
      orderBy: {
        createdAt: 'desc',
      },
    });

    this.logger.log(`Found ${users.length} users for tenant: ${tenantId}`);
    return users;
  }

  /**
   * Find one user by ID
   * @param tenantId - Tenant UUID
   * @param id - User UUID
   * @returns User entity
   * @throws NotFoundException if user not found
   */
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
        status: true,
        createdAt: true,
        updatedAt: true,
      },
    });

    if (!user) {
      this.logger.warn(`User ${id} not found for tenant: ${tenantId}`);
      throw new NotFoundException(`User with ID ${id} not found`);
    }

    return user;
  }

  /**
   * Create new user
   * @param tenantId - Tenant UUID
   * @param dto - User creation data
   * @returns Created user entity
   * @throws ConflictException if email already exists
   */
  async create(tenantId: string, dto: CreateUserDto): Promise<User> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Verificar email único dentro del tenant
    const existing = await this.prisma.user.findFirst({
      where: { email: dto.email },
    });

    if (existing) {
      this.logger.warn(`Email ${dto.email} already exists for tenant: ${tenantId}`);
      throw new ConflictException('Email already exists');
    }

    // ✅ SIEMPRE hashear password con bcrypt
    const hashedPassword = await bcrypt.hash(dto.password, 10);

    const user = await this.prisma.user.create({
      data: {
        tenant_id: tenantId, // ✅ EXPLÍCITO
        email: dto.email,
        name: dto.name,
        password: hashedPassword,
        role: dto.role,
        status: 'active',
      },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        status: true,
        createdAt: true,
        updatedAt: true,
      },
    });

    this.logger.log(`User created: ${user.id} for tenant: ${tenantId}`);
    return user;
  }

  /**
   * Update user
   * @param tenantId - Tenant UUID
   * @param id - User UUID
   * @param dto - User update data
   * @returns Updated user entity
   * @throws NotFoundException if user not found
   */
  async update(tenantId: string, id: string, dto: UpdateUserDto): Promise<User> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Verify user exists
    await this.findOne(tenantId, id);

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
        status: true,
        createdAt: true,
        updatedAt: true,
      },
    });

    this.logger.log(`User updated: ${id} for tenant: ${tenantId}`);
    return user;
  }

  /**
   * Delete user
   * @param tenantId - Tenant UUID
   * @param id - User UUID
   * @throws NotFoundException if user not found
   */
  async remove(tenantId: string, id: string): Promise<void> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Verify exists
    await this.findOne(tenantId, id);

    await this.prisma.user.delete({
      where: { id },
    });

    this.logger.log(`User deleted: ${id} for tenant: ${tenantId}`);
  }
}
```

### Reglas para Services

1. **SIEMPRE** incluir JSDoc comments para métodos públicos
2. **SIEMPRE** ejecutar `set_config` antes de queries
3. **SIEMPRE** usar Logger (NO `console.log`)
4. **SIEMPRE** lanzar excepciones específicas (`NotFoundException`, `ConflictException`)
5. **SIEMPRE** hashear passwords con bcrypt (10 rounds)
6. **SIEMPRE** incluir `tenant_id` explícitamente en inserts
7. **NUNCA** retornar passwords en responses
8. **SIEMPRE** validar unicidad de campos dentro del tenant

---

## 📋 DTOs

### Create DTO

```typescript
import { IsEmail, IsString, MinLength, MaxLength, IsEnum, IsOptional } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { UserRole } from '@/shared/enums/user-role.enum';

export class CreateUserDto {
  @ApiProperty({
    example: 'user@example.com',
    description: 'User email address',
  })
  @IsEmail({}, { message: 'Invalid email format' })
  email: string;

  @ApiProperty({
    example: 'John Doe',
    description: 'User full name',
    minLength: 3,
    maxLength: 100,
  })
  @IsString()
  @MinLength(3, { message: 'Name must be at least 3 characters' })
  @MaxLength(100, { message: 'Name must not exceed 100 characters' })
  name: string;

  @ApiProperty({
    example: 'SecurePass123!',
    description: 'User password (min 8 characters)',
    minLength: 8,
  })
  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  password: string;

  @ApiProperty({
    enum: UserRole,
    example: UserRole.USER,
    description: 'User role',
  })
  @IsEnum(UserRole, { message: 'Invalid role' })
  role: UserRole;

  @ApiPropertyOptional({
    example: '+1234567890',
    description: 'User phone number',
  })
  @IsOptional()
  @IsString()
  phone?: string;
}
```

### Update DTO

```typescript
import { PartialType } from '@nestjs/swagger';
import { CreateUserDto } from './create-user.dto';

// ✅ MEJOR PRÁCTICA: Usar PartialType para reutilizar validaciones
export class UpdateUserDto extends PartialType(CreateUserDto) {}

// Alternativamente, si necesitas personalizar:
import { IsEmail, IsString, MinLength, IsEnum, IsOptional } from 'class-validator';
import { ApiPropertyOptional } from '@nestjs/swagger';
import { UserRole } from '@/shared/enums/user-role.enum';

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

### Filters DTO

```typescript
import { IsOptional, IsEnum, IsString, IsInt, Min, Max } from 'class-validator';
import { ApiPropertyOptional } from '@nestjs/swagger';
import { Type } from 'class-transformer';
import { UserRole } from '@/shared/enums/user-role.enum';
import { UserStatus } from '../enums/user-status.enum';

export class UserFiltersDto {
  @ApiPropertyOptional({ enum: UserRole, description: 'Filter by role' })
  @IsOptional()
  @IsEnum(UserRole)
  role?: UserRole;

  @ApiPropertyOptional({ enum: UserStatus, description: 'Filter by status' })
  @IsOptional()
  @IsEnum(UserStatus)
  status?: UserStatus;

  @ApiPropertyOptional({ description: 'Search in name and email' })
  @IsOptional()
  @IsString()
  search?: string;

  @ApiPropertyOptional({ description: 'Page number', minimum: 1, default: 1 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  page?: number = 1;

  @ApiPropertyOptional({ description: 'Items per page', minimum: 1, maximum: 100, default: 20 })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  limit?: number = 20;

  @ApiPropertyOptional({ description: 'Sort field', default: 'createdAt' })
  @IsOptional()
  @IsString()
  sortBy?: string = 'createdAt';

  @ApiPropertyOptional({ enum: ['asc', 'desc'], default: 'desc' })
  @IsOptional()
  @IsEnum(['asc', 'desc'])
  sortOrder?: 'asc' | 'desc' = 'desc';
}
```

### Reglas para DTOs

1. **SIEMPRE** usar `class-validator` decorators
2. **SIEMPRE** documentar con `@ApiProperty` y `@ApiPropertyOptional`
3. **SIEMPRE** incluir mensajes de error personalizados
4. **SIEMPRE** usar `@Type()` para transformar query params
5. **SIEMPRE** usar `PartialType` para Update DTOs cuando sea posible
6. **NUNCA** usar `any` type
7. **SIEMPRE** incluir examples en Swagger

---

## 📦 Entities

```typescript
import { ApiProperty } from '@nestjs/swagger';
import { UserRole } from '@/shared/enums/user-role.enum';

export class User {
  @ApiProperty({ example: '123e4567-e89b-12d3-a456-426614174000' })
  id: string;

  @ApiProperty({ example: 'user@example.com' })
  email: string;

  @ApiProperty({ example: 'John Doe' })
  name: string;

  @ApiProperty({ enum: UserRole, example: UserRole.USER })
  role: UserRole;

  @ApiProperty({ example: 'active' })
  status: string;

  @ApiProperty({ example: '+1234567890', required: false })
  phone?: string;

  @ApiProperty({ example: '2025-01-01T00:00:00.000Z' })
  createdAt: Date;

  @ApiProperty({ example: '2025-01-01T00:00:00.000Z' })
  updatedAt: Date;

  // ❌ NUNCA incluir password en entity
}
```

---

## 🛡️ Guards

### Tenant Guard Example

```typescript
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  ForbiddenException,
  Logger,
} from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';

@Injectable()
export class TenantGuard implements CanActivate {
  private readonly logger = new Logger(TenantGuard.name);

  constructor(private readonly prisma: PrismaService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const user = request.user; // Set by JwtAuthGuard
    const tenantId = request.headers['x-tenant-id'];

    if (!tenantId) {
      this.logger.warn('Missing X-Tenant-ID header');
      throw new ForbiddenException('Missing X-Tenant-ID header');
    }

    // Verify user has access to this tenant
    const userTenant = await this.prisma.userTenant.findFirst({
      where: {
        user_id: user.userId,
        tenant_id: tenantId,
        is_active: true,
      },
    });

    if (!userTenant) {
      this.logger.warn(`User ${user.userId} has no access to tenant ${tenantId}`);
      throw new ForbiddenException('Access to this tenant is forbidden');
    }

    // Attach tenant to request
    request.tenantId = tenantId;

    return true;
  }
}
```

---

## ⚡ Interceptors

### Tenant Interceptor Example

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
  Logger,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { PrismaService } from '@/database/prisma.service';

@Injectable()
export class TenantInterceptor implements NestInterceptor {
  private readonly logger = new Logger(TenantInterceptor.name);

  constructor(private readonly prisma: PrismaService) {}

  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const request = context.switchToHttp().getRequest();
    const tenantId = request.headers['x-tenant-id'];

    if (tenantId) {
      // Fetch tenant details
      const tenant = await this.prisma.tenant.findUnique({
        where: { id: tenantId },
        select: {
          id: true,
          name: true,
          plan: true,
          is_active: true,
        },
      });

      if (tenant) {
        request.tenant = tenant;
        this.logger.log(`Tenant context set: ${tenant.name} (${tenant.id})`);
      }
    }

    return next.handle();
  }
}
```

---

## ❌ Error Handling

### Custom Exception Filter

```typescript
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
  Logger,
} from '@nestjs/common';
import { Response } from 'express';

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  private readonly logger = new Logger(HttpExceptionFilter.name);

  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest();
    const status = exception.getStatus();
    const exceptionResponse = exception.getResponse();

    const errorResponse = {
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      method: request.method,
      message: typeof exceptionResponse === 'string'
        ? exceptionResponse
        : (exceptionResponse as any).message || 'Internal server error',
    };

    this.logger.error(
      `${request.method} ${request.url} ${status} - ${errorResponse.message}`,
      exception.stack,
    );

    response.status(status).json(errorResponse);
  }
}
```

---

## 📊 Logging

### Service Logging Example

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class UsersService {
  private readonly logger = new Logger(UsersService.name);

  async findAll(tenantId: string): Promise<User[]> {
    // ✅ CORRECTO: Usar Logger
    this.logger.log(`Finding all users for tenant: ${tenantId}`);
    this.logger.debug(`Filters applied: ${JSON.stringify(filters)}`);

    try {
      const users = await this.prisma.user.findMany();
      this.logger.log(`Found ${users.length} users`);
      return users;
    } catch (error) {
      this.logger.error(`Error finding users: ${error.message}`, error.stack);
      throw error;
    }
  }

  // ❌ INCORRECTO: Usar console.log
  async create(dto: CreateUserDto): Promise<User> {
    console.log('Creating user');  // ❌ NUNCA
  }
}
```

---

## 🧪 Testing

### Reglas para Tests

1. **Unit tests:** Mockear dependencies (PrismaService, ConfigService, etc.)
2. **E2E tests:** Usar base de datos de test
3. **Coverage mínimo:** 80% general, 95% archivos críticos
4. **Naming:** `describe('ClassName')`, `it('should...')`

---

<div align="center">

**Sigue estos estándares SIEMPRE**

Código limpio, type-safe y escalable

</div>
