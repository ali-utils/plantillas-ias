# bMOI - CPanel Multi-Tenant BO

<div align="center">

![bMOI Logo](./docs/assets/logo-bmoi.svg)

**Sistema de Control Multi-Tenant Empresarial**

[![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue.svg)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-18.2-61dafb.svg)](https://reactjs.org/)
[![Vite](https://img.shields.io/badge/Vite-5.0-646cff.svg)](https://vitejs.dev/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)

[🚀 Inicio Rápido](#-inicio-rápido) • [📖 Documentación](#-documentación) • [🏗️ Arquitectura](#️-arquitectura) • [🤝 Contribuir](#-contribuir)

</div>

---

## 📋 Tabla de Contenidos

- [Descripción](#-descripción)
- [Características Principales](#-características-principales)
- [Stack Tecnológico](#-stack-tecnológico)
- [Inicio Rápido](#-inicio-rápido)
- [Estructura del Proyecto](#-estructura-del-proyecto)
- [Documentación](#-documentación)
- [Arquitectura Multi-Tenant](#️-arquitectura-multi-tenant)
- [Sistema de Memoria IA](#-sistema-de-memoria-ia)
- [Scripts Disponibles](#-scripts-disponibles)
- [Contribuir](#-contribuir)

---

## 🎯 Descripción

**bMOI CPanel Multi-Tenant BO** es un panel de control empresarial diseñado para gestionar múltiples organizaciones (tenants) de forma aislada y segura. Cada tenant puede administrar sus propios usuarios, productos, pedidos y configuraciones de manera independiente, con diferentes niveles de planes y permisos.

### ✨ Casos de Uso

- **E-commerce Multi-Vendor**: Gestión de múltiples tiendas independientes
- **SaaS B2B**: Panel administrativo para clientes empresariales
- **Marketplaces**: Control de vendedores y productos por organización
- **Plataformas Educativas**: Gestión de instituciones y cursos

---

## 🌟 Características Principales

### 🏢 Multi-Tenancy Avanzado
- ✅ **Aislamiento Total de Datos** por tenant
- ✅ **Cambio Dinámico de Tenant** sin re-login
- ✅ **Sistema de Planes**: Free, Plan1, Plan2, Plan3
- ✅ **Límites por Plan**: Usuarios, productos, almacenamiento
- ✅ **Branding Personalizado** por tenant

### 🔐 Seguridad Enterprise
- ✅ **Autenticación JWT** con refresh tokens
- ✅ **RBAC** (Role-Based Access Control)
- ✅ **Headers de Tenant** obligatorios (`X-Tenant-ID`)
- ✅ **Validación de Permisos** en frontend y backend
- ✅ **Auditoría de Acciones** por tenant

### 🌍 Internacionalización
- ✅ **Multi-idioma** (ES, EN) - Extensible
- ✅ **i18n** con React i18next
- ✅ **Formatos de Fecha/Moneda** por región
- ✅ **RTL Support** preparado

### 🎨 Sistema de Diseño
- ✅ **Paleta de Colores Profesional**
  - Primary: `#2563EB` (Blue 600) - Confianza y profesionalismo
  - Secondary: `#7C3AED` (Violet 600) - Innovación
  - Success: `#059669` (Emerald 600)
  - Warning: `#D97706` (Amber 600)
  - Danger: `#DC2626` (Red 600)
- ✅ **Design System** basado en Atomic Design
- ✅ **Componentes Reutilizables** con Ant Design
- ✅ **Responsive**: Mobile-first approach

### 🤖 IA-Friendly Development
- ✅ **Sistema de Memoria Persistente** (`.claude/`, `.copilot/`)
- ✅ **Anti-Olvido**: Checkpoints automáticos cada 5 mensajes
- ✅ **Convenciones Estrictas**: Validación automática
- ✅ **Documentación Inline**: Comentarios contextuales
- ✅ **Templates de Código**: Generación automática

### 📊 Dashboard y Analytics
- ✅ **Métricas en Tiempo Real** por tenant
- ✅ **Gráficos Interactivos** (Chart.js / Recharts)
- ✅ **Reportes Exportables** (PDF, Excel, CSV)
- ✅ **KPIs Personalizados** por plan

---

## 🛠️ Stack Tecnológico

### Frontend Core
```
React 18.2          → UI Library
TypeScript 5.3      → Type Safety
Vite 5.0            → Build Tool & Dev Server
```

### Estado y Datos
```
Zustand             → Client State Management
React Query v5      → Server State & Caching
React Hook Form     → Form Management
Zod                 → Schema Validation
```

### UI/UX
```
Ant Design 5        → Component Library
Tailwind CSS 3      → Utility-first CSS
CSS Modules         → Scoped Styles
Framer Motion       → Animations
```

### Routing y Seguridad
```
React Router v6     → Client-side Routing
Axios               → HTTP Client
React i18next       → Internationalization
```

### Testing
```
Vitest              → Unit Testing
React Testing Lib   → Component Testing
Playwright          → E2E Testing
MSW                 → API Mocking
```

### DevTools
```
ESLint              → Linting
Prettier            → Code Formatting
Husky               → Git Hooks
Commitlint          → Commit Conventions
```

---

## 🚀 Inicio Rápido

### 📚 Guías Disponibles

Este proyecto ofrece dos guías de instalación según tu nivel de experiencia:

- **[SETUP.md](./.docs/SETUP.md)** → Guía completa paso a paso desde cero (recomendado para principiantes)
- **Quickstart** (abajo) → Instalación rápida para desarrolladores experimentados

---

### Prerrequisitos
```bash
node >= 18.0.0
npm >= 9.0.0  (o pnpm >= 8.0.0)
```

### Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-org/cpanel-multitenant-bo.git
cd cpanel-multitenant-bo

# 2. Instalar dependencias
npm install
# o
pnpm install

# 3. Configurar variables de entorno
cp .env.example .env.local

# 4. Iniciar servidor de desarrollo
npm run dev
```

### Primer Uso con IA

Si vas a desarrollar con **Claude Code** o **GitHub Copilot**, lee primero:

1. **[START.md](./START.md)** → Pasos para comenzar desarrollo con IA
2. **[PROJECT_INSTRUCTIONS.md](./PROJECT_INSTRUCTIONS.md)** → Guía completa del proyecto
3. **[CODING_STANDARDS.md](./CODING_STANDARDS.md)** → Estándares de código

---

## 📁 Estructura del Proyecto

```
cpanel-multitenant-bo/
│
├── 📄 Documentación Raíz
│   ├── README.md                      # Este archivo
│   ├── START.md                       # 🆕 Guía de inicio con IA
│   ├── PROJECT_INSTRUCTIONS.md        # 🆕 Instrucciones completas para IA
│   ├── CODING_STANDARDS.md            # 🆕 Estándares de código
│   ├── ARQUITECTURA.md                # 🆕 Decisiones arquitectónicas
│   ├── QUICKSTART.md                  # 🆕 Inicio rápido paso a paso
│   └── CHANGELOG.md                   # Historial de cambios
│
├── 🤖 Configuración IA (Multi-IA Compatible)
│   ├── .claude/                       # Configuración Claude Code
│   │   ├── context.md                 # Contexto del proyecto
│   │   ├── conventions.md             # Convenciones obligatorias
│   │   ├── memory-checkpoints.md      # Sistema anti-olvido
│   │   ├── session-state.md           # Estado de sesión
│   │   └── skills/                    # Skills personalizados
│   │
│   ├── .copilot/                      # Configuración GitHub Copilot
│   │   ├── instructions.md            # Instrucciones Copilot
│   │   └── templates/                 # Templates de código
│   │
│   └── .cursorrules                   # Configuración Cursor AI
│
├── 📦 Código Fuente
│   └── src/
│       ├── main.tsx                   # Entry point
│       ├── App.tsx                    # Componente raíz
│       │
│       ├── components/                # Atomic Design
│       │   ├── atoms/                 # Componentes básicos
│       │   ├── molecules/             # Componentes compuestos
│       │   ├── organisms/             # Componentes complejos
│       │   └── templates/             # Templates de página
│       │
│       ├── pages/                     # Páginas/Rutas
│       ├── hooks/                     # Custom hooks
│       ├── context/                   # React Context
│       ├── store/                     # Zustand stores
│       ├── services/                  # API services
│       ├── types/                     # TypeScript types
│       ├── utils/                     # Utilidades
│       ├── config/                    # Configuraciones
│       ├── locales/                   # Traducciones i18n
│       └── styles/                    # Estilos globales
│
├── 🧪 Testing
│   ├── tests/
│   │   ├── unit/
│   │   ├── integration/
│   │   └── e2e/
│   │
│   └── __mocks__/                     # Mocks para testing
│
├── 🔧 Scripts y Automatización
│   ├── scripts/
│   │   ├── generate-component.js      # Generador de componentes
│   │   ├── validate-conventions.js    # Validador de convenciones
│   │   ├── memory-checkpoint.js       # Sistema anti-olvido
│   │   └── i18n-sync.js              # Sincronización traducciones
│   │
│   └── .husky/                        # Git hooks
│
├── 📚 Documentación Adicional
│   └── docs/
│       ├── architecture/              # Documentación arquitectura
│       ├── api/                       # Documentación API
│       ├── components/                # Catálogo componentes
│       └── guides/                    # Guías de desarrollo
│
└── 📋 Configuración
    ├── package.json
    ├── tsconfig.json
    ├── vite.config.ts
    ├── tailwind.config.js
    ├── .eslintrc.json
    ├── .prettierrc
    └── .env.example
```

---

## 📖 Documentación

### 📑 Documentos Principales (Raíz del Proyecto)

| Documento | Propósito | Audiencia |
|-----------|-----------|-----------|
| **[SETUP.md](./.docs/SETUP.md)** | Configuración completa paso a paso desde cero | Nuevos desarrolladores |
| **[START.md](./START.md)** | Guía de inicio para desarrollo con IA | Desarrolladores + IA |
| **[PROJECT_INSTRUCTIONS.md](./PROJECT_INSTRUCTIONS.md)** | Instrucciones completas del proyecto | Claude/Copilot/Cursor |
| **[CODING_STANDARDS.md](./CODING_STANDARDS.md)** | Estándares de código detallados | Desarrolladores + IA |
| **[ARQUITECTURA.md](./ARQUITECTURA.md)** | Decisiones técnicas y arquitectura | Arquitectos + IA |
| **[QUICKSTART.md](./QUICKSTART.md)** | Inicio rápido en 5 minutos | Nuevos desarrolladores |

### 🤖 Configuración IA

#### Claude Code (`.claude/`)
- `context.md` - Contexto del proyecto y objetivos
- `conventions.md` - Convenciones obligatorias
- `memory-checkpoints.md` - Sistema anti-olvido
- `session-state.md` - Estado actual del desarrollo

#### GitHub Copilot (`.copilot/`)
- `instructions.md` - Instrucciones específicas
- `templates/` - Templates de código reutilizables

---

## 🏗️ Arquitectura Multi-Tenant

### Estrategia de Aislamiento

```typescript
// Cada request incluye el tenant ID en header
Headers: {
  'X-Tenant-ID': 'tenant-abc-123',
  'Authorization': 'Bearer <token>'
}

// Context global de tenant
const { currentTenant, switchTenant } = useTenant()

// Query keys incluyen tenant
queryKey: ['users', currentTenant.id, filters]
```

### Flujo de Autenticación Multi-Tenant

```
1. Login → JWT con lista de tenants
2. Selección de Tenant → Actualizar contexto
3. API Calls → Header X-Tenant-ID automático
4. Validación Backend → Verificar permisos y plan
5. Datos → Filtrados por tenant
```

### Niveles de Plan

| Plan | Usuarios | Productos | Almacenamiento | Features |
|------|----------|-----------|----------------|----------|
| **Free** | 10 | 50 | 1 GB | Básico |
| **Plan1** | 50 | 500 | 10 GB | Dashboard + Reports |
| **Plan2** | 200 | 2,000 | 50 GB | Analytics + API |
| **Plan3** | Ilimitado | Ilimitado | 500 GB | Todo + White Label |

---

## 🧠 Sistema de Memoria IA

### Anti-Olvido Automático

Este proyecto incluye un sistema de **memoria persistente** para IAs:

#### Checkpoints Automáticos
```bash
# Cada 5 mensajes con la IA, ejecutar:
npm run ai:checkpoint

# Esto actualiza:
# - .claude/session-state.md
# - .claude/memory-checkpoints.md
# - Progreso y decisiones recientes
```

#### Recordatorios
```bash
# Antes de una sesión larga:
npm run ai:remind

# Genera un resumen del estado actual del proyecto
```

#### Validación de Convenciones
```bash
# Verificar que el código sigue las convenciones:
npm run validate

# Esto verifica:
# - Nomenclatura (camelCase, PascalCase, etc.)
# - Estructura de archivos
# - Imports ordenados
# - Tests de cobertura
```

### Documentación Inline

Todos los componentes críticos incluyen:
- ✅ Comentarios explicando decisiones arquitectónicas
- ✅ JSDoc para funciones complejas
- ✅ Ejemplos de uso
- ✅ Referencias a documentación

---

## 🎯 Scripts Disponibles

### Desarrollo
```bash
npm run dev              # Servidor de desarrollo (puerto 3000)
npm run dev:host         # Desarrollo con acceso en red local
npm run build            # Build de producción
npm run preview          # Preview del build
```

### Testing
```bash
npm run test             # Ejecutar tests
npm run test:watch       # Tests en modo watch
npm run test:coverage    # Coverage report
npm run test:e2e         # Tests end-to-end
```

### Calidad de Código
```bash
npm run lint             # Ejecutar ESLint
npm run lint:fix         # Fix automático de ESLint
npm run format           # Formatear con Prettier
npm run type-check       # Verificar tipos TypeScript
npm run validate         # Validación completa
```

### Generadores
```bash
npm run generate:component [nombre] [tipo]
# Ejemplo: npm run generate:component UserCard molecule

npm run generate:page [nombre]
npm run generate:hook [nombre]
npm run generate:service [nombre]
```

### IA Helpers
```bash
npm run ai:checkpoint    # Guardar checkpoint de memoria
npm run ai:remind        # Recordar estado del proyecto
npm run ai:validate      # Validar convenciones IA
```

### Internacionalización
```bash
npm run i18n:extract     # Extraer strings para traducir
npm run i18n:sync        # Sincronizar archivos de idioma
npm run i18n:validate    # Validar traducciones
```

---

## 🧪 Testing

### Cobertura Objetivo
- **Unit Tests**: > 80%
- **Integration Tests**: > 60%
- **E2E Tests**: Flujos críticos

### Estrategia
```
E2E Tests (10%)
├── Login multi-tenant
├── Cambio de tenant
├── CRUD usuarios
└── Validación de límites

Integration Tests (30%)
├── API integration
├── Context providers
└── State management

Unit Tests (60%)
├── Components
├── Hooks
├── Utils
└── Services
```

---

## 🎨 Paleta de Colores

### Colores Principales
```css
--primary-50:  #EFF6FF;
--primary-600: #2563EB;  /* Principal */
--primary-700: #1D4ED8;

--secondary-50:  #F5F3FF;
--secondary-600: #7C3AED;  /* Secundario */
--secondary-700: #6D28D9;

--success-600: #059669;
--warning-600: #D97706;
--danger-600:  #DC2626;
```

### Colores por Plan
```css
--plan-free:  #6B7280;  /* Gray */
--plan-plan1: #3B82F6;  /* Blue */
--plan-plan2: #7C3AED;  /* Violet */
--plan-plan3: #F59E0B;  /* Amber */
```

---

## 🤝 Contribuir

### Flujo de Trabajo

1. **Fork** el proyecto
2. **Crear rama** feature: `git checkout -b feature/nueva-funcionalidad`
3. **Commits** siguiendo [Conventional Commits](https://www.conventionalcommits.org/)
4. **Push** a la rama: `git push origin feature/nueva-funcionalidad`
5. **Pull Request** con descripción detallada

### Convenciones de Commit
```
feat: nueva funcionalidad
fix: corrección de bug
docs: cambios en documentación
style: cambios de formato
refactor: refactorización de código
test: añadir o modificar tests
chore: tareas de mantenimiento
```

---

## 📄 Licencia

Este proyecto está bajo la licencia **MIT**. Ver [LICENSE](./LICENSE) para más detalles.

---

## 👥 Equipo

Desarrollado con ❤️ por **Team bMOI**

- **Proyecto**: cpanel-multitenant-bo
- **Versión**: 1.0.0
- **Última actualización**: Noviembre 2025

---

## 🔗 Enlaces Útiles

- [📖 Documentación Completa](./docs/)
- [⚙️ Configuración Desde Cero](./.docs/SETUP.md)
- [🚀 Guía de Inicio Rápido](./QUICKSTART.md)
- [🏗️ Arquitectura](./ARQUITECTURA.md)
- [📝 Estándares de Código](./CODING_STANDARDS.md)
- [🤖 Desarrollo con IA](./START.md)

---

<div align="center">

**¿Tienes preguntas? [Abre un issue](https://github.com/tu-org/cpanel-multitenant-bo/issues)**

</div>
