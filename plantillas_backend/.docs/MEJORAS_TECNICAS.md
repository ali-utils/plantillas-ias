# Mejoras Técnicas y Optimizaciones - bMOI Backend API

> **Recomendaciones técnicas para un backend de producción enterprise-grade**
> **Estas mejoras garantizan escalabilidad, seguridad y mantenibilidad**

---

## 📋 Índice

1. [Arquitectura y Patrones](#arquitectura-y-patrones)
2. [Seguridad Avanzada](#seguridad-avanzada)
3. [Performance y Escalabilidad](#performance-y-escalabilidad)
4. [Observabilidad y Monitoreo](#observabilidad-y-monitoreo)
5. [Resiliencia y Tolerancia a Fallos](#resiliencia-y-tolerancia-a-fallos)
6. [Testing Avanzado](#testing-avanzado)
7. [DevOps y CI/CD](#devops-y-cicd)
8. [Documentación y DX](#documentación-y-dx)

---

## 1. Arquitectura y Patrones

### 1.1 Repository Pattern

**Problema**: Prisma está acoplado directamente en los servicios, dificultando el testing y el cambio de ORM.

**Solución**: Implementar Repository Pattern

```typescript
// src/common/repositories/base.repository.ts
export abstract class BaseRepository<T> {
  abstract findAll(tenantId: string, filters?: any): Promise<T[]>;
  abstract findOne(tenantId: string, id: string): Promise<T | null>;
  abstract create(tenantId: string, data: any): Promise<T>;
  abstract update(tenantId: string, id: string, data: any): Promise<T>;
  abstract remove(tenantId: string, id: string): Promise<void>;
}

// src/modules/users/users.repository.ts
@Injectable()
export class UsersRepository extends BaseRepository<User> {
  constructor(private readonly prisma: PrismaService) {
    super();
  }

  async findAll(tenantId: string, filters?: UserFiltersDto): Promise<User[]> {
    await this.setTenantContext(tenantId);
    return this.prisma.user.findMany({
      where: this.buildWhereClause(filters),
    });
  }

  private async setTenantContext(tenantId: string): Promise<void> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;
  }

  private buildWhereClause(filters?: UserFiltersDto) {
    // Build complex where clauses
  }
}

// src/modules/users/users.service.ts
@Injectable()
export class UsersService {
  constructor(private readonly usersRepository: UsersRepository) {}

  async findAll(tenantId: string, filters: UserFiltersDto) {
    return this.usersRepository.findAll(tenantId, filters);
  }
}
```

**Beneficios**:
- ✅ Separación de responsabilidades
- ✅ Fácil testing con mocks
- ✅ Cambio de ORM sin afectar servicios
- ✅ Lógica de queries centralizada

---

### 1.2 CQRS Pattern (Command Query Responsibility Segregation)

**Problema**: Las operaciones de lectura y escritura compiten por recursos.

**Solución**: Separar comandos (write) de queries (read)

```typescript
// src/modules/users/commands/create-user.command.ts
export class CreateUserCommand {
  constructor(
    public readonly tenantId: string,
    public readonly dto: CreateUserDto,
  ) {}
}

// src/modules/users/handlers/create-user.handler.ts
@CommandHandler(CreateUserCommand)
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(private readonly usersRepository: UsersRepository) {}

  async execute(command: CreateUserCommand): Promise<User> {
    const { tenantId, dto } = command;
    return this.usersRepository.create(tenantId, dto);
  }
}

// src/modules/users/queries/get-users.query.ts
export class GetUsersQuery {
  constructor(
    public readonly tenantId: string,
    public readonly filters: UserFiltersDto,
  ) {}
}

// src/modules/users/handlers/get-users.handler.ts
@QueryHandler(GetUsersQuery)
export class GetUsersHandler implements IQueryHandler<GetUsersQuery> {
  constructor(private readonly usersRepository: UsersRepository) {}

  async execute(query: GetUsersQuery): Promise<User[]> {
    return this.usersRepository.findAll(query.tenantId, query.filters);
  }
}

// src/modules/users/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus,
  ) {}

  async create(tenantId: string, dto: CreateUserDto): Promise<User> {
    return this.commandBus.execute(new CreateUserCommand(tenantId, dto));
  }

  async findAll(tenantId: string, filters: UserFiltersDto): Promise<User[]> {
    return this.queryBus.execute(new GetUsersQuery(tenantId, filters));
  }
}
```

**Beneficios**:
- ✅ Escalabilidad independiente de lecturas/escrituras
- ✅ Optimización específica para cada operación
- ✅ Fácil implementar cache solo en queries
- ✅ Event sourcing ready

**Instalación**:
```bash
npm install @nestjs/cqrs
```

---

### 1.3 Response DTOs

**Problema**: Retornamos entidades de Prisma directamente, exponiendo campos internos.

**Solución**: DTOs de respuesta con class-transformer

```typescript
// src/modules/users/dto/user-response.dto.ts
import { Exclude, Expose, Transform } from 'class-transformer';
import { ApiProperty } from '@nestjs/swagger';

@Exclude()
export class UserResponseDto {
  @Expose()
  @ApiProperty()
  id: string;

  @Expose()
  @ApiProperty()
  email: string;

  @Expose()
  @ApiProperty()
  name: string;

  @Expose()
  @ApiProperty({ enum: ['ADMIN', 'MANAGER', 'USER'] })
  role: string;

  @Expose()
  @ApiProperty()
  status: string;

  @Expose()
  @ApiProperty()
  @Transform(({ value }) => value?.toISOString())
  createdAt: Date;

  @Expose()
  @ApiProperty()
  @Transform(({ value }) => value?.toISOString())
  updatedAt: Date;

  // ❌ password está excluido por @Exclude() en la clase
  // ❌ tenantId está excluido
}

// src/modules/users/users.service.ts
async findAll(tenantId: string, filters: UserFiltersDto): Promise<UserResponseDto[]> {
  const users = await this.usersRepository.findAll(tenantId, filters);
  return plainToClass(UserResponseDto, users, { excludeExtraneousValues: true });
}
```

**Beneficios**:
- ✅ Control total sobre qué se expone
- ✅ Transformaciones automáticas (dates, enums)
- ✅ Swagger documentation automática
- ✅ Seguridad (no exponer campos sensibles)

---

### 1.4 Result Pattern (Error Handling)

**Problema**: Excepciones hacen el flujo difícil de seguir y testear.

**Solución**: Result pattern para manejar errores explícitamente

```typescript
// src/common/types/result.type.ts
export class Result<T> {
  private constructor(
    public readonly isSuccess: boolean,
    public readonly value?: T,
    public readonly error?: string,
    public readonly errorCode?: string,
  ) {}

  static ok<U>(value: U): Result<U> {
    return new Result<U>(true, value);
  }

  static fail<U>(error: string, errorCode?: string): Result<U> {
    return new Result<U>(false, undefined, error, errorCode);
  }
}

// src/modules/users/users.service.ts
async create(tenantId: string, dto: CreateUserDto): Promise<Result<User>> {
  // Verificar email único
  const existing = await this.usersRepository.findByEmail(tenantId, dto.email);
  if (existing) {
    return Result.fail('Email already exists', 'USER_EMAIL_EXISTS');
  }

  // Verificar plan limits
  const limitCheck = await this.checkPlanLimits(tenantId);
  if (!limitCheck) {
    return Result.fail('Plan limit reached', 'PLAN_LIMIT_REACHED');
  }

  const user = await this.usersRepository.create(tenantId, dto);
  return Result.ok(user);
}

// src/modules/users/users.controller.ts
@Post()
async create(@Tenant() tenant: ITenantContext, @Body() dto: CreateUserDto) {
  const result = await this.usersService.create(tenant.id, dto);

  if (!result.isSuccess) {
    throw new BadRequestException({
      message: result.error,
      code: result.errorCode,
    });
  }

  return result.value;
}
```

**Beneficios**:
- ✅ Errores explícitos
- ✅ Fácil testing
- ✅ Error codes consistentes
- ✅ No pollution con try-catch

---

### 1.5 Specification Pattern (Queries Complejas)

**Problema**: Queries complejas dificultan la legibilidad y reutilización.

**Solución**: Specification pattern

```typescript
// src/common/specifications/specification.interface.ts
export interface ISpecification<T> {
  toQuery(): any; // Prisma where clause
}

// src/modules/users/specifications/active-users.specification.ts
export class ActiveUsersSpecification implements ISpecification<User> {
  toQuery() {
    return {
      status: 'ACTIVE',
    };
  }
}

export class UsersByRoleSpecification implements ISpecification<User> {
  constructor(private readonly role: UserRole) {}

  toQuery() {
    return {
      role: this.role,
    };
  }
}

export class UserSearchSpecification implements ISpecification<User> {
  constructor(private readonly search: string) {}

  toQuery() {
    return {
      OR: [
        { email: { contains: this.search, mode: 'insensitive' } },
        { name: { contains: this.search, mode: 'insensitive' } },
      ],
    };
  }
}

// Combinar especificaciones
export class CompositeSpecification<T> implements ISpecification<T> {
  constructor(private readonly specs: ISpecification<T>[]) {}

  toQuery() {
    return {
      AND: this.specs.map((spec) => spec.toQuery()),
    };
  }
}

// Uso
const specs = [
  new ActiveUsersSpecification(),
  new UsersByRoleSpecification('ADMIN'),
  new UserSearchSpecification('john'),
];

const compositeSpec = new CompositeSpecification(specs);
const users = await prisma.user.findMany({
  where: compositeSpec.toQuery(),
});
```

---

## 2. Seguridad Avanzada

### 2.1 Helmet - HTTP Headers Security

**Instalación**:
```bash
npm install helmet
```

**Configuración**:
```typescript
// src/main.ts
import helmet from 'helmet';

app.use(
  helmet({
    contentSecurityPolicy: {
      directives: {
        defaultSrc: ["'self'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        scriptSrc: ["'self'"],
        imgSrc: ["'self'", 'data:', 'https:'],
      },
    },
    hsts: {
      maxAge: 31536000,
      includeSubDomains: true,
      preload: true,
    },
  }),
);
```

---

### 2.2 Rate Limiting Granular

**Configuración avanzada por endpoint**:

```typescript
// src/common/decorators/rate-limit.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const RATE_LIMIT_KEY = 'rateLimit';

export interface RateLimitConfig {
  ttl: number; // seconds
  limit: number; // requests
}

export const RateLimit = (config: RateLimitConfig) =>
  SetMetadata(RATE_LIMIT_KEY, config);

// Uso en controllers
@Post('login')
@RateLimit({ ttl: 60, limit: 5 }) // 5 intentos por minuto
async login() {}

@Get('users')
@RateLimit({ ttl: 60, limit: 100 }) // 100 requests por minuto
async getUsers() {}
```

---

### 2.3 Request ID Tracking

**Para trazabilidad de requests**:

```typescript
// src/common/interceptors/request-id.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class RequestIdInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    const request = context.switchToHttp().getRequest();
    const requestId = request.headers['x-request-id'] || uuidv4();

    request.id = requestId;
    request.headers['x-request-id'] = requestId;

    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        response.setHeader('X-Request-ID', requestId);
      }),
    );
  }
}

// Aplicar globalmente en main.ts
app.useGlobalInterceptors(new RequestIdInterceptor());
```

---

### 2.4 Secrets Management con Vault (Producción)

**Para producción, no usar .env**:

```typescript
// src/config/vault.config.ts
import * as Vault from 'node-vault';

export class VaultService {
  private vault: Vault.client;

  constructor() {
    this.vault = Vault({
      endpoint: process.env.VAULT_ADDR,
      token: process.env.VAULT_TOKEN,
    });
  }

  async getSecret(path: string): Promise<any> {
    const result = await this.vault.read(path);
    return result.data;
  }
}

// Uso
const secrets = await vaultService.getSecret('secret/data/bmoi/jwt');
const JWT_SECRET = secrets.secret;
```

---

## 3. Performance y Escalabilidad

### 3.1 Database Connection Pooling

**Configuración óptima de Prisma**:

```typescript
// src/database/prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
  constructor() {
    super({
      datasources: {
        db: {
          url: process.env.DATABASE_URL,
        },
      },
      log: [
        { emit: 'event', level: 'query' },
        { emit: 'event', level: 'error' },
        { emit: 'event', level: 'warn' },
      ],
      // Connection pool optimizado
      connectionTimeout: 10000,
      poolTimeout: 10000,
      connectionLimit: 20, // Ajustar según carga
    });
  }

  async onModuleInit() {
    await this.$connect();

    // Query logging
    this.$on('query', (e) => {
      console.log('Query: ' + e.query);
      console.log('Duration: ' + e.duration + 'ms');
    });
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }

  // Helper para transacciones
  async transaction<T>(fn: (prisma: PrismaService) => Promise<T>): Promise<T> {
    return this.$transaction(async (prisma) => {
      return fn(prisma as PrismaService);
    });
  }
}
```

---

### 3.2 Redis Caching Strategy

**Implementar cache en múltiples niveles**:

```typescript
// src/common/decorators/cache.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const CACHE_KEY_METADATA = 'cacheKey';
export const CACHE_TTL_METADATA = 'cacheTTL';

export function Cacheable(key: string, ttl: number = 300) {
  return (target: any, propertyKey: string, descriptor: PropertyDescriptor) => {
    SetMetadata(CACHE_KEY_METADATA, key)(target, propertyKey, descriptor);
    SetMetadata(CACHE_TTL_METADATA, ttl)(target, propertyKey, descriptor);
  };
}

// src/common/interceptors/cache.interceptor.ts
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(
    private readonly reflector: Reflector,
    private readonly redisService: RedisService,
  ) {}

  async intercept(context: ExecutionContext, next: CallHandler) {
    const cacheKey = this.reflector.get(CACHE_KEY_METADATA, context.getHandler());
    const ttl = this.reflector.get(CACHE_TTL_METADATA, context.getHandler());

    if (!cacheKey) {
      return next.handle();
    }

    const request = context.switchToHttp().getRequest();
    const tenantId = request.tenant?.id;
    const fullKey = `${tenantId}:${cacheKey}:${JSON.stringify(request.query)}`;

    // Intentar obtener del cache
    const cached = await this.redisService.get(fullKey);
    if (cached) {
      return of(JSON.parse(cached));
    }

    // Si no está en cache, ejecutar y guardar
    return next.handle().pipe(
      tap(async (data) => {
        await this.redisService.set(fullKey, JSON.stringify(data), ttl);
      }),
    );
  }
}

// Uso en servicios
@Cacheable('users:list', 300) // Cache 5 minutos
async findAll(tenantId: string, filters: UserFiltersDto) {
  return this.usersRepository.findAll(tenantId, filters);
}
```

---

### 3.3 Query Optimization con Indexes

**Agregar en migrations**:

```sql
-- Indexes compuestos para queries frecuentes
CREATE INDEX CONCURRENTLY idx_users_tenant_status_created
  ON users(tenant_id, status, created_at DESC);

CREATE INDEX CONCURRENTLY idx_users_tenant_role_status
  ON users(tenant_id, role, status)
  WHERE status = 'ACTIVE';

-- Partial indexes para queries específicas
CREATE INDEX CONCURRENTLY idx_active_users
  ON users(tenant_id, email)
  WHERE status = 'ACTIVE';

-- Full-text search
CREATE INDEX CONCURRENTLY idx_users_search
  ON users USING gin(to_tsvector('english', name || ' ' || email));

-- Explicar queries antes de deployar
EXPLAIN ANALYZE
SELECT * FROM users
WHERE tenant_id = 'xxx'
  AND status = 'ACTIVE'
ORDER BY created_at DESC
LIMIT 20;
```

---

### 3.4 Pagination Helper

```typescript
// src/common/dto/pagination.dto.ts
import { Type } from 'class-transformer';
import { IsInt, Min, Max, IsOptional, IsEnum } from 'class-validator';
import { ApiPropertyOptional } from '@nestjs/swagger';

export class PaginationDto {
  @ApiPropertyOptional({ minimum: 1, default: 1 })
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @IsOptional()
  page?: number = 1;

  @ApiPropertyOptional({ minimum: 1, maximum: 100, default: 20 })
  @Type(() => Number)
  @IsInt()
  @Min(1)
  @Max(100)
  @IsOptional()
  limit?: number = 20;

  @ApiPropertyOptional()
  @IsOptional()
  sortBy?: string = 'createdAt';

  @ApiPropertyOptional({ enum: ['asc', 'desc'], default: 'desc' })
  @IsOptional()
  @IsEnum(['asc', 'desc'])
  sortOrder?: 'asc' | 'desc' = 'desc';
}

// Response wrapper
export class PaginatedResponseDto<T> {
  data: T[];
  meta: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}

// Helper function
export async function paginate<T>(
  query: any,
  pagination: PaginationDto,
): Promise<PaginatedResponseDto<T>> {
  const { page, limit, sortBy, sortOrder } = pagination;
  const skip = (page - 1) * limit;

  const [data, total] = await Promise.all([
    query.findMany({
      skip,
      take: limit,
      orderBy: { [sortBy]: sortOrder },
    }),
    query.count(),
  ]);

  return {
    data,
    meta: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
      hasNext: page < Math.ceil(total / limit),
      hasPrev: page > 1,
    },
  };
}
```

---

## 4. Observabilidad y Monitoreo

### 4.1 Structured Logging con Winston

```typescript
// src/config/logger.config.ts
import { WinstonModule } from 'nest-winston';
import * as winston from 'winston';

export const loggerConfig = WinstonModule.createLogger({
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.ms(),
        winston.format.errors({ stack: true }),
        winston.format.json(),
      ),
    }),
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
    new winston.transports.File({
      filename: 'logs/combined.log',
      format: winston.format.combine(
        winston.format.timestamp(),
        winston.format.json(),
      ),
    }),
  ],
});

// Uso con contexto
this.logger.log({
  message: 'User created',
  userId: user.id,
  tenantId: tenant.id,
  requestId: request.id,
  duration: Date.now() - start,
});
```

---

### 4.2 Prometheus Metrics

```bash
npm install @willsoto/nestjs-prometheus prom-client
```

```typescript
// src/app.module.ts
import { PrometheusModule } from '@willsoto/nestjs-prometheus';

@Module({
  imports: [
    PrometheusModule.register({
      defaultMetrics: {
        enabled: true,
      },
      path: '/metrics',
    }),
  ],
})
export class AppModule {}

// Custom metrics
import { Counter, Histogram } from 'prom-client';

const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status'],
});

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration',
  labelNames: ['method', 'route'],
  buckets: [0.1, 0.5, 1, 2, 5],
});
```

---

### 4.3 Health Checks Completos

```typescript
// src/modules/health/health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  PrismaHealthIndicator,
  MemoryHealthIndicator,
  DiskHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private prisma: PrismaHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      // Database
      () => this.prisma.pingCheck('database'),

      // Memory (heap should not exceed 150MB)
      () => this.memory.checkHeap('memory_heap', 150 * 1024 * 1024),

      // Disk (should have at least 250GB available)
      () =>
        this.disk.checkStorage('storage', {
          path: '/',
          thresholdPercent: 0.9,
        }),
    ]);
  }

  @Get('liveness')
  liveness() {
    return { status: 'ok' };
  }

  @Get('readiness')
  @HealthCheck()
  readiness() {
    return this.health.check([
      () => this.prisma.pingCheck('database'),
    ]);
  }
}
```

---

## 5. Resiliencia y Tolerancia a Fallos

### 5.1 Retry Logic con Exponential Backoff

```bash
npm install @nestjs/axios axios-retry
```

```typescript
// src/common/utils/retry.util.ts
import axiosRetry from 'axios-retry';

export function configureRetry(axios: any) {
  axiosRetry(axios, {
    retries: 3,
    retryDelay: axiosRetry.exponentialDelay,
    retryCondition: (error) => {
      return (
        axiosRetry.isNetworkOrIdempotentRequestError(error) ||
        error.response?.status === 429 ||
        error.response?.status >= 500
      );
    },
  });
}

// Retry decorator
export function Retry(attempts: number = 3, delay: number = 1000) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      let lastError: any;

      for (let i = 0; i < attempts; i++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error;
          if (i < attempts - 1) {
            await new Promise((resolve) =>
              setTimeout(resolve, delay * Math.pow(2, i)),
            );
          }
        }
      }

      throw lastError;
    };

    return descriptor;
  };
}

// Uso
@Retry(3, 1000)
async sendEmail(to: string, subject: string, body: string) {
  // Código que puede fallar
}
```

---

### 5.2 Circuit Breaker Pattern

```bash
npm install opossum
```

```typescript
// src/common/utils/circuit-breaker.util.ts
import CircuitBreaker from 'opossum';

export function createCircuitBreaker(fn: Function, options?: any) {
  const breaker = new CircuitBreaker(fn, {
    timeout: 3000, // 3 segundos
    errorThresholdPercentage: 50,
    resetTimeout: 30000, // 30 segundos
    ...options,
  });

  breaker.on('open', () => {
    console.warn('Circuit breaker opened');
  });

  breaker.on('halfOpen', () => {
    console.info('Circuit breaker half-open');
  });

  breaker.on('close', () => {
    console.info('Circuit breaker closed');
  });

  return breaker;
}

// Uso
const getUsersBreaker = createCircuitBreaker(
  async (tenantId: string) => {
    return this.usersRepository.findAll(tenantId);
  },
);

const users = await getUsersBreaker.fire(tenantId);
```

---

### 5.3 Graceful Shutdown

```typescript
// src/main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Enable graceful shutdown
  app.enableShutdownHooks();

  // Handle SIGTERM
  process.on('SIGTERM', async () => {
    console.log('SIGTERM received, closing application gracefully');
    await app.close();
  });

  // Handle SIGINT
  process.on('SIGINT', async () => {
    console.log('SIGINT received, closing application gracefully');
    await app.close();
  });

  await app.listen(3000);
}
```

---

## 6. Testing Avanzado

### 6.1 Test Containers para E2E

```bash
npm install -D testcontainers
```

```typescript
// test/setup-e2e.ts
import { PostgreSqlContainer } from 'testcontainers';

let postgresContainer: PostgreSqlContainer;

export async function setupE2E() {
  postgresContainer = await new PostgreSqlContainer()
    .withExposedPorts(5432)
    .start();

  process.env.DATABASE_URL = postgresContainer.getConnectionUri();
}

export async function teardownE2E() {
  await postgresContainer.stop();
}
```

---

### 6.2 Factory Pattern para Tests

```typescript
// test/factories/user.factory.ts
import { User } from '@prisma/client';
import * as faker from 'faker';

export class UserFactory {
  static create(overrides?: Partial<User>): User {
    return {
      id: faker.datatype.uuid(),
      tenantId: faker.datatype.uuid(),
      email: faker.internet.email(),
      name: faker.name.findName(),
      password: faker.internet.password(),
      role: 'USER',
      status: 'ACTIVE',
      createdAt: new Date(),
      updatedAt: new Date(),
      ...overrides,
    };
  }

  static createMany(count: number, overrides?: Partial<User>): User[] {
    return Array.from({ length: count }, () => this.create(overrides));
  }
}

// Uso en tests
const user = UserFactory.create({ role: 'ADMIN' });
const users = UserFactory.createMany(10, { tenantId: 'tenant-1' });
```

---

## 7. DevOps y CI/CD

### 7.1 GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm run test:cov
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test

      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Build Docker image
        run: docker build -t bmoi-backend:${{ github.sha }} .

      - name: Push to registry
        run: |
          echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
          docker push bmoi-backend:${{ github.sha }}
```

---

### 7.2 Docker Multi-Stage Optimizado

Ver `SETUP_INICIAL.md` - Ya incluido

---

## 8. Documentación y DX

### 8.1 OpenAPI/Swagger Avanzado

```typescript
// src/main.ts
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('bMOI Backend API')
  .setDescription('Multi-tenant backend API for bMOI')
  .setVersion('1.0')
  .addBearerAuth()
  .addServer('http://localhost:3000', 'Development')
  .addServer('https://api-staging.bmoi.com', 'Staging')
  .addServer('https://api.bmoi.com', 'Production')
  .addTag('auth', 'Authentication endpoints')
  .addTag('users', 'User management')
  .addTag('tenants', 'Tenant management')
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document, {
  customSiteTitle: 'bMOI API Docs',
  customCss: '.swagger-ui .topbar { display: none }',
  swaggerOptions: {
    persistAuthorization: true,
    tagsSorter: 'alpha',
    operationsSorter: 'alpha',
  },
});
```

---

## 📊 Resumen de Prioridades

### Fase 1 (Inmediato)
- ✅ Repository Pattern
- ✅ Response DTOs
- ✅ Request ID Tracking
- ✅ Structured Logging
- ✅ Health Checks
- ✅ Helmet

### Fase 2 (Corto Plazo)
- ✅ Caching Strategy
- ✅ Rate Limiting Granular
- ✅ Database Indexes
- ✅ Pagination Helper
- ✅ Retry Logic

### Fase 3 (Mediano Plazo)
- ✅ CQRS
- ✅ Circuit Breaker
- ✅ Prometheus Metrics
- ✅ Test Containers
- ✅ CI/CD Pipeline

### Fase 4 (Largo Plazo)
- ✅ Specification Pattern
- ✅ Result Pattern
- ✅ Secrets Management (Vault)
- ✅ APM Integration

---

<div align="center">

**Estas mejoras garantizan un backend enterprise-grade**

Escalable | Seguro | Mantenible | Observable

</div>
