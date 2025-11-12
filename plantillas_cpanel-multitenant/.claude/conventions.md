# Convenciones de Código - OBLIGATORIAS

> **Reglas que NUNCA se violan**

---

## 🔤 Nomenclatura

### Variables y Funciones: `camelCase`

```typescript
// ✅ CORRECTO
const userName = "John"
const tenantId = "abc-123"
const isAuthenticated = true
const getUserData = () => {}
const handleSubmit = () => {}

// ❌ INCORRECTO
const UserName = "John"      // PascalCase
const user_name = "John"     // snake_case PROHIBIDO
const TENANT_ID = "abc"      // UPPER_CASE
```

### Componentes: `PascalCase`

```typescript
// ✅ CORRECTO
export const UserProfile: React.FC = () => {}
export const TenantDashboard: React.FC = () => {}

// ❌ INCORRECTO
export const userProfile = () => {}
export const user_profile = () => {}
```

### Hooks: `use` + `camelCase`

```typescript
// ✅ CORRECTO
export const useAuth = () => {}
export const useTenant = () => {}
export const useLocalStorage = (key: string) => {}

// ❌ INCORRECTO
export const UseAuth = () => {}
export const use_auth = () => {}
export const auth = () => {}
```

### Constantes: `UPPER_SNAKE_CASE`

```typescript
// ✅ CORRECTO
export const API_BASE_URL = "https://api.example.com"
export const MAX_RETRIES = 3
export const TENANT_HEADER_KEY = "X-Tenant-ID"

// ❌ INCORRECTO
export const apiBaseUrl = "..."
export const ApiBaseUrl = "..."
```

### Interfaces: `I` + `PascalCase`

```typescript
// ✅ CORRECTO
interface IUser {
  id: string
  name: string
}

interface ITenant {
  id: string
  plan: PlanType
}

// ❌ INCORRECTO
interface User {}
interface user {}
interface i_user {}
```

---

## 🏢 Multi-Tenant OBLIGATORIO

### Siempre Verificar Tenant

```typescript
// ✅ CORRECTO
export const MyComponent = () => {
  const { currentTenant } = useTenant()

  if (!currentTenant) return <TenantRequired />

  return <div data-tenant={currentTenant.slug}>Content</div>
}

// ❌ INCORRECTO
export const MyComponent = () => {
  // No verifica tenant
  return <div>Content</div>
}
```

### Query Keys con Tenant ID

```typescript
// ✅ CORRECTO
queryKey: ['users', currentTenant.id, filters]

// ❌ INCORRECTO
queryKey: ['users']  // Falta tenant ID
queryKey: ['users', filters]  // Falta tenant ID
```

### API Calls con Tenant

```typescript
// ✅ CORRECTO - Interceptor automático
const { data } = await apiClient.get('/users')
// Header X-Tenant-ID agregado automáticamente

// ❌ INCORRECTO - Headers manuales innecesarios
const { data } = await axios.get('/users', {
  headers: { 'X-Tenant-ID': 'hardcoded' }
})
```

---

## 📦 TypeScript Estricto

### NO usar `any`

```typescript
// ✅ CORRECTO
function fetchUsers(tenantId: string): Promise<IUser[]> {
  return apiClient.get<IUser[]>('/users')
}

// Para tipos desconocidos, usar unknown
function processData(data: unknown): void {
  if (typeof data === 'string') {
    console.log(data.toUpperCase())
  }
}

// ❌ INCORRECTO
function fetchUsers(tenantId: any): Promise<any> {
  const data: any = await fetch('/users')
  return data
}
```

---

## 🧪 Testing OBLIGATORIO

### Cobertura Mínima: 80%

```typescript
// test para componentes multi-tenant
describe('MyComponent', () => {
  it('should render with tenant', () => {
    const mockTenant = { id: 'test', name: 'Test' }
    render(<MyComponent />, { tenant: mockTenant })
    expect(screen.getByText(/test/i)).toBeInTheDocument()
  })

  it('should show tenant required when no tenant', () => {
    render(<MyComponent />, { tenant: null })
    expect(screen.getByText(/selecciona.*tenant/i)).toBeInTheDocument()
  })
})
```

---

## 📋 Imports Ordenados

```typescript
// 1. React
import React, { useState, useEffect } from 'react'

// 2. Librerías externas
import { useQuery } from '@tanstack/react-query'
import { Button } from 'antd'

// 3. Componentes internos
import { UserCard } from '@/components/molecules/UserCard'

// 4. Hooks
import { useTenant } from '@/hooks/useTenant'

// 5. Services y utils
import { apiClient } from '@/services/api/client'

// 6. Types
import type { IUser } from '@/types'

// 7. Estilos (siempre al final)
import styles from './Component.module.css'
```

---

## 🚫 PROHIBIDO

```typescript
// ❌ snake_case en JS/TS
const user_name = "John"

// ❌ any type
const data: any = fetchData()

// ❌ console.log en producción
console.log('Debug')

// ❌ Queries sin tenant
queryKey: ['users']

// ❌ Componentes > 300 líneas
export const HugeComponent = () => {
  // 500 líneas...
}

// ❌ Mezclar datos entre tenants
const allData = [...tenant1Data, ...tenant2Data]
```

---

## ✅ TEMPLATE de Componente

```typescript
import React, { useState, useEffect } from 'react'
import { useTenant } from '@/hooks/useTenant'
import type { IComponentProps } from './Component.types'
import styles from './Component.module.css'

export const ComponentName: React.FC<IComponentProps> = ({
  prop1,
  prop2,
}) => {
  // 1. Estado
  const [state, setState] = useState()

  // 2. Hooks
  const { currentTenant } = useTenant()

  // 3. Efectos
  useEffect(() => {
    // ...
  }, [currentTenant])

  // 4. Handlers
  const handleClick = () => {
    // ...
  }

  // 5. Early returns
  if (!currentTenant) return <TenantRequired />

  // 6. Render
  return (
    <div className={styles.container} data-tenant={currentTenant.slug}>
      {/* Content */}
    </div>
  )
}
```

---

<div align="center">

**Estas convenciones NO son negociables**

</div>
