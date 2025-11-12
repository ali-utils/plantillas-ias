# Setup Inicial - bMOI Backend API Multi-Tenant

> **Guía completa para inicializar el proyecto desde cero**
> **Tiempo estimado**: 30-45 minutos

---

## 📋 Índice

1. [Pre-requisitos](#pre-requisitos)
2. [Paso 1: Inicializar Proyecto NestJS](#paso-1-inicializar-proyecto-nestjs)
3. [Paso 2: Configurar Docker](#paso-2-configurar-docker)
4. [Paso 3: Configurar Prisma](#paso-3-configurar-prisma)
5. [Paso 4: Configurar Variables de Entorno](#paso-4-configurar-variables-de-entorno)
6. [Paso 5: Instalar Dependencias](#paso-5-instalar-dependencias)
7. [Paso 6: Configurar Herramientas de Desarrollo](#paso-6-configurar-herramientas-de-desarrollo)
8. [Paso 7: Crear Estructura de Carpetas](#paso-7-crear-estructura-de-carpetas)
9. [Paso 8: Configurar PostgreSQL RLS](#paso-8-configurar-postgresql-rls)
10. [Paso 9: Primer Inicio](#paso-9-primer-inicio)
11. [Verificación Final](#verificación-final)

---

## 🔧 Pre-requisitos

Antes de comenzar, asegúrate de tener instalado:

```bash
# Verificar versiones
node --version    # >= 20.x LTS
npm --version     # >= 10.x
docker --version  # >= 24.x
docker compose version  # >= 2.x
git --version     # >= 2.x
```

**Versiones requeridas:**
- Node.js: **20 LTS** o superior
- npm: **10.x** o superior
- Docker: **24.x** o superior
- Docker Compose: **v2.x** o superior
- Git: **2.x** o superior

---

## Paso 1: Inicializar Proyecto NestJS

### 1.1 Instalar NestJS CLI (si no lo tienes)

```bash
npm install -g @nestjs/cli
```

### 1.2 Crear proyecto NestJS

```bash
# Navegar al directorio padre
cd D:\code\pBmoi

# Crear proyecto (SI EL DIRECTORIO NO EXISTE)
nest new bo-backend-api --package-manager npm --skip-git

# O si ya estás dentro del directorio bo-backend-api
cd bo-backend-api
npm init -y
npm install @nestjs/common @nestjs/core @nestjs/platform-express reflect-metadata rxjs
```

### 1.3 Verificar estructura inicial

```bash
# Deberías tener:
bo-backend-api/
├── src/
│   ├── app.controller.ts
│   ├── app.controller.spec.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test/
├── node_modules/
├── package.json
├── tsconfig.json
├── tsconfig.build.json
└── nest-cli.json
```

### 1.4 Limpiar archivos no necesarios

```bash
# Eliminar archivos de ejemplo
rm src/app.controller.ts
rm src/app.controller.spec.ts
rm src/app.service.ts
```

---

## Paso 2: Configurar Docker

### 2.1 Crear Dockerfile

Crear archivo `Dockerfile` en la raíz del proyecto:

```dockerfile
# ========================================
# Dockerfile - bMOI Backend API
# Multi-stage build para optimización
# ========================================

# Stage 1: Dependencies
FROM node:20-alpine AS dependencies
WORKDIR /app

# Instalar dependencias de build
RUN apk add --no-cache libc6-compat python3 make g++

# Copiar archivos de dependencias
COPY package*.json ./
COPY prisma ./prisma/

# Instalar dependencias
RUN npm ci --only=production && npm cache clean --force
RUN npm ci

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app

# Copiar dependencias del stage anterior
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .

# Generar Prisma Client
RUN npx prisma generate

# Build de la aplicación
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS production
WORKDIR /app

# Crear usuario no-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nestjs -u 1001

# Copiar archivos necesarios
COPY --from=builder --chown=nestjs:nodejs /app/dist ./dist
COPY --from=builder --chown=nestjs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nestjs:nodejs /app/package*.json ./
COPY --from=builder --chown=nestjs:nodejs /app/prisma ./prisma

# Variables de entorno
ENV NODE_ENV=production
ENV PORT=3000

# Exponer puerto
EXPOSE 3000

# Cambiar a usuario no-root
USER nestjs

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

# Comando de inicio
CMD ["node", "dist/main"]
```

### 2.2 Crear docker-compose.yml

Crear archivo `docker-compose.yml` en la raíz:

```yaml
# ========================================
# Docker Compose - bMOI Backend API
# Entorno completo de desarrollo
# ========================================

version: '3.9'

services:
  # ========================================
  # Backend NestJS
  # ========================================
  backend:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: bmoi-backend
    restart: unless-stopped
    ports:
      - "${PORT:-3000}:3000"
    environment:
      - NODE_ENV=${NODE_ENV:-development}
      - PORT=3000
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?schema=public
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - MINIO_ENDPOINT=minio
      - MINIO_PORT=9000
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      minio:
        condition: service_healthy
    volumes:
      - ./src:/app/src
      - ./prisma:/app/prisma
      - /app/node_modules
    networks:
      - bmoi-network
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # ========================================
  # PostgreSQL 15 con RLS
  # ========================================
  postgres:
    image: postgres:15-alpine
    container_name: bmoi-postgres
    restart: unless-stopped
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    environment:
      - POSTGRES_USER=${POSTGRES_USER:-bmoi}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD:-bmoi_password_dev}
      - POSTGRES_DB=${POSTGRES_DB:-bmoi_db}
      - POSTGRES_INITDB_ARGS=--encoding=UTF-8 --lc-collate=en_US.UTF-8 --lc-ctype=en_US.UTF-8
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init:/docker-entrypoint-initdb.d
    networks:
      - bmoi-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-bmoi} -d ${POSTGRES_DB:-bmoi_db}"]
      interval: 10s
      timeout: 5s
      retries: 5
    command:
      - "postgres"
      - "-c"
      - "max_connections=200"
      - "-c"
      - "shared_buffers=256MB"
      - "-c"
      - "effective_cache_size=1GB"
      - "-c"
      - "maintenance_work_mem=64MB"
      - "-c"
      - "checkpoint_completion_target=0.9"
      - "-c"
      - "wal_buffers=16MB"
      - "-c"
      - "default_statistics_target=100"
      - "-c"
      - "random_page_cost=1.1"
      - "-c"
      - "effective_io_concurrency=200"
      - "-c"
      - "work_mem=1310kB"
      - "-c"
      - "min_wal_size=1GB"
      - "-c"
      - "max_wal_size=4GB"

  # ========================================
  # Redis 7 - Cache & Queue
  # ========================================
  redis:
    image: redis:7-alpine
    container_name: bmoi-redis
    restart: unless-stopped
    ports:
      - "${REDIS_PORT:-6379}:6379"
    volumes:
      - redis_data:/data
      - ./docker/redis/redis.conf:/usr/local/etc/redis/redis.conf
    networks:
      - bmoi-network
    command: redis-server /usr/local/etc/redis/redis.conf
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  # ========================================
  # MinIO - S3-compatible Storage
  # ========================================
  minio:
    image: minio/minio:latest
    container_name: bmoi-minio
    restart: unless-stopped
    ports:
      - "${MINIO_PORT:-9000}:9000"
      - "${MINIO_CONSOLE_PORT:-9001}:9001"
    environment:
      - MINIO_ROOT_USER=${MINIO_ROOT_USER:-minioadmin}
      - MINIO_ROOT_PASSWORD=${MINIO_ROOT_PASSWORD:-minioadmin123}
    volumes:
      - minio_data:/data
    networks:
      - bmoi-network
    command: server /data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  # ========================================
  # Prisma Studio (Opcional - Solo Dev)
  # ========================================
  prisma-studio:
    image: node:20-alpine
    container_name: bmoi-prisma-studio
    restart: unless-stopped
    ports:
      - "5555:5555"
    environment:
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}?schema=public
    volumes:
      - ./prisma:/app/prisma
      - ./node_modules:/app/node_modules
    working_dir: /app
    depends_on:
      - postgres
    networks:
      - bmoi-network
    command: npx prisma studio --port 5555
    profiles:
      - dev

# ========================================
# Volumes
# ========================================
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  minio_data:
    driver: local

# ========================================
# Networks
# ========================================
networks:
  bmoi-network:
    driver: bridge
```

### 2.3 Crear .dockerignore

```bash
# .dockerignore
node_modules
npm-debug.log
dist
.git
.gitignore
.env
.env.*
!.env.example
.DS_Store
*.md
.vscode
.idea
coverage
.nyc_output
*.log
```

### 2.4 Crear archivos de configuración adicionales Docker

**docker/postgres/init/01-enable-extensions.sql:**

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Enable pgcrypto for encryption functions
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

-- Enable pg_trgm for text search
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
```

**docker/redis/redis.conf:**

```conf
# Redis configuration for bMOI Backend

# Persistence
save 900 1
save 300 10
save 60 10000

# Memory
maxmemory 512mb
maxmemory-policy allkeys-lru

# Security
requirepass ${REDIS_PASSWORD:-redis_password_dev}

# Networking
bind 0.0.0.0
protected-mode no
port 6379

# Performance
tcp-backlog 511
timeout 0
tcp-keepalive 300
```

---

## Paso 3: Configurar Prisma

### 3.1 Instalar Prisma

```bash
npm install -D prisma
npm install @prisma/client
```

### 3.2 Inicializar Prisma

```bash
npx prisma init
```

### 3.3 Crear schema.prisma completo

Editar `prisma/schema.prisma`:

```prisma
// ========================================
// Prisma Schema - bMOI Backend API
// ========================================

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["fullTextSearch", "fullTextIndex"]
  binaryTargets   = ["native", "linux-musl-openssl-3.0.x"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ========================================
// Enums
// ========================================

enum UserRole {
  ADMIN
  MANAGER
  USER
}

enum PlanType {
  FREE
  PLAN1
  PLAN2
  PLAN3
}

enum TenantStatus {
  ACTIVE
  SUSPENDED
  INACTIVE
}

enum UserStatus {
  ACTIVE
  INACTIVE
  PENDING
}

enum AuditAction {
  CREATE
  UPDATE
  DELETE
  LOGIN
  LOGOUT
  PASSWORD_CHANGE
  PERMISSION_CHANGE
}

// ========================================
// Models
// ========================================

model Tenant {
  id          String       @id @default(uuid()) @db.Uuid
  name        String       @db.VarChar(255)
  slug        String       @unique @db.VarChar(255)
  plan        PlanType     @default(FREE)
  status      TenantStatus @default(ACTIVE)
  settings    Json?        @db.JsonB
  limits      Json?        @db.JsonB // Plan limits

  // Timestamps
  createdAt   DateTime     @default(now()) @map("created_at") @db.Timestamptz
  updatedAt   DateTime     @updatedAt @map("updated_at") @db.Timestamptz

  // Relations
  users       User[]
  auditLogs   AuditLog[]

  @@map("tenants")
  @@index([slug])
  @@index([status])
  @@index([plan])
  @@index([createdAt(sort: Desc)])
}

model User {
  id          String     @id @default(uuid()) @db.Uuid
  tenantId    String     @map("tenant_id") @db.Uuid
  email       String     @db.VarChar(255)
  name        String     @db.VarChar(255)
  password    String     @db.VarChar(255)
  role        UserRole   @default(USER)
  status      UserStatus @default(PENDING)
  phone       String?    @db.VarChar(50)
  avatar      String?    @db.Text
  settings    Json?      @db.JsonB

  // Timestamps
  lastLoginAt DateTime?  @map("last_login_at") @db.Timestamptz
  createdAt   DateTime   @default(now()) @map("created_at") @db.Timestamptz
  updatedAt   DateTime   @updatedAt @map("updated_at") @db.Timestamptz

  // Relations
  tenant      Tenant     @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  refreshTokens RefreshToken[]
  auditLogs   AuditLog[]

  @@unique([tenantId, email])
  @@map("users")
  @@index([tenantId, email])
  @@index([tenantId, status])
  @@index([tenantId, role])
  @@index([email])
}

model RefreshToken {
  id          String   @id @default(uuid()) @db.Uuid
  userId      String   @map("user_id") @db.Uuid
  token       String   @unique @db.Text
  expiresAt   DateTime @map("expires_at") @db.Timestamptz
  isRevoked   Boolean  @default(false) @map("is_revoked")

  // Timestamps
  createdAt   DateTime @default(now()) @map("created_at") @db.Timestamptz

  // Relations
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("refresh_tokens")
  @@index([userId])
  @@index([token])
  @@index([expiresAt])
}

model AuditLog {
  id          String      @id @default(uuid()) @db.Uuid
  tenantId    String      @map("tenant_id") @db.Uuid
  userId      String?     @map("user_id") @db.Uuid
  action      AuditAction
  entity      String      @db.VarChar(100)
  entityId    String?     @map("entity_id") @db.Uuid
  changes     Json?       @db.JsonB
  metadata    Json?       @db.JsonB
  ipAddress   String?     @map("ip_address") @db.VarChar(45)
  userAgent   String?     @map("user_agent") @db.Text

  // Timestamps
  createdAt   DateTime    @default(now()) @map("created_at") @db.Timestamptz

  // Relations
  tenant      Tenant      @relation(fields: [tenantId], references: [id], onDelete: Cascade)
  user        User?       @relation(fields: [userId], references: [id], onDelete: SetNull)

  @@map("audit_logs")
  @@index([tenantId, createdAt(sort: Desc)])
  @@index([userId])
  @@index([action])
  @@index([entity, entityId])
}

// ========================================
// Row-Level Security Policies
// ========================================
// NOTE: RLS policies are applied via migrations
```

### 3.4 Crear migración inicial

```bash
npx prisma migrate dev --name init
```

---

## Paso 4: Configurar Variables de Entorno

### 4.1 Crear .env.example

```bash
# ========================================
# Environment Variables - bMOI Backend API
# Copy this file to .env and fill in the values
# ========================================

# ========================================
# Application
# ========================================
NODE_ENV=development
PORT=3000
API_PREFIX=api/v1

# ========================================
# Database (PostgreSQL)
# ========================================
DATABASE_URL=postgresql://bmoi:bmoi_password_dev@localhost:5432/bmoi_db?schema=public
POSTGRES_USER=bmoi
POSTGRES_PASSWORD=bmoi_password_dev
POSTGRES_DB=bmoi_db
POSTGRES_PORT=5432

# ========================================
# JWT
# ========================================
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_REFRESH_SECRET=your-super-secret-refresh-key-change-in-production
JWT_ACCESS_EXPIRATION=15m
JWT_REFRESH_EXPIRATION=7d

# ========================================
# Redis
# ========================================
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=redis_password_dev
REDIS_DB=0

# ========================================
# MinIO (S3-compatible)
# ========================================
MINIO_ENDPOINT=localhost
MINIO_PORT=9000
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin123
MINIO_USE_SSL=false
MINIO_BUCKET_NAME=bmoi-files

# ========================================
# CORS
# ========================================
CORS_ORIGIN=http://localhost:5173,http://localhost:3001
CORS_CREDENTIALS=true

# ========================================
# Rate Limiting
# ========================================
RATE_LIMIT_TTL=60
RATE_LIMIT_LIMIT=100

# ========================================
# Logging
# ========================================
LOG_LEVEL=debug
LOG_FORMAT=json

# ========================================
# Security
# ========================================
BCRYPT_ROUNDS=10

# ========================================
# Email (Optional - for future)
# ========================================
SMTP_HOST=
SMTP_PORT=587
SMTP_USER=
SMTP_PASSWORD=
SMTP_FROM=noreply@bmoi.com

# ========================================
# Monitoring (Optional)
# ========================================
SENTRY_DSN=
APM_SERVER_URL=
```

### 4.2 Crear .env local

```bash
cp .env.example .env
```

---

## Paso 5: Instalar Dependencias

### 5.1 Dependencias de producción

```bash
npm install @nestjs/common @nestjs/core @nestjs/platform-express
npm install @nestjs/config @nestjs/swagger
npm install @nestjs/jwt @nestjs/passport passport passport-jwt
npm install @nestjs/throttler @nestjs/bull bull bullmq
npm install @prisma/client
npm install class-validator class-transformer
npm install bcrypt
npm install helmet compression
npm install ioredis
npm install winston nest-winston
```

### 5.2 Dependencias de desarrollo

```bash
npm install -D @nestjs/cli @nestjs/schematics @nestjs/testing
npm install -D @types/node @types/express @types/jest @types/supertest
npm install -D @types/passport-jwt @types/bcrypt
npm install -D @typescript-eslint/eslint-plugin @typescript-eslint/parser
npm install -D eslint eslint-config-prettier eslint-plugin-prettier
npm install -D prettier
npm install -D jest ts-jest supertest
npm install -D prisma
npm install -D husky lint-staged
npm install -D @commitlint/cli @commitlint/config-conventional
```

---

## Paso 6: Configurar Herramientas de Desarrollo

### 6.1 Configurar TypeScript (tsconfig.json)

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2021",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "strictBindCallApply": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "paths": {
      "@/*": ["src/*"],
      "@common/*": ["src/common/*"],
      "@modules/*": ["src/modules/*"],
      "@config/*": ["src/config/*"],
      "@database/*": ["src/database/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "test"]
}
```

### 6.2 Configurar ESLint (.eslintrc.js)

```javascript
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: 'tsconfig.json',
    tsconfigRootDir: __dirname,
    sourceType: 'module',
  },
  plugins: ['@typescript-eslint/eslint-plugin'],
  extends: [
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
  ],
  root: true,
  env: {
    node: true,
    jest: true,
  },
  ignorePatterns: ['.eslintrc.js'],
  rules: {
    '@typescript-eslint/interface-name-prefix': 'off',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'no-console': ['warn', { allow: ['warn', 'error'] }],
  },
};
```

### 6.3 Configurar Prettier (.prettierrc)

```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### 6.4 Configurar Jest (jest.config.js)

```javascript
module.exports = {
  moduleFileExtensions: ['js', 'json', 'ts'],
  rootDir: 'src',
  testRegex: '.*\\.spec\\.ts$',
  transform: {
    '^.+\\.(t|j)s$': 'ts-jest',
  },
  collectCoverageFrom: [
    '**/*.(t|j)s',
    '!**/*.spec.ts',
    '!**/node_modules/**',
    '!**/dist/**',
  ],
  coverageDirectory: '../coverage',
  testEnvironment: 'node',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/$1',
    '^@common/(.*)$': '<rootDir>/common/$1',
    '^@modules/(.*)$': '<rootDir>/modules/$1',
    '^@config/(.*)$': '<rootDir>/config/$1',
    '^@database/(.*)$': '<rootDir>/database/$1',
  },
};
```

### 6.5 Configurar Husky y Lint-Staged

```bash
# Inicializar Husky
npx husky install
npm pkg set scripts.prepare="husky install"

# Crear hook pre-commit
npx husky add .husky/pre-commit "npx lint-staged"

# Crear hook commit-msg
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

**package.json (agregar):**

```json
{
  "lint-staged": {
    "*.ts": [
      "eslint --fix",
      "prettier --write"
    ]
  }
}
```

**commitlint.config.js:**

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',
        'fix',
        'docs',
        'style',
        'refactor',
        'perf',
        'test',
        'build',
        'ci',
        'chore',
        'revert',
      ],
    ],
  },
};
```

---

## Paso 7: Crear Estructura de Carpetas

```bash
# Crear estructura completa
mkdir -p src/{common/{decorators,guards,interceptors,pipes,filters,interfaces},config,database,modules/{auth,tenants,users,health},shared/{enums,constants,types}}

# Crear subcarpetas para common
mkdir -p src/common/decorators
mkdir -p src/common/guards
mkdir -p src/common/interceptors
mkdir -p src/common/pipes
mkdir -p src/common/filters
mkdir -p src/common/interfaces

# Crear carpetas de config
mkdir -p src/config

# Crear carpetas de database
mkdir -p src/database

# Crear módulos base
mkdir -p src/modules/auth
mkdir -p src/modules/tenants
mkdir -p src/modules/users
mkdir -p src/modules/health

# Crear carpetas shared
mkdir -p src/shared/enums
mkdir -p src/shared/constants
mkdir -p src/shared/types

# Crear carpetas Docker
mkdir -p docker/postgres/init
mkdir -p docker/redis
mkdir -p docker/nginx
```

**Estructura final:**

```
src/
├── common/
│   ├── decorators/
│   ├── guards/
│   ├── interceptors/
│   ├── pipes/
│   ├── filters/
│   └── interfaces/
├── config/
│   ├── app.config.ts
│   ├── database.config.ts
│   ├── jwt.config.ts
│   └── redis.config.ts
├── database/
│   ├── prisma.service.ts
│   └── prisma.module.ts
├── modules/
│   ├── auth/
│   ├── tenants/
│   ├── users/
│   └── health/
├── shared/
│   ├── enums/
│   ├── constants/
│   └── types/
├── app.module.ts
└── main.ts
```

---

## Paso 8: Configurar PostgreSQL RLS

### 8.1 Crear migración para RLS

Crear archivo `prisma/migrations/<timestamp>_add_rls/migration.sql`:

```sql
-- ========================================
-- Enable Row-Level Security (RLS)
-- ========================================

-- Enable RLS on tenants table
ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;

-- Enable RLS on users table
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Enable RLS on refresh_tokens table
ALTER TABLE refresh_tokens ENABLE ROW LEVEL SECURITY;

-- Enable RLS on audit_logs table
ALTER TABLE audit_logs ENABLE ROW LEVEL SECURITY;

-- ========================================
-- RLS Policies for USERS table
-- ========================================

-- Policy: Users can only see their tenant's data
CREATE POLICY tenant_isolation_users ON users
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant_id', true)::uuid);

-- Policy: Insert must have correct tenant_id
CREATE POLICY tenant_isolation_users_insert ON users
  FOR INSERT
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id', true)::uuid);

-- ========================================
-- RLS Policies for REFRESH_TOKENS table
-- ========================================

CREATE POLICY tenant_isolation_refresh_tokens ON refresh_tokens
  FOR ALL
  USING (
    user_id IN (
      SELECT id FROM users
      WHERE tenant_id = current_setting('app.current_tenant_id', true)::uuid
    )
  );

-- ========================================
-- RLS Policies for AUDIT_LOGS table
-- ========================================

CREATE POLICY tenant_isolation_audit_logs ON audit_logs
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant_id', true)::uuid);

CREATE POLICY tenant_isolation_audit_logs_insert ON audit_logs
  FOR INSERT
  WITH CHECK (tenant_id = current_setting('app.current_tenant_id', true)::uuid);

-- ========================================
-- Indexes for Performance
-- ========================================

CREATE INDEX IF NOT EXISTS idx_users_tenant_email ON users(tenant_id, email);
CREATE INDEX IF NOT EXISTS idx_users_tenant_status ON users(tenant_id, status);
CREATE INDEX IF NOT EXISTS idx_users_tenant_role ON users(tenant_id, role);
CREATE INDEX IF NOT EXISTS idx_refresh_tokens_user ON refresh_tokens(user_id);
CREATE INDEX IF NOT EXISTS idx_refresh_tokens_token ON refresh_tokens(token);
CREATE INDEX IF NOT EXISTS idx_audit_logs_tenant_created ON audit_logs(tenant_id, created_at DESC);
```

---

## Paso 9: Primer Inicio

### 9.1 Levantar Docker Compose

```bash
# Levantar todos los servicios
docker compose up -d

# Ver logs
docker compose logs -f backend

# Verificar servicios
docker compose ps
```

### 9.2 Ejecutar migraciones

```bash
# Dentro del contenedor
docker compose exec backend npx prisma migrate deploy

# O localmente (si tienes PostgreSQL accesible)
npx prisma migrate deploy
```

### 9.3 Generar Prisma Client

```bash
docker compose exec backend npx prisma generate
```

### 9.4 Seed inicial (opcional)

Crear `prisma/seed.ts`:

```typescript
import { PrismaClient } from '@prisma/client';
import * as bcrypt from 'bcrypt';

const prisma = new PrismaClient();

async function main() {
  console.log('🌱 Seeding database...');

  // Create default tenant
  const tenant = await prisma.tenant.create({
    data: {
      name: 'Demo Tenant',
      slug: 'demo-tenant',
      plan: 'PLAN1',
      status: 'ACTIVE',
    },
  });

  console.log('✅ Tenant created:', tenant.name);

  // Create admin user
  const hashedPassword = await bcrypt.hash('Admin123!', 10);
  const admin = await prisma.user.create({
    data: {
      tenantId: tenant.id,
      email: 'admin@demo.com',
      name: 'Admin User',
      password: hashedPassword,
      role: 'ADMIN',
      status: 'ACTIVE',
    },
  });

  console.log('✅ Admin user created:', admin.email);
  console.log('🎉 Seeding completed!');
}

main()
  .catch((e) => {
    console.error('❌ Seeding failed:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Ejecutar seed:

```bash
docker compose exec backend npx prisma db seed
```

---

## Verificación Final

### ✅ Checklist de Verificación

```bash
# 1. Verificar que Docker Compose está corriendo
docker compose ps

# Deberías ver:
# - bmoi-backend (running)
# - bmoi-postgres (healthy)
# - bmoi-redis (healthy)
# - bmoi-minio (healthy)

# 2. Verificar PostgreSQL
docker compose exec postgres psql -U bmoi -d bmoi_db -c "\dt"

# Deberías ver las tablas: tenants, users, refresh_tokens, audit_logs

# 3. Verificar Redis
docker compose exec redis redis-cli ping

# Debería responder: PONG

# 4. Verificar MinIO
curl http://localhost:9000/minio/health/live

# 5. Verificar Backend (si está corriendo)
curl http://localhost:3000/health

# 6. Verificar Prisma Studio (opcional)
# Abrir en navegador: http://localhost:5555
```

### 🎯 Siguientes Pasos

1. **Implementar Health Check Module** → Ver `FASE_1_AUTH.md`
2. **Implementar Auth Module** → JWT + Refresh Tokens
3. **Implementar Guards** → JWT, Tenant, Permission
4. **Implementar Interceptors** → Tenant, Audit, Logging
5. **Tests** → Unit + E2E

---

## 🔧 Comandos Útiles

```bash
# Docker
docker compose up -d                    # Iniciar servicios
docker compose down                     # Detener servicios
docker compose down -v                  # Detener y eliminar volúmenes
docker compose logs -f backend          # Ver logs del backend
docker compose exec backend sh          # Shell en el contenedor

# Prisma
docker compose exec backend npx prisma generate         # Generar cliente
docker compose exec backend npx prisma migrate dev      # Crear migración
docker compose exec backend npx prisma migrate deploy   # Aplicar migraciones
docker compose exec backend npx prisma studio           # Abrir Prisma Studio
docker compose exec backend npx prisma db seed          # Ejecutar seed

# NestJS
docker compose exec backend npm run start:dev           # Desarrollo
docker compose exec backend npm run build               # Build
docker compose exec backend npm run test                # Tests
docker compose exec backend npm run test:e2e            # E2E tests
docker compose exec backend npm run lint                # Lint
docker compose exec backend npm run format              # Format

# Logs
docker compose logs -f                  # Todos los servicios
docker compose logs -f backend          # Solo backend
docker compose logs -f postgres         # Solo PostgreSQL
docker compose logs -f redis            # Solo Redis
```

---

<div align="center">

**Setup completado! 🎉**

Ahora estás listo para comenzar a desarrollar el backend multi-tenant

Próximo paso: [FASE_1_AUTH.md](.docs/FASE_1_AUTH.md)

</div>
