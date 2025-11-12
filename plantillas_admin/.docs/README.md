# 🎨 Plantillas Admin Web - bMOI Platform

> **Plantillas completas para la aplicación Admin Web del ecosistema bMOI**
> **Framework**: React 18 + TypeScript + Vite
> **Fecha**: 2025-11-04

---

## 📋 Índice

1. [Visión General](#visión-general)
2. [Arquitectura](#arquitectura)
3. [Stack Tecnológico](#stack-tecnológico)
4. [Estructura del Proyecto](#estructura-del-proyecto)
5. [Setup Inicial](#setup-inicial)
6. [Documentación](#documentación)
7. [Integración con Backend](#integración-con-backend)

---

## 🎯 Visión General

La **Admin Web** es una aplicación de panel de control para administradores del ecosistema bMOI que permite:

- 🏢 **Gestión de Tenants**: CRUD completo de organizaciones
- 👥 **Gestión de Usuarios**: Administración de usuarios y roles
- 📊 **Dashboard Analytics**: Métricas y estadísticas en tiempo real
- 🎨 **Gestión de Artesanías**: Aprobación y moderación de contenido
- 💰 **Gestión de Transacciones**: Monitoreo de pagos y órdenes
- 🔍 **Auditoría**: Logs y trazabilidad de acciones
- ⚙️ **Configuración**: Planes, límites, y configuraciones globales

### Características Principales

- ✅ **Multi-Tenant Aware**: Administración multi-organización
- ✅ **Role-Based Access Control**: Super Admin, Admin, Moderator
- ✅ **Real-time Dashboard**: Métricas y notificaciones en tiempo real
- ✅ **Responsive Design**: Desktop-first, mobile-friendly
- ✅ **Dark Mode**: Soporte para tema claro/oscuro
- ✅ **Type-Safe**: TypeScript end-to-end
- ✅ **Offline-Ready**: Service Workers para funcionalidad offline

---

## 🏗️ Arquitectura

### Posición en el Ecosistema

```
┌─────────────────────────────────────────────────────────────┐
│                    Ecosistema bMOI                          │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────┐      ┌──────────────┐      ┌──────────────┐
    │   Landing    │      │  Web Panel   │      │  App Móvil   │
    │ React + Vite │      │ React + Vite │      │   Flutter    │
    └──────────────┘      └──────────────┘      └──────────────┘
            │                     │                     │
            └─────────────────────┴─────────────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │      Admin Web (ESTE)     │
                    │     React + TypeScript    │
                    │         + Vite            │
                    └─────────────┬─────────────┘
                                  │
                                  │ HTTPS + JWT
                                  ▼
                    ┌──────────────────────────┐
                    │   API Gateway / WAF      │
                    │     (Envoy/Nginx)        │
                    └──────────────────────────┘
                                  │
                    ┌─────────────┴──────────────┐
                    │                            │
                    ▼                            ▼
            ┌───────────────┐          ┌───────────────┐
            │ BFF Web/Móvil │          │   Backend     │
            │  (NestJS/Go)  │          │   Services    │
            └───────────────┘          └───────────────┘
```

### Arquitectura Frontend

```
┌─────────────────────────────────────────────────────────────┐
│                    Admin Web Architecture                   │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────────────┐
    │              Presentation Layer                  │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
    │  │  Pages   │  │ Layouts  │  │Components│      │
    │  └──────────┘  └──────────┘  └──────────┘      │
    └──────────────────────────────────────────────────┘
                        │
    ┌──────────────────────────────────────────────────┐
    │              Application Layer                   │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
    │  │  Hooks   │  │ Context  │  │  Store   │      │
    │  └──────────┘  └──────────┘  └──────────┘      │
    └──────────────────────────────────────────────────┘
                        │
    ┌──────────────────────────────────────────────────┐
    │               Domain Layer                       │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
    │  │ Services │  │  Models  │  │  Types   │      │
    │  └──────────┘  └──────────┘  └──────────┘      │
    └──────────────────────────────────────────────────┘
                        │
    ┌──────────────────────────────────────────────────┐
    │            Infrastructure Layer                  │
    │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
    │  │ API      │  │  Cache   │  │ Storage  │      │
    │  └──────────┘  └──────────┘  └──────────┘      │
    └──────────────────────────────────────────────────┘
```

---

## 💻 Stack Tecnológico

### Core

| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| **React** | 18.x | UI Framework |
| **TypeScript** | 5.x | Type Safety |
| **Vite** | 5.x | Build Tool |
| **React Router** | 6.x | Routing |

### State Management

| Tecnología | Propósito |
|------------|-----------|
| **Zustand** | Global State (ligero y simple) |
| **React Query** | Server State & Caching |
| **Context API** | Auth & Theme Context |

### UI & Styling

| Tecnología | Propósito |
|------------|-----------|
| **Tailwind CSS** | Utility-First Styling |
| **Shadcn/UI** | Component Library |
| **Lucide Icons** | Icon Set |
| **Recharts** | Charts & Graphs |

### Data & API

| Tecnología | Propósito |
|------------|-----------|
| **Axios** | HTTP Client |
| **Zod** | Schema Validation |
| **React Query** | Data Fetching & Caching |

### Forms & Validation

| Tecnología | Propósito |
|------------|-----------|
| **React Hook Form** | Form Management |
| **Zod** | Schema Validation |

### Testing

| Tecnología | Propósito |
|------------|-----------|
| **Vitest** | Unit Testing |
| **Testing Library** | Component Testing |
| **Playwright** | E2E Testing |
| **MSW** | API Mocking |

### DevOps

| Tecnología | Propósito |
|------------|-----------|
| **Docker** | Containerization |
| **Nginx** | Web Server |
| **GitHub Actions** | CI/CD |

---

## 📁 Estructura del Proyecto

```
admin-web/
├── public/
│   ├── favicon.ico
│   ├── logo.svg
│   └── manifest.json
│
├── src/
│   ├── app/                        # Application Core
│   │   ├── App.tsx
│   │   ├── routes.tsx
│   │   └── providers.tsx
│   │
│   ├── pages/                      # Pages (Route Components)
│   │   ├── auth/
│   │   │   ├── LoginPage.tsx
│   │   │   └── ForgotPasswordPage.tsx
│   │   ├── dashboard/
│   │   │   └── DashboardPage.tsx
│   │   ├── tenants/
│   │   │   ├── TenantsListPage.tsx
│   │   │   ├── TenantDetailPage.tsx
│   │   │   └── CreateTenantPage.tsx
│   │   ├── users/
│   │   │   ├── UsersListPage.tsx
│   │   │   ├── UserDetailPage.tsx
│   │   │   └── CreateUserPage.tsx
│   │   ├── artesanias/
│   │   │   ├── ArtesaniasListPage.tsx
│   │   │   └── ArtesaniaDetailPage.tsx
│   │   ├── transactions/
│   │   │   └── TransactionsListPage.tsx
│   │   ├── audit/
│   │   │   └── AuditLogsPage.tsx
│   │   └── settings/
│   │       └── SettingsPage.tsx
│   │
│   ├── components/                 # Reusable Components
│   │   ├── ui/                     # Base UI Components (shadcn)
│   │   │   ├── button.tsx
│   │   │   ├── input.tsx
│   │   │   ├── card.tsx
│   │   │   ├── table.tsx
│   │   │   └── ...
│   │   ├── common/                 # Common Components
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── Breadcrumbs.tsx
│   │   │   └── Loading.tsx
│   │   ├── forms/                  # Form Components
│   │   │   ├── TenantForm.tsx
│   │   │   ├── UserForm.tsx
│   │   │   └── ...
│   │   └── charts/                 # Chart Components
│   │       ├── BarChart.tsx
│   │       ├── LineChart.tsx
│   │       └── PieChart.tsx
│   │
│   ├── layouts/                    # Layout Components
│   │   ├── AuthLayout.tsx
│   │   ├── DashboardLayout.tsx
│   │   └── BlankLayout.tsx
│   │
│   ├── features/                   # Feature Modules
│   │   ├── auth/
│   │   │   ├── hooks/
│   │   │   │   ├── useAuth.ts
│   │   │   │   └── useLogin.ts
│   │   │   ├── services/
│   │   │   │   └── auth.service.ts
│   │   │   ├── types/
│   │   │   │   └── auth.types.ts
│   │   │   └── store/
│   │   │       └── authStore.ts
│   │   ├── tenants/
│   │   │   ├── hooks/
│   │   │   ├── services/
│   │   │   ├── types/
│   │   │   └── store/
│   │   └── users/
│   │       ├── hooks/
│   │       ├── services/
│   │       ├── types/
│   │       └── store/
│   │
│   ├── lib/                        # Utilities & Helpers
│   │   ├── api/
│   │   │   ├── client.ts           # Axios instance
│   │   │   ├── interceptors.ts
│   │   │   └── endpoints.ts
│   │   ├── utils/
│   │   │   ├── formatters.ts
│   │   │   ├── validators.ts
│   │   │   └── helpers.ts
│   │   └── constants/
│   │       ├── routes.ts
│   │       ├── permissions.ts
│   │       └── config.ts
│   │
│   ├── hooks/                      # Global Hooks
│   │   ├── useAuth.ts
│   │   ├── usePermissions.ts
│   │   ├── useToast.ts
│   │   └── useDebounce.ts
│   │
│   ├── types/                      # Global Types
│   │   ├── api.types.ts
│   │   ├── models.types.ts
│   │   └── common.types.ts
│   │
│   ├── styles/                     # Global Styles
│   │   ├── globals.css
│   │   └── tailwind.css
│   │
│   ├── assets/                     # Static Assets
│   │   ├── images/
│   │   └── icons/
│   │
│   └── main.tsx                    # Entry Point
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── docker/
│   ├── Dockerfile
│   ├── Dockerfile.dev
│   └── nginx.conf
│
├── .env.example
├── .env.development
├── .env.production
├── .eslintrc.cjs
├── .prettierrc
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
├── package.json
└── README.md
```

---

## 🚀 Setup Inicial

### Prerequisitos

```bash
node --version    # >= 20.x
npm --version     # >= 10.x
```

### Instalación Rápida

```bash
# 1. Crear proyecto con Vite
npm create vite@latest admin-web -- --template react-ts
cd admin-web

# 2. Instalar dependencias base
npm install

# 3. Instalar dependencias adicionales (ver SETUP_COMPLETO.md)
npm install react-router-dom zustand @tanstack/react-query axios zod

# 4. Configurar Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# 5. Instalar Shadcn/UI
npx shadcn-ui@latest init

# 6. Levantar en desarrollo
npm run dev
```

---

## 📚 Documentación

### Documentos Principales

1. **[SETUP_COMPLETO.md](SETUP_COMPLETO.md)** - Setup paso a paso completo
2. **[ARQUITECTURA_FRONTEND.md](ARQUITECTURA_FRONTEND.md)** - Arquitectura detallada
3. **[GUIA_DESARROLLO.md](GUIA_DESARROLLO.md)** - Guía de desarrollo
4. **[ESTANDARES_CODIGO.md](ESTANDARES_CODIGO.md)** - Estándares de código
5. **[COMPONENTES.md](COMPONENTES.md)** - Guía de componentes
6. **[STATE_MANAGEMENT.md](STATE_MANAGEMENT.md)** - Gestión de estado
7. **[API_INTEGRATION.md](API_INTEGRATION.md)** - Integración con API
8. **[TESTING.md](TESTING.md)** - Guía de testing
9. **[DEPLOYMENT.md](DEPLOYMENT.md)** - Guía de deployment
10. **[CHECKLIST_CALIDAD.md](CHECKLIST_CALIDAD.md)** - Checklist de calidad

### Guías Rápidas

- **[QUICK_START.md](QUICK_START.md)** - Inicio rápido (5 minutos)
- **[COMPONENTES_UI.md](COMPONENTES_UI.md)** - Componentes UI disponibles
- **[EJEMPLOS.md](EJEMPLOS.md)** - Ejemplos de código común

---

## 🔌 Integración con Backend

### Conexión con API Gateway

```typescript
// lib/api/client.ts
import axios from 'axios';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL, // http://localhost:3000/api/v1
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor - Agregar JWT token
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  // Multi-tenant: Agregar X-Tenant-ID si está disponible
  const tenantId = localStorage.getItem('selected_tenant_id');
  if (tenantId) {
    config.headers['X-Tenant-ID'] = tenantId;
  }

  return config;
});

// Response interceptor - Manejar errores
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    if (error.response?.status === 401) {
      // Token expirado, intentar refresh
      // ...
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

### Endpoints Principales

```typescript
// lib/api/endpoints.ts
export const ENDPOINTS = {
  // Auth
  LOGIN: '/auth/login',
  LOGOUT: '/auth/logout',
  REFRESH: '/auth/refresh',

  // Tenants
  TENANTS: '/tenants',
  TENANT_BY_ID: (id: string) => `/tenants/${id}`,

  // Users
  USERS: '/users',
  USER_BY_ID: (id: string) => `/users/${id}`,

  // Dashboard
  DASHBOARD_STATS: '/dashboard/stats',

  // Audit
  AUDIT_LOGS: '/audit',
} as const;
```

---

## 🎨 Convenciones de Código

### Nomenclatura

```typescript
// ✅ CORRECTO

// Componentes: PascalCase
export const UserCard = () => { }

// Archivos: kebab-case
// user-card.tsx
// auth-service.ts

// Hooks: camelCase con prefijo "use"
export const useAuth = () => { }

// Constantes: UPPER_SNAKE_CASE
export const API_BASE_URL = 'http://localhost:3000'

// Tipos e Interfaces: PascalCase
interface User { }
type AuthState = { }

// ❌ INCORRECTO
const user_card = () => { }  // snake_case
const UseAuth = () => { }    // PascalCase en hook
const api_base_url = '...'   // camelCase en constante
```

---

## 🔐 Roles y Permisos

### Roles del Admin

| Rol | Descripción | Permisos |
|-----|-------------|----------|
| **Super Admin** | Control total del sistema | Todo |
| **Admin** | Administrador de tenant | Gestión de su tenant |
| **Moderator** | Moderador de contenido | Aprobar/Rechazar contenido |

### Permisos

```typescript
export const PERMISSIONS = {
  // Tenants
  TENANTS_VIEW: 'tenants.view',
  TENANTS_CREATE: 'tenants.create',
  TENANTS_EDIT: 'tenants.edit',
  TENANTS_DELETE: 'tenants.delete',

  // Users
  USERS_VIEW: 'users.view',
  USERS_CREATE: 'users.create',
  USERS_EDIT: 'users.edit',
  USERS_DELETE: 'users.delete',

  // Artesanías
  ARTESANIAS_MODERATE: 'artesanias.moderate',

  // Transactions
  TRANSACTIONS_VIEW: 'transactions.view',

  // Audit
  AUDIT_VIEW: 'audit.view',
} as const;
```

---

## 📊 Features Principales

### 1. Dashboard

- 📈 Métricas en tiempo real
- 📊 Gráficos interactivos
- 🔔 Notificaciones
- 📋 Actividad reciente

### 2. Gestión de Tenants

- ➕ Crear nuevo tenant
- ✏️ Editar información
- 🔄 Cambiar plan
- ❌ Suspender/Activar

### 3. Gestión de Usuarios

- 👥 Lista de usuarios
- 🔍 Búsqueda y filtros
- ✏️ Editar perfiles
- 🔑 Gestión de roles

### 4. Moderación de Contenido

- 🎨 Aprobar artesanías
- ❌ Rechazar contenido
- 📝 Comentarios de revisión
- 📸 Galería de imágenes

### 5. Auditoría

- 📜 Logs de actividad
- 🔍 Búsqueda avanzada
- 📥 Exportar logs
- 📊 Reportes

---

## 🚀 Próximos Pasos

1. **Setup Completo** → [SETUP_COMPLETO.md](SETUP_COMPLETO.md)
2. **Entender Arquitectura** → [ARQUITECTURA_FRONTEND.md](ARQUITECTURA_FRONTEND.md)
3. **Desarrollar Primera Feature** → [GUIA_DESARROLLO.md](GUIA_DESARROLLO.md)
4. **Ver Ejemplos** → [EJEMPLOS.md](EJEMPLOS.md)

---

<div align="center">

**Admin Web - Panel de Control Enterprise**

React 18 | TypeScript | Vite | Tailwind CSS

Type-Safe | Modern | Responsive | Fast

**Próximo**: [Setup Completo](SETUP_COMPLETO.md)

</div>
