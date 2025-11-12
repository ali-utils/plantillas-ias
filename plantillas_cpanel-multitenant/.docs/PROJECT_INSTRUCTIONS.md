# PROJECT INSTRUCTIONS - bMOI CPanel Multi-Tenant BO

> **Documento maestro para Asistentes de IA (Claude, Copilot, Cursor, y otros)**
> Versión: 2.0 | Última actualización: Noviembre 2025

---

## 📌 PROPÓSITO DE ESTE DOCUMENTO

Este archivo contiene **TODA la información crítica** que necesitas para trabajar en este proyecto de manera consistente y profesional.

### 🎯 Objetivos
1. **Mantener coherencia** entre sesiones de trabajo con diferentes IAs
2. **Prevenir regresiones** y errores de convenciones
3. **Acelerar el desarrollo** con contexto completo
4. **Documentar decisiones** arquitectónicas

### 📖 Cómo Usar Este Documento

- **Antes de empezar cualquier tarea**: Lee las secciones relevantes
- **Cada 5 mensajes**: Revisa "Checkpoints de Memoria"
- **Al finalizar sesión**: Actualiza `.claude/session-state.md`
- **Ante dudas**: Consulta este documento primero

---

## 🏢 INFORMACIÓN DEL PROYECTO

### Identidad
```yaml
Nombre: bMOI CPanel Multi-Tenant BO
Logo: bMOI
Tipo: Panel de Control Multi-Tenant Empresarial
Versión: 1.0.0
Inicio: Noviembre 2025
Equipo: Team bMOI
```

### Objetivo Principal
Crear un **panel de control multi-tenant profesional** que permita a múltiples organizaciones gestionar de forma independiente y segura sus:
- 👥 Usuarios y roles
- 📦 Productos y catálogos
- 🛒 Pedidos y transacciones
- 📊 Métricas y reportes
- ⚙️ Configuraciones y branding

### Casos de Uso
1. **E-commerce Multi-Vendor**: Marketplace con múltiples vendedores
2. **SaaS B2B**: Panel administrativo para empresas clientes
3. **Plataformas Educativas**: Gestión de instituciones educativas
4. **Sistemas Corporativos**: Multi-subsidiarias de una empresa

---

## 🎨 IDENTIDAD VISUAL Y DISEÑO

### Paleta de Colores Principal

```typescript
// Colores Primarios - Profesionalismo y Confianza
export const COLORS = {
  // Primary (Blue) - Acción principal, CTAs, navegación activa
  primary: {
    50:  '#EFF6FF',
    100: '#DBEAFE',
    200: '#BFDBFE',
    300: '#93C5FD',
    400: '#60A5FA',
    500: '#3B82F6',
    600: '#2563EB', // ⭐ PRINCIPAL
    700: '#1D4ED8',
    800: '#1E40AF',
    900: '#1E3A8A',
  },

  // Secondary (Violet) - Innovación, acciones secundarias
  secondary: {
    50:  '#F5F3FF',
    100: '#EDE9FE',
    200: '#DDD6FE',
    300: '#C4B5FD',
    400: '#A78BFA',
    500: '#8B5CF6',
    600: '#7C3AED', // ⭐ SECUNDARIO
    700: '#6D28D9',
    800: '#5B21B6',
    900: '#4C1D95',
  },

  // Success (Emerald) - Estados positivos, confirmaciones
  success: {
    50:  '#ECFDF5',
    100: '#D1FAE5',
    500: '#10B981',
    600: '#059669', // ⭐ SUCCESS
    700: '#047857',
  },

  // Warning (Amber) - Advertencias, límites de plan
  warning: {
    50:  '#FFFBEB',
    100: '#FEF3C7',
    500: '#F59E0B',
    600: '#D97706', // ⭐ WARNING
    700: '#B45309',
  },

  // Danger (Red) - Errores, acciones destructivas
  danger: {
    50:  '#FEF2F2',
    100: '#FEE2E2',
    500: '#EF4444',
    600: '#DC2626', // ⭐ DANGER
    700: '#B91C1C',
  },

  // Neutrals (Slate) - Textos, fondos, bordes
  gray: {
    50:  '#F8FAFC',
    100: '#F1F5F9',
    200: '#E2E8F0',
    300: '#CBD5E1',
    400: '#94A3B8',
    500: '#64748B',
    600: '#475569',
    700: '#334155',
    800: '#1E293B',
    900: '#0F172A',
  },
}
```

### Colores por Plan (Multi-Tenant)

```typescript
export const PLAN_COLORS = {
  free: {
    color: '#6B7280',    // Gray-500
    bg: '#F3F4F6',       // Gray-100
    badge: 'Gratis',
    icon: '🆓'
  },
  plan1: {
    color: '#3B82F6',    // Blue-500
    bg: '#DBEAFE',       // Blue-100
    badge: 'Básico',
    icon: '📦'
  },
  plan2: {
    color: '#7C3AED',    // Violet-600
    bg: '#DDD6FE',       // Violet-200
    badge: 'Pro',
    icon: '⭐'
  },
  plan3: {
    color: '#F59E0B',    // Amber-500
    bg: '#FEF3C7',       // Amber-100
    badge: 'Enterprise',
    icon: '👑'
  }
}
```

### Razones de Elección de Colores

1. **Azul (#2563EB)** como primario:
   - Inspira **confianza y profesionalismo**
   - Universal en aplicaciones empresariales
   - Buen contraste con fondos claros/oscuros

2. **Violeta (#7C3AED)** como secundario:
   - Representa **innovación y tecnología**
   - Se distingue del azul sin competir
   - Moderno y diferenciador

3. **Verde Esmeralda (#059669)** para éxito:
   - Más profesional que verde brillante
   - Mejor accesibilidad (WCAG AA+)

4. **Degradados por Plan**:
   - Permite identificación visual rápida del nivel
   - Free → Enterprise va de gris a dorado

---

## 🏗️ ARQUITECTURA TÉCNICA

### Stack Tecnológico Completo

#### Frontend Core
```json
{
  "react": "^18.2.0",           // UI Library
  "react-dom": "^18.2.0",
  "typescript": "^5.3.0",       // Type Safety
  "vite": "^5.0.0"              // Build Tool
}
```

#### Estado y Datos
```json
{
  "zustand": "^4.4.0",          // Client State (simple, performante)
  "@tanstack/react-query": "^5.0.0",  // Server State + Caching
  "react-hook-form": "^7.48.0", // Forms
  "zod": "^3.22.0"              // Schema Validation
}
```

#### UI/Styling
```json
{
  "antd": "^5.11.0",            // Component Library
  "tailwindcss": "^3.3.0",      // Utility CSS
  "@ant-design/icons": "^5.2.0", // Icons
  "framer-motion": "^10.16.0"   // Animations
}
```

#### Routing
```json
{
  "react-router-dom": "^6.20.0" // Client Routing
}
```

#### HTTP & API
```json
{
  "axios": "^1.6.0"             // HTTP Client
}
```

#### Internacionalización
```json
{
  "react-i18next": "^13.5.0",   // i18n
  "i18next": "^23.7.0"
}
```

#### Testing
```json
{
  "vitest": "^1.0.0",           // Unit Testing
  "@testing-library/react": "^14.1.0",  // Component Testing
  "@testing-library/user-event": "^14.5.0",
  "msw": "^2.0.0",              // API Mocking
  "@playwright/test": "^1.40.0" // E2E Testing
}
```

#### DevTools
```json
{
  "eslint": "^8.54.0",
  "prettier": "^3.1.0",
  "husky": "^8.0.0",
  "lint-staged": "^15.1.0",
  "commitlint": "^18.4.0"
}
```

### Razones de Elección Técnica

#### ¿Por qué Zustand y no Redux?
```
✅ Zustand:
- API más simple (menos boilerplate)
- Mejor performance (menos re-renders)
- Bundle size: ~1KB vs ~20KB (Redux)
- Curva de aprendizaje suave
- Perfecto para state pequeño/mediano

❌ Redux:
- Demasiado complejo para este proyecto
- Overkill para multi-tenant simple
- Requiere más tiempo de desarrollo
```

#### ¿Por qué React Query?
```
✅ React Query:
- Manejo automático de cache
- Invalidación inteligente por tenant
- Optimistic updates out-of-the-box
- DevTools excelentes
- Retry y error handling automático

❌ Alternativas:
- SWR: Menos features para multi-tenant
- Apollo: Overkill (no usamos GraphQL)
```

#### ¿Por qué Ant Design?
```
✅ Ant Design:
- Componentes enterprise-ready
- Table avanzado con filtros/sorting
- Form system robusto
- Tema personalizable
- Soporte TypeScript nativo

❌ Material-UI:
- Bundle más pesado
- Look&Feel muy reconocible (menos custom)
```

---

## 🔐 ARQUITECTURA MULTI-TENANT

### Estrategia de Aislamiento

```typescript
/**
 * FLUJO MULTI-TENANT
 *
 * 1. Usuario hace login
 * 2. Backend retorna JWT con lista de tenants
 * 3. Usuario selecciona tenant activo
 * 4. Frontend guarda tenant en contexto
 * 5. Todas las requests incluyen X-Tenant-ID header
 * 6. Backend valida y filtra datos por tenant
 */

// Estructura del JWT
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
}

// Context de Tenant (SIEMPRE usar)
interface ITenantContext {
  currentTenant: ITenant | null
  tenantList: ITenant[]
  isLoading: boolean
  switchTenant: (tenantId: string) => Promise<void>
  hasPermission: (permission: string) => boolean
  isWithinLimits: (resource: string, count?: number) => boolean
  planLimits: IPlanLimits | null
}
```

### Headers API Obligatorios

```typescript
// TODAS las requests deben incluir:
{
  'Authorization': 'Bearer <jwt_token>',
  'X-Tenant-ID': '<current_tenant_id>',
  'Content-Type': 'application/json',
  'Accept-Language': 'es-ES' // Para i18n
}
```

### Límites por Plan

```typescript
export const PLAN_LIMITS = {
  free: {
    users: 10,
    products: 50,
    orders: 100,
    storage: 1024, // MB
    apiCallsPerDay: 1000,
    features: ['dashboard', 'users', 'products']
  },
  plan1: {
    users: 50,
    products: 500,
    orders: 1000,
    storage: 10240, // 10GB
    apiCallsPerDay: 10000,
    features: ['dashboard', 'users', 'products', 'orders', 'reports']
  },
  plan2: {
    users: 200,
    products: 2000,
    orders: 10000,
    storage: 51200, // 50GB
    apiCallsPerDay: 100000,
    features: ['dashboard', 'users', 'products', 'orders', 'reports', 'analytics', 'api']
  },
  plan3: {
    users: -1, // Ilimitado
    products: -1,
    orders: -1,
    storage: 512000, // 500GB
    apiCallsPerDay: -1,
    features: ['all', 'white-label', 'priority-support']
  }
} as const
```

---

## 📋 CONVENCIONES DE CÓDIGO (OBLIGATORIAS)

### 🔤 Nomenclatura

#### Variables y Funciones: camelCase
```typescript
// ✅ CORRECTO
const userName = "John Doe"
const tenantId = "tenant-123"
const isAuthenticated = true
const currentPlan = "plan1"

function calculateTotalPrice(items: ICartItem[]): number {
  return items.reduce((sum, item) => sum + item.price, 0)
}

const handleTenantSwitch = async (tenantId: string) => {
  await switchTenant(tenantId)
}

// ❌ INCORRECTO
const UserName = "John"        // PascalCase para variable
const user_name = "John"       // snake_case prohibido
const TENANT_ID = "123"        // UPPER_CASE para variable
```

#### Componentes React: PascalCase
```typescript
// ✅ CORRECTO
export const UserProfile: React.FC<IUserProfileProps> = ({ user }) => {
  return <div>{user.name}</div>
}

export const TenantDashboard: React.FC = () => {}
export const PlanUpgradeModal: React.FC = () => {}

// ❌ INCORRECTO
export const userProfile = () => {}  // camelCase para componente
export const user_profile = () => {} // snake_case prohibido
```

#### Hooks Personalizados: use + camelCase
```typescript
// ✅ CORRECTO
export const useAuth = () => { /* ... */ }
export const useTenant = () => { /* ... */ }
export const useLocalStorage = (key: string) => { /* ... */ }
export const usePlanLimits = (plan: PlanType) => { /* ... */ }

// ❌ INCORRECTO
export const UseAuth = () => {}      // PascalCase incorrecto
export const use_auth = () => {}     // snake_case prohibido
```

#### Constantes: UPPER_SNAKE_CASE
```typescript
// ✅ CORRECTO
export const API_BASE_URL = "https://api.example.com"
export const MAX_RETRY_ATTEMPTS = 3
export const DEFAULT_PAGE_SIZE = 20
export const TENANT_HEADER_KEY = "X-Tenant-ID"

// ❌ INCORRECTO
export const apiBaseUrl = "..."     // camelCase para constante
export const MaxRetry = 3           // PascalCase incorrecto
```

#### Interfaces y Types: I + PascalCase
```typescript
// ✅ CORRECTO
interface IUser {
  id: string
  name: string
  email: string
  tenantId: string
}

interface ITenant {
  id: string
  name: string
  plan: PlanType
}

type PlanType = 'free' | 'plan1' | 'plan2' | 'plan3'
type UserRole = 'admin' | 'manager' | 'user'

// ❌ INCORRECTO
interface User {}               // Sin prefijo I
interface user {}               // camelCase incorrecto
type plan_type = string         // snake_case prohibido
```

### 📁 Estructura de Archivos

```
Componentes:
  ComponentName/
  ├── ComponentName.tsx         # Componente principal
  ├── ComponentName.types.ts    # Interfaces y types
  ├── ComponentName.module.css  # Estilos (si CSS Modules)
  ├── ComponentName.test.tsx    # Tests
  └── index.ts                  # Re-export

Páginas:
  PageName/
  ├── PageName.tsx
  ├── PageName.types.ts
  ├── hooks/                    # Hooks específicos de la página
  │   └── usePageData.ts
  └── components/               # Componentes locales
      └── LocalComponent.tsx
```

### 📦 Orden de Imports (ESTRICTO)

```typescript
// 1. React y hooks principales
import React, { useState, useEffect, useMemo } from 'react'

// 2. Librerías externas (alfabéticamente)
import { useQuery, useMutation } from '@tanstack/react-query'
import { Button, Card, Modal, Table } from 'antd'
import axios from 'axios'

// 3. Componentes internos (orden: atoms, molecules, organisms, templates)
import { Button as CustomButton } from '@/components/atoms/Button'
import { FormField } from '@/components/molecules/FormField'
import { Header } from '@/components/organisms/Header'
import { MainLayout } from '@/components/templates/MainLayout'

// 4. Hooks personalizados (tenant hooks primero, luego alfabético)
import { useTenant } from '@/hooks/useTenant'
import { useAuth } from '@/hooks/useAuth'
import { useDebounce } from '@/hooks/useDebounce'

// 5. Servicios, utils y config
import { tenantService } from '@/services/api/tenant.service'
import { formatDate } from '@/utils/dateHelpers'
import { API_BASE_URL } from '@/config/api.config'

// 6. Types e interfaces
import type { IUser, ITenant, IPlanLimits } from '@/types'
import type { IComponentProps } from './Component.types'

// 7. Estilos (siempre al final)
import styles from './Component.module.css'
```

### 🏗️ Estructura de Componente (TEMPLATE OBLIGATORIO)

```typescript
// ComponentName.tsx
import React, { useState, useEffect } from 'react'
import { useTenant } from '@/hooks/useTenant'
import type { IComponentProps } from './ComponentName.types'
import styles from './ComponentName.module.css'

/**
 * ComponentName - Descripción breve del componente
 *
 * @description Descripción detallada de qué hace este componente
 * y por qué existe. Incluir consideraciones multi-tenant si aplica.
 *
 * @example
 * <ComponentName
 *   prop1="value"
 *   prop2={123}
 * />
 */
export const ComponentName: React.FC<IComponentProps> = ({
  prop1,
  prop2,
  onAction,
  ...rest
}) => {
  // ========================================
  // 1. HOOKS DE ESTADO
  // ========================================
  const [isLoading, setIsLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  // ========================================
  // 2. HOOKS PERSONALIZADOS Y CONTEXT
  // ========================================
  const { currentTenant, hasPermission } = useTenant()
  const { user } = useAuth()

  // ========================================
  // 3. REACT QUERY (si aplica)
  // ========================================
  const { data, isLoading: queryLoading } = useQuery({
    queryKey: ['resource', currentTenant?.id],
    queryFn: () => fetchData(currentTenant!.id),
    enabled: !!currentTenant
  })

  // ========================================
  // 4. EFECTOS
  // ========================================
  useEffect(() => {
    if (!currentTenant) return

    // Lógica específica del tenant
    console.log('Tenant changed:', currentTenant.name)

    // Cleanup si es necesario
    return () => {
      console.log('Cleanup')
    }
  }, [currentTenant])

  // ========================================
  // 5. HANDLERS (usar handle + Acción)
  // ========================================
  const handleClick = () => {
    if (!hasPermission('action.execute')) {
      console.warn('Usuario sin permisos')
      return
    }

    onAction?.()
  }

  const handleSubmit = async (data: FormData) => {
    setIsLoading(true)
    try {
      await submitData(data)
    } catch (err) {
      setError(err.message)
    } finally {
      setIsLoading(false)
    }
  }

  // ========================================
  // 6. RENDER HELPERS (si son complejos)
  // ========================================
  const renderContent = () => {
    if (isLoading) return <Spinner />
    if (error) return <ErrorMessage message={error} />
    if (!currentTenant) return <TenantSelector />

    return <ActualContent data={data} />
  }

  // ========================================
  // 7. EARLY RETURNS (si aplica)
  // ========================================
  if (!currentTenant) {
    return <div>Por favor selecciona un tenant</div>
  }

  if (!hasPermission('component.view')) {
    return <NoPermission />
  }

  // ========================================
  // 8. RETURN PRINCIPAL
  // ========================================
  return (
    <div
      className={styles.container}
      data-tenant={currentTenant.slug}
      data-plan={currentTenant.plan}
      {...rest}
    >
      {renderContent()}
    </div>
  )
}

// Solo exportar como default si es una PÁGINA
// ComponentName.displayName = 'ComponentName' // Para DevTools
```

---

## 🧪 TESTING OBLIGATORIO

### Cobertura Mínima
- **Unit Tests**: 80%
- **Integration Tests**: 60%
- **E2E Tests**: Flujos críticos

### Template de Test

```typescript
// ComponentName.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { TenantProvider } from '@/context/TenantContext'
import { ComponentName } from './ComponentName'

// Mock de tenant para tests
const mockTenant: ITenant = {
  id: 'tenant-test-123',
  name: 'Test Tenant',
  slug: 'test-tenant',
  plan: 'plan1',
  status: 'active',
  permissions: ['users.read', 'users.write']
}

// Helper para renderizar con providers
const renderWithProviders = (
  component: React.ReactElement,
  tenant: ITenant | null = mockTenant
) => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  })

  return render(
    <QueryClientProvider client={queryClient}>
      <TenantProvider initialTenant={tenant}>
        {component}
      </TenantProvider>
    </QueryClientProvider>
  )
}

describe('ComponentName', () => {
  it('debe renderizar correctamente con tenant', () => {
    renderWithProviders(<ComponentName prop1="test" />)

    expect(screen.getByText(/test/i)).toBeInTheDocument()
  })

  it('debe mostrar mensaje cuando no hay tenant', () => {
    renderWithProviders(<ComponentName />, null)

    expect(screen.getByText(/selecciona un tenant/i)).toBeInTheDocument()
  })

  it('debe validar permisos antes de mostrar contenido', () => {
    const tenantSinPermisos = {
      ...mockTenant,
      permissions: []
    }

    renderWithProviders(<ComponentName />, tenantSinPermisos)

    expect(screen.getByText(/sin permisos/i)).toBeInTheDocument()
  })

  it('debe manejar acciones con tenant correcto', async () => {
    const onAction = vi.fn()

    renderWithProviders(<ComponentName onAction={onAction} />)

    const button = screen.getByRole('button', { name: /acción/i })
    fireEvent.click(button)

    await waitFor(() => {
      expect(onAction).toHaveBeenCalledTimes(1)
    })
  })
})
```

---

## 🚫 PROHIBICIONES ABSOLUTAS

### ❌ NUNCA HACER ESTO

```typescript
// ❌ 1. NO usar snake_case en variables/funciones
const user_name = "John"
const get_user_data = () => {}

// ❌ 2. NO usar `any` type
const data: any = fetchData()
function process(input: any): any {}

// ❌ 3. NO console.log en código de producción
console.log('Debug info')  // Usar logger service

// ❌ 4. NO mutar estado directamente
state.user.name = "New Name"  // Usar setState

// ❌ 5. NO inline styles (excepto valores dinámicos)
<div style={{ color: 'red', fontSize: 16 }}>

// ❌ 6. NO componentes > 300 líneas
export const HugeComponent = () => {
  // ... 500 líneas de código
}

// ❌ 7. NO queries sin tenant ID
queryKey: ['users']  // Falta tenant ID

// ❌ 8. NO requests sin verificar tenant
await fetchUsers()  // Falta context de tenant

// ❌ 9. NO mezclar datos entre tenants
const allData = [...tenant1Data, ...tenant2Data]

// ❌ 10. NO hardcodear IDs de tenant
if (tenantId === 'abc-123')  // Usar contexto
```

### ✅ EN SU LUGAR, HACER ESTO

```typescript
// ✅ 1. camelCase correcto
const userName = "John"
const getUserData = () => {}

// ✅ 2. Tipos específicos
const data: IUserData = fetchData()
function process(input: IInput): IOutput {}

// ✅ 3. Logger service
import { logger } from '@/utils/logger'
logger.debug('Debug info', { context })

// ✅ 4. Immutability
setState(prev => ({ ...prev, user: { ...prev.user, name: "New" } }))

// ✅ 5. CSS Modules o Tailwind
<div className={styles.error}>
<div className="text-red-600">

// ✅ 6. Componentes pequeños
export const SmallComponent = () => {
  // ... max 100 líneas
}

// ✅ 7. Query keys con tenant
queryKey: ['users', currentTenant.id]

// ✅ 8. Verificar tenant siempre
if (!currentTenant) return null
await fetchUsers(currentTenant.id)

// ✅ 9. Datos separados por tenant
const tenantData = data.filter(d => d.tenantId === currentTenant.id)

// ✅ 10. Usar contexto
const { currentTenant } = useTenant()
if (currentTenant.plan === 'plan3')
```

---

## 🧠 SISTEMA ANTI-OLVIDO PARA IAs

### Checkpoints de Memoria

**CADA 5 MENSAJES**, ejecuta mentalmente este checklist:

```markdown
## Checkpoint de Memoria

### ✅ Contexto del Proyecto
- [ ] Nombre del proyecto: bMOI CPanel Multi-Tenant BO
- [ ] Logo: bMOI
- [ ] Stack: React + TypeScript + Vite
- [ ] Arquitectura: Multi-Tenant con aislamiento de datos

### ✅ Convenciones Activas
- [ ] Variables: camelCase
- [ ] Componentes: PascalCase
- [ ] Constantes: UPPER_SNAKE_CASE
- [ ] Interfaces: I + PascalCase
- [ ] NO usar snake_case NUNCA
- [ ] NO usar `any` type

### ✅ Reglas Multi-Tenant
- [ ] Siempre verificar currentTenant antes de queries
- [ ] Incluir tenant.id en query keys de React Query
- [ ] Validar permisos con hasPermission()
- [ ] Verificar límites con isWithinLimits()
- [ ] Headers X-Tenant-ID en todas las requests

### ✅ Testing
- [ ] Cobertura mínima: 80%
- [ ] Tests con tenant context
- [ ] No usar any en mocks

### ✅ Estado Actual (actualizar)
- Última feature: ___________
- Componentes creados: ___________
- Bugs conocidos: ___________
- Próximo paso: ___________
```

### Recordatorios Automáticos

Antes de **cada respuesta**, verifica:

1. ✅ ¿Estoy usando las convenciones correctas?
2. ✅ ¿He verificado el tenant context?
3. ✅ ¿Los tipos están correctamente definidos?
4. ✅ ¿He incluido tests si es código nuevo?
5. ✅ ¿He documentado decisiones importantes?

### Frases Trigger para Revisar Contexto

Si el usuario dice:
- "Recuérdame..." → Lee este documento completo
- "¿Cuáles son las convenciones?" → Sección CONVENCIONES
- "¿Cómo funciona multi-tenant?" → Sección ARQUITECTURA MULTI-TENANT
- "Empecemos de nuevo" → Lee START.md y este documento

---

## 📝 ACTUALIZACIÓN DE DOCUMENTACIÓN

### Al Finalizar Cada Sesión

Actualiza `.claude/session-state.md` con:

```markdown
## Última Actualización
- Fecha: YYYY-MM-DD HH:mm
- IA: Claude/Copilot/Cursor
- Sprint: Sprint X
- Fase: Fase X

## Trabajo Realizado
- [x] Feature/componente completado
- [x] Bug corregido
- [x] Refactor realizado

## Decisiones Tomadas
1. Decisión técnica importante
2. Razón de la decisión
3. Alternativas consideradas

## Próximos Pasos
- [ ] Siguiente tarea
- [ ] Feature pendiente
- [ ] Mejora sugerida

## Problemas Encontrados
- Bug X: Descripción y solución
- Bloqueo Y: Razón y workaround

## Notas para Próxima Sesión
- Información importante a recordar
- Contexto crítico
```

---

## 🔄 FLUJO DE TRABAJO CON IA

### Inicio de Sesión

1. **Leer START.md**
2. **Revisar `.claude/session-state.md`**
3. **Verificar convenciones** en este documento
4. **Confirmar objetivos** de la sesión

### Durante el Desarrollo

1. **Cada 5 mensajes**: Checkpoint de memoria
2. **Antes de commit**: Ejecutar `npm run validate`
3. **Después de código nuevo**: Escribir tests
4. **Al cambiar arquitectura**: Documentar en ARQUITECTURA.md

### Fin de Sesión

1. **Actualizar session-state.md**
2. **Documentar decisiones** importantes
3. **Listar próximos pasos**
4. **Revisar que no queden console.log**

---

## 🎯 PRIORIDADES Y DECISIONES

### Orden de Prioridades

1. **Seguridad y Aislamiento de Datos** (Crítico)
   - Validación de tenant en todas las operaciones
   - Permisos verificados

2. **Tipo Safety** (Alto)
   - No usar `any`
   - Interfaces completas

3. **Performance** (Alto)
   - Lazy loading de rutas
   - Memoización donde corresponda

4. **Testing** (Medio-Alto)
   - 80% de cobertura

5. **UX/UI** (Medio)
   - Responsive
   - Accesibilidad WCAG AA

### Cuando Hay Conflicto

**Prioridad**: Seguridad > Performance > UX > Features

Ejemplo:
```
Si una feature cool requiere comprometer el aislamiento de datos,
LA FEATURE NO SE IMPLEMENTA hasta encontrar solución segura.
```

---

## 🆘 TROUBLESHOOTING

### Problemas Comunes y Soluciones

#### "El componente no tiene acceso al tenant"

```typescript
// ❌ Problema
export const MyComponent = () => {
  // No hay useTenant()
  return <div>Content</div>
}

// ✅ Solución
export const MyComponent = () => {
  const { currentTenant } = useTenant()

  if (!currentTenant) return <TenantRequired />

  return <div>Content for {currentTenant.name}</div>
}
```

#### "Los datos se mezclan entre tenants"

```typescript
// ❌ Problema
queryKey: ['users']  // Sin tenant ID

// ✅ Solución
queryKey: ['users', currentTenant.id]
```

#### "TypeScript muestra errores de tipo"

```typescript
// ❌ Problema
const data: any = fetchData()

// ✅ Solución
const data: IUserData = fetchData()
// O si es desconocido:
const data: unknown = fetchData()
if (isUserData(data)) {
  // Ahora TypeScript sabe que es IUserData
}
```

---

## 📚 RECURSOS ADICIONALES

### Documentos Relacionados

- **[START.md](./START.md)** - Guía paso a paso para comenzar
- **[CODING_STANDARDS.md](./CODING_STANDARDS.md)** - Estándares detallados con ejemplos
- **[ARQUITECTURA.md](./ARQUITECTURA.md)** - Decisiones técnicas profundas
- **[QUICKSTART.md](./QUICKSTART.md)** - Inicio en 5 minutos

### Links Útiles

- [React Query Docs](https://tanstack.com/query/latest)
- [Zustand Docs](https://docs.pmnd.rs/zustand)
- [Ant Design Components](https://ant.design/components)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)

---

## ✅ CHECKLIST FINAL

Antes de marcar una tarea como "completada":

- [ ] Código sigue convenciones de nomenclatura
- [ ] Imports están ordenados correctamente
- [ ] Componente verifica tenant context
- [ ] Types/Interfaces definidos (no `any`)
- [ ] Tests escritos con >80% cobertura
- [ ] No hay console.log
- [ ] Documentación inline agregada
- [ ] No supera 300 líneas por archivo
- [ ] Ejecuta `npm run validate` sin errores
- [ ] session-state.md actualizado

---

<div align="center">

**Este documento es tu fuente de verdad. Consúltalo con frecuencia.**

Última actualización: Noviembre 2025 | Versión 2.0

</div>
