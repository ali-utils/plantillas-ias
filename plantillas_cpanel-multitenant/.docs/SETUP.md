# SETUP - bMOI CPanel Multi-Tenant BO (Desde Cero)

**Guía completa paso a paso para crear el proyecto desde cero.**

---

## 📋 Tabla de Contenidos

- [Prerrequisitos](#-prerrequisitos)
- [Paso 1: Inicializar Proyecto](#-paso-1-inicializar-proyecto-vite--react--typescript)
- [Paso 2: Instalar Dependencias](#-paso-2-instalar-dependencias)
- [Paso 3: Configurar Tailwind CSS](#-paso-3-configurar-tailwind-css)
- [Paso 4: Configurar Ant Design](#-paso-4-configurar-ant-design)
- [Paso 5: Estructura de Carpetas](#-paso-5-crear-estructura-de-carpetas)
- [Paso 6: Configuración de TypeScript](#-paso-6-configurar-typescript)
- [Paso 7: Configurar ESLint + Prettier](#-paso-7-configurar-eslint--prettier)
- [Paso 8: Variables de Entorno](#-paso-8-variables-de-entorno)
- [Paso 9: Configurar Docker](#-paso-9-configurar-docker)
- [Paso 10: Primer Run](#-paso-10-primer-run)

---

## ✅ Prerrequisitos

### Software Necesario

| Software | Versión Mínima | Verificar |
|----------|----------------|-----------|
| **Node.js** | 18.0.0 | `node --version` |
| **npm** | 9.0.0 | `npm --version` |
| **Git** | 2.x | `git --version` |
| **Docker** | 24.x | `docker --version` |
| **Docker Compose** | 2.x | `docker compose version` |

### Instalación de Prerrequisitos

#### Windows

```powershell
# Node.js (con nvm-windows recomendado)
# Descargar: https://nodejs.org/en/download/

# Docker Desktop
# Descargar: https://www.docker.com/products/docker-desktop

# Git
# Descargar: https://git-scm.com/download/win
```

#### macOS

```bash
# Node.js con nvm
brew install nvm
nvm install 20
nvm use 20

# Docker Desktop
brew install --cask docker

# Git (ya viene con macOS)
```

#### Linux (Ubuntu/Debian)

```bash
# Node.js con nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20

# Docker
sudo apt-get update
sudo apt-get install docker.io docker-compose-plugin

# Git
sudo apt-get install git
```

### Verificar Instalación

```bash
node --version    # Debe mostrar v18.x.x o superior
npm --version     # Debe mostrar 9.x.x o superior
git --version     # Debe mostrar 2.x.x
docker --version  # Debe mostrar 24.x.x o superior
```

---

## 🚀 Paso 1: Inicializar Proyecto (Vite + React + TypeScript)

### 1.1 Crear Directorio del Proyecto

```bash
# Crear carpeta
mkdir cpanel-multitenant-bo
cd cpanel-multitenant-bo

# Inicializar Git
git init
```

### 1.2 Crear Proyecto con Vite

```bash
# Inicializar proyecto Vite con template React + TypeScript
npm create vite@latest . -- --template react-ts

# Responder:
# ? Current directory is not empty. Remove existing files and continue? › Yes
# ✔ Scaffolding project in /path/to/cpanel-multitenant-bo...
```

### 1.3 Verificar Estructura Inicial

```
cpanel-multitenant-bo/
├── index.html
├── package.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
├── public/
│   └── vite.svg
└── src/
    ├── App.css
    ├── App.tsx
    ├── index.css
    ├── main.tsx
    └── vite-env.d.ts
```

### 1.4 Primera Instalación

```bash
npm install
```

### 1.5 Probar que Funciona

```bash
npm run dev
```

Abrir http://localhost:5173 → Deberías ver la página de inicio de Vite.

**✅ Checkpoint:** Si ves la página de Vite funcionando, continúa al Paso 2.

---

## 📦 Paso 2: Instalar Dependencias

### 2.1 Dependencias de Producción

```bash
npm install \
  react-router-dom \
  axios \
  zustand \
  @tanstack/react-query \
  antd \
  react-hook-form \
  zod \
  @hookform/resolvers \
  dayjs \
  recharts \
  react-i18next \
  i18next
```

**¿Qué hace cada una?**
- `react-router-dom` → Routing
- `axios` → HTTP client
- `zustand` → State management (client state)
- `@tanstack/react-query` → Server state & caching
- `antd` → Ant Design components
- `react-hook-form` → Form management
- `zod` → Schema validation
- `@hookform/resolvers` → Integración zod + react-hook-form
- `dayjs` → Date manipulation
- `recharts` → Charts
- `react-i18next` + `i18next` → Internationalization

### 2.2 Dependencias de Desarrollo

```bash
npm install -D \
  tailwindcss \
  postcss \
  autoprefixer \
  @types/node \
  eslint \
  @typescript-eslint/eslint-plugin \
  @typescript-eslint/parser \
  prettier \
  eslint-config-prettier \
  eslint-plugin-prettier \
  eslint-plugin-react \
  eslint-plugin-react-hooks \
  @vitejs/plugin-react \
  vitest \
  @testing-library/react \
  @testing-library/jest-dom \
  @testing-library/user-event \
  jsdom
```

**¿Qué hace cada una?**
- `tailwindcss` + `postcss` + `autoprefixer` → Tailwind CSS
- `@types/node` → Types para Node.js
- `eslint` + plugins → Linting
- `prettier` + plugins → Code formatting
- `vitest` + `@testing-library/*` → Testing
- `jsdom` → DOM simulation para tests

---

## 🎨 Paso 3: Configurar Tailwind CSS

### 3.1 Inicializar Tailwind

```bash
npx tailwindcss init -p
```

Esto crea:
- `tailwind.config.js`
- `postcss.config.js`

### 3.2 Configurar `tailwind.config.js`

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      colors: {
        // Colores del design system
        primary: {
          50: '#EFF6FF',
          100: '#DBEAFE',
          200: '#BFDBFE',
          300: '#93C5FD',
          400: '#60A5FA',
          500: '#3B82F6',
          600: '#2563EB',  // Principal
          700: '#1D4ED8',
          800: '#1E40AF',
          900: '#1E3A8A',
        },
        secondary: {
          50: '#F5F3FF',
          100: '#EDE9FE',
          200: '#DDD6FE',
          300: '#C4B5FD',
          400: '#A78BFA',
          500: '#8B5CF6',
          600: '#7C3AED',  // Secundario
          700: '#6D28D9',
          800: '#5B21B6',
          900: '#4C1D95',
        },
      },
    },
  },
  plugins: [],
}
```

### 3.3 Actualizar `src/index.css`

**Borrar todo** el contenido existente y reemplazar con:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom CSS global (opcional) */
@layer base {
  html,
  body,
  #root {
    @apply h-full;
  }
}
```

---

## 🐜 Paso 4: Configurar Ant Design

### 4.1 Crear `src/config/antd-theme.ts`

```bash
mkdir -p src/config
```

```typescript
// src/config/antd-theme.ts
import type { ThemeConfig } from 'antd';

export const antdTheme: ThemeConfig = {
  token: {
    // Color primario (blue-600)
    colorPrimary: '#2563EB',

    // Color secundario
    colorInfo: '#7C3AED',

    // Estados
    colorSuccess: '#059669',
    colorWarning: '#D97706',
    colorError: '#DC2626',

    // Border radius
    borderRadius: 8,

    // Font
    fontFamily: "'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif",
  },
  components: {
    Button: {
      controlHeight: 40,
      fontSize: 14,
    },
    Input: {
      controlHeight: 40,
    },
  },
};
```

### 4.2 Actualizar `src/main.tsx`

```typescript
import React from 'react';
import ReactDOM from 'react-dom/client';
import { ConfigProvider } from 'antd';
import { antdTheme } from './config/antd-theme';
import App from './App';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <ConfigProvider theme={antdTheme}>
      <App />
    </ConfigProvider>
  </React.StrictMode>,
);
```

---

## 📁 Paso 5: Crear Estructura de Carpetas

### 5.1 Crear Carpetas Base

```bash
# Desde la raíz del proyecto
mkdir -p src/{components/{atoms,molecules,organisms,templates},pages,hooks,context,services/{api},store,types,utils,config,styles,locales/{en,es}}
```

### 5.2 Estructura Resultante

```
src/
├── components/
│   ├── atoms/           # Componentes básicos (Button, Input, etc.)
│   ├── molecules/       # Componentes compuestos (Card, Form Field, etc.)
│   ├── organisms/       # Componentes complejos (Table, Header, etc.)
│   └── templates/       # Templates de página
├── pages/               # Páginas/Rutas
├── hooks/               # Custom hooks
├── context/             # React Context providers
├── services/
│   └── api/             # API clients
├── store/               # Zustand stores
├── types/               # TypeScript types
├── utils/               # Utilidades
├── config/              # Configuración
├── styles/              # CSS/SCSS global
└── locales/             # Traducciones i18n
    ├── en/
    └── es/
```

### 5.3 Crear Archivos Iniciales

```bash
# Types
cat > src/types/common.types.ts << 'EOF'
// Common types
export interface ITenant {
  id: string;
  name: string;
  slug: string;
  plan: 'free' | 'plan1' | 'plan2' | 'plan3';
  createdAt: string;
}

export interface IUser {
  id: string;
  email: string;
  name: string;
  role: 'owner' | 'admin' | 'editor' | 'viewer';
  avatar?: string;
}

export interface ITenantContext {
  id: string;
  name: string;
  plan: string;
  role: string;
  permissions: string[];
}
EOF

# API Client base
cat > src/services/api/client.ts << 'EOF'
import axios from 'axios';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api/v1',
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor (add auth token)
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('accessToken');
    const tenantId = localStorage.getItem('currentTenantId');

    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    if (tenantId) {
      config.headers['X-Tenant-ID'] = tenantId;
    }

    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor (handle errors)
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // Token expired - redirect to login
      localStorage.removeItem('accessToken');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);
EOF
```

---

## 🔧 Paso 6: Configurar TypeScript

### 6.1 Actualizar `tsconfig.json`

Reemplazar contenido con:

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
      "@/*": ["./src/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 6.2 Actualizar `vite.config.ts`

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    port: 3000,
    host: true,
  },
});
```

---

## 🔍 Paso 7: Configurar ESLint + Prettier

### 7.1 Crear `.eslintrc.cjs`

```javascript
module.exports = {
  root: true,
  env: {
    browser: true,
    es2020: true,
  },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react/jsx-runtime',
    'plugin:react-hooks/recommended',
    'prettier',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs'],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module',
    project: ['./tsconfig.json', './tsconfig.node.json'],
    tsconfigRootDir: __dirname,
  },
  plugins: ['react', '@typescript-eslint', 'prettier'],
  rules: {
    'react/react-in-jsx-scope': 'off',
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-unused-vars': [
      'warn',
      { argsIgnorePattern: '^_' },
    ],
    'prettier/prettier': 'error',
  },
  settings: {
    react: {
      version: 'detect',
    },
  },
};
```

### 7.2 Crear `.prettierrc`

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### 7.3 Crear `.prettierignore`

```
dist
node_modules
.env
.env.local
coverage
```

### 7.4 Actualizar `package.json` con Scripts

Agregar en la sección `"scripts"`:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css,md}\"",
    "type-check": "tsc --noEmit",
    "validate": "npm run lint && npm run type-check && npm run test",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  }
}
```

---

## 🔐 Paso 8: Variables de Entorno

### 8.1 Crear `.env.example`

```bash
# API
VITE_API_BASE_URL=http://localhost:3000/api/v1

# App
VITE_APP_NAME=bMOI CPanel

# Multi-tenant
VITE_TENANT_HEADER=X-Tenant-ID

# Features
VITE_ENABLE_DEV_LOGS=true
```

### 8.2 Crear `.env.local`

```bash
cp .env.example .env.local
```

Editar `.env.local` con valores locales.

### 8.3 Crear `.gitignore`

```
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

node_modules
dist
dist-ssr
*.local

# Editor
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?

# Environment
.env.local
.env.*.local

# Testing
coverage
```

---

## 🐳 Paso 9: Configurar Docker

### 9.1 Crear `Dockerfile`

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build
RUN npm run build

# Stage 2: Production
FROM nginx:alpine

# Copy built assets
COPY --from=builder /app/dist /usr/share/nginx/html

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### 9.2 Crear `nginx.conf`

```nginx
server {
  listen 80;
  server_name localhost;

  root /usr/share/nginx/html;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }

  # Cache static assets
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
  }
}
```

### 9.3 Crear `docker-compose.yml`

```yaml
version: '3.8'

services:
  # Development profile
  frontend-dev:
    profiles: ['dev']
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - '3000:3000'
    volumes:
      - ./src:/app/src
      - ./public:/app/public
      - ./index.html:/app/index.html
      - /app/node_modules
    environment:
      - VITE_API_BASE_URL=http://localhost:5000/api/v1
    command: npm run dev

  # Production profile
  frontend-prod:
    profiles: ['prod']
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '80:80'
```

### 9.4 Crear `Dockerfile.dev`

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "run", "dev", "--", "--host"]
```

---

## ✅ Paso 10: Primer Run

### 10.1 Sin Docker (Local)

```bash
# Instalar dependencias
npm install

# Verificar que no hay errores de TypeScript
npm run type-check

# Verificar linting
npm run lint

# Iniciar desarrollo
npm run dev
```

Abrir http://localhost:3000

### 10.2 Con Docker (Recomendado)

```bash
# Construir e iniciar
docker compose --profile dev up -d

# Ver logs
docker compose logs -f frontend-dev

# Verificar está corriendo
curl http://localhost:3000
```

Abrir http://localhost:3000

---

## 🎉 Verificación Final

### Checklist de Completitud

- [ ] Proyecto corre sin errores (`npm run dev`)
- [ ] TypeScript compila sin errores (`npm run type-check`)
- [ ] ESLint pasa sin errores (`npm run lint`)
- [ ] Estructura de carpetas creada
- [ ] Variables de entorno configuradas
- [ ] Docker funciona (`docker compose --profile dev up`)
- [ ] Puedes acceder a http://localhost:3000
- [ ] Imports con `@/` funcionan

### Comandos de Verificación

```bash
# Verificar todo
npm run validate

# Debería mostrar:
# ✓ ESLint: 0 errors
# ✓ TypeScript: 0 errors
# ✓ Tests: passing
```

---

## 🚀 Próximos Pasos

Ahora que el proyecto está configurado:

1. **Leer documentación:**
   - [START.md](./START.md) - Desarrollo con IA
   - [PROJECT_INSTRUCTIONS.md](./PROJECT_INSTRUCTIONS.md) - Instrucciones completas
   - [CODING_STANDARDS.md](./CODING_STANDARDS.md) - Estándares de código
   - [RESPONSIVE_DESIGN.md](./RESPONSIVE_DESIGN.md) - Diseño responsive

2. **Comenzar desarrollo:**
   ```bash
   # Generar primer componente
   npm run generate:component Button atom

   # Crear primera página
   npm run generate:page Login
   ```

3. **Implementar features:**
   - Autenticación (Login/Register)
   - Selector de Tenant
   - Dashboard
   - Gestión de Usuarios

---

## 🆘 Troubleshooting

### Puerto 3000 ocupado

```bash
# Cambiar puerto
# Opción 1: Variable de entorno
PORT=3001 npm run dev

# Opción 2: Modificar vite.config.ts
# server: { port: 3001 }
```

### Error: Cannot find module '@/*'

```bash
# Verificar tsconfig.json tiene:
# "baseUrl": "."
# "paths": { "@/*": ["./src/*"] }

# Verificar vite.config.ts tiene:
# resolve: { alias: { '@': path.resolve(__dirname, './src') } }

# Reiniciar servidor
npm run dev
```

### Errores de TypeScript

```bash
# Limpiar cache
rm -rf node_modules dist
npm install
npm run type-check
```

### Docker no levanta

```bash
# Ver logs detallados
docker compose --profile dev logs

# Reconstruir desde cero
docker compose --profile dev down -v
docker compose --profile dev build --no-cache
docker compose --profile dev up
```

---

## 📚 Recursos Adicionales

- [Vite Documentation](https://vitejs.dev/)
- [React Documentation](https://react.dev/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Ant Design](https://ant.design/components/overview/)
- [Docker Docs](https://docs.docker.com/)

---

**Última actualización:** 2025-01-04
**Versión:** 1.0.0
