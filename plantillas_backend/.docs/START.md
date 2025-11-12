# START - Guía Paso a Paso para Empezar con el Backend

> **Guía completa para desarrolladores que empiezan con el backend bMOI**
>
> Incluye instrucciones específicas para trabajar con Claude Code, GitHub Copilot y Cursor AI

---

## 📋 Tabla de Contenidos

- [Antes de Empezar](#antes-de-empezar)
- [Setup Inicial](#setup-inicial)
- [Primer Desarrollo: Crear un Módulo](#primer-desarrollo-crear-un-módulo)
- [Testing](#testing)
- [Trabajar con IAs](#trabajar-con-ias)
- [Workflow Diario](#workflow-diario)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Antes de Empezar

### Prerequisitos

- ✅ **Docker** (20.10+) y **Docker Compose** (2.0+)
- ✅ **Git**
- ✅ **VS Code** (recomendado) o tu editor favorito
- ✅ **Extensiones VS Code recomendadas:**
  - Prisma
  - ESLint
  - Prettier
  - Docker
  - Thunder Client o REST Client (para testing)

### Lectura Obligatoria

**Antes de escribir una sola línea de código, lee estos documentos en orden:**

1. **`QUICKSTART.md`** (5 min) - Setup rápido
2. **`PROJECT_INSTRUCTIONS.md`** (15 min) - Guía completa
3. **`.claude/context.md`** (10 min) - Contexto del proyecto
4. **`.claude/conventions.md`** (10 min) - Convenciones obligatorias
5. **`CODING_STANDARDS.md`** (15 min) - Estándares de código
6. **`ARQUITECTURA.md`** (15 min) - Decisiones arquitectónicas

**Total: ~70 minutos de lectura que te ahorrarán horas de confusión**

---

## 🚀 Setup Inicial

### Paso 1: Clonar y Configurar (5 min)

```bash
# Clonar repositorio
git clone https://github.com/your-org/bmoi-backend-api.git
cd bmoi-backend-api

# Copiar variables de entorno
cp .env.example .env

# Editar .env con tus valores
# (Para desarrollo local, los valores por defecto funcionan)
code .env
```

### Paso 2: Iniciar Docker (2 min)

```bash
# Iniciar todos los servicios
docker compose up -d

# Ver logs
docker compose logs -f backend
```

**Espera hasta ver:**
```
[Nest] LOG [NestApplication] Nest application successfully started
[Nest] LOG [Bootstrap] Application is running on: http://localhost:5000
```

### Paso 3: Ejecutar Migraciones (1 min)

```bash
# Ejecutar migraciones
docker compose exec backend npx prisma migrate dev

# Seed data inicial
docker compose exec backend npm run seed
```

### Paso 4: Verificar (1 min)

```bash
# Health check
curl http://localhost:5000/health

# Abrir Swagger
open http://localhost:5000/api/docs

# Abrir Prisma Studio
docker compose exec backend npx prisma studio
```

---

## 👨‍💻 Primer Desarrollo: Crear un Módulo

### Escenario: Crear módulo de "Products"

Vamos a crear un módulo completo de productos paso a paso.

### Paso 1: Crear Estructura del Módulo (2 min)

```bash
# Generar módulo, controller y service
docker compose exec backend nest g module modules/products
docker compose exec backend nest g controller modules/products
docker compose exec backend nest g service modules/products

# Crear carpetas adicionales
mkdir -p src/modules/products/dto
mkdir -p src/modules/products/entities
mkdir -p src/modules/products/tests
```

**Estructura creada:**
```
src/modules/products/
├── products.module.ts
├── products.controller.ts
├── products.service.ts
├── dto/
├── entities/
└── tests/
```

### Paso 2: Crear el Schema de Prisma (5 min)

Edita `prisma/schema.prisma`:

```prisma
// Modelo Product
model Product {
  id          String   @id @default(uuid()) @db.Uuid
  tenant_id   String   @db.Uuid
  name        String   @db.VarChar(255)
  description String?  @db.Text
  price       Decimal  @db.Decimal(10, 2)
  stock       Int      @default(0)
  sku         String   @db.VarChar(100)
  is_active   Boolean  @default(true)
  created_at  DateTime @default(now()) @db.Timestamptz
  updated_at  DateTime @updatedAt @db.Timestamptz

  // Relación con tenant
  tenant      Tenant   @relation(fields: [tenant_id], references: [id], onDelete: Cascade)

  // Índices
  @@index([tenant_id, created_at(sort: Desc)])
  @@index([tenant_id, sku])
  @@unique([tenant_id, sku])
  @@map("products")
}
```

**Crear migration:**

```bash
docker compose exec backend npx prisma migrate dev --name add_products_table
```

### Paso 3: Crear DTOs (10 min)

**`src/modules/products/dto/create-product.dto.ts`:**

```typescript
import { IsString, IsNumber, IsOptional, MinLength, Min, IsBoolean } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateProductDto {
  @ApiProperty({ example: 'Product Name', minLength: 3 })
  @IsString()
  @MinLength(3)
  name: string;

  @ApiPropertyOptional({ example: 'Product description' })
  @IsOptional()
  @IsString()
  description?: string;

  @ApiProperty({ example: 99.99, minimum: 0 })
  @IsNumber()
  @Min(0)
  price: number;

  @ApiProperty({ example: 100, minimum: 0 })
  @IsNumber()
  @Min(0)
  stock: number;

  @ApiProperty({ example: 'SKU-001' })
  @IsString()
  sku: string;

  @ApiPropertyOptional({ example: true })
  @IsOptional()
  @IsBoolean()
  isActive?: boolean;
}
```

**`src/modules/products/dto/update-product.dto.ts`:**

```typescript
import { PartialType } from '@nestjs/swagger';
import { CreateProductDto } from './create-product.dto';

export class UpdateProductDto extends PartialType(CreateProductDto) {}
```

### Paso 4: Crear Entity (5 min)

**`src/modules/products/entities/product.entity.ts`:**

```typescript
import { ApiProperty } from '@nestjs/swagger';

export class Product {
  @ApiProperty({ example: '123e4567-e89b-12d3-a456-426614174000' })
  id: string;

  @ApiProperty({ example: 'Product Name' })
  name: string;

  @ApiProperty({ example: 'Product description' })
  description?: string;

  @ApiProperty({ example: 99.99 })
  price: number;

  @ApiProperty({ example: 100 })
  stock: number;

  @ApiProperty({ example: 'SKU-001' })
  sku: string;

  @ApiProperty({ example: true })
  isActive: boolean;

  @ApiProperty({ example: '2025-01-01T00:00:00.000Z' })
  createdAt: Date;

  @ApiProperty({ example: '2025-01-01T00:00:00.000Z' })
  updatedAt: Date;
}
```

### Paso 5: Implementar Service (15 min)

**`src/modules/products/products.service.ts`:**

```typescript
import {
  Injectable,
  Logger,
  NotFoundException,
  ConflictException,
} from '@nestjs/common';
import { PrismaService } from '@/database/prisma.service';

import { CreateProductDto } from './dto/create-product.dto';
import { UpdateProductDto } from './dto/update-product.dto';
import { Product } from './entities/product.entity';

@Injectable()
export class ProductsService {
  private readonly logger = new Logger(ProductsService.name);

  constructor(private readonly prisma: PrismaService) {}

  async findAll(tenantId: string): Promise<Product[]> {
    // ✅ CRÍTICO: Set tenant context
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    const products = await this.prisma.product.findMany({
      orderBy: { createdAt: 'desc' },
    });

    this.logger.log(`Found ${products.length} products for tenant: ${tenantId}`);
    return products;
  }

  async findOne(tenantId: string, id: string): Promise<Product> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    const product = await this.prisma.product.findUnique({
      where: { id },
    });

    if (!product) {
      throw new NotFoundException(`Product with ID ${id} not found`);
    }

    return product;
  }

  async create(tenantId: string, dto: CreateProductDto): Promise<Product> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    // Verificar SKU único dentro del tenant
    const existing = await this.prisma.product.findFirst({
      where: { sku: dto.sku },
    });

    if (existing) {
      throw new ConflictException('SKU already exists');
    }

    const product = await this.prisma.product.create({
      data: {
        tenant_id: tenantId, // ✅ EXPLÍCITO
        ...dto,
      },
    });

    this.logger.log(`Product created: ${product.id} for tenant: ${tenantId}`);
    return product;
  }

  async update(tenantId: string, id: string, dto: UpdateProductDto): Promise<Product> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    await this.findOne(tenantId, id); // Verify exists

    const product = await this.prisma.product.update({
      where: { id },
      data: dto,
    });

    return product;
  }

  async remove(tenantId: string, id: string): Promise<void> {
    await this.prisma.$executeRaw`
      SELECT set_config('app.current_tenant_id', ${tenantId}, true)
    `;

    await this.findOne(tenantId, id); // Verify exists

    await this.prisma.product.delete({
      where: { id },
    });

    this.logger.log(`Product deleted: ${id} for tenant: ${tenantId}`);
  }
}
```

### Paso 6: Implementar Controller (10 min)

**`src/modules/products/products.controller.ts`:**

```typescript
import {
  Controller,
  Get,
  Post,
  Patch,
  Delete,
  Body,
  Param,
  UseGuards,
  UseInterceptors,
  Logger,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';

import { JwtAuthGuard } from '@/common/guards/jwt-auth.guard';
import { TenantGuard } from '@/common/guards/tenant.guard';
import { TenantInterceptor } from '@/common/interceptors/tenant.interceptor';
import { Tenant } from '@/common/decorators/tenant.decorator';
import { RequirePermission } from '@/common/decorators/permissions.decorator';
import { CheckPlanLimit } from '@/common/decorators/plan-limit.decorator';
import { ITenantContext } from '@/common/interfaces/tenant-context.interface';

import { ProductsService } from './products.service';
import { CreateProductDto } from './dto/create-product.dto';
import { UpdateProductDto } from './dto/update-product.dto';
import { Product } from './entities/product.entity';

@Controller('products')
@ApiTags('products')
@ApiBearerAuth()
@UseGuards(JwtAuthGuard, TenantGuard)
@UseInterceptors(TenantInterceptor)
export class ProductsController {
  private readonly logger = new Logger(ProductsController.name);

  constructor(private readonly productsService: ProductsService) {}

  @Get()
  @RequirePermission('products.read')
  @ApiOperation({ summary: 'Get all products' })
  @ApiResponse({ status: 200, type: [Product] })
  async findAll(@Tenant() tenant: ITenantContext): Promise<Product[]> {
    return this.productsService.findAll(tenant.id);
  }

  @Get(':id')
  @RequirePermission('products.read')
  @ApiOperation({ summary: 'Get product by ID' })
  async findOne(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ): Promise<Product> {
    return this.productsService.findOne(tenant.id, id);
  }

  @Post()
  @RequirePermission('products.create')
  @CheckPlanLimit('products')
  @ApiOperation({ summary: 'Create product' })
  async create(
    @Tenant() tenant: ITenantContext,
    @Body() createProductDto: CreateProductDto,
  ): Promise<Product> {
    return this.productsService.create(tenant.id, createProductDto);
  }

  @Patch(':id')
  @RequirePermission('products.update')
  @ApiOperation({ summary: 'Update product' })
  async update(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
    @Body() updateProductDto: UpdateProductDto,
  ): Promise<Product> {
    return this.productsService.update(tenant.id, id, updateProductDto);
  }

  @Delete(':id')
  @RequirePermission('products.delete')
  @HttpCode(HttpStatus.NO_CONTENT)
  @ApiOperation({ summary: 'Delete product' })
  async remove(
    @Tenant() tenant: ITenantContext,
    @Param('id') id: string,
  ): Promise<void> {
    return this.productsService.remove(tenant.id, id);
  }
}
```

### Paso 7: Registrar en App Module (2 min)

Edita `src/app.module.ts`:

```typescript
import { ProductsModule } from './modules/products/products.module';

@Module({
  imports: [
    // ... otros módulos
    ProductsModule,
  ],
})
export class AppModule {}
```

### Paso 8: Probar el Módulo (5 min)

```bash
# Verificar que compila
docker compose exec backend npm run build

# Verificar endpoints en Swagger
open http://localhost:5000/api/docs

# Probar con curl (con token de login)
TOKEN="your-jwt-token"
TENANT_ID="your-tenant-id"

curl -X POST http://localhost:5000/products \
  -H "Authorization: Bearer $TOKEN" \
  -H "X-Tenant-ID: $TENANT_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test Product",
    "price": 99.99,
    "stock": 100,
    "sku": "TEST-001"
  }'
```

---

## 🧪 Testing

### Paso 1: Crear Unit Tests (15 min)

**`src/modules/products/tests/products.service.spec.ts`:**

```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { ProductsService } from '../products.service';
import { PrismaService } from '@/database/prisma.service';

describe('ProductsService', () => {
  let service: ProductsService;
  let prisma: PrismaService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ProductsService,
        {
          provide: PrismaService,
          useValue: {
            $executeRaw: jest.fn(),
            product: {
              findMany: jest.fn(),
              findUnique: jest.fn(),
              findFirst: jest.fn(),
              create: jest.fn(),
              update: jest.fn(),
              delete: jest.fn(),
            },
          },
        },
      ],
    }).compile();

    service = module.get<ProductsService>(ProductsService);
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

    it('should return array of products', async () => {
      const mockProducts = [
        { id: '1', name: 'Product 1', price: 99.99 },
        { id: '2', name: 'Product 2', price: 199.99 },
      ];

      (prisma.product.findMany as jest.Mock).mockResolvedValue(mockProducts);

      const result = await service.findAll('tenant-123');

      expect(result).toEqual(mockProducts);
    });
  });

  // ... más tests
});
```

### Paso 2: Ejecutar Tests

```bash
# Unit tests
docker compose exec backend npm run test

# Con coverage
docker compose exec backend npm run test:cov

# Watch mode
docker compose exec backend npm run test:watch products.service
```

---

## 🤖 Trabajar con IAs

### Claude Code

**Primer mensaje al empezar sesión:**

```
Hola Claude, voy a trabajar en el backend bMOI (NestJS multi-tenant).

Por favor lee estos archivos para entender el proyecto:
- PROJECT_INSTRUCTIONS.md
- .claude/context.md
- .claude/conventions.md

Confirma que entiendes:
1. Framework: NestJS 10
2. Multi-tenant: PostgreSQL RLS
3. Convenciones: kebab-case, PascalCase, camelCase
4. NUNCA usar: snake_case, any type
5. SIEMPRE: set_config para RLS

Voy a crear el módulo de [NOMBRE]. ¿Estás listo?
```

### GitHub Copilot

1. Abre `.copilot/instructions.md`
2. Copilot lee automáticamente ese archivo
3. Empieza a escribir código, Copilot sugerirá según las instrucciones

### Cursor AI

1. Cursor lee `.cursorrules` automáticamente
2. Usa `Cmd+K` para chat rápido
3. Usa `Cmd+L` para chat largo

**Ejemplo de prompt:**

```
Crea el service de products siguiendo las convenciones del proyecto.
Debe incluir:
- Set tenant context con $executeRaw
- Métodos findAll, findOne, create, update, remove
- Logger en cada método
- Validación de SKU único dentro del tenant
```

---

## 📅 Workflow Diario

### Inicio del Día

```bash
# 1. Pull últimos cambios
git pull origin main

# 2. Iniciar Docker
docker compose up -d

# 3. Ver logs
docker compose logs -f backend

# 4. Verificar que todo funciona
curl http://localhost:5000/health
```

### Durante el Desarrollo

```bash
# Crear nueva branch
git checkout -b feature/nombre-feature

# Desarrollar con hot-reload activo
# (edita archivos, backend se recarga automáticamente)

# Ejecutar tests frecuentemente
docker compose exec backend npm run test

# Verificar linting
docker compose exec backend npm run lint
```

### Fin del Día

```bash
# 1. Ejecutar todos los tests
docker compose exec backend npm run test

# 2. Verificar coverage
docker compose exec backend npm run test:cov

# 3. Commit
git add .
git commit -m "feat: descripción del cambio"

# 4. Push
git push origin feature/nombre-feature

# 5. Crear PR en GitHub

# 6. Actualizar session-state.md
# Editar .claude/session-state.md con:
# - Módulos creados hoy
# - Endpoints implementados
# - Tests escritos
# - Próximos pasos
```

---

## 🐛 Troubleshooting

### Error: "Port 5000 already in use"

```bash
# Cambiar puerto en docker-compose.yml
services:
  backend:
    ports:
      - "3001:5000"
```

### Error: "Database connection refused"

```bash
# Verificar PostgreSQL
docker compose ps postgres

# Restart
docker compose restart postgres

# Ver logs
docker compose logs postgres
```

### Hot-reload no funciona

```bash
# Rebuild backend image
docker compose build backend

# Restart backend
docker compose restart backend
```

### Tests fallan

```bash
# Ver logs detallados
docker compose exec backend npm run test -- --verbose

# Limpiar cache de jest
docker compose exec backend npm run test -- --clearCache
```

---

## 📚 Recursos Adicionales

- **NestJS Docs:** https://docs.nestjs.com
- **Prisma Docs:** https://www.prisma.io/docs
- **PostgreSQL RLS:** https://www.postgresql.org/docs/current/ddl-rowsecurity.html

---

<div align="center">

**¡Listo para empezar!**

Sigue esta guía paso a paso y tendrás tu primer módulo funcionando en <2 horas

[QUICKSTART](QUICKSTART.md) • [Arquitectura](ARQUITECTURA.md) • [Standards](CODING_STANDARDS.md)

</div>
