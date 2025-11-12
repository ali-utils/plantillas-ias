# bMOI - Guía de Integración de Componentes

**Versión:** 1.0.0
**Fecha:** 2025-01-04
**Propósito:** Documentación oficial de integración entre todos los componentes del ecosistema bMOI

---

## 📋 Tabla de Contenidos

- [URLs Oficiales](#-urls-oficiales)
- [Headers Obligatorios](#-headers-obligatorios)
- [Autenticación Multi-Tenant](#-autenticación-multi-tenant)
- [Versionado de API](#-versionado-de-api)
- [Flujos End-to-End](#-flujos-end-to-end)
- [Convenciones por Lenguaje](#-convenciones-por-lenguaje)
- [Arquitectura de Comunicación](#-arquitectura-de-comunicación)

---

## 🌐 URLs Oficiales

### Entornos

#### Desarrollo Local

```bash
# Backend API
API_BASE_URL=http://localhost:3000/api/v1

# Frontend CPanel Multi-Tenant
CPANEL_URL=http://localhost:3001

# Landing Page (Static)
LANDING_URL=http://localhost:4321

# Mobile App (emulador)
MOBILE_API_URL=http://10.0.2.2:3000/api/v1  # Android
MOBILE_API_URL=http://localhost:3000/api/v1  # iOS
```

#### Staging

```bash
# Backend API
API_BASE_URL=https://api-staging.bmoi.com/api/v1

# Frontend CPanel
CPANEL_URL=https://cpanel-staging.bmoi.com

# Landing Page
LANDING_URL=https://staging.bmoi.com

# Mobile App
MOBILE_API_URL=https://api-staging.bmoi.com/api/v1
```

#### Producción

```bash
# Backend API
API_BASE_URL=https://api.bmoi.com/api/v1

# API Gateway (si se implementa)
API_GATEWAY_URL=https://gateway.bmoi.com

# BFF WebMovil (si se implementa)
BFF_URL=https://bff.bmoi.com/api/v1

# Frontend CPanel
CPANEL_URL=https://cpanel.bmoi.com

# Landing Page
LANDING_URL=https://bmoi.com

# Mobile App
MOBILE_API_URL=https://api.bmoi.com/api/v1
```

---

## 🔑 Headers Obligatorios

### Todas las Requests Autenticadas

```http
Authorization: Bearer <JWT_TOKEN>
X-Tenant-ID: <TENANT_UUID>
Content-Type: application/json
Accept: application/json
```

### Requests Públicas (Landing Page - Registro/Contacto)

```http
X-Tenant-ID: public
Content-Type: application/json
Accept: application/json
```

### Implementación por Tecnología

#### React (CPanel) - Axios

```typescript
// src/services/api/client.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
});

// Interceptor para agregar headers automáticamente
apiClient.interceptors.request.use(config => {
  const token = localStorage.getItem('accessToken');
  const tenantId = localStorage.getItem('currentTenantId');

  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  if (tenantId) {
    config.headers['X-Tenant-ID'] = tenantId;
  }

  return config;
});
```

#### Flutter (Mobile) - Dio

```dart
// lib/core/network/dio_client.dart
import 'package:dio/dio.dart';
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class DioClient {
  late final Dio dio;
  final FlutterSecureStorage secureStorage;

  DioClient({required this.secureStorage}) {
    dio = Dio(BaseOptions(
      baseUrl: 'https://api.bmoi.com/api/v1',
      connectTimeout: const Duration(seconds: 30),
      receiveTimeout: const Duration(seconds: 30),
    ));

    dio.interceptors.add(AuthInterceptor(secureStorage));
  }
}

class AuthInterceptor extends Interceptor {
  final FlutterSecureStorage secureStorage;

  AuthInterceptor(this.secureStorage);

  @override
  void onRequest(
    RequestOptions options,
    RequestInterceptorHandler handler,
  ) async {
    final token = await secureStorage.read(key: 'jwt_token');
    final tenantId = await secureStorage.read(key: 'tenant_id');

    if (token != null) {
      options.headers['Authorization'] = 'Bearer $token';
    }

    if (tenantId != null) {
      options.headers['X-Tenant-ID'] = tenantId;
    }

    handler.next(options);
  }
}
```

#### Astro (Landing Page) - Fetch

```typescript
// src/utils/api-client.ts
const API_BASE_URL = import.meta.env.PUBLIC_API_BASE_URL;

export async function apiPost(endpoint: string, data: any, tenantId = 'public') {
  const response = await fetch(`${API_BASE_URL}${endpoint}`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-Tenant-ID': tenantId,
    },
    body: JSON.stringify(data),
  });

  if (!response.ok) {
    throw new Error(`API Error: ${response.statusText}`);
  }

  return response.json();
}
```

---

## 🔐 Autenticación Multi-Tenant

### Flujo Completo de Login

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUJO DE AUTENTICACIÓN                    │
└─────────────────────────────────────────────────────────────┘

1. Usuario ingresa email + password
   ↓
2. POST /api/v1/auth/login
   Body: { email, password }
   Headers: { X-Tenant-ID: "public" }  ← Tenant público para login
   ↓
3. Backend valida credenciales
   ↓
4. Backend retorna:
   {
     accessToken: "eyJhbGc...",
     refreshToken: "eyJhbGc...",
     user: {
       id: "uuid",
       email: "user@example.com",
       name: "John Doe"
     },
     tenants: [  ← ⚠️ PLURAL - Lista de tenants del usuario
       {
         id: "tenant-uuid-1",
         name: "Company A",
         slug: "company-a",
         plan: "plan2",
         role: "admin"
       },
       {
         id: "tenant-uuid-2",
         name: "Company B",
         slug: "company-b",
         plan: "free",
         role: "user"
       }
     ]
   }
   ↓
5. Frontend guarda:
   - accessToken (localStorage/SecureStorage)
   - refreshToken (localStorage/SecureStorage)
   - user (state/context)
   - tenants[] (state/context)
   ↓
6. Frontend evalúa:
   if (tenants.length === 1) {
     // Seleccionar automáticamente el único tenant
     setCurrentTenant(tenants[0]);
   } else if (tenants.length > 1) {
     // Mostrar selector de tenant
     showTenantSelector(tenants);
   }
   ↓
7. Una vez seleccionado el tenant:
   - Guardar currentTenantId
   - Todas las requests subsiguientes incluyen:
     * Authorization: Bearer {accessToken}
     * X-Tenant-ID: {currentTenantId}
```

### Endpoints de Autenticación

#### POST /api/v1/auth/register

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "name": "John Doe",
  "companyName": "My Company",
  "plan": "free"
}
```

**Headers:**
```http
X-Tenant-ID: public
Content-Type: application/json
```

**Response:** (201 Created)
```json
{
  "message": "User registered successfully",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "tenant": {
    "id": "tenant-uuid",
    "name": "My Company",
    "slug": "my-company",
    "plan": "free"
  }
}
```

#### POST /api/v1/auth/login

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Headers:**
```http
X-Tenant-ID: public
Content-Type: application/json
```

**Response:** (200 OK)
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "John Doe"
  },
  "tenants": [
    {
      "id": "tenant-uuid-1",
      "name": "Company A",
      "slug": "company-a",
      "plan": "plan2",
      "role": "admin",
      "permissions": ["users.read", "users.create", "products.read", ...]
    }
  ]
}
```

#### POST /api/v1/auth/refresh

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:** (200 OK)
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."  // Rotación de refresh token
}
```

#### POST /api/v1/auth/logout

**Headers:**
```http
Authorization: Bearer {accessToken}
X-Tenant-ID: {tenantId}
```

**Response:** (200 OK)
```json
{
  "message": "Logged out successfully"
}
```

---

## 🔄 Versionado de API

### Estrategia de Versionado

**Método:** URL-based versioning

```
https://api.bmoi.com/api/v1/users
https://api.bmoi.com/api/v2/users  (futura)
```

### Política de Versiones

- **Versión Actual:** `v1`
- **Deprecación:** Versión `v(n-1)` se mantiene **6 meses** después de lanzar `v(n)`
- **Breaking Changes:** Solo en nuevas versiones mayores
- **Non-Breaking Changes:** Se añaden a versión actual

### Headers de Versión (Futuro)

```http
Accept: application/vnd.bmoi.v1+json
API-Version: v1
```

### Endpoints por Versión

#### v1 (Actual)

```
GET    /api/v1/tenants
GET    /api/v1/users
POST   /api/v1/users
GET    /api/v1/products
POST   /api/v1/products
GET    /api/v1/orders
POST   /api/v1/orders
```

---

## 🔁 Flujos End-to-End

### 1. Registro de Usuario desde Landing Page

```
┌────────────┐         ┌────────────┐         ┌────────────┐
│  Landing   │────────>│  Backend   │────────>│ PostgreSQL │
│   (Astro)  │         │  (NestJS)  │         │    +RLS    │
└────────────┘         └────────────┘         └────────────┘

1. Usuario llena formulario de registro
2. Landing: POST /api/v1/auth/register
   Headers: { X-Tenant-ID: "public" }
   Body: { email, password, name, companyName, plan: "free" }
3. Backend:
   - Valida datos
   - Crea tenant en DB
   - Crea usuario con rol "owner"
   - Asocia usuario a tenant
4. Backend responde con datos de usuario + tenant
5. Landing: Redirige a CPanel con mensaje de éxito
6. CPanel: Muestra login
```

### 2. Login Multi-Tenant desde CPanel

```
┌────────────┐         ┌────────────┐         ┌────────────┐
│   CPanel   │────────>│  Backend   │────────>│ PostgreSQL │
│  (React)   │         │  (NestJS)  │         │    +RLS    │
└────────────┘         └────────────┘         └────────────┘

1. Usuario ingresa email + password
2. CPanel: POST /api/v1/auth/login
   Headers: { X-Tenant-ID: "public" }
   Body: { email, password }
3. Backend:
   - Valida credenciales
   - Busca tenants del usuario
   - Genera JWT con RS256
   - Retorna accessToken + refreshToken + tenants[]
4. CPanel:
   - Guarda tokens en localStorage
   - if (tenants.length > 1):
       Muestra TenantSelector
     else:
       Selecciona automáticamente único tenant
5. CPanel: Redirige a dashboard del tenant seleccionado
```

### 3. Consulta de Productos desde CPanel

```
┌────────────┐         ┌────────────┐         ┌────────────┐
│   CPanel   │────────>│  Backend   │────────>│ PostgreSQL │
│  (React)   │         │  (NestJS)  │         │    +RLS    │
└────────────┘         └────────────┘         └────────────┘

1. Usuario navega a sección "Productos"
2. CPanel: GET /api/v1/products?page=1&limit=20
   Headers: {
     Authorization: Bearer {accessToken},
     X-Tenant-ID: {currentTenantId}
   }
3. Backend:
   - JwtAuthGuard valida token
   - TenantGuard valida X-Tenant-ID
   - ProductsService:
       await prisma.$executeRaw`
         SELECT set_config('app.current_tenant_id', ${tenantId}, true)
       `;
       return prisma.product.findMany();  // RLS filtra automáticamente
4. PostgreSQL RLS aplica:
   WHERE tenant_id = current_setting('app.current_tenant_id')::UUID
5. Backend retorna solo productos del tenant
6. CPanel: Renderiza tabla de productos
```

### 4. Creación de Producto desde Mobile App

```
┌────────────┐         ┌────────────┐         ┌────────────┐
│   Mobile   │────────>│  Backend   │────────>│ PostgreSQL │
│ (Flutter)  │         │  (NestJS)  │         │    +RLS    │
└────────────┘         └────────────┘         └────────────┘

1. Usuario llena formulario de producto
2. Mobile: POST /api/v1/products
   Headers: {
     Authorization: Bearer {jwtToken},
     X-Tenant-ID: {currentTenantId}
   }
   Body: { name, price, description, ... }
3. Backend:
   - JwtAuthGuard valida token
   - TenantGuard valida X-Tenant-ID y permisos
   - @CheckPlanLimit('products') verifica límites del plan
   - ProductsService:
       await prisma.$executeRaw`
         SELECT set_config('app.current_tenant_id', ${tenantId}, true)
       `;
       return prisma.product.create({
         data: {
           tenant_id: tenantId,  // EXPLÍCITO
           ...productData
         }
       });
4. Backend retorna producto creado
5. Mobile: Actualiza caché local (Hive)
6. Mobile: Muestra mensaje de éxito
```

---

## 🌍 Convenciones por Lenguaje

### Por qué Diferentes Convenciones

Cada framework/lenguaje tiene sus propias convenciones oficiales. **Aceptamos estas diferencias** siempre que se documenten claramente.

#### TypeScript (Backend NestJS)

```typescript
// Archivos
users.controller.ts         // kebab-case
create-user.dto.ts           // kebab-case

// Clases
export class UsersController  // PascalCase
export class CreateUserDto    // PascalCase

// Métodos/Variables
async findAll()              // camelCase
const currentTenant          // camelCase

// Constantes
const JWT_SECRET             // UPPER_SNAKE_CASE
const MAX_RETRIES            // UPPER_SNAKE_CASE
```

#### TypeScript (Frontend React)

```typescript
// Archivos componentes
UserProfile.tsx              // PascalCase
TenantSelector.tsx           // PascalCase

// Archivos utilities
formatDate.ts                // camelCase
apiClient.ts                 // camelCase

// Componentes
export const UserProfile     // PascalCase

// Variables/Funciones
const userName               // camelCase
function fetchData()         // camelCase

// Constantes
const API_BASE_URL           // UPPER_SNAKE_CASE
```

#### TypeScript (Astro)

```typescript
// Archivos componentes
Header.astro                 // PascalCase
Hero.astro                   // PascalCase

// Archivos páginas
index.astro                  // kebab-case
pricing.astro                // kebab-case

// Constantes (Astro convention)
const apiUrl                 // camelCase ← DIFERENTE (Astro docs usan esto)
const siteTitle              // camelCase
```

#### Dart (Flutter)

```dart
// Archivos
login_screen.dart            // snake_case (Dart Style Guide)
product_model.dart           // snake_case

// Clases
class LoginScreen            // PascalCase
class ProductModel           // PascalCase

// Variables/Funciones
String userName              // lowerCamelCase
void fetchProducts()         // lowerCamelCase

// Constantes (Dart convention)
const String apiUrl          // lowerCamelCase ← DIFERENTE (Dart Style Guide)
const int maxRetries         // lowerCamelCase
```

### Tabla Comparativa

| Elemento | Backend (NestJS) | Frontend (React) | Landing (Astro) | Mobile (Dart) |
|----------|------------------|------------------|-----------------|---------------|
| **Archivos** | kebab-case | PascalCase (comp) / camelCase (utils) | PascalCase (comp) / kebab-case (pages) | snake_case |
| **Clases** | PascalCase | PascalCase | PascalCase | PascalCase |
| **Variables** | camelCase | camelCase | camelCase | lowerCamelCase |
| **Constantes** | UPPER_SNAKE_CASE | UPPER_SNAKE_CASE | camelCase | lowerCamelCase |

**Conclusión:** Estas diferencias son **intencionales** y siguen los style guides oficiales de cada tecnología.

---

## 🏗️ Arquitectura de Comunicación

### Arquitectura Actual (Simplificada)

```
┌──────────────────────────────────────────────────────────────┐
│                         FRONTENDS                             │
├──────────────┬──────────────┬──────────────┬─────────────────┤
│ Landing Page │  Web Panel   │  Mobile App  │  Admin Panel    │
│   (Astro)    │   (React)    │  (Flutter)   │  (React?)       │
└──────┬───────┴──────┬───────┴──────┬───────┴──────┬──────────┘
       │              │              │              │
       │              │              │              │
       │     POST /auth/register     │              │
       │     POST /contact           │              │
       │              │              │              │
       └──────────────┴──────────────┴──────────────┘
                      │
                      ▼
       ┌──────────────────────────────────────────┐
       │         Backend API (NestJS)              │
       │  ┌────────────────────────────────────┐  │
       │  │  Guards & Interceptors              │  │
       │  │  - JwtAuthGuard                     │  │
       │  │  - TenantGuard                      │  │
       │  │  - TenantInterceptor                │  │
       │  └────────────────────────────────────┘  │
       │                                           │
       │  Modules:                                 │
       │  - Auth  (login, register, refresh)       │
       │  - Users (CRUD multi-tenant)              │
       │  - Products (CRUD multi-tenant)           │
       │  - Orders (CRUD multi-tenant)             │
       │  - Tenants (management)                   │
       └─────────────────┬─────────────────────────┘
                         │
                         ▼
       ┌──────────────────────────────────────────┐
       │        PostgreSQL 15 + RLS               │
       │  ┌────────────────────────────────────┐  │
       │  │  Row-Level Security Policies        │  │
       │  │  WHERE tenant_id =                  │  │
       │  │    current_setting(                 │  │
       │  │      'app.current_tenant_id'        │  │
       │  │    )::UUID                          │  │
       │  └────────────────────────────────────┘  │
       └──────────────────────────────────────────┘
                         │
                         ▼
       ┌──────────────────────────────────────────┐
       │             Redis (Cache)                 │
       │  - Sessions                               │
       │  - Cache de queries                       │
       │  - Rate limiting                          │
       └──────────────────────────────────────────┘
```

### Arquitectura Futura (Con API Gateway / BFF)

```
┌──────────────────────────────────────────────────────────────┐
│                         FRONTENDS                             │
├──────────────┬──────────────┬──────────────┬─────────────────┤
│ Landing Page │  Web Panel   │  Mobile App  │  Admin Panel    │
└──────┬───────┴──────┬───────┴──────┬───────┴──────┬──────────┘
       │              │              │              │
       └──────────────┴──────────────┴──────────────┘
                      │
                      ▼
       ┌──────────────────────────────────────────┐
       │     API Gateway (Nginx/Envoy)            │
       │  - Rate Limiting                         │
       │  - CORS                                  │
       │  - SSL Termination                       │
       │  - Request Routing                       │
       └─────────────────┬────────────────────────┘
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼
┌──────────────────────┐   ┌──────────────────────┐
│ BFF Web/Mobile       │   │  Auth Service        │
│ (NestJS/Go)          │   │  (NodeGo)            │
│ - Aggregation        │   └──────────────────────┘
│ - Transformation     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────────────────────────┐
│           Domain Services (NodeGo)                │
├──────────────┬──────────────┬────────────────────┤
│ Products     │  Orders      │  Payments          │
└──────────────┴──────────────┴────────────────────┘
```

**Decisión Pendiente:** Definir si se implementará API Gateway / BFF o conexión directa.

---

## 🔐 Seguridad

### Headers de Seguridad

Todos los endpoints deben retornar:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
```

### Rate Limiting

```
/api/v1/auth/login     → 5 requests / minuto por IP
/api/v1/auth/register  → 3 requests / hora por IP
/api/v1/*              → 100 requests / minuto por usuario autenticado
```

### CORS

**Orígenes Permitidos (Producción):**
```
https://bmoi.com
https://cpanel.bmoi.com
```

**Orígenes Permitidos (Desarrollo):**
```
http://localhost:3001
http://localhost:4321
http://localhost:*
```

---

## 📱 Manejo de Errores

### Formato de Error Estándar

Todos los endpoints deben retornar errores en este formato:

```json
{
  "statusCode": 400,
  "message": "Validation failed",
  "error": "Bad Request",
  "details": [
    {
      "field": "email",
      "message": "Email is already registered"
    }
  ],
  "timestamp": "2025-01-04T10:30:00.000Z",
  "path": "/api/v1/auth/register"
}
```

### Códigos de Estado HTTP

| Código | Significado | Uso |
|--------|-------------|-----|
| 200 | OK | Request exitoso |
| 201 | Created | Recurso creado |
| 400 | Bad Request | Validación falló |
| 401 | Unauthorized | Token inválido/expirado |
| 403 | Forbidden | Sin permisos para el recurso |
| 404 | Not Found | Recurso no existe |
| 409 | Conflict | Conflicto (ej: email duplicado) |
| 422 | Unprocessable Entity | Datos inválidos |
| 429 | Too Many Requests | Rate limit excedido |
| 500 | Internal Server Error | Error del servidor |

---

## 🧪 Testing de Integración

### Variables de Entorno para Tests

```bash
# .env.test
API_BASE_URL=http://localhost:3000/api/v1
TEST_USER_EMAIL=test@bmoi.com
TEST_USER_PASSWORD=TestPassword123!
TEST_TENANT_ID=test-tenant-uuid
```

### Ejemplo de Test E2E (Playwright)

```typescript
// tests/e2e/auth-flow.spec.ts
import { test, expect } from '@playwright/test';

test('complete auth flow', async ({ page }) => {
  // 1. Login
  await page.goto('http://localhost:3001/login');
  await page.fill('[name="email"]', 'test@bmoi.com');
  await page.fill('[name="password"]', 'TestPassword123!');
  await page.click('button[type="submit"]');

  // 2. Verificar redirección a selector de tenant (si tiene múltiples)
  await expect(page).toHaveURL(/\/select-tenant|\/dashboard/);

  // 3. Si tiene múltiples tenants, seleccionar uno
  if (await page.locator('[data-testid="tenant-selector"]').isVisible()) {
    await page.click('[data-testid="tenant-card"]:first-child');
  }

  // 4. Verificar llegó al dashboard
  await expect(page).toHaveURL(/\/dashboard/);

  // 5. Verificar que el header X-Tenant-ID se envía
  page.on('request', request => {
    if (request.url().includes('/api/v1/')) {
      expect(request.headers()['x-tenant-id']).toBeTruthy();
      expect(request.headers()['authorization']).toContain('Bearer ');
    }
  });
});
```

---

## 📚 Recursos Adicionales

- **Plantillas Backend:** `plantillas_backend/`
- **Plantillas CPanel:** `plantillas_cpanel-multitenant/`
- **Plantillas Landing:** `plantillas_landing/`
- **Plantillas Mobile:** `plantillas_mobile/`
- **Análisis Completo:** `ANALISIS_PLANTILLAS_BMOI.md`

---

**Mantenido por:** Equipo bMOI
**Última actualización:** 2025-01-04
**Versión:** 1.0.0
