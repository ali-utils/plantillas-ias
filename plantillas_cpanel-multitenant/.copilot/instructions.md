# Instrucciones para GitHub Copilot

> **Configuración para GitHub Copilot en bMOI CPanel Multi-Tenant BO**

---

## 🏢 Proyecto

```
Nombre: bMOI CPanel Multi-Tenant BO
Tipo: Panel de Control Multi-Tenant Empresarial
Stack: React 18 + TypeScript 5 + Vite 5
Entorno: Docker (OBLIGATORIO para desarrollo)
```

## 🐳 Desarrollo con Docker (OBLIGATORIO)

**⚠️ TODO el desarrollo se hace en Docker**

```bash
# ✅ Iniciar entorno de desarrollo
docker compose --profile dev up -d

# ✅ Ver logs
docker compose logs -f frontend-dev

# ✅ Ejecutar comandos dentro del container
docker compose exec frontend-dev npm install [package]
docker compose exec frontend-dev npm run lint
docker compose exec frontend-dev npm test

# ✅ Detener entorno
docker compose --profile dev down

# ❌ NUNCA ejecutar directamente:
npm install  # NO
npm run dev  # NO
```

**Razón:** Consistencia entre devs, CI/CD, aislamiento

---

## 🎨 Identidad

```typescript
// Paleta de Colores
const COLORS = {
  primary: '#2563EB',    // Blue-600
  secondary: '#7C3AED',  // Violet-600
  success: '#059669',
  warning: '#D97706',
  danger: '#DC2626',
}

// Logo
const LOGO = 'bMOI'
```

---

## 🔤 Convenciones OBLIGATORIAS

### Nomenclatura

```typescript
// ✅ Variables y funciones: camelCase
const userName = "John"
const getUserData = () => {}
const handleSubmit = (event) => {}

// ✅ Componentes: PascalCase
export const UserProfile: React.FC = () => {}
export const TenantDashboard: React.FC = () => {}

// ✅ Hooks: use + camelCase
export const useAuth = () => {}
export const useTenant = () => {}

// ✅ Constantes: UPPER_SNAKE_CASE
export const API_BASE_URL = "https://api.example.com"
export const MAX_RETRIES = 3

// ✅ Interfaces: I + PascalCase
interface IUser {
  id: string
  name: string
}

// ❌ PROHIBIDO: snake_case
const user_name = "John"  // NUNCA
const get_user_data = () => {}  // NUNCA
```

---

## 🏢 Multi-Tenant Patterns

### Siempre Verificar Tenant

```typescript
import { useTenant } from '@/hooks/useTenant'

export const MyComponent: React.FC = () => {
  const { currentTenant, hasPermission } = useTenant()

  // Siempre verificar
  if (!currentTenant) {
    return <div>Por favor selecciona un tenant</div>
  }

  // Validar permisos
  if (!hasPermission('feature.access')) {
    return <div>Sin permisos</div>
  }

  return (
    <div data-tenant={currentTenant.slug}>
      Contenido para {currentTenant.name}
    </div>
  )
}
```

### Query Keys con Tenant ID

```typescript
import { useQuery } from '@tanstack/react-query'
import { useTenant } from '@/hooks/useTenant'

export const useUsersData = () => {
  const { currentTenant } = useTenant()

  return useQuery({
    // ✅ SIEMPRE incluir tenant.id en la key
    queryKey: ['users', currentTenant?.id, filters],
    queryFn: () => fetchUsers(currentTenant!.id, filters),
    enabled: !!currentTenant
  })
}
```

---

## 📦 TypeScript Estricto

```typescript
// ✅ Tipos específicos
interface IUser {
  id: string
  name: string
  email: string
  tenantId: string
}

async function fetchUsers(tenantId: string): Promise<IUser[]> {
  const response = await apiClient.get<IUser[]>('/users')
  return response.data
}

// ❌ NO usar any
const data: any = fetchData()  // PROHIBIDO
function process(input: any) {}  // PROHIBIDO
```

---

## ⚛️ Template de Componente

```typescript
import React, { useState, useEffect } from 'react'
import { useTenant } from '@/hooks/useTenant'
import type { IComponentProps } from './Component.types'
import styles from './Component.module.css'

/**
 * ComponentName - Descripción breve
 */
export const ComponentName: React.FC<IComponentProps> = ({
  prop1,
  prop2,
}) => {
  // 1. Estado
  const [loading, setLoading] = useState(false)

  // 2. Hooks
  const { currentTenant } = useTenant()

  // 3. Efectos
  useEffect(() => {
    if (!currentTenant) return
    // Lógica
  }, [currentTenant])

  // 4. Handlers
  const handleClick = () => {
    // ...
  }

  // 5. Early returns
  if (!currentTenant) return <div>Selecciona tenant</div>

  // 6. Render
  return (
    <div className={styles.container} data-tenant={currentTenant.slug}>
      {/* Content */}
    </div>
  )
}
```

---

## 🪝 Template de Hook

```typescript
import { useState, useEffect } from 'react'
import { useQuery } from '@tanstack/react-query'
import { useTenant } from '@/hooks/useTenant'

export const useCustomHook = (initialValue?: any) => {
  const { currentTenant } = useTenant()
  const [state, setState] = useState(initialValue)

  const { data, isLoading } = useQuery({
    queryKey: ['resource', currentTenant?.id],
    queryFn: () => fetchData(currentTenant!.id),
    enabled: !!currentTenant
  })

  useEffect(() => {
    if (!currentTenant) return
    // Lógica
  }, [currentTenant])

  return {
    state,
    setState,
    data,
    isLoading
  }
}
```

---

## 🔌 Template de Service

```typescript
import { apiClient } from '@/services/api/client'
import type { IUser, IApiResponse } from '@/types'

export const userService = {
  getAll: async (tenantId: string, filters = {}) => {
    const response = await apiClient.get<IApiResponse<IUser[]>>('/users', {
      params: filters
    })
    return response.data.data
  },

  getById: async (tenantId: string, userId: string) => {
    const response = await apiClient.get<IApiResponse<IUser>>(
      `/users/${userId}`
    )
    return response.data.data
  },

  create: async (tenantId: string, data: Partial<IUser>) => {
    const response = await apiClient.post<IApiResponse<IUser>>('/users', data)
    return response.data.data
  },

  update: async (tenantId: string, userId: string, data: Partial<IUser>) => {
    const response = await apiClient.put<IApiResponse<IUser>>(
      `/users/${userId}`,
      data
    )
    return response.data.data
  },

  delete: async (tenantId: string, userId: string) => {
    await apiClient.delete(`/users/${userId}`)
  },
}
```

---

## 📋 Orden de Imports

```typescript
// 1. React
import React, { useState, useEffect } from 'react'

// 2. Librerías externas
import { useQuery } from '@tanstack/react-query'
import { Button, Card } from 'antd'

// 3. Componentes internos
import { UserCard } from '@/components/molecules/UserCard'

// 4. Hooks
import { useTenant } from '@/hooks/useTenant'

// 5. Services
import { userService } from '@/services/user.service'

// 6. Types
import type { IUser } from '@/types'

// 7. Estilos
import styles from './Component.module.css'
```

---

## 🧪 Template de Test

```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { TenantProvider } from '@/context/TenantContext'
import { ComponentName } from './ComponentName'

const mockTenant = {
  id: 'tenant-123',
  name: 'Test Tenant',
  plan: 'plan1',
}

const renderWithProviders = (component: React.ReactElement) => {
  const queryClient = new QueryClient()

  return render(
    <QueryClientProvider client={queryClient}>
      <TenantProvider initialTenant={mockTenant}>
        {component}
      </TenantProvider>
    </QueryClientProvider>
  )
}

describe('ComponentName', () => {
  it('should render with tenant', () => {
    renderWithProviders(<ComponentName />)
    expect(screen.getByText(/test/i)).toBeInTheDocument()
  })

  it('should show tenant required when no tenant', () => {
    renderWithProviders(<ComponentName />)
    // Test tenant required message
  })
})
```

---

## 🚫 PROHIBIDO

```typescript
// ❌ snake_case
const user_name = "John"

// ❌ any type
const data: any = fetchData()

// ❌ console.log en producción
console.log('Debug')

// ❌ Queries sin tenant
queryKey: ['users']

// ❌ Componentes sin verificar tenant
export const MyComponent = () => {
  return <div>Content</div>  // Falta useTenant()
}
```

---

## ✅ SIEMPRE

```typescript
// ✅ Verificar tenant
const { currentTenant } = useTenant()
if (!currentTenant) return null

// ✅ Query con tenant
queryKey: ['users', currentTenant.id]

// ✅ Tipos específicos
interface IUser { id: string }

// ✅ Validar permisos
if (!hasPermission('action')) return
```

---

## 📚 Referencia Rápida

```typescript
// Hooks del proyecto
useTenant()     // Tenant context
useAuth()       // Authentication
usePermissions() // Permisos

// Services
apiClient       // HTTP client
userService     // Users CRUD
tenantService   // Tenants

// Types
IUser           // User interface
ITenant         // Tenant interface
IApiResponse<T> // API response wrapper

// Constants
API_BASE_URL
TENANT_HEADER_KEY
PLAN_LIMITS
```

---

<div align="center">

**Sigue estas instrucciones SIEMPRE**

</div>
