# ARQUITECTURA - Backend bMOI Multi-Tenant

> **Decisiones Arquitectónicas y Justificaciones Técnicas**

---

## 📋 Tabla de Contenidos

- [Visión General](#visión-general)
- [Decisiones Arquitectónicas](#decisiones-arquitectónicas)
- [Estrategia Multi-Tenant](#estrategia-multi-tenant)
- [Stack Tecnológico](#stack-tecnológico)
- [Capas de Seguridad](#capas-de-seguridad)
- [Escalabilidad](#escalabilidad)
- [Performance](#performance)
- [Monitoreo](#monitoreo)

---

## 🎯 Visión General

### Objetivo

Construir una **API REST multi-tenant** que:

1. ✅ Garantice **aislamiento total de datos** entre tenants
2. ✅ Sea **escalable** de 10 a 10,000+ tenants
3. ✅ Sea **type-safe** con TypeScript end-to-end
4. ✅ Sea **segura** con 7 capas de seguridad
5. ✅ Sea **mantenible** con arquitectura modular
6. ✅ Sea **observable** con logging y monitoring

### Principios de Diseño

- **Security First:** Aislamiento multi-tenant a nivel de base de datos
- **Type Safety:** TypeScript en todo el stack
- **Modular:** Fácil evolución a microservicios
- **Docker-First:** Mismo container en dev, staging, prod
- **API-First:** Swagger auto-generado, contratos claros
- **Testing:** Coverage >80%, crítico >95%

---

## 🏗️ Decisiones Arquitectónicas

### 1. Framework Backend: NestJS 10

**Decisión:** Usar **NestJS** sobre Express, Fastify o Go

#### Justificación

| Criterio | NestJS | Express | Fastify | Go |
|----------|--------|---------|---------|-----|
| TypeScript nativo | ✅ Sí | ⚠️ Requiere setup | ⚠️ Requiere setup | ❌ No (lenguaje diferente) |
| Dependency Injection | ✅ Built-in | ❌ Manual | ❌ Manual | ⚠️ Frameworks externos |
| Decorators | ✅ Excelente para multi-tenant | ❌ No | ❌ No | ❌ No |
| Swagger auto-generado | ✅ Sí | ❌ Manual | ❌ Manual | ⚠️ Frameworks externos |
| Modularidad | ✅ Nativo | ⚠️ Manual | ⚠️ Manual | ✅ Sí |
| Learning curve | ⚠️ Media | ✅ Baja | ✅ Baja | ⚠️ Media-Alta |
| Performance | ✅ Buena | ✅ Buena | ✅ Excelente | ✅ Excelente |

#### Trade-offs Aceptados

- ✅ **Pros:**
  - TypeScript end-to-end con frontend
  - Decorators perfectos para multi-tenant (@UseGuards, @Tenant)
  - Dependency Injection automático
  - Swagger auto-generado
  - Arquitectura modular de fábrica

- ⚠️ **Contras:**
  - Más opinionado que Express
  - Bundle size ligeramente mayor
  - Performance ligeramente menor que Fastify o Go

**Conclusión:** Los beneficios de productividad y type-safety superan los pequeños trade-offs de performance.

---

### 2. Base de Datos: PostgreSQL 15 con Row-Level Security

**Decisión:** PostgreSQL 15 con **Row-Level Security (RLS)** sobre MongoDB, MySQL o Database-per-Tenant

#### Comparación de Estrategias Multi-Tenant

| Estrategia | Aislamiento | Costo | Complejidad | Escalabilidad |
|------------|-------------|-------|-------------|---------------|
| **RLS (Nuestra elección)** | ✅ Garantizado DB | ✅ Bajo | ✅ Baja | ✅ 1-10K tenants |
| Schema per tenant | ⚠️ Bueno | ⚠️ Medio | ⚠️ Media | ⚠️ 1-1K tenants |
| Database per tenant | ✅ Excelente | ❌ Muy alto | ❌ Alta | ❌ 1-100 tenants |
| Discriminator column | ❌ Solo app-level | ✅ Bajo | ✅ Baja | ✅ Muy alta |

#### Row-Level Security (RLS) - Cómo Funciona

**1. Todas las tablas tienen `tenant_id`:**

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id UUID NOT NULL REFERENCES tenants(id) ON DELETE CASCADE,
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Índice compuesto para queries rápidas
CREATE INDEX idx_users_tenant_id ON users(tenant_id, created_at DESC);
```

**2. Habilitar RLS y crear políticas:**

```sql
-- Habilitar RLS
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Política para SELECT, UPDATE, DELETE
CREATE POLICY tenant_isolation ON users
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Política para INSERT
CREATE POLICY tenant_isolation_insert ON users
  FOR INSERT
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id')::UUID);
```

**3. Backend configura el contexto:**

```typescript
// En cada service method
async findAll(tenantId: string) {
  // Set PostgreSQL session variable
  await this.prisma.$executeRaw`
    SELECT set_config('app.current_tenant_id', ${tenantId}, true)
  `;

  // Query automáticamente filtrado por RLS
  return this.prisma.user.findMany();
}
```

#### Ventajas de RLS

1. **Aislamiento garantizado a nivel de base de datos**
   - Imposible accidentalmente mezclar datos
   - Funciona incluso si hay bugs en aplicación

2. **Una sola base de datos**
   - Costo-efectivo hasta 10K tenants
   - Backups simples
   - Migraciones simples

3. **Performance excelente**
   - Índices compuestos (tenant_id + otros campos)
   - PostgreSQL optimiza queries automáticamente

4. **Auditoría simplificada**
   - Una sola tabla de audit para todos los tenants
   - Fácil análisis cross-tenant para admins

#### Alternativas Descartadas

**❌ MongoDB:**
- No soporta RLS nativo
- Aislamiento solo a nivel de aplicación
- Mayor riesgo de data leakage

**❌ MySQL:**
- RLS disponible desde 8.0 pero menos maduro
- PostgreSQL tiene mejor ecosistema para RLS

**❌ Database per Tenant:**
- Muy costoso (10K databases = pesadilla operacional)
- Migraciones complejas
- Backups complejos

**❌ Schema per Tenant:**
- Demasiado complejo para 1-10K tenants
- Migraciones requieren scripts para cada schema

---

### 3. ORM: Prisma

**Decisión:** Usar **Prisma** sobre TypeORM o Sequelize

#### Comparación

| Criterio | Prisma | TypeORM | Sequelize |
|----------|--------|---------|-----------|
| Type-safety | ✅ Compile-time | ⚠️ Runtime | ❌ Débil |
| Auto-completion | ✅ Excelente | ⚠️ Bueno | ❌ Pobre |
| Migrations | ✅ Simples | ⚠️ Buenas | ⚠️ Complejas |
| Developer Experience | ✅ Excelente | ⚠️ Buena | ❌ Pobre |
| Raw SQL support | ✅ Sí ($executeRaw) | ✅ Sí | ✅ Sí |
| Madurez | ⚠️ Relativamente nuevo | ✅ Muy maduro | ✅ Maduro |

#### Justificación

- ✅ **Type-safe a nivel de compilación:** Errores detectados antes de runtime
- ✅ **Auto-completion excelente:** Productividad ++
- ✅ **Migrations simples:** `prisma migrate dev`
- ✅ **Schema declarativo:** Fácil de leer y mantener
- ✅ **Support para raw SQL:** Necesario para RLS (`$executeRaw`)

#### Trade-offs Aceptados

- ⚠️ Menos maduro que TypeORM
- ⚠️ Menos flexible para queries muy complejas
- ⚠️ No soporta todos los features de PostgreSQL (pero los esenciales sí)

**Conclusión:** Los beneficios de type-safety y DX superan las limitaciones.

---

### 4. Cache: Redis 7

**Decisión:** Redis para cache, sessions y queue

#### Justificación

- ✅ **In-memory:** Extremadamente rápido (<1ms latency)
- ✅ **Versatil:** Cache + Sessions + Pub/Sub + Queue
- ✅ **Escalable:** Fácil clustering
- ✅ **Ecosistema maduro:** BullMQ, ioredis

#### Uso en bMOI

1. **Cache de queries:**
   ```typescript
   // Cache users list por tenant
   const cacheKey = `users:${tenantId}:list`;
   const cached = await this.redis.get(cacheKey);
   if (cached) return JSON.parse(cached);

   const users = await this.prisma.user.findMany();
   await this.redis.set(cacheKey, JSON.stringify(users), 'EX', 300); // 5min TTL
   ```

2. **Sessions:**
   ```typescript
   // Store session data
   await this.redis.set(`session:${userId}`, sessionData, 'EX', 86400); // 24h
   ```

3. **Queue (BullMQ):**
   ```typescript
   // Agregar job a queue
   await this.emailQueue.add('send-welcome', {
     email: user.email,
     name: user.name,
   });
   ```

---

### 5. Autenticación: JWT con RS256

**Decisión:** JWT con algoritmo **RS256** (asymmetric) sobre HS256

#### RS256 vs HS256

| Criterio | RS256 (Nuestra elección) | HS256 |
|----------|--------------------------|-------|
| Seguridad | ✅ Mayor (clave pública/privada) | ⚠️ Menor (clave compartida) |
| Performance | ⚠️ Más lento | ✅ Más rápido |
| Key distribution | ✅ Fácil (public key) | ❌ Riesgoso (secret compartido) |
| Microservices ready | ✅ Sí | ❌ No |

#### Justificación

1. **Mayor seguridad:**
   - Private key solo en backend
   - Public key puede distribuirse a otros servicios

2. **Escalabilidad:**
   - Fácil agregar servicios que validen tokens
   - No necesitan secret compartido

3. **Best practice para producción:**
   - Recomendado por OWASP
   - Standard en empresas enterprise

#### Implementación

```typescript
// Generate token (backend)
const payload = {
  userId: user.id,
  tenantId: tenant.id,
  role: user.role,
};

const accessToken = this.jwtService.sign(payload, {
  algorithm: 'RS256',
  expiresIn: '15m',
  privateKey: this.configService.get('JWT_PRIVATE_KEY'),
});

// Verify token (backend o otros servicios)
const decoded = this.jwtService.verify(token, {
  publicKey: this.configService.get('JWT_PUBLIC_KEY'),
});
```

#### Trade-offs Aceptados

- ⚠️ Ligeramente más lento que HS256
- ⚠️ Requiere generar y gestionar key pair

**Conclusión:** Seguridad > Performance en autenticación.

---

### 6. File Storage: MinIO (S3-compatible)

**Decisión:** MinIO en dev/staging, AWS S3 en producción

#### Justificación

- ✅ **S3-compatible API:** Mismo código para dev y prod
- ✅ **Self-hosted en dev:** Gratis, sin AWS account
- ✅ **Sin vendor lock-in:** Fácil migrar a S3, Azure Blob, etc.
- ✅ **Open source:** Control total

#### Uso

```typescript
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

// Same code works with MinIO and S3
const s3Client = new S3Client({
  endpoint: process.env.MINIO_ENDPOINT, // MinIO en dev, undefined en prod (usa S3)
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY,
    secretAccessKey: process.env.AWS_SECRET_KEY,
  },
});

// Upload file
await s3Client.send(new PutObjectCommand({
  Bucket: `tenant-${tenantId}-files`,
  Key: file.name,
  Body: file.buffer,
}));
```

---

## 🔐 Capas de Seguridad

### 7 Capas de Defensa en Profundidad

#### 1. Network Layer

```typescript
// HTTPS enforcement
app.use(helmet());

// CORS configurado
app.enableCors({
  origin: process.env.ALLOWED_ORIGINS.split(','),
  credentials: true,
});

// Rate limiting
@UseGuards(ThrottlerGuard)
@Throttle(10, 60) // 10 requests por minuto
```

#### 2. Authentication Layer

```typescript
// JWT validation
@UseGuards(JwtAuthGuard)
export class UsersController {
  // Solo usuarios autenticados
}
```

#### 3. Authorization Layer

```typescript
// RBAC
@RequirePermission('users.read')
async findAll() {
  // Solo usuarios con permiso 'users.read'
}
```

#### 4. Tenant Isolation Layer

```typescript
// Tenant validation + RLS
@UseGuards(TenantGuard)
@UseInterceptors(TenantInterceptor)
export class UsersController {
  async findAll(@Tenant() tenant: ITenantContext) {
    // RLS garantiza aislamiento
  }
}
```

#### 5. Input Validation Layer

```typescript
// DTO validation
export class CreateUserDto {
  @IsEmail()
  email: string;

  @MinLength(8)
  password: string;
}
```

#### 6. Plan Limits Layer

```typescript
// Enforce plan limits
@CheckPlanLimit('users')
async create(@Tenant() tenant, @Body() dto) {
  // Verificado que tenant no excede límite
}
```

#### 7. Audit Layer

```typescript
// Audit logging
@UseInterceptors(AuditInterceptor)
async delete(@Param('id') id: string) {
  // Log: user X deleted resource Y at timestamp Z
}
```

---

## 📈 Escalabilidad

### Arquitectura: Monolito Modular → Microservicios

**Fase 1: Monolito Modular (actual)**

```
┌─────────────────────────────┐
│     NestJS Backend API      │
│                             │
│  ┌──────┐  ┌──────┐        │
│  │ Auth │  │Users │  ...   │
│  └──────┘  └──────┘        │
│                             │
│  Shared: Guards, Prisma     │
└─────────────────────────────┘
       │
       ▼
┌─────────────────┐
│   PostgreSQL    │
│   + Redis       │
└─────────────────┘
```

**Fase 2: Microservicios (futura)**

```
┌──────────┐   ┌──────────┐   ┌──────────┐
│  Auth    │   │  Users   │   │ Products │
│ Service  │   │ Service  │   │ Service  │
└──────────┘   └──────────┘   └──────────┘
      │              │              │
      └──────────────┴──────────────┘
                     │
              ┌──────────────┐
              │  API Gateway │
              └──────────────┘
```

### Estrategia de Migración

1. **Actualmente:** Monolito modular con módulos bien separados
2. **Cuando sea necesario:** Extraer módulos a microservicios
3. **Sin refactoring masivo:** Módulos ya están preparados

### Límites de Escalabilidad

| Componente | Límite Actual | Siguiente Paso |
|------------|---------------|----------------|
| Tenants | 10,000 | Sharding de DB |
| Users per tenant | 10,000 | Partition tables |
| Requests/sec | 1,000 | Load balancer + replicas |
| Database size | 100GB | Read replicas |

---

## ⚡ Performance

### Optimizaciones Implementadas

#### 1. Database Indexes

```sql
-- Índices compuestos para queries multi-tenant
CREATE INDEX idx_users_tenant_created ON users(tenant_id, created_at DESC);
CREATE INDEX idx_products_tenant_category ON products(tenant_id, category_id);

-- Índices parciales para queries frecuentes
CREATE INDEX idx_active_users ON users(tenant_id) WHERE status = 'active';
```

#### 2. Query Optimization

```typescript
// ✅ CORRECTO: Select solo campos necesarios
const users = await this.prisma.user.findMany({
  select: {
    id: true,
    email: true,
    name: true,
    // No cargar password, timestamps innecesarios
  },
});

// ❌ INCORRECTO: Select all
const users = await this.prisma.user.findMany();
```

#### 3. Caching Strategy

```typescript
// Cache por tenant con TTL
const cacheKey = `users:${tenantId}:list`;
const ttl = 300; // 5 minutos

// Try cache first
const cached = await this.redis.get(cacheKey);
if (cached) return JSON.parse(cached);

// Query DB
const users = await this.prisma.user.findMany();

// Cache result
await this.redis.set(cacheKey, JSON.stringify(users), 'EX', ttl);

return users;
```

#### 4. Pagination

```typescript
// Cursor-based pagination para mejor performance
async findAll(tenantId: string, filters: UserFiltersDto) {
  await this.setTenantContext(tenantId);

  return this.prisma.user.findMany({
    take: filters.limit,
    skip: (filters.page - 1) * filters.limit,
    cursor: filters.cursor ? { id: filters.cursor } : undefined,
    orderBy: { [filters.sortBy]: filters.sortOrder },
  });
}
```

---

## 📊 Monitoreo

### Logging Strategy

```typescript
// Winston logger con niveles
this.logger.log('User created');      // INFO
this.logger.debug('Query executed');  // DEBUG
this.logger.warn('Cache miss');       // WARN
this.logger.error('DB error', trace); // ERROR
```

### Metrics (Future)

- **Prometheus:** Métricas de aplicación
- **Grafana:** Dashboards
- **Alerting:** PagerDuty/Slack

---

<div align="center">

**Arquitectura Sólida, Escalable y Segura**

NestJS + PostgreSQL RLS + Redis

Type-safe | Isolated | Performant

</div>
