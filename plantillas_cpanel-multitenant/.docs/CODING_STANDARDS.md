# CODING STANDARDS - bMOI CPanel Multi-Tenant BO

> **Estándares de código obligatorios con ejemplos prácticos**
> Versión: 2.0 | Última actualización: Noviembre 2025

---

## 📖 Índice

1. [Convenciones de Nomenclatura](#-convenciones-de-nomenclatura)
2. [Estructura de Archivos](#-estructura-de-archivos)
3. [TypeScript Standards](#-typescript-standards)
4. [React Best Practices](#-react-best-practices)
5. [Multi-Tenant Patterns](#-multi-tenant-patterns)
6. [Estado y Data Fetching](#-estado-y-data-fetching)
7. [Estilos y CSS](#-estilos-y-css)
8. [Testing Standards](#-testing-standards)
9. [Performance y Optimización](#-performance-y-optimización)
10. [Seguridad](#-seguridad)

---

## 🔤 Convenciones de Nomenclatura

### Variables y Funciones: `camelCase`

```typescript
// ========================================
// ✅ CORRECTO
// ========================================

// Variables
const userName = "John Doe"
const isAuthenticated = true
const userAge = 25
const tenantId = "tenant-123"
const currentPlan = "plan1"
const hasPermission = false

// Funciones
function calculateTotalPrice(items: ICartItem[]): number {
  return items.reduce((sum, item) => sum + item.price, 0)
}

const getUserData = async (userId: string) => {
  const response = await fetch(`/api/users/${userId}`)
  return response.json()
}

const handleSubmit = (event: React.FormEvent) => {
  event.preventDefault()
  // ...
}

// Arrays y objetos
const userList = [...]
const configOptions = {...}
const filterParams = {...}

// ========================================
// ❌ INCORRECTO
// ========================================

// PascalCase en variables (reservado para componentes/clases)
const UserName = "John"
const IsAuthenticated = true

// snake_case PROHIBIDO en JavaScript/TypeScript
const user_name = "John"
const is_authenticated = true
const get_user_data = () => {}

// UPPER_CASE en variables normales (reservado para constantes)
const USERID = "123"
const USERNAME = "John"

// Nombres poco descriptivos
const x = "John"
const temp = true
const fn = () => {}
```

### Componentes React: `PascalCase`

```typescript
// ========================================
// ✅ CORRECTO
// ========================================

// Componentes funcionales
export const UserProfile: React.FC<IUserProfileProps> = ({ user }) => {
  return <div>{user.name}</div>
}

export const TenantDashboard: React.FC = () => {
  return <main>Dashboard</main>
}

export const PlanUpgradeModal: React.FC<IModalProps> = ({ isOpen }) => {
  return <Modal open={isOpen}>...</Modal>
}

// Componentes de orden superior (HOC)
export const withTenantContext = <P extends object>(
  Component: React.ComponentType<P>
) => {
  return (props: P) => (
    <TenantProvider>
      <Component {...props} />
    </TenantProvider>
  )
}

// ========================================
// ❌ INCORRECTO
// ========================================

// camelCase en componentes
export const userProfile = () => {
  return <div>Profile</div>
}

// snake_case PROHIBIDO
export const user_profile = () => {
  return <div>Profile</div>
}

// UPPER_CASE incorrecto para componentes
export const USERPROFILE = () => {
  return <div>Profile</div>
}

// Nombres no descriptivos
export const UP = () => {}
export const Component1 = () => {}
```

### Hooks Personalizados: `use` + `camelCase`

```typescript
// ========================================
// ✅ CORRECTO
// ========================================

// Hooks simples
export const useAuth = () => {
  const [user, setUser] = useState<IUser | null>(null)
  // ...
  return { user, login, logout }
}

export const useTenant = () => {
  // ...
}

// Hooks con parámetros
export const useLocalStorage = <T,>(key: string, initialValue: T) => {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key)
      return item ? JSON.parse(item) : initialValue
    } catch (error) {
      return initialValue
    }
  })
  // ...
  return [storedValue, setValue] as const
}

export const useFetchData = <T,>(url: string, options?: RequestInit) => {
  // ...
}

export const usePlanLimits = (plan: PlanType) => {
  // ...
}

export const useDebounce = <T,>(value: T, delay: number) => {
  // ...
}

// ========================================
// ❌ INCORRECTO
// ❌=========================================

// Sin prefijo "use"
export const auth = () => {}
export const tenant = () => {}

// PascalCase incorrecto
export const UseAuth = () => {}

// snake_case PROHIBIDO
export const use_auth = () => {}
export const use_local_storage = () => {}
```

### Constantes: `UPPER_SNAKE_CASE`

```typescript
// ========================================
// ✅ CORRECTO
// ========================================

// URLs y endpoints
export const API_BASE_URL = "https://api.controlpanel.com/v1"
export const AUTH_ENDPOINT = "/auth/login"
export const WS_ENDPOINT = "wss://api.controlpanel.com"

// Límites y configuración
export const MAX_RETRY_ATTEMPTS = 3
export const DEFAULT_PAGE_SIZE = 20
export const MIN_PASSWORD_LENGTH = 8
export const SESSION_TIMEOUT_MS = 3600000 // 1 hora

// Headers
export const TENANT_HEADER_KEY = "X-Tenant-ID"
export const AUTH_HEADER_KEY = "Authorization"

// Mensajes y textos fijos
export const ERROR_MESSAGES = {
  UNAUTHORIZED: "No autorizado",
  FORBIDDEN: "Sin permisos",
  NOT_FOUND: "No encontrado"
} as const

// Configuraciones complejas
export const PLAN_LIMITS = {
  FREE: { users: 10, products: 50 },
  PLAN1: { users: 50, products: 500 },
  PLAN2: { users: 200, products: 2000 },
  PLAN3: { users: -1, products: -1 } // Ilimitado
} as const

// ========================================
// ❌ INCORRECTO
// ========================================

// camelCase en constantes
export const apiBaseUrl = "..."
export const maxRetries = 3

// PascalCase incorrecto
export const ApiBaseUrl = "..."

// snake_case minúsculas (debería ser mayúsculas)
export const api_base_url = "..."
```

### Interfaces y Types: `I` + `PascalCase` / `PascalCase`

```typescript
// ========================================
// ✅ CORRECTO
// ========================================

// Interfaces con prefijo I
interface IUser {
  id: string
  name: string
  email: string
  tenantId: string
  role: UserRole
}

interface ITenant {
  id: string
  name: string
  slug: string
  plan: PlanType
  status: TenantStatus
}

interface IComponentProps {
  title: string
  onClose?: () => void
  children?: React.ReactNode
}

// Types sin prefijo (para unions, intersections, etc.)
type PlanType = 'free' | 'plan1' | 'plan2' | 'plan3'
type UserRole = 'admin' | 'manager' | 'user'
type TenantStatus = 'active' | 'inactive' | 'suspended'

type UserWithTenant = IUser & { tenant: ITenant }

type ApiResponse<T> = {
  data: T
  status: number
  message?: string
}

// ========================================
// ❌ INCORRECTO
// ========================================

// Sin prefijo I en interfaces
interface User {
  id: string
}

// camelCase incorrecto
interface iUser {
  id: string
}

// snake_case PROHIBIDO
interface i_user {
  id: string
}

type user_role = string
type plan_type = string
```

### Archivos y Carpetas

```
✅ CORRECTO:
  UserProfile.tsx
  UserProfile.types.ts
  UserProfile.module.css
  UserProfile.test.tsx

  useAuth.ts
  useTenant.ts

  user.service.ts
  tenant.service.ts

  constants.ts
  helpers.ts

❌ INCORRECTO:
  userProfile.tsx
  user_profile.tsx
  user-profile.tsx (excepto en carpetas)

  UseAuth.ts
  use_auth.ts

  userService.ts
  user_service.ts
```

---

## 📁 Estructura de Archivos

### Componente Completo

```
UserCard/
├── UserCard.tsx              # Componente principal
├── UserCard.types.ts         # Interfaces y types
├── UserCard.module.css       # Estilos (si usa CSS Modules)
├── UserCard.test.tsx         # Tests
├── components/               # Sub-componentes (opcional)
│   └── UserCardHeader.tsx
└── index.ts                  # Re-export
```

#### UserCard.tsx
```typescript
import React from 'react'
import { Card, Avatar } from 'antd'
import { useTenant } from '@/hooks/useTenant'
import type { IUserCardProps } from './UserCard.types'
import styles from './UserCard.module.css'

export const UserCard: React.FC<IUserCardProps> = ({
  user,
  onEdit,
  onDelete
}) => {
  const { currentTenant, hasPermission } = useTenant()

  return (
    <Card
      className={styles.card}
      data-tenant={currentTenant?.slug}
    >
      <Avatar src={user.avatar} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>

      {hasPermission('users.edit') && (
        <button onClick={() => onEdit(user.id)}>Editar</button>
      )}
    </Card>
  )
}
```

#### UserCard.types.ts
```typescript
import type { IUser } from '@/types'

export interface IUserCardProps {
  user: IUser
  onEdit?: (userId: string) => void
  onDelete?: (userId: string) => void
  variant?: 'default' | 'compact' | 'detailed'
}
```

#### index.ts
```typescript
export { UserCard } from './UserCard'
export type { IUserCardProps } from './UserCard.types'
```

### Página con Estructura

```
UsersPage/
├── UsersPage.tsx
├── UsersPage.types.ts
├── components/
│   ├── UsersList.tsx
│   ├── UsersFilters.tsx
│   └── UserCreateModal.tsx
├── hooks/
│   ├── useUsersData.ts
│   └── useUsersFilters.ts
└── index.ts
```

---

## 📘 TypeScript Standards

### Type Safety: NO usar `any`

```typescript
// ========================================
// ✅ CORRECTO
// ========================================

// Tipos específicos
interface IApiResponse<T> {
  data: T
  status: number
  message?: string
}

async function fetchUsers(tenantId: string): Promise<IUser[]> {
  const response = await api.get<IApiResponse<IUser[]>>('/users', {
    params: { tenantId }
  })
  return response.data.data
}

// unknown cuando el tipo es realmente desconocido
function processData(data: unknown): void {
  if (typeof data === 'string') {
    console.log(data.toUpperCase())
  } else if (isUser(data)) {
    console.log(data.name)
  }
}

// Type guard
function isUser(obj: unknown): obj is IUser {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj &&
    'email' in obj
  )
}

// Generics
function createArray<T>(length: number, value: T): T[] {
  return Array(length).fill(value)
}

// ========================================
// ❌ INCORRECTO
// ========================================

// any type sin justificación
async function fetchUsers(tenantId: any): Promise<any> {
  const response: any = await api.get('/users')
  return response.data
}

function processData(data: any) {
  console.log(data.something)
}

const user: any = {
  id: '123',
  name: 'John'
}

// Conversión forzada sin validación
const data = response as IUser  // Peligroso si no estás seguro
```

### Interfaces vs Types

```typescript
// ========================================
// ✅ USAR INTERFACES PARA
// ========================================

// Objetos y estructuras de datos
interface IUser {
  id: string
  name: string
  email: string
}

// Props de componentes
interface IButtonProps {
  text: string
  onClick: () => void
  disabled?: boolean
}

// Cuando se necesita extends
interface IAdmin extends IUser {
  permissions: string[]
}

// ========================================
// ✅ USAR TYPES PARA
// ========================================

// Unions
type PlanType = 'free' | 'plan1' | 'plan2' | 'plan3'
type Status = 'loading' | 'success' | 'error'

// Intersections
type UserWithPermissions = IUser & { permissions: string[] }

// Tipos complejos
type ApiHandler = (
  req: Request,
  res: Response
) => Promise<void> | void

// Mapped types
type Partial<T> = {
  [P in keyof T]?: T[P]
}

// Conditional types
type NonNullable<T> = T extends null | undefined ? never : T
```

### Enums vs Union Types

```typescript
// ========================================
// ✅ PREFERIR Union Types
// ========================================

// Más simple y type-safe
type PlanType = 'free' | 'plan1' | 'plan2' | 'plan3'

const plan: PlanType = 'plan1' // ✅
// const plan: PlanType = 'invalid' // ❌ Error en compile time

// Para constantes relacionadas
const PLAN_NAMES = {
  free: 'Gratis',
  plan1: 'Básico',
  plan2: 'Pro',
  plan3: 'Enterprise'
} as const

type PlanKey = keyof typeof PLAN_NAMES

// ========================================
// ⚠️ Usar Enums SOLO si se necesita
// ========================================

// Solo cuando necesitas valores numéricos o lógica compleja
enum HttpStatus {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  Unauthorized = 401,
  Forbidden = 403,
  NotFound = 404
}

const status = HttpStatus.OK // 200
```

---

## ⚛️ React Best Practices

### Componente Template (ESTÁNDAR OBLIGATORIO)

```typescript
import React, { useState, useEffect, useMemo } from 'react'
import { useQuery } from '@tanstack/react-query'
import { Card, Button, Spin } from 'antd'
import { useTenant } from '@/hooks/useTenant'
import { useAuth } from '@/hooks/useAuth'
import { ErrorBoundary } from '@/components/atoms/ErrorBoundary'
import type { IComponentProps } from './Component.types'
import styles from './Component.module.css'

/**
 * ComponentName - Descripción breve
 *
 * @description Explicación detallada del propósito del componente.
 * Incluir consideraciones multi-tenant si aplica.
 *
 * @example
 * ```tsx
 * <ComponentName
 *   title="Ejemplo"
 *   data={data}
 *   onAction={handleAction}
 * />
 * ```
 */
export const ComponentName: React.FC<IComponentProps> = ({
  title,
  data,
  onAction,
  variant = 'default',
  ...rest
}) => {
  // ===========================================
  // 1. HOOKS DE ESTADO
  // ===========================================
  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [selectedItems, setSelectedItems] = useState<string[]>([])

  // ===========================================
  // 2. HOOKS PERSONALIZADOS Y CONTEXT
  // ===========================================
  const { currentTenant, hasPermission, isWithinLimits } = useTenant()
  const { user } = useAuth()

  // ===========================================
  // 3. REACT QUERY / DATA FETCHING
  // ===========================================
  const {
    data: remoteData,
    isLoading: queryLoading,
    error: queryError,
    refetch
  } = useQuery({
    queryKey: ['resource', currentTenant?.id, filters],
    queryFn: () => fetchData(currentTenant!.id),
    enabled: !!currentTenant,
    staleTime: 5 * 60 * 1000, // 5 minutos
  })

  // ===========================================
  // 4. EFECTOS
  // ===========================================
  useEffect(() => {
    if (!currentTenant) return

    // Lógica específica del tenant
    console.log('Tenant cambió:', currentTenant.name)

    // Cleanup
    return () => {
      console.log('Cleanup')
    }
  }, [currentTenant])

  // ===========================================
  // 5. MEMOIZACIÓN (si es necesario)
  // ===========================================
  const processedData = useMemo(() => {
    if (!data) return []

    return data
      .filter(item => item.tenantId === currentTenant?.id)
      .map(item => ({ ...item, processed: true }))
  }, [data, currentTenant])

  const totalPrice = useMemo(() => {
    return processedData.reduce((sum, item) => sum + item.price, 0)
  }, [processedData])

  // ===========================================
  // 6. CALLBACKS (con useCallback si se pasan a hijos)
  // ===========================================
  const handleAction = useCallback((itemId: string) => {
    if (!hasPermission('action.execute')) {
      setError('Sin permisos para esta acción')
      return
    }

    if (!isWithinLimits('actions')) {
      setError('Límite de plan alcanzado')
      return
    }

    setIsLoading(true)
    onAction?.(itemId)
      .then(() => {
        // Success
        refetch()
      })
      .catch((err) => {
        setError(err.message)
      })
      .finally(() => {
        setIsLoading(false)
      })
  }, [hasPermission, isWithinLimits, onAction, refetch])

  // ===========================================
  // 7. RENDER HELPERS
  // ===========================================
  const renderLoading = () => (
    <div className={styles.loading}>
      <Spin tip="Cargando..." />
    </div>
  )

  const renderError = (message: string) => (
    <div className={styles.error}>
      <p>{message}</p>
      <Button onClick={() => refetch()}>Reintentar</Button>
    </div>
  )

  const renderContent = () => {
    if (!processedData.length) {
      return <p>No hay datos disponibles</p>
    }

    return (
      <div className={styles.content}>
        {processedData.map(item => (
          <Card key={item.id}>
            {item.name}
          </Card>
        ))}
      </div>
    )
  }

  // ===========================================
  // 8. EARLY RETURNS
  // ===========================================
  if (!currentTenant) {
    return <div>Por favor selecciona un tenant</div>
  }

  if (!hasPermission('component.view')) {
    return <div>Sin permisos para ver este componente</div>
  }

  // ===========================================
  // 9. RENDER PRINCIPAL
  // ===========================================
  return (
    <ErrorBoundary>
      <div
        className={`${styles.container} ${styles[variant]}`}
        data-tenant={currentTenant.slug}
        data-plan={currentTenant.plan}
        {...rest}
      >
        <h2>{title}</h2>

        {isLoading || queryLoading ? renderLoading() : null}
        {error || queryError ? renderError(error || queryError.message) : null}
        {!isLoading && !error ? renderContent() : null}

        <div className={styles.actions}>
          <Button onClick={() => handleAction('item-1')}>
            Ejecutar Acción
          </Button>
        </div>
      </div>
    </ErrorBoundary>
  )
}

// Display name para DevTools
ComponentName.displayName = 'ComponentName'
```

### Props Drilling vs Context

```typescript
// ========================================
// ✅ CORRECTO: Usar Context para datos globales
// ========================================

// TenantContext.tsx
interface ITenantContext {
  currentTenant: ITenant | null
  switchTenant: (id: string) => Promise<void>
}

export const TenantContext = createContext<ITenantContext | undefined>(undefined)

export const TenantProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [currentTenant, setCurrentTenant] = useState<ITenant | null>(null)

  const switchTenant = async (id: string) => {
    const tenant = await fetchTenant(id)
    setCurrentTenant(tenant)
  }

  return (
    <TenantContext.Provider value={{ currentTenant, switchTenant }}>
      {children}
    </TenantContext.Provider>
  )
}

// Componente que usa el context
export const UsersList = () => {
  const { currentTenant } = useTenant() // Custom hook

  return <div>Users for {currentTenant?.name}</div>
}

// ========================================
// ❌ INCORRECTO: Props drilling excesivo
// ========================================

// App -> Layout -> Sidebar -> UserMenu -> UserInfo
<App>
  <Layout tenant={tenant}>
    <Sidebar tenant={tenant}>
      <UserMenu tenant={tenant}>
        <UserInfo tenant={tenant} />
      </UserMenu>
    </Sidebar>
  </Layout>
</App>

// Pasar tenant a través de 5 niveles es ineficiente
```

---

## 🏢 Multi-Tenant Patterns

### Hook `useTenant` (OBLIGATORIO)

```typescript
import { useContext } from 'react'
import { TenantContext } from '@/context/TenantContext'

export const useTenant = () => {
  const context = useContext(TenantContext)

  if (!context) {
    throw new Error('useTenant debe usarse dentro de TenantProvider')
  }

  return context
}

// USO en componentes
export const MyComponent = () => {
  const { currentTenant, hasPermission, isWithinLimits } = useTenant()

  if (!currentTenant) {
    return <div>Selecciona un tenant</div>
  }

  if (!hasPermission('feature.access')) {
    return <div>Sin permisos</div>
  }

  if (!isWithinLimits('users', currentUserCount + 1)) {
    return <div>Límite de plan alcanzado</div>
  }

  return <div>Contenido para {currentTenant.name}</div>
}
```

### Query Keys con Tenant

```typescript
// ========================================
// ✅ CORRECTO: Siempre incluir tenant ID
// ========================================

// queryKeys.ts
export const queryKeys = {
  // Base
  all: (tenantId: string) => ['tenant', tenantId] as const,

  // Users
  users: (tenantId: string) => [...queryKeys.all(tenantId), 'users'] as const,
  usersList: (tenantId: string, filters: IUserFilters) =>
    [...queryKeys.users(tenantId), 'list', filters] as const,
  userDetail: (tenantId: string, userId: string) =>
    [...queryKeys.users(tenantId), 'detail', userId] as const,

  // Products
  products: (tenantId: string) => [...queryKeys.all(tenantId), 'products'] as const,
  productsList: (tenantId: string, filters: IProductFilters) =>
    [...queryKeys.products(tenantId), 'list', filters] as const,

  // Dashboard
  dashboard: (tenantId: string, dateRange: IDateRange) =>
    [...queryKeys.all(tenantId), 'dashboard', dateRange] as const,
}

// USO
const { data } = useQuery({
  queryKey: queryKeys.usersList(currentTenant.id, filters),
  queryFn: () => fetchUsers(currentTenant.id, filters),
  enabled: !!currentTenant
})

// ========================================
// ❌ INCORRECTO: Sin tenant ID
// ========================================
queryKey: ['users']  // Datos se mezclarán entre tenants
queryKey: ['users', filters]  // Falta tenant
```

### API Client con Tenant Header

```typescript
// ========================================
// ✅ CORRECTO: Interceptor automático
// ========================================

// api.ts
import axios, { AxiosInstance } from 'axios'
import type { ITenant } from '@/types'

class ApiClient {
  private instance: AxiosInstance
  private currentTenant: ITenant | null = null

  constructor() {
    this.instance = axios.create({
      baseURL: import.meta.env.VITE_API_URL,
      timeout: 10000,
    })

    // Request interceptor
    this.instance.interceptors.request.use((config) => {
      // Tenant header
      if (this.currentTenant) {
        config.headers['X-Tenant-ID'] = this.currentTenant.id
      }

      // Auth token
      const token = localStorage.getItem('auth_token')
      if (token) {
        config.headers.Authorization = `Bearer ${token}`
      }

      return config
    })

    // Response interceptor
    this.instance.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 403) {
          // Sin permisos para este tenant
          console.error('Tenant forbidden:', this.currentTenant?.id)
        }

        if (error.response?.status === 402) {
          // Límite de plan excedido
          console.warn('Plan limit exceeded')
        }

        return Promise.reject(error)
      }
    )
  }

  setTenant(tenant: ITenant | null) {
    this.currentTenant = tenant
  }

  async get<T>(...args) {
    return this.instance.get<T>(...args)
  }

  async post<T>(...args) {
    return this.instance.post<T>(...args)
  }

  // ... put, delete, etc.
}

export const apiClient = new ApiClient()
```

---

## 📊 Estado y Data Fetching

### Zustand Store Pattern

```typescript
// ========================================
// ✅ CORRECTO: Store con TypeScript
// ========================================

// authStore.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'
import type { IUser } from '@/types'

interface IAuthState {
  // State
  user: IUser | null
  token: string | null
  isAuthenticated: boolean

  // Actions
  login: (user: IUser, token: string) => void
  logout: () => void
  updateUser: (userData: Partial<IUser>) => void
}

export const useAuthStore = create<IAuthState>()(
  persist(
    (set) => ({
      // Initial state
      user: null,
      token: null,
      isAuthenticated: false,

      // Actions
      login: (user, token) => set({
        user,
        token,
        isAuthenticated: true
      }),

      logout: () => set({
        user: null,
        token: null,
        isAuthenticated: false
      }),

      updateUser: (userData) => set((state) => ({
        user: state.user ? { ...state.user, ...userData } : null
      })),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        token: state.token,
        user: state.user,
      }),
    }
  )
)

// USO
const MyComponent = () => {
  const { user, login, logout } = useAuthStore()

  return (
    <div>
      {user ? (
        <p>Hola, {user.name}</p>
      ) : (
        <button onClick={() => login(userData, token)}>Login</button>
      )}
    </div>
  )
}
```

### React Query Best Practices

```typescript
// ========================================
// ✅ CORRECTO: Query con todas las opciones
// ========================================

const { data, isLoading, error, refetch, isFetching } = useQuery({
  queryKey: ['users', currentTenant.id, filters],

  queryFn: async () => {
    const response = await apiClient.get<IApiResponse<IUser[]>>('/users', {
      params: filters
    })
    return response.data.data
  },

  // Opciones importantes
  enabled: !!currentTenant,  // Solo ejecutar si hay tenant
  staleTime: 5 * 60 * 1000,  // 5 minutos (datos "frescos")
  cacheTime: 10 * 60 * 1000, // 10 minutos (en cache)
  retry: 3,                   // Reintentos en caso de error
  retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),

  // Callbacks
  onSuccess: (data) => {
    console.log('Datos cargados:', data.length)
  },

  onError: (error) => {
    console.error('Error al cargar:', error)
  }
})

// Mutation con invalidación
const mutation = useMutation({
  mutationFn: (newUser: ICreateUserDTO) => {
    return apiClient.post('/users', newUser)
  },

  onSuccess: () => {
    // Invalidar y refetch
    queryClient.invalidateQueries({
      queryKey: ['users', currentTenant.id]
    })
  },

  onError: (error) => {
    console.error('Error al crear usuario:', error)
  }
})

// USO
const handleCreateUser = () => {
  mutation.mutate({
    name: 'John',
    email: 'john@example.com'
  })
}
```

---

*[Continuará en la siguiente respuesta debido a límites de longitud]*

---

## 🎯 Resumen de Reglas Críticas

### OBLIGATORIO ✅
1. Variables: `camelCase`
2. Componentes: `PascalCase`
3. Constantes: `UPPER_SNAKE_CASE`
4. Interfaces: `I` + `PascalCase`
5. Hooks: `use` + `camelCase`
6. Tenant context: Siempre verificar
7. Query keys: Incluir tenant ID
8. Tests: Cobertura > 80%

### PROHIBIDO ❌
1. `snake_case` en JS/TS
2. `any` type sin justificación
3. `console.log` en producción
4. Queries sin tenant ID
5. Componentes > 300 líneas
6. Mezclar datos entre tenants

---

<div align="center">

**Estos estándares son obligatorios. No hay excepciones.**

Última actualización: Noviembre 2025 | Versión 2.0

</div>
