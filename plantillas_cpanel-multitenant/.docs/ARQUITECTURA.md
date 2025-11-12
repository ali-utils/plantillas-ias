# ARQUITECTURA - bMOI CPanel Multi-Tenant BO

> **Decisiones arquitectónicas y diseño técnico del sistema**
> Versión: 2.0 | Última actualización: Noviembre 2025

---

## 📑 Índice

1. [Visión General](#-visión-general)
2. [Arquitectura Multi-Tenant](#-arquitectura-multi-tenant)
3. [Stack Tecnológico](#-stack-tecnológico)
4. [Patrones de Diseño](#-patrones-de-diseño)
5. [Gestión de Estado](#-gestión-de-estado)
6. [Seguridad](#-seguridad)
7. [Performance](#-performance)
8. [Escalabilidad](#-escalabilidad)

---

## 🏛️ Visión General

### Principios Arquitectónicos

```
1. Multi-Tenant First: Todo diseño considera múltiples tenants
2. Type Safety: TypeScript estricto, cero `any`
3. Separation of Concerns: Responsabilidades bien definidas
4. DRY (Don't Repeat Yourself): Componentes reutilizables
5. SOLID Principles: Código mantenible y extensible
6. Security by Default: Seguridad no es opcional
```

### Diagrama de Alto Nivel

```
┌─────────────────────────────────────────────────────────────┐
│                         FRONTEND                             │
│                    (React + TypeScript)                      │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   UI Layer   │  │  Logic Layer │  │  Data Layer  │      │
│  │              │  │              │  │              │      │
│  │ Components   │→ │ Hooks        │→ │ Services     │      │
│  │ Pages        │  │ Context      │  │ API Client   │      │
│  │ Templates    │  │ Store        │  │ React Query  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                    TENANT ISOLATION                          │
│         Context API + Header Interceptors                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│                    API REST (Backend)                        │
│              https://api.controlpanel.com/v1                 │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 🏢 Arquitectura Multi-Tenant

### Estrategia: Single Instance, Data Isolation

```
Frontend Único
    ↓
Contexto de Tenant (React Context)
    ↓
API Client (Axios Interceptors)
    ↓
Header: X-Tenant-ID
    ↓
Backend (Valida y filtra por tenant)
    ↓
Database (Datos separados por tenant_id)
```

### Flujo de Autenticación Multi-Tenant

```typescript
/**
 * FLUJO COMPLETO
 *
 * 1. Usuario ingresa credenciales (email + password)
 * 2. Backend valida y retorna JWT con lista de tenants
 * 3. Frontend muestra selector de tenant (si tiene múltiples)
 * 4. Usuario selecciona tenant activo
 * 5. Frontend guarda tenant en Context + LocalStorage
 * 6. API Client configura header X-Tenant-ID automáticamente
 * 7. Todas las requests subsecuentes incluyen header de tenant
 * 8. Backend valida que el usuario tenga acceso al tenant solicitado
 * 9. Backend filtra todos los datos por tenant_id
 */

// JWT Structure
interface JWTPayload {
  userId: string
  email: string
  tenants: Array<{
    id: string
    name: string
    role: 'admin' | 'manager' | 'user'
    plan: 'free' | 'plan1' | 'plan2' | 'plan3'
  }>
  currentTenantId: string
  exp: number
  iat: number
}

// Request Headers
{
  'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIs...',
  'X-Tenant-ID': 'tenant-abc-123',
  'Content-Type': 'application/json',
  'Accept-Language': 'es-ES'
}
```

### Capas de Validación Multi-Tenant

```
┌────────────────────────────────────────────┐
│ Layer 1: Frontend Route Guards             │
│ ✓ ¿Usuario autenticado?                    │
│ ✓ ¿Tenant seleccionado?                    │
│ ✓ ¿Tiene permisos para esta ruta?          │
└────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────┐
│ Layer 2: Component Level Validation        │
│ ✓ ¿Tiene permisos para ver componente?     │
│ ✓ ¿Está dentro de límites del plan?        │
│ ✓ ¿Datos pertenecen al tenant actual?      │
└────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────┐
│ Layer 3: API Request Interceptor           │
│ ✓ Agregar X-Tenant-ID header               │
│ ✓ Validar que tenant existe en localStorage│
│ ✓ Agregar token de autenticación           │
└────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────┐
│ Layer 4: Backend Validation (API)          │
│ ✓ Validar JWT                              │
│ ✓ Verificar que user pertenece al tenant   │
│ ✓ Filtrar datos por tenant_id              │
│ ✓ Validar límites del plan                 │
└────────────────────────────────────────────┘
```

### Límites por Plan (Configuración)

```typescript
export const PLAN_LIMITS = {
  free: {
    users: 10,
    products: 50,
    orders: 100,
    storageGB: 1,
    apiCallsPerDay: 1000,
    features: [
      'dashboard.basic',
      'users.read',
      'users.create',
      'products.read',
      'products.create'
    ]
  },

  plan1: {
    users: 50,
    products: 500,
    orders: 1000,
    storageGB: 10,
    apiCallsPerDay: 10000,
    features: [
      'dashboard.advanced',
      'users.full',
      'products.full',
      'orders.read',
      'orders.create',
      'reports.basic'
    ]
  },

  plan2: {
    users: 200,
    products: 2000,
    orders: 10000,
    storageGB: 50,
    apiCallsPerDay: 100000,
    features: [
      'dashboard.pro',
      'users.full',
      'products.full',
      'orders.full',
      'reports.advanced',
      'analytics.full',
      'api.access'
    ]
  },

  plan3: {
    users: -1, // Ilimitado
    products: -1,
    orders: -1,
    storageGB: 500,
    apiCallsPerDay: -1,
    features: [
      'all',
      'white-label',
      'custom-domain',
      'priority-support',
      'sla-99.9'
    ]
  }
} as const

// Helper para validar límites
export const checkPlanLimit = (
  plan: PlanType,
  resource: keyof PlanLimits,
  currentCount: number
): boolean => {
  const limit = PLAN_LIMITS[plan][resource]
  if (limit === -1) return true // Ilimitado
  return currentCount < limit
}
```

---

## 💻 Stack Tecnológico

### Decisiones y Justificaciones

#### 1. React 18 (UI Library)

**¿Por qué React?**
```
✅ Ecosistema maduro y robusto
✅ Concurrent rendering (mejor performance)
✅ Server Components (futuro)
✅ Hooks bien establecidos
✅ Gran comunidad y recursos

❌ Alternativas consideradas:
  - Vue 3: Ecosistema más pequeño para enterprise
  - Svelte: Menos librerías multi-tenant
  - Angular: Demasiado opinionado y pesado
```

#### 2. TypeScript 5.3 (Type Safety)

**¿Por qué TypeScript?**
```
✅ Type safety en tiempo de compilación
✅ Mejor DX (autocomplete, refactoring)
✅ Previene bugs en producción
✅ Documentación implícita
✅ Mejor mantenibilidad a largo plazo

❌ JavaScript puro: Muy propenso a errores
```

#### 3. Vite 5 (Build Tool)

**¿Por qué Vite sobre Create React App?**
```
✅ HMR ultra-rápido (< 100ms)
✅ Build optimizado con Rollup
✅ ES Modules nativos
✅ Plugin ecosystem moderno
✅ Mejor performance en desarrollo

❌ CRA: Lento, deprecated, mal mantenido
❌ Webpack: Configuración compleja
```

#### 4. Zustand (Client State)

**¿Por qué Zustand sobre Redux?**
```
✅ API simple (menos boilerplate)
✅ Bundle size: ~1KB vs ~20KB (Redux)
✅ No requiere Provider wrapping
✅ DevTools integrado
✅ Middleware (persist, immer)

Perfecto para:
  - UI state (modals, sidebar)
  - User preferences
  - Temporal data

❌ Redux: Overkill para este proyecto
❌ MobX: Menos predecible
```

#### 5. React Query v5 (Server State)

**¿Por qué React Query?**
```
✅ Cache automático por query key
✅ Invalidación inteligente
✅ Retry y error handling
✅ Optimistic updates
✅ Prefetching y background refetch
✅ Perfect para multi-tenant (cache por tenant)

Perfecto para:
  - API data fetching
  - Cache de datos del servidor
  - Sincronización automática

❌ SWR: Menos features para multi-tenant
❌ Apollo: Requiere GraphQL (no lo usamos)
```

#### 6. Ant Design 5 (Component Library)

**¿Por qué Ant Design?**
```
✅ Componentes enterprise-ready
✅ Table avanzado (sorting, filtering, pagination)
✅ Form system robusto
✅ Tema completamente personalizable
✅ TypeScript nativo
✅ Diseño consistente
✅ Internacionalización built-in

❌ Material-UI: Bundle más pesado
❌ Chakra UI: Menos componentes complejos
❌ Custom: Demasiado tiempo de desarrollo
```

#### 7. Tailwind CSS 3 (Utility CSS)

**¿Por qué Tailwind + Ant Design?**
```
✅ Tailwind: Para layouts y spacing custom
✅ Ant Design: Para componentes complejos
✅ Complementarios, no compiten

Uso:
  - Tailwind: margin, padding, flex, grid
  - Ant Design: Button, Table, Form, Modal
  - CSS Modules: Componentes custom específicos
```

---

## 🎨 Patrones de Diseño

### Atomic Design

```
atoms/
  ├── Button           → Elementos básicos, no divisibles
  ├── Input
  ├── Icon
  └── Avatar

molecules/
  ├── FormField        → Combinación de atoms (Input + Label + Error)
  ├── SearchBox        → Input + Button
  ├── UserCard         → Avatar + Text + Button
  └── TenantBadge      → Icon + Text (con color por plan)

organisms/
  ├── Header           → Navigation + UserMenu + TenantSwitcher
  ├── Sidebar          → Navigation + Logo
  ├── DataTable        → Ant Table + Filters + Pagination
  └── UserForm         → Multiple FormFields + Validation

templates/
  ├── DashboardLayout  → Header + Sidebar + Content + Footer
  ├── AuthLayout       → Centered form + Background
  └── FullPageLayout   → Simple full-page container

pages/
  ├── LoginPage        → AuthLayout + LoginForm
  ├── DashboardPage    → DashboardLayout + Widgets
  └── UsersPage        → DashboardLayout + UsersList
```

### Feature-Based Structure (Pages)

```
pages/
  ├── users/
  │   ├── UsersPage.tsx
  │   ├── components/
  │   │   ├── UsersList.tsx
  │   │   ├── UserFilters.tsx
  │   │   └── UserCreateModal.tsx
  │   ├── hooks/
  │   │   ├── useUsersData.ts
  │   │   └── useUserFilters.ts
  │   └── types/
  │       └── users.types.ts
  │
  └── dashboard/
      ├── DashboardPage.tsx
      ├── components/
      │   ├── MetricCard.tsx
      │   └── RecentOrders.tsx
      └── hooks/
          └── useDashboardData.ts
```

### Container/Presentational Pattern

```typescript
// ============================================
// Container (Lógica)
// ============================================
export const UsersPageContainer = () => {
  const { currentTenant } = useTenant()
  const { data, isLoading } = useUsersData(currentTenant.id)
  const [filters, setFilters] = useState<IUserFilters>({})

  return (
    <UsersPageView
      users={data}
      isLoading={isLoading}
      filters={filters}
      onFilterChange={setFilters}
    />
  )
}

// ============================================
// Presentational (UI)
// ============================================
export const UsersPageView: React.FC<IUsersPageViewProps> = ({
  users,
  isLoading,
  filters,
  onFilterChange
}) => {
  return (
    <div>
      <UserFilters filters={filters} onChange={onFilterChange} />
      {isLoading ? <Spinner /> : <UsersList users={users} />}
    </div>
  )
}
```

### Custom Hooks Pattern

```typescript
// useUsersData.ts
export const useUsersData = (tenantId: string) => {
  const { data, isLoading, error } = useQuery({
    queryKey: queryKeys.usersList(tenantId, filters),
    queryFn: () => fetchUsers(tenantId, filters),
    enabled: !!tenantId
  })

  const createUser = useMutation({
    mutationFn: (userData: ICreateUserDTO) =>
      createUserApi(tenantId, userData),
    onSuccess: () => {
      queryClient.invalidateQueries(queryKeys.users(tenantId))
    }
  })

  return {
    users: data,
    isLoading,
    error,
    createUser
  }
}
```

---

## 📊 Gestión de Estado

### División de Responsabilidades

```
┌─────────────────────────────────────────────────────┐
│ Client State (Zustand)                              │
│ • UI state (modals, drawers, sidebar)               │
│ • User preferences (theme, language)                │
│ • Temporal data (form drafts)                       │
│ • Navigation state                                  │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ Server State (React Query)                          │
│ • API data (users, products, orders)                │
│ • Cache management                                  │
│ • Background synchronization                        │
│ • Optimistic updates                                │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ Context API                                         │
│ • Authentication (user, token)                      │
│ • Current Tenant (tenant, permissions, limits)      │
│ • Theme (colors, dark mode)                         │
│ • i18n (language, locale)                           │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ Local State (useState)                              │
│ • Component-specific state                          │
│ • Form inputs                                       │
│ • Toggle states                                     │
└─────────────────────────────────────────────────────┘
```

### Ejemplo de Store Zustand

```typescript
// uiStore.ts
interface IUIState {
  sidebarOpen: boolean
  modalOpen: string | null
  theme: 'light' | 'dark'

  toggleSidebar: () => void
  openModal: (modalId: string) => void
  closeModal: () => void
  setTheme: (theme: 'light' | 'dark') => void
}

export const useUIStore = create<IUIState>((set) => ({
  sidebarOpen: true,
  modalOpen: null,
  theme: 'light',

  toggleSidebar: () => set((state) => ({
    sidebarOpen: !state.sidebarOpen
  })),

  openModal: (modalId) => set({ modalOpen: modalId }),
  closeModal: () => set({ modalOpen: null }),
  setTheme: (theme) => set({ theme }),
}))
```

---

## 🔐 Seguridad

### Capas de Seguridad

```
1. HTTPS Only
   └─ Todas las comunicaciones encriptadas

2. JWT Authentication
   └─ Tokens firmados, expirables, con refresh

3. Tenant Isolation
   └─ Header X-Tenant-ID obligatorio
   └─ Validación backend

4. RBAC (Role-Based Access Control)
   └─ Permisos granulares por rol

5. Plan Limits Validation
   └─ Frontend y backend verifican límites

6. Input Validation
   └─ Zod schemas en forms
   └─ Sanitización de inputs

7. XSS Prevention
   └─ React escapa automáticamente
   └─ dangerouslySetInnerHTML solo cuando necesario

8. CSRF Protection
   └─ SameSite cookies
   └─ CSRF tokens para mutaciones críticas
```

### Validación de Permisos

```typescript
// permissions.ts
export const PERMISSIONS = {
  // Users
  'users.read': ['admin', 'manager', 'user'],
  'users.create': ['admin', 'manager'],
  'users.edit': ['admin', 'manager'],
  'users.delete': ['admin'],

  // Products
  'products.read': ['admin', 'manager', 'user'],
  'products.create': ['admin', 'manager'],
  'products.edit': ['admin', 'manager'],
  'products.delete': ['admin'],

  // Orders
  'orders.read': ['admin', 'manager', 'user'],
  'orders.create': ['admin', 'manager', 'user'],
  'orders.edit': ['admin', 'manager'],
  'orders.delete': ['admin'],

  // Reports (plan-based)
  'reports.basic': ['admin', 'manager'], // Plan1+
  'reports.advanced': ['admin'], // Plan2+
  'analytics.full': ['admin'], // Plan2+
} as const

// Hook para validar permisos
export const usePermissions = () => {
  const { currentTenant } = useTenant()
  const { user } = useAuth()

  const hasPermission = (permission: keyof typeof PERMISSIONS): boolean => {
    if (!currentTenant || !user) return false

    // Validar rol
    const allowedRoles = PERMISSIONS[permission]
    if (!allowedRoles.includes(user.role)) return false

    // Validar plan (para features premium)
    if (permission.startsWith('analytics') && currentTenant.plan < 'plan2') {
      return false
    }

    return true
  }

  return { hasPermission }
}
```

---

## ⚡ Performance

### Optimizaciones Implementadas

```typescript
// 1. Code Splitting (Lazy Loading)
const DashboardPage = lazy(() => import('@/pages/dashboard/DashboardPage'))
const UsersPage = lazy(() => import('@/pages/users/UsersPage'))

<Suspense fallback={<PageLoader />}>
  <Routes>
    <Route path="/dashboard" element={<DashboardPage />} />
    <Route path="/users" element={<UsersPage />} />
  </Routes>
</Suspense>

// 2. Memoization
const MemoizedUserCard = React.memo(UserCard, (prev, next) => {
  return prev.user.id === next.user.id &&
         prev.user.updatedAt === next.user.updatedAt
})

// 3. useMemo para cálculos pesados
const totalPrice = useMemo(() => {
  return items.reduce((sum, item) => sum + item.price, 0)
}, [items])

// 4. useCallback para funciones que se pasan a hijos
const handleDelete = useCallback((userId: string) => {
  deleteUser(userId)
}, [deleteUser])

// 5. React Query con staleTime
const { data } = useQuery({
  queryKey: ['users', tenantId],
  queryFn: () => fetchUsers(tenantId),
  staleTime: 5 * 60 * 1000, // 5 minutos
  cacheTime: 10 * 60 * 1000, // 10 minutos
})

// 6. Virtualización para listas largas
import { FixedSizeList } from 'react-window'

<FixedSizeList
  height={600}
  itemCount={users.length}
  itemSize={50}
>
  {({ index, style }) => (
    <div style={style}>
      <UserCard user={users[index]} />
    </div>
  )}
</FixedSizeList>
```

### Métricas Objetivo

```yaml
Performance Targets:
  - First Contentful Paint: < 1.5s
  - Time to Interactive: < 3.5s
  - Largest Contentful Paint: < 2.5s
  - Cumulative Layout Shift: < 0.1
  - First Input Delay: < 100ms

Bundle Size:
  - Initial Bundle: < 200KB (gzipped)
  - Total Bundle: < 500KB (gzipped)
  - Code Coverage: > 80%

API Performance:
  - Average Response Time: < 200ms
  - P95 Response Time: < 500ms
  - Error Rate: < 0.1%
```

---

## 📈 Escalabilidad

### Estrategia de Crecimiento

```
Fase 1: MVP (1-1000 tenants)
  ✓ Single frontend instance
  ✓ Single API instance
  ✓ PostgreSQL database
  ✓ Redis cache

Fase 2: Growth (1K-10K tenants)
  ✓ Multiple frontend instances (CDN)
  ✓ Load-balanced API instances
  ✓ Database read replicas
  ✓ Redis cluster

Fase 3: Scale (10K+ tenants)
  ✓ Global CDN distribution
  ✓ Microservices architecture (si necesario)
  ✓ Database sharding by tenant
  ✓ Message queue (RabbitMQ/Kafka)
  ✓ Kubernetes orchestration
```

### Cache Strategy

```typescript
// Multi-level caching
┌────────────────────────────────────────────┐
│ Level 1: React Query (Client Cache)        │
│ • 5-10 minutes stale time                  │
│ • Invalidación por tenant                  │
│ • Automatic background refetch             │
└────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────┐
│ Level 2: Service Worker (PWA)             │
│ • Static assets                            │
│ • API responses (offline)                  │
└────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────┐
│ Level 3: CDN (CloudFlare/AWS)             │
│ • Static files                             │
│ • Images                                   │
└────────────────────────────────────────────┘
                    ↓
┌────────────────────────────────────────────┐
│ Level 4: Redis (Backend Cache)            │
│ • User sessions                            │
│ • Tenant data                              │
│ • Query results                            │
└────────────────────────────────────────────┘
```

---

## 🔄 CI/CD Pipeline

```yaml
# .github/workflows/ci.yml

name: CI/CD Pipeline

on: [push, pull_request]

jobs:
  # 1. Lint & Type Check
  lint:
    - ESLint
    - Prettier
    - TypeScript compiler

  # 2. Unit Tests
  test:
    - Vitest (coverage > 80%)

  # 3. Build
  build:
    - Vite build
    - Bundle size check (< 500KB)

  # 4. E2E Tests
  e2e:
    - Playwright tests

  # 5. Deploy (solo en main)
  deploy:
    - Staging (on develop)
    - Production (on main)
```

---

<div align="center">

**Esta arquitectura está diseñada para escalar y evolucionar con el proyecto**

Última actualización: Noviembre 2025 | Versión 2.0

</div>
