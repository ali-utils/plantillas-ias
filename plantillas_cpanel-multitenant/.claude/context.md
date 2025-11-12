# Contexto del Proyecto - bMOI CPanel Multi-Tenant BO

> **Contexto completo para Claude Code y otros asistentes de IA**
> Última actualización: Noviembre 2025

---

## 🏢 Información del Proyecto

```yaml
Nombre: bMOI CPanel Multi-Tenant BO
Logo: bMOI
Tipo: Panel de Control Multi-Tenant Empresarial
Versión: 1.0.0
Fecha Inicio: Noviembre 2025
Estado: En Desarrollo - Fase 1
```

---

## 🎯 Objetivo Principal

Crear un **panel de control multi-tenant profesional** que permita a múltiples organizaciones gestionar de forma independiente y segura:

- 👥 Usuarios y roles
- 📦 Productos y catálogos
- 🛒 Pedidos y transacciones
- 📊 Métricas y reportes
- ⚙️ Configuraciones personalizadas

---

## 🎨 Identidad Visual

### Paleta de Colores

```typescript
// Colores Principales
Primary:   #2563EB  // Blue-600 - Confianza, profesionalismo
Secondary: #7C3AED  // Violet-600 - Innovación, tecnología

// Semánticos
Success:   #059669  // Emerald-600
Warning:   #D97706  // Amber-600
Danger:    #DC2626  // Red-600

// Colores por Plan
Free:      #6B7280  // Gray-500
Plan1:     #3B82F6  // Blue-500
Plan2:     #7C3AED  // Violet-600
Plan3:     #F59E0B  // Amber-500
```

### Logo

```
bMOI - Simple, profesional, sin decoraciones
```

---

## 🐳 Entorno de Desarrollo: DOCKER (OBLIGATORIO)

**⚠️ CRÍTICO: Este proyecto SE DESARROLLA COMPLETAMENTE EN DOCKER.**

```bash
# ✅ SIEMPRE usa Docker para desarrollo
docker compose --profile dev up -d

# ❌ NUNCA ejecutes comandos directamente en tu máquina
npm install  # ❌ NO
npm run dev  # ❌ NO

# ✅ Ejecuta comandos DENTRO del container
docker compose exec frontend-dev npm install [package]
docker compose exec frontend-dev npm run lint
```

**Razones:**
- Consistencia: Mismo entorno para todos
- Onboarding: 5 minutos para ser productivo
- No contamina: Dependencias aisladas
- CI/CD: Mismo container en dev/staging/prod

Ver `DOCKER.md` y `TECHNICAL_ANALYSIS.md` para detalles.

## 💻 Stack Tecnológico

```typescript
// Frontend Core
React:       18.2
TypeScript:  5.3
Vite:        5.0
Docker:      ✅ OBLIGATORIO

// Estado
Zustand:         4.4    // Client state
React Query:     5.0    // Server state

// UI
Ant Design:      5.11   // Components
Tailwind CSS:    3.3    // Utility CSS

// Forms
React Hook Form: 7.48
Zod:             3.22   // Validation

// Routing
React Router:    6.20

// HTTP
Axios:           1.6    // ✅ Decisión confirmada (interceptors para multi-tenant)

// i18n
React i18next:   13.5

// Testing
Vitest:          1.0
Testing Library: 14.1
Playwright:      1.40
```

---

## 🏗️ Arquitectura Multi-Tenant

### Estrategia

```
Single Instance + Data Isolation

Frontend → Tenant Context → API Client → Header (X-Tenant-ID) → Backend
```

### Flujo de Autenticación

```
1. Login → JWT con lista de tenants
2. Selección de Tenant → Guardar en context
3. API Calls → Header X-Tenant-ID automático
4. Backend → Validar y filtrar por tenant_id
```

### Límites por Plan

```typescript
Free:   10 users,  50 products,  100 orders
Plan1:  50 users,  500 products, 1K orders
Plan2:  200 users, 2K products,  10K orders
Plan3:  Ilimitado, Ilimitado,    Ilimitado
```

---

## 📋 Convenciones OBLIGATORIAS

### Nomenclatura

```typescript
// Variables y funciones: camelCase
const userName = "John"
const getUserData = () => {}

// Componentes: PascalCase
const UserProfile = () => {}

// Constantes: UPPER_SNAKE_CASE
const API_BASE_URL = "https://api.example.com"

// Interfaces: I + PascalCase
interface IUser { id: string }

// Hooks: use + camelCase
const useAuth = () => {}

// ❌ PROHIBIDO: snake_case en JS/TS
const user_name = "John"  // NUNCA
```

### Reglas Multi-Tenant

```typescript
// ✅ SIEMPRE verificar tenant
const { currentTenant } = useTenant()
if (!currentTenant) return null

// ✅ Query keys con tenant.id
queryKey: ['users', currentTenant.id]

// ✅ Validar permisos
if (!hasPermission('users.create')) return

// ✅ Verificar límites
if (!isWithinLimits('users')) {
  showUpgradeModal()
  return
}
```

---

## 📂 Estructura del Proyecto

```
src/
├── components/      # Atomic Design
│   ├── atoms/
│   ├── molecules/
│   ├── organisms/
│   └── templates/
├── pages/          # Rutas principales
├── hooks/          # Custom hooks
├── context/        # React Context
├── store/          # Zustand stores
├── services/       # API services
├── types/          # TypeScript types
├── utils/          # Utilidades
├── config/         # Configuraciones
├── locales/        # i18n
└── styles/         # Estilos globales
```

---

## 🔐 Seguridad

```typescript
// Headers obligatorios
{
  'Authorization': 'Bearer <jwt>',
  'X-Tenant-ID': '<current_tenant_id>',
  'Content-Type': 'application/json'
}

// Validación de permisos
hasPermission(permission: string): boolean

// Validación de límites
isWithinLimits(resource: string, count?: number): boolean
```

---

## 📝 Fases del Proyecto

```
✅ Fase 0: Setup y configuración inicial
🔄 Fase 1: Autenticación y Layout Multi-Tenant (En progreso)
📋 Fase 2: Dashboard con Métricas por Tenant
📋 Fase 3: Gestión de Usuarios
📋 Fase 4: Productos y Pedidos
```

---

## 🚫 Prohibiciones ABSOLUTAS

```typescript
// ❌ NO usar snake_case
const user_name = "John"

// ❌ NO usar any type
const data: any = fetchData()

// ❌ NO queries sin tenant ID
queryKey: ['users']

// ❌ NO componentes sin validar tenant
const MyComponent = () => {
  // Falta verificar currentTenant
  return <div>Content</div>
}

// ❌ NO console.log en producción
console.log('Debug')
```

---

## ✅ Patrones Correctos

```typescript
// ✅ Componente con tenant validation
export const MyComponent: React.FC = () => {
  const { currentTenant, hasPermission } = useTenant()

  if (!currentTenant) return <TenantRequired />
  if (!hasPermission('view')) return <NoPermission />

  return <div data-tenant={currentTenant.slug}>Content</div>
}

// ✅ Query con tenant
const { data } = useQuery({
  queryKey: ['users', currentTenant.id, filters],
  queryFn: () => fetchUsers(currentTenant.id, filters),
  enabled: !!currentTenant
})

// ✅ Mutation con invalidación
const mutation = useMutation({
  mutationFn: (user: IUser) => createUser(currentTenant.id, user),
  onSuccess: () => {
    queryClient.invalidateQueries(['users', currentTenant.id])
  }
})
```

---

## 🧠 Sistema Anti-Olvido

### Cada 5 Mensajes

```markdown
🧠 Checkpoint:
- Proyecto: bMOI Multi-Tenant BO
- Convenciones: camelCase, PascalCase, UPPER_SNAKE_CASE
- Multi-tenant: Siempre verificar currentTenant
- NO usar: snake_case, any type
```

### Al Final de Sesión

Actualizar `.claude/session-state.md` con:
- Fecha y hora
- Features completadas
- Decisiones tomadas
- Próximos pasos

---

## 📚 Documentos Críticos

**LEER ANTES DE PROGRAMAR:**

1. [PROJECT_INSTRUCTIONS.md](../PROJECT_INSTRUCTIONS.md) - Guía completa para IAs
2. [CODING_STANDARDS.md](../CODING_STANDARDS.md) - Estándares con ejemplos
3. [ARQUITECTURA.md](../ARQUITECTURA.md) - Decisiones técnicas
4. [START.md](../START.md) - Comenzar desarrollo con IA

---

## 🎯 Estado Actual

```yaml
Fase Actual: Fase 1 - Autenticación Multi-Tenant
Última Actualización: 2025-11-03
Componentes Creados: 0
Tests Coverage: 0%
```

---

<div align="center">

**Este contexto es tu fuente de verdad. Consúltalo frecuentemente.**

</div>
