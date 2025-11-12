# Setup Completo - Admin Web bMOI

> **Guía paso a paso para configurar la aplicación Admin Web desde cero**
> **Tiempo estimado**: 20-30 minutos

---

## 📋 Tabla de Contenidos

1. [Pre-requisitos](#pre-requisitos)
2. [Paso 1: Crear Proyecto con Vite](#paso-1-crear-proyecto-con-vite)
3. [Paso 2: Instalar Dependencias](#paso-2-instalar-dependencias)
4. [Paso 3: Configurar TypeScript](#paso-3-configurar-typescript)
5. [Paso 4: Configurar Tailwind CSS](#paso-4-configurar-tailwind-css)
6. [Paso 5: Configurar Shadcn/UI](#paso-5-configurar-shadcnui)
7. [Paso 6: Configurar ESLint y Prettier](#paso-6-configurar-eslint-y-prettier)
8. [Paso 7: Configurar Router](#paso-7-configurar-router)
9. [Paso 8: Configurar API Client](#paso-8-configurar-api-client)
10. [Paso 9: Crear Estructura de Carpetas](#paso-9-crear-estructura-de-carpetas)
11. [Paso 10: Configurar Variables de Entorno](#paso-10-configurar-variables-de-entorno)
12. [Paso 11: Primer Inicio](#paso-11-primer-inicio)
13. [Verificación Final](#verificación-final)

---

## 🔧 Pre-requisitos

Verificar que tienes instalado:

```bash
node --version    # >= 20.x
npm --version     # >= 10.x
git --version     # >= 2.x
```

---

## Paso 1: Crear Proyecto con Vite

### 1.1 Crear proyecto

```bash
# Navegar a la carpeta de proyectos
cd D:\code\pBmoi

# Crear proyecto con Vite y template React + TypeScript
npm create vite@latest bo-admin-web -- --template react-ts

# Entrar al proyecto
cd admin-web
```

### 1.2 Instalar dependencias base

```bash
npm install
```

### 1.3 Verificar que funciona

```bash
npm run dev
```

Debería abrir en `http://localhost:5173`

---

## Paso 2: Instalar Dependencias

### 2.1 Routing

```bash
npm install react-router-dom
npm install -D @types/react-router-dom
```

### 2.2 State Management

```bash
# Zustand (state global ligero)
npm install zustand

# React Query (server state)
npm install @tanstack/react-query
npm install @tanstack/react-query-devtools
```

### 2.3 HTTP Client

```bash
npm install axios
```

### 2.4 Forms & Validation

```bash
# React Hook Form
npm install react-hook-form

# Zod (schema validation)
npm install zod
npm install @hookform/resolvers
```

### 2.5 UI & Icons

```bash
# Lucide Icons
npm install lucide-react

# Class variance authority (para variants de componentes)
npm install class-variance-authority clsx tailwind-merge
```

### 2.6 Charts

```bash
npm install recharts
```

### 2.7 Utilities

```bash
# Date handling
npm install date-fns

# Toast notifications
npm install sonner

# Copy to clipboard
npm install copy-to-clipboard
```

### 2.8 Development

```bash
npm install -D @types/node
```

---

## Paso 3: Configurar TypeScript

### 3.1 Actualizar `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    /* Path aliases */
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/types/*": ["./src/types/*"],
      "@/features/*": ["./src/features/*"],
      "@/layouts/*": ["./src/layouts/*"],
      "@/pages/*": ["./src/pages/*"],
      "@/styles/*": ["./src/styles/*"],
      "@/assets/*": ["./src/assets/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 3.2 Actualizar `vite.config.ts` para path aliases

```typescript
import path from 'path';
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@/components': path.resolve(__dirname, './src/components'),
      '@/lib': path.resolve(__dirname, './src/lib'),
      '@/hooks': path.resolve(__dirname, './src/hooks'),
      '@/types': path.resolve(__dirname, './src/types'),
      '@/features': path.resolve(__dirname, './src/features'),
      '@/layouts': path.resolve(__dirname, './src/layouts'),
      '@/pages': path.resolve(__dirname, './src/pages'),
      '@/styles': path.resolve(__dirname, './src/styles'),
      '@/assets': path.resolve(__dirname, './src/assets'),
    },
  },
});
```

---

## Paso 4: Configurar Tailwind CSS

### 4.1 Instalar Tailwind CSS

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 4.2 Configurar `tailwind.config.js`

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: ['class'],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: '2rem',
      screens: {
        '2xl': '1400px',
      },
    },
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      keyframes: {
        'accordion-down': {
          from: { height: 0 },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: 0 },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};
```

### 4.3 Instalar plugin de animaciones

```bash
npm install -D tailwindcss-animate
```

### 4.4 Crear `src/styles/globals.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

### 4.5 Importar en `src/main.tsx`

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.tsx';
import './styles/globals.css'; // <-- Importar aquí

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

## Paso 5: Configurar Shadcn/UI

### 5.1 Inicializar Shadcn/UI

```bash
npx shadcn-ui@latest init
```

Responder:
- **Style**: Default
- **Base color**: Slate
- **CSS variables**: Yes

### 5.2 Instalar componentes base

```bash
# UI Components
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
npx shadcn-ui@latest add label
npx shadcn-ui@latest add table
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add select
npx shadcn-ui@latest add checkbox
npx shadcn-ui@latest add switch
npx shadcn-ui@latest add tabs
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add avatar
npx shadcn-ui@latest add toast
npx shadcn-ui@latest add skeleton
npx shadcn-ui@latest add alert
npx shadcn-ui@latest add breadcrumb
```

---

## Paso 6: Configurar ESLint y Prettier

### 6.1 Instalar ESLint

```bash
npm install -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

### 6.2 Crear `.eslintrc.cjs`

```javascript
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs'],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh'],
  rules: {
    'react-refresh/only-export-components': [
      'warn',
      { allowConstantExport: true },
    ],
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'no-console': ['warn', { allow: ['warn', 'error'] }],
  },
};
```

### 6.3 Instalar Prettier

```bash
npm install -D prettier eslint-config-prettier eslint-plugin-prettier
```

### 6.4 Crear `.prettierrc`

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### 6.5 Agregar scripts en `package.json`

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css,md}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,css,md}\"",
    "preview": "vite preview"
  }
}
```

---

## Paso 7: Configurar Router

### 7.1 Crear `src/app/routes.tsx`

```typescript
import { createBrowserRouter, Navigate } from 'react-router-dom';
import { DashboardLayout } from '@/layouts/DashboardLayout';
import { AuthLayout } from '@/layouts/AuthLayout';

// Pages
import { LoginPage } from '@/pages/auth/LoginPage';
import { DashboardPage } from '@/pages/dashboard/DashboardPage';
import { TenantsListPage } from '@/pages/tenants/TenantsListPage';
import { UsersListPage } from '@/pages/users/UsersListPage';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <Navigate to="/dashboard" replace />,
  },
  {
    path: '/auth',
    element: <AuthLayout />,
    children: [
      {
        path: 'login',
        element: <LoginPage />,
      },
    ],
  },
  {
    path: '/',
    element: <DashboardLayout />,
    children: [
      {
        path: 'dashboard',
        element: <DashboardPage />,
      },
      {
        path: 'tenants',
        element: <TenantsListPage />,
      },
      {
        path: 'users',
        element: <UsersListPage />,
      },
    ],
  },
]);
```

### 7.2 Actualizar `src/App.tsx`

```typescript
import { RouterProvider } from 'react-router-dom';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { router } from './app/routes';
import { Toaster } from 'sonner';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false,
      retry: 1,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
      <Toaster position="top-right" richColors />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}

export default App;
```

---

## Paso 8: Configurar API Client

### 8.1 Crear `src/lib/api/client.ts`

```typescript
import axios, { AxiosError, InternalAxiosRequestConfig } from 'axios';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api/v1',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor
apiClient.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    // Agregar JWT token
    const token = localStorage.getItem('access_token');
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    // Agregar X-Tenant-ID header
    const tenantId = localStorage.getItem('selected_tenant_id');
    if (tenantId && config.headers) {
      config.headers['X-Tenant-ID'] = tenantId;
    }

    return config;
  },
  (error: AxiosError) => {
    return Promise.reject(error);
  }
);

// Response interceptor
apiClient.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    const originalRequest = error.config as InternalAxiosRequestConfig & { _retry?: boolean };

    // Handle 401 Unauthorized
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Intentar refresh token
        const refreshToken = localStorage.getItem('refresh_token');
        const response = await axios.post(
          `${import.meta.env.VITE_API_URL}/auth/refresh`,
          { refreshToken }
        );

        const { accessToken } = response.data;
        localStorage.setItem('access_token', accessToken);

        // Retry original request
        if (originalRequest.headers) {
          originalRequest.headers.Authorization = `Bearer ${accessToken}`;
        }
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Refresh failed, logout
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        window.location.href = '/auth/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);

export default apiClient;
```

### 8.2 Crear `src/lib/api/endpoints.ts`

```typescript
export const ENDPOINTS = {
  // Auth
  LOGIN: '/auth/login',
  LOGOUT: '/auth/logout',
  REFRESH: '/auth/refresh',
  ME: '/auth/me',

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

## Paso 9: Crear Estructura de Carpetas

```bash
# Crear carpetas principales
mkdir -p src/{pages,components,layouts,features,lib,hooks,types,styles,assets}

# Páginas
mkdir -p src/pages/{auth,dashboard,tenants,users,artesanias,transactions,audit,settings}

# Componentes
mkdir -p src/components/{ui,common,forms,charts}

# Features
mkdir -p src/features/{auth,tenants,users}/{hooks,services,types,store}

# Lib
mkdir -p src/lib/{api,utils,constants}

# Layouts
mkdir -p src/layouts

# Assets
mkdir -p src/assets/{images,icons}
```

---

## Paso 10: Configurar Variables de Entorno

### 10.1 Crear `.env.example`

```bash
# API
VITE_API_URL=http://localhost:3000/api/v1

# App
VITE_APP_NAME=bMOI Admin
VITE_APP_VERSION=1.0.0

# Features
VITE_ENABLE_DEVTOOLS=true
VITE_ENABLE_MOCK_API=false

# Sentry (opcional)
VITE_SENTRY_DSN=
```

### 10.2 Crear `.env.development`

```bash
VITE_API_URL=http://localhost:3000/api/v1
VITE_ENABLE_DEVTOOLS=true
VITE_ENABLE_MOCK_API=false
```

### 10.3 Crear `.env.production`

```bash
VITE_API_URL=https://api.bmoi.com/api/v1
VITE_ENABLE_DEVTOOLS=false
VITE_ENABLE_MOCK_API=false
```

### 10.4 Crear archivo local `.env`

```bash
cp .env.example .env
```

---

## Paso 11: Primer Inicio

### 11.1 Levantar en desarrollo

```bash
npm run dev
```

Debería abrir en `http://localhost:5173`

### 11.2 Verificar que funciona

- [ ] El servidor levanta sin errores
- [ ] La página carga correctamente
- [ ] Tailwind CSS está funcionando
- [ ] No hay errores en consola

---

## Verificación Final

### ✅ Checklist

```bash
# 1. Dependencias instaladas
npm list react react-dom react-router-dom zustand @tanstack/react-query axios

# 2. TypeScript configurado
cat tsconfig.json

# 3. Tailwind CSS configurado
cat tailwind.config.js

# 4. ESLint funciona
npm run lint

# 5. Prettier funciona
npm run format:check

# 6. Build funciona
npm run build

# 7. Preview funciona
npm run preview
```

### Estructura de Archivos

```
admin-web/
├── src/
│   ├── app/
│   │   └── routes.tsx
│   ├── pages/
│   ├── components/
│   │   └── ui/        (shadcn components)
│   ├── layouts/
│   ├── features/
│   ├── lib/
│   │   ├── api/
│   │   │   ├── client.ts
│   │   │   └── endpoints.ts
│   │   ├── utils/
│   │   └── constants/
│   ├── hooks/
│   ├── types/
│   ├── styles/
│   │   └── globals.css
│   ├── assets/
│   ├── App.tsx
│   └── main.tsx
├── .env
├── .env.example
├── .eslintrc.cjs
├── .prettierrc
├── tailwind.config.js
├── tsconfig.json
├── vite.config.ts
└── package.json
```

---

## 🚀 Próximos Pasos

1. **Crear primer layout** → Ver [ARQUITECTURA_FRONTEND.md](ARQUITECTURA_FRONTEND.md)
2. **Crear primera página** → Ver [GUIA_DESARROLLO.md](GUIA_DESARROLLO.md)
3. **Conectar con backend** → Ver [API_INTEGRATION.md](API_INTEGRATION.md)
4. **Ver ejemplos** → Ver [EJEMPLOS.md](EJEMPLOS.md)

---

<div align="center">

**Setup Completo! 🎉**

Ahora estás listo para desarrollar el Admin Web

**Próximo**: [Arquitectura Frontend](ARQUITECTURA_FRONTEND.md)

</div>
