# 🧠 Context - Admin Web Frontend

> **Contexto completo del proyecto Admin Web para Claude Code**

---

## 📌 Información Esencial del Proyecto

### Nombre del Proyecto
**bMOI Admin Web** - Panel de administración multi-tenant

### Propósito
Aplicación web SPA (Single Page Application) que permite a los tenants del ecosistema bMOI gestionar:
- Usuarios y permisos (RBAC)
- Productos, categorías y variantes
- Órdenes de compra y facturación
- Dashboard con métricas en tiempo real
- Configuración del tenant
- Auditoría y logs de actividad

### Posición en el Ecosistema

```
┌─────────────────────────────────────────────────────────────┐
│                     ECOSISTEMA bMOI                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐ │
│  │  Admin Web   │───▶│ API Gateway  │───▶│  Services    │ │
│  │  (React SPA) │    │  (NestJS)    │    │  (Backend)   │ │
│  │  Puerto 5173 │    │  Puerto 3000 │    │              │ │
│  └──────────────┘    └──────────────┘    └──────────────┘ │
│         │                    │                    │         │
│         │                    │                    │         │
│         └────────────────────┴────────────────────┘         │
│                              │                               │
│                    ┌─────────▼─────────┐                   │
│                    │   PostgreSQL 15   │                   │
│                    │   + Redis + MinIO │                   │
│                    └───────────────────┘                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Estado Actual del Proyecto
**Fase:** Fase 0 - Setup Frontend (0% completado)

El proyecto inicia **desde cero**. No existe código previo.

---

## 🛠️ Stack Tecnológico

### Frontend Core
- **Framework:** React 18.3+ con Concurrent Features
- **Lenguaje:** TypeScript 5.6+ (modo strict)
- **Build Tool:** Vite 5+ (desarrollo y producción)
- **Package Manager:** npm (no yarn, no pnpm)

### Routing & Navigation
- **Router:** React Router 6.x con Data APIs (loaders, actions)
- **Navegación:** SPA con lazy loading de rutas

### State Management
- **Client State:** Zustand 4+ con persist middleware
- **Server State:** TanStack Query (React Query) 5+
- **Forms:** React Hook Form 7+ con @hookform/resolvers

### UI & Styling
- **Framework CSS:** Tailwind CSS 3.4+
- **Componentes:** Shadcn/UI (copy-paste approach)
- **Primitivos:** Radix UI (accesibilidad WCAG AA)
- **Icons:** Lucide React

### Validación
- **Schema Validation:** Zod 3+ (frontend y backend compatible)
- **Form Validation:** Integrado con React Hook Form

### HTTP Client
- **Cliente:** Axios 1.7+ con interceptors
- **Base URL:** `http://localhost:3000/api/v1` (desarrollo)
- **Headers obligatorios:**
  - `Authorization: Bearer {token}`
  - `X-Tenant-ID: {tenantId}`

### Testing
- **Test Runner:** Vitest (compatible con Vite)
- **Component Testing:** React Testing Library
- **API Mocking:** MSW (Mock Service Worker)
- **E2E:** Playwright (opcional, Fase 4)

### Deployment
- **Contenedor:** Docker multi-stage
- **Web Server:** Nginx con configuración SPA
- **CI/CD:** GitHub Actions

---

## 📂 Estructura del Proyecto

```
cpanel-multitenant-bo/  (nombre sugerido del repositorio)
├── .claude/
│   ├── context.md           ← Este archivo
│   ├── conventions.md       ← Convenciones obligatorias
│   └── session-state.md     ← Estado actual del desarrollo
│
├── .copilot/
│   └── instructions.md      ← Instrucciones para GitHub Copilot
│
├── .docs/
│   ├── README.md            ← Overview del proyecto
│   ├── INDICE.md            ← Navegación de documentación
│   ├── SETUP_INICIAL.md     ← Setup paso a paso
│   ├── QUICKSTART.md        ← Setup rápido (5 min)
│   ├── START.md             ← Primeros pasos después del setup
│   ├── ARQUITECTURA.md      ← Decisiones arquitectónicas
│   ├── CODING_STANDARDS.md  ← Estándares de código
│   ├── EJEMPLOS_CODIGO.md   ← Ejemplos completos
│   └── CONFIGURACIONES.md   ← Archivos de configuración
│
├── public/
│   ├── favicon.ico
│   └── logo.svg
│
├── src/
│   ├── assets/              # Imágenes, fonts
│   ├── components/          # Componentes React
│   │   ├── ui/              # Shadcn/UI components
│   │   ├── atoms/           # Componentes básicos
│   │   ├── molecules/       # Agrupaciones simples
│   │   └── organisms/       # Componentes complejos
│   ├── pages/               # Páginas enrutables
│   ├── layouts/             # Layouts reutilizables
│   ├── hooks/               # Custom hooks
│   │   ├── queries/         # React Query hooks
│   │   └── mutations/       # React Query mutations
│   ├── services/            # API services
│   ├── stores/              # Zustand stores
│   ├── types/               # TypeScript types
│   ├── lib/                 # Utils y helpers
│   ├── router/              # React Router config
│   ├── styles/              # CSS global
│   ├── App.tsx              # Componente raíz
│   └── main.tsx             # Entry point
│
├── .cursorrules             ← Reglas para Cursor AI
├── .dockerignore
├── .env.example
├── .eslintrc.cjs
├── .gitignore
├── .prettierrc
├── CLAUDE.md                ← Guía rápida para Claude
├── docker-compose.yml       ← Desarrollo con Docker
├── Dockerfile               ← Producción multi-stage
├── index.html
├── nginx.conf               ← Nginx para producción
├── package.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
└── vitest.config.ts
```

---

## 🔑 Conceptos Clave

### 1. Multi-Tenancy

**Crítico:** Toda la aplicación está diseñada para multi-tenant.

**Flujo de Tenant:**
```
1. User se loguea
2. Backend retorna lista de tenants accesibles
3. User selecciona un tenant
4. Frontend guarda tenant_id en localStorage
5. TODAS las requests incluyen header X-Tenant-ID
6. Backend valida acceso y aplica RLS
```

**Implementación:**
```typescript
// Axios interceptor (OBLIGATORIO)
apiClient.interceptors.request.use((config) => {
  const tenantId = localStorage.getItem('selected_tenant_id');
  if (tenantId && config.headers) {
    config.headers['X-Tenant-ID'] = tenantId;
  }
  return config;
});
```

### 2. Autenticación JWT

**Flow completo:**
```
┌──────────┐                ┌─────────┐                ┌──────────┐
│  Login   │─── POST ──────▶│   API   │───── 200 ─────▶│ Response │
│  Form    │  credentials   │ Gateway │   tokens+user  │          │
└──────────┘                └─────────┘                └──────────┘
                                  │
                                  ▼
                    Save to localStorage:
                    - access_token (JWT)
                    - refresh_token
                    - user object
                                  │
                                  ▼
                    Update Zustand Auth Store
                                  │
                                  ▼
                    Redirect to /dashboard
                                  │
                                  ▼
            ┌───────────────────────────────────────┐
            │   All subsequent requests include:    │
            │   Authorization: Bearer {access_token}│
            │   X-Tenant-ID: {selected_tenant_id}   │
            └───────────────────────────────────────┘
```

**Refresh Token (automático en 401):**
```typescript
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401 && !error.config._retry) {
      error.config._retry = true;
      const newAccessToken = await authService.refreshToken();
      error.config.headers.Authorization = `Bearer ${newAccessToken}`;
      return apiClient(error.config);
    }
    return Promise.reject(error);
  }
);
```

### 3. RBAC (Role-Based Access Control)

**4 niveles de permisos:**
1. **SUPER_ADMIN:** Acceso total multi-tenant
2. **ADMIN:** Administrador del tenant
3. **USER:** Usuario con permisos limitados
4. **GUEST:** Solo lectura

**Sistema de permisos granular:**
```typescript
const permissions = [
  'users.read',
  'users.create',
  'users.update',
  'users.delete',
  'products.read',
  'products.create',
  // ... etc
];

// Verificación en componentes
const { hasPermission } = usePermissions();

{hasPermission('users.create') && (
  <Button onClick={handleCreate}>Create User</Button>
)}

// Verificación en rutas
loader: protectedLoaderWithPermission('users.read')
```

### 4. Plan-Based Limits

**4 planes con límites:**
- **Free:** 10 users, 50 products, 1GB storage
- **Plan1:** 50 users, 500 products, 10GB storage
- **Plan2:** 200 users, 2000 products, 50GB storage
- **Plan3:** Unlimited users/products, 500GB storage

**El frontend debe:**
- Mostrar límites actuales en dashboard
- Advertir cuando se acerque al límite (80%)
- Prevenir acciones cuando se alcance el límite
- Sugerir upgrade de plan

**Nota:** La validación real es en el backend, frontend solo UX.

---

## 🎨 Decisiones de Diseño

### React 18 ✅ (no Vue, no Angular)

**Por qué:**
- Ecosistema más grande
- Mejor soporte TypeScript
- Concurrent features para mejor UX
- Preparado para Server Components (futuro)

### Vite ✅ (no Webpack, no CRA)

**Por qué:**
- Dev server ultra-rápido (< 1s start)
- HMR instantáneo (< 50ms)
- Build optimizado con Rollup
- TypeScript nativo sin configuración

### Zustand ✅ (no Redux, no Context API pura)

**Por qué:**
- Minimalista: 1KB vs 3KB Redux
- Sin boilerplate (no actions/reducers)
- TypeScript first-class
- Persist middleware built-in

### React Query ✅ (no SWR, no fetch manual)

**Por qué:**
- Caching automático inteligente
- Background refetch
- Optimistic updates
- Retry logic built-in
- DevTools excelentes

### Shadcn/UI ✅ (no MUI, no Ant Design)

**Por qué:**
- Copy-paste approach (código en tu proyecto)
- Totalmente customizable
- Tailwind-first (no CSS-in-JS runtime)
- Radix UI (accesibilidad WCAG AA)
- Bundle size óptimo

### Docker ✅ (Desarrollo y Producción)

**Por qué:**
- Consistencia dev/staging/prod
- Onboarding rápido (5 min)
- Nginx configurado y optimizado
- Multi-stage build (imagen ligera)

---

## 🔒 Seguridad

### Headers de Seguridad (Nginx)

```nginx
add_header Content-Security-Policy "default-src 'self'; ...";
add_header X-Content-Type-Options "nosniff";
add_header X-Frame-Options "DENY";
add_header X-XSS-Protection "1; mode=block";
add_header Referrer-Policy "strict-origin-when-cross-origin";
```

### Validación de Inputs

**SIEMPRE usar Zod:**
```typescript
const createUserSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).regex(/[A-Z]/).regex(/[0-9]/),
  name: z.string().min(3),
});
```

### XSS Protection

- React escapa por defecto (no usar `dangerouslySetInnerHTML`)
- Validar con Zod antes de enviar al backend
- CSP headers en Nginx

### CSRF Protection

- No necesario (JWT en header, no cookies)
- Backend valida token en cada request

---

## ⚡ Performance

### Code Splitting

```typescript
// Lazy loading de rutas
const DashboardPage = lazy(() => import('@/pages/dashboard/DashboardPage'));
const UsersPage = lazy(() => import('@/pages/users/UsersPage'));
```

### Bundle Size Optimization

```typescript
// vite.config.ts - Manual chunks
manualChunks: {
  'react-vendor': ['react', 'react-dom', 'react-router-dom'],
  'ui-vendor': ['@radix-ui/...'],
  'form-vendor': ['react-hook-form', 'zod'],
  'query-vendor': ['@tanstack/react-query'],
}
```

**Target:**
- Initial bundle < 500KB
- Total < 2MB
- First Contentful Paint < 1.5s
- Time to Interactive < 3s

### React Query Caching

```typescript
queryClient.setDefaultOptions({
  queries: {
    staleTime: 5 * 60 * 1000, // 5 min
    gcTime: 10 * 60 * 1000,   // 10 min
    retry: 3,
  },
});
```

### Image Optimization

- Lazy loading: `<img loading="lazy" />`
- Responsive: `<picture>` con srcset
- Formatos modernos: WebP, AVIF

---

## 🧪 Testing

### Test Pyramid

```
         ┌─────────┐
         │   E2E   │  10% (Critical flows)
         ├─────────┤
         │  Integ  │  30% (Components + API)
         ├─────────┤
         │  Unit   │  60% (Utils, hooks, logic)
         └─────────┘
```

**Cobertura mínima:** 80%

### Ejemplo Unit Test

```typescript
describe('formatDate', () => {
  it('should format date correctly', () => {
    const date = new Date('2024-01-15');
    expect(formatDate(date)).toBe('15/01/2024');
  });
});
```

### Ejemplo Component Test

```typescript
describe('UserCard', () => {
  it('should call onEdit when button clicked', async () => {
    const onEdit = vi.fn();
    render(<UserCard user={mockUser} onEdit={onEdit} />);

    await user.click(screen.getByRole('button', { name: /edit/i }));

    expect(onEdit).toHaveBeenCalledWith(mockUser);
  });
});
```

---

## 📋 Convenciones OBLIGATORIAS

### Nomenclatura

```typescript
// Componentes: PascalCase
UserCard.tsx
DashboardLayout.tsx

// Hooks: camelCase + prefijo "use"
use-users.ts
use-pagination.ts

// Services: kebab-case + sufijo ".service"
users.service.ts
auth.service.ts

// Utils: kebab-case
date-utils.ts
string-utils.ts

// Types: PascalCase
User, UserFilters, CreateUserDto

// Variables: camelCase
const userName = 'John';
const isAuthenticated = true;

// Constantes: UPPER_SNAKE_CASE
const MAX_USERS = 100;
const API_BASE_URL = 'http://...';
```

### Imports Order

```typescript
// 1. React y librerías externas
import React, { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

// 2. Componentes internos
import { Button } from '@/components/ui/button';

// 3. Hooks
import { useAuth } from '@/hooks/use-auth';

// 4. Services y utils
import { usersService } from '@/services/users.service';

// 5. Types
import type { User } from '@/types/user.types';

// 6. Styles
import styles from './Component.module.css';
```

### Reglas de Oro

1. **NUNCA usar `any`** → Usar tipos específicos o `unknown`
2. **SIEMPRE tipar props** → Interfaces/types para componentes
3. **SIEMPRE cleanup en useEffect** → Evitar memory leaks
4. **SIEMPRE validar permisos** → Frontend (UX) + Backend (seguridad)
5. **NUNCA inline functions en JSX** → Declarar fuera (performance)
6. **SIEMPRE usar keys en listas** → Unique y stable
7. **NUNCA console.log en producción** → Eliminar antes de commit
8. **SIEMPRE handle loading/error states** → Skeleton, ErrorBoundary

---

## 🐳 Docker (OBLIGATORIO)

### Por qué Docker es Mandatorio

1. **Consistencia:** Mismo ambiente dev/staging/prod
2. **Onboarding:** Nuevo dev productivo en 5 min
3. **Nginx optimizado:** Configuración de producción lista
4. **No contaminar:** Dependencias aisladas
5. **CI/CD ready:** Mismo contenedor en todos los ambientes

### Comandos Docker

```bash
# Development (Vite dev server)
docker compose up -d

# View logs
docker compose logs -f frontend

# Install dependencies
docker compose exec frontend npm install [package]

# Run tests
docker compose exec frontend npm run test

# Stop
docker compose down

# Production build
docker build -t bmoi-admin-web:latest .
docker run -p 80:80 bmoi-admin-web:latest
```

### NO correr comandos directamente

```bash
# ❌ NUNCA (corre en tu máquina):
npm install
npm run dev
npm run build

# ✅ SIEMPRE (corre en Docker):
docker compose up -d
docker compose exec frontend npm install
```

---

## 🚀 Fases de Desarrollo

### Fase 0: Setup Frontend (0% - ACTUAL)

**Objetivo:** Crear proyecto base funcional

**Tareas:**
1. ✅ Inicializar proyecto Vite + React + TypeScript
2. ✅ Configurar Tailwind CSS + Shadcn/UI
3. ✅ Configurar ESLint + Prettier
4. ✅ Configurar Docker (development + production)
5. ✅ Configurar React Router
6. ✅ Crear estructura de carpetas
7. ✅ Configurar Axios con interceptors
8. ✅ Crear primer componente de prueba
9. ✅ Verificar hot-reload funciona
10. ✅ Documentar setup en README

**Entregable:** Proyecto React funcionando en Docker con hot-reload

---

### Fase 1: Auth & Layout (0%)

**Objetivo:** Sistema de autenticación completo

**Tareas:**
1. Crear LoginPage con formulario
2. Implementar Auth Service (login, logout, refresh)
3. Crear Auth Store (Zustand)
4. Implementar JWT interceptors
5. Crear ProtectedRoute component
6. Crear DashboardLayout (Sidebar + Header)
7. Implementar TenantSelector
8. Testing de flujo de login completo

**Entregable:** Login funcional con redirect a dashboard vacío

---

### Fase 2: Core Modules (0%)

**Objetivo:** Módulos principales del backoffice

**Módulos a implementar:**
1. **Dashboard:** Métricas y gráficos
2. **Users:** CRUD completo con permisos
3. **Products:** Gestión de productos y categorías
4. **Orders:** Visualización y gestión de órdenes

**Por cada módulo:**
- Services (API calls)
- Hooks (queries + mutations)
- Pages (List, Detail, Create, Edit)
- Forms con validación Zod
- Tests (unit + integration)

**Entregable:** Backoffice funcional con CRUD de users y products

---

### Fase 3: Advanced Features (0%)

**Objetivo:** Features avanzadas y UX

**Features:**
- Audit logs visualization
- Dashboard con gráficos (Chart.js o Recharts)
- Export data (CSV, Excel, PDF)
- Bulk actions
- Advanced filters
- Real-time notifications
- Theme switcher (light/dark)

**Entregable:** Backoffice con features avanzadas

---

### Fase 4: Testing & Optimization (0%)

**Objetivo:** Calidad y performance

**Tareas:**
1. Alcanzar 80% test coverage
2. E2E tests con Playwright
3. Performance optimization
4. Bundle size analysis
5. Accessibility audit (WCAG AA)
6. SEO optimization
7. Lighthouse score > 90

**Entregable:** Aplicación optimizada y testeada

---

## 📚 Documentación Crítica

### Archivos que DEBES leer primero:

1. **Este archivo (context.md)** - Contexto completo ✅
2. **.claude/conventions.md** - Convenciones obligatorias
3. **.docs/CODING_STANDARDS.md** - Estándares de código
4. **.docs/SETUP_INICIAL.md** - Setup paso a paso
5. **.docs/ARQUITECTURA.md** - Decisiones arquitectónicas

### Archivos de referencia:

- **.docs/EJEMPLOS_CODIGO.md** - Código listo para copiar
- **.docs/CONFIGURACIONES.md** - Todos los archivos de config
- **CLAUDE.md** (raíz) - Guía rápida de referencia

---

## 🧠 Anti-Forgetting Checkpoints

### Cada 5 mensajes, verifica mentalmente:

- [ ] Proyecto: **bMOI Admin Web** (React SPA)
- [ ] Stack: **React 18 + TypeScript 5 + Vite + Tailwind**
- [ ] State: **Zustand** (client) + **React Query** (server)
- [ ] Multi-tenant: **X-Tenant-ID header OBLIGATORIO**
- [ ] Auth: **JWT con refresh token automático**
- [ ] Convenciones: **PascalCase** (componentes), **camelCase** (variables)
- [ ] NUNCA usar `any` en TypeScript
- [ ] SIEMPRE tipar props de componentes
- [ ] SIEMPRE incluir X-Tenant-ID en requests
- [ ] SIEMPRE usar Zod para validación
- [ ] SIEMPRE handle loading/error states

### Al finalizar sesión, actualizar:

`.claude/session-state.md` con:
- Componentes creados hoy
- Features implementadas
- Decisiones técnicas tomadas
- Problemas encontrados y soluciones
- Próximos pasos
- Progreso de la fase actual

---

## 🔗 Integración con Backend

### API Gateway (NestJS)

**Base URL:** `http://localhost:3000/api/v1`

**Endpoints principales:**

```
POST   /auth/login
POST   /auth/refresh
POST   /auth/logout
GET    /auth/me

GET    /users
GET    /users/:id
POST   /users
PATCH  /users/:id
DELETE /users/:id

GET    /products
GET    /products/:id
POST   /products
PATCH  /products/:id
DELETE /products/:id

GET    /orders
GET    /orders/:id
POST   /orders
PATCH  /orders/:id

GET    /dashboard/metrics
GET    /audit-logs
```

**Headers obligatorios:**
```typescript
{
  'Authorization': 'Bearer {access_token}',
  'X-Tenant-ID': '{selected_tenant_id}',
  'Content-Type': 'application/json'
}
```

**Responses estándar:**

```typescript
// Success
{
  data: T,
  meta?: {
    total: number,
    page: number,
    limit: number
  }
}

// Error
{
  statusCode: number,
  message: string | string[],
  error: string
}
```

---

## 💡 Troubleshooting AI

### Si olvido convenciones:

```
🧠 RECORDATORIO - FRONTEND:

Convenciones ESTRICTAS:
- Componentes: PascalCase (UserCard.tsx)
- Hooks: camelCase + "use" (use-users.ts)
- Services: kebab-case + ".service" (users.service.ts)
- Variables: camelCase (userName)
- Constantes: UPPER_SNAKE_CASE (API_URL)
- NUNCA usar "any" en TypeScript

Multi-tenant OBLIGATORIO:
- X-Tenant-ID en TODAS las requests
- Verificar tenant en cada página
- Mostrar tenant actual en Header

Por favor confirma que recordarás esto.
```

### Si creo componente sin tipos:

```
⚠️ ERROR: Falta tipado

TODOS los componentes DEBEN:
1. Tener interface Props definida
2. Tipar el componente correctamente
3. NO usar "any" nunca

Ejemplo:
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}

export const UserCard = ({ user, onEdit }: UserCardProps) => {
  // ...
};

Por favor corrige el código anterior.
```

---

## 📊 Estado Actual

**Fase:** Fase 0 - Setup Frontend
**Progreso:** 0%
**Último update:** 2025-11-04

**Próximo paso inmediato:**
Inicializar proyecto Vite con React + TypeScript + Docker

**Bloqueadores:** Ninguno

---

<div align="center">

**Admin Web Multi-Tenant con React + TypeScript**

Type-safe | Performante | Escalable | Docker-first

Documentación completa en `.docs/`

</div>
