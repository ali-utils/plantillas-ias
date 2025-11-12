# 📦 Plantillas Admin Web - bMOI

> **Plantillas completas y listas para usar para el frontend Admin Web del proyecto bMOI**

---

## 🎯 ¿Qué es esto?

Este directorio contiene **plantillas completas** para crear el proyecto frontend **Admin Web** del ecosistema bMOI desde cero. Incluye:

✅ Documentación completa para IAs (Claude, Copilot, Cursor)
✅ Configuración Docker lista para desarrollo y producción
✅ Estructura de proyecto profesional
✅ Ejemplos de código completos
✅ Estándares y convenciones definidas
✅ Guías paso a paso de setup

---

## 📂 Estructura de Plantillas

```
plantillas_admin/
├── .claude/                         # Instrucciones para Claude Code
│   ├── context.md                   # Contexto completo del proyecto
│   ├── conventions.md               # Convenciones obligatorias
│   └── session-state.md             # Estado del desarrollo
│
├── .copilot/                        # Instrucciones para GitHub Copilot
│   └── instructions.md
│
├── .docs/                           # Documentación completa
│   ├── README.md                    # Overview del proyecto
│   ├── SETUP_INICIAL.md             # Setup paso a paso (11 pasos)
│   ├── ARQUITECTURA.md              # Decisiones arquitectónicas
│   ├── CODING_STANDARDS.md          # Estándares de código
│   ├── RESPONSIVE_DESIGN.md         # Diseño responsive mobile-first
│   ├── EJEMPLOS_CODIGO.md           # Código listo para copiar
│   ├── CONFIGURACIONES.md           # Archivos de configuración
│   │
│   ├── # 🎯 bMOI-Specific Docs (CRITICAL)
│   ├── BMOI_STACK_EVALUATION.md     # Stack evaluado para bMOI Admin
│   ├── BMOI_KYC_WORKFLOWS.md        # Sistema completo de KYC
│   └── BMOI_SECURITY_BEST_PRACTICES.md  # MFA, RBAC, auditoría, buenas prácticas
│
├── .cursorrules                     # Reglas para Cursor AI
├── .dockerignore                    # Archivos excluidos de Docker
├── .env.example                     # Variables de entorno template
├── CLAUDE.md                        # Guía rápida para Claude (raíz)
├── docker-compose.yml               # Orquestación Docker
├── Dockerfile                       # Build de producción
├── Dockerfile.dev                   # Build de desarrollo
├── nginx.conf                       # Configuración Nginx
└── README_PLANTILLAS.md             # Este archivo
```

---

## 🚀 Cómo Usar Estas Plantillas

### Opción 1: Copiar Todo (Recomendado)

```bash
# 1. Crear directorio del nuevo proyecto frontend
mkdir cpanel-multitenant-bo
cd cpanel-multitenant-bo

# 2. Copiar todas las plantillas
cp -r ../bo-backend-api/plantillas_admin/. .

# 3. Inicializar git
git init
git add .
git commit -m "chore: initial commit with templates"

# 4. Inicializar proyecto Vite
npm create vite@latest . -- --template react-ts

# 5. Instalar dependencias
npm install

# 6. Copiar .env.example a .env
cp .env.example .env

# 7. Levantar con Docker
docker compose up -d

# 8. Abrir navegador
open http://localhost:5173
```

### Opción 2: Usar como Referencia

Si prefieres crear tu proyecto manualmente, usa las plantillas como referencia:

1. Lee `.claude/context.md` para entender el proyecto
2. Sigue `.docs/SETUP_INICIAL.md` paso a paso
3. Copia los archivos de configuración de `.docs/CONFIGURACIONES.md`
4. Usa `.docs/EJEMPLOS_CODIGO.md` como guía para escribir código

---

## 📚 Documentación para IAs

### Para Claude Code

**Archivos principales:**
1. `.claude/context.md` - Contexto completo (LEER PRIMERO)
2. `.claude/conventions.md` - Convenciones obligatorias
3. `.claude/session-state.md` - Estado del desarrollo
4. `CLAUDE.md` (raíz) - Guía rápida de referencia

**Cómo usar:**
Cuando trabajes con Claude Code en este proyecto, Claude leerá automáticamente estos archivos para entender el contexto.

### Para GitHub Copilot

**Archivo principal:**
- `.copilot/instructions.md`

**Cómo usar:**
GitHub Copilot leerá automáticamente este archivo para generar sugerencias de código consistentes con el proyecto.

### Para Cursor AI

**Archivo principal:**
- `.cursorrules`

**Cómo usar:**
Cursor AI leerá automáticamente este archivo al abrir el proyecto para aplicar las reglas definidas.

---

## 🐳 Docker Setup

### Desarrollo (Solo Frontend)

```bash
# Levantar frontend en modo desarrollo
docker compose up -d

# Ver logs
docker compose logs -f frontend

# Ejecutar comandos dentro del container
docker compose exec frontend npm install [package]
docker compose exec frontend npm run test

# Detener
docker compose down
```

### Full Stack (Frontend + Backend)

```bash
# Levantar frontend + backend + postgres + redis
docker compose --profile full up -d

# Ver todos los logs
docker compose logs -f

# Detener todo
docker compose --profile full down
```

### Producción

```bash
# Build de producción
docker build -t bmoi-admin-web:latest .

# Ejecutar
docker run -p 80:80 bmoi-admin-web:latest

# Con variables de entorno
docker run -p 80:80 --env-file .env.production bmoi-admin-web:latest
```

---

## 🛠️ Stack Tecnológico

### Frontend Core
- **Framework:** React 18 + TypeScript 5 (strict mode)
- **Build Tool:** Vite 5
- **Package Manager:** npm

### Routing & State
- **Router:** React Router 6 con Data APIs
- **Client State:** Zustand 4
- **Server State:** TanStack Query (React Query) 5

### Forms & Validation
- **Forms:** React Hook Form 7
- **Validation:** Zod 3

### UI & Styling
- **CSS Framework:** Tailwind CSS 3.4
- **Components:** Shadcn/UI (Radix UI)
- **Icons:** Lucide React

### HTTP & API
- **Client:** Axios con interceptors
- **Base URL:** `http://localhost:3000/api/v1`

### Testing
- **Runner:** Vitest
- **Component Testing:** React Testing Library
- **API Mocking:** MSW

### DevOps
- **Container:** Docker + Docker Compose
- **Web Server:** Nginx
- **CI/CD:** GitHub Actions (template en .docs/CONFIGURACIONES.md)

---

## 📏 Convenciones Principales

### Nomenclatura de Archivos

```
UserCard.tsx              # Componentes: PascalCase
use-users.ts              # Hooks: kebab-case + "use"
users.service.ts          # Services: kebab-case + ".service"
date-utils.ts             # Utils: kebab-case
user.types.ts             # Types: kebab-case + ".types"
```

### Nomenclatura de Código

```typescript
// Componentes: PascalCase
export const UserCard = () => { };

// Variables: camelCase
const userName = 'John';

// Constantes: UPPER_SNAKE_CASE
const MAX_USERS = 100;

// Interfaces: PascalCase SIN "I"
interface UserCardProps { }

// Hooks: "use" + camelCase
export const useAuth = () => { };
```

### Reglas Estrictas

1. ❌ **NUNCA** usar `any` → Usar tipos específicos
2. ✅ **SIEMPRE** tipar props con interfaces
3. ✅ **SIEMPRE** incluir X-Tenant-ID en requests
4. ✅ **SIEMPRE** validar con Zod
5. ✅ **SIEMPRE** manejar loading/error states
6. ❌ **NUNCA** inline functions en JSX
7. ✅ **SIEMPRE** usar alias `@/` para imports
8. ✅ **SIEMPRE** usar Tailwind CSS para estilos

Consulta `.claude/conventions.md` para la lista completa.

---

## 📖 Guías de Setup

### Setup Completo (30-45 min)

Sigue `.docs/SETUP_INICIAL.md` para:
1. Inicializar proyecto Vite
2. Configurar Docker
3. Instalar dependencias
4. Configurar Tailwind CSS + Shadcn/UI
5. Configurar ESLint + Prettier
6. Configurar TypeScript
7. Configurar React Router
8. Configurar Axios con interceptors
9. Crear estructura de carpetas
10. Verificar todo funciona
11. Primer componente de prueba

### Setup Rápido (5-10 min)

Si ya conoces el stack:

```bash
# 1. Crear proyecto
npm create vite@latest cpanel-multitenant-bo -- --template react-ts
cd cpanel-multitenant-bo

# 2. Instalar dependencias principales
npm install react-router-dom @tanstack/react-query zustand axios zod react-hook-form @hookform/resolvers

# 3. Instalar Tailwind
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# 4. Instalar Shadcn/UI
npx shadcn-ui@latest init

# 5. Copiar archivos de configuración desde plantillas
cp ../bo-backend-api/plantillas_admin/.docs/CONFIGURACIONES.md .
# (Copiar manualmente las configs de este archivo)

# 6. Levantar
docker compose up -d
```

---

## 🎨 Ejemplos de Código

Todos los ejemplos están en `.docs/EJEMPLOS_CODIGO.md`:

### Componentes
- DashboardLayout (Sidebar + Header + Content)
- LoginPage (con formulario Zod)
- UsersPage (lista con React Query)
- UserCard (componente reutilizable)

### Hooks
- useAuth (autenticación)
- useUsers (queries)
- usePagination (paginación)

### Services
- api-client.ts (Axios configurado)
- auth.service.ts (login, logout, refresh)
- users.service.ts (CRUD)

### Stores
- auth.store.ts (Zustand)
- ui.store.ts (tema, sidebar)

### Forms
- UserForm (React Hook Form + Zod)
- LoginForm (con validación)

---

## 🔐 Multi-Tenant

**CRÍTICO:** Todas las requests deben incluir `X-Tenant-ID`.

El interceptor de Axios ya está configurado en las plantillas:

```typescript
// api-client.ts (ejemplo en plantillas)
apiClient.interceptors.request.use((config) => {
  const tenantId = localStorage.getItem('selected_tenant_id');
  if (tenantId && config.headers) {
    config.headers['X-Tenant-ID'] = tenantId;
  }
  return config;
});
```

### Flujo Multi-Tenant

1. User se loguea
2. Backend retorna lista de tenants accesibles
3. User selecciona un tenant
4. Frontend guarda `tenant_id` en localStorage
5. Todas las requests incluyen header `X-Tenant-ID`
6. Backend valida acceso y aplica Row-Level Security

---

## 🧪 Testing

### Ejecutar Tests

```bash
# Unit tests
docker compose exec frontend npm run test

# Watch mode
docker compose exec frontend npm run test:watch

# Coverage
docker compose exec frontend npm run test:coverage
```

### Template de Test

Ver ejemplos completos en `.docs/CODING_STANDARDS.md` sección Testing.

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import { UserCard } from '../UserCard';

describe('UserCard', () => {
  it('should render user information', () => {
    const mockUser = { id: '1', name: 'John', email: 'john@example.com' };
    render(<UserCard user={mockUser} onEdit={vi.fn()} />);
    expect(screen.getByText('John')).toBeInTheDocument();
  });
});
```

---

## 📊 Fases de Desarrollo

### Fase 0: Setup Frontend (0%)
- Inicializar proyecto Vite
- Configurar Docker
- Instalar dependencias
- Configurar herramientas (Tailwind, ESLint, etc.)
- Crear estructura de carpetas

### Fase 1: Auth & Layout (0%)
- Login page
- Auth service
- JWT interceptors
- Dashboard layout
- Tenant selector

### Fase 2: Core Modules (0%)
- Users CRUD
- Products CRUD
- Orders visualization
- Dashboard con métricas

### Fase 3: Advanced Features (0%)
- Audit logs
- Export data
- Advanced filters
- Real-time notifications

### Fase 4: Testing & Optimization (0%)
- 80% test coverage
- Performance optimization
- Lighthouse score > 90
- Accessibility audit

Ver progreso detallado en `.claude/session-state.md`.

---

## 🔗 Integración con Backend

### API Gateway

**Base URL:** `http://localhost:3000/api/v1`

**Headers obligatorios:**
```
Authorization: Bearer {access_token}
X-Tenant-ID: {selected_tenant_id}
Content-Type: application/json
```

### Endpoints Principales

```
POST   /auth/login        # Login
POST   /auth/refresh      # Refresh token
GET    /auth/me           # User info

GET    /users             # List users
POST   /users             # Create user
GET    /users/:id         # Get user
PATCH  /users/:id         # Update user
DELETE /users/:id         # Delete user

GET    /products          # List products
GET    /orders            # List orders
GET    /dashboard/metrics # Dashboard metrics
```

Ver documentación completa del backend en `../bo-backend-api/.docs/`.

---

## 📚 Recursos Adicionales

### Documentación del Proyecto
- `.claude/context.md` - Contexto completo
- `.docs/ARQUITECTURA.md` - Decisiones técnicas
- `.docs/CODING_STANDARDS.md` - Estándares detallados
- `.docs/EJEMPLOS_CODIGO.md` - Código completo
- `.docs/CONFIGURACIONES.md` - Todos los archivos de config

### Documentación Externa
- [React 18](https://react.dev/)
- [TypeScript](https://www.typescriptlang.org/)
- [Vite](https://vitejs.dev/)
- [React Router](https://reactrouter.com/)
- [TanStack Query](https://tanstack.com/query/latest)
- [Zustand](https://zustand-demo.pmnd.rs/)
- [Tailwind CSS](https://tailwindcss.com/)
- [Shadcn/UI](https://ui.shadcn.com/)
- [Zod](https://zod.dev/)

---

## 💡 Tips

### Para Desarrollo

1. **Usa Docker siempre:** `docker compose up -d`
2. **Hot reload funciona:** Cambios se reflejan automáticamente
3. **Logs:** `docker compose logs -f frontend`
4. **Instalar paquetes:** `docker compose exec frontend npm install [package]`

### Para Trabajar con IAs

1. **Claude Code:** Leerá automáticamente `.claude/context.md`
2. **Copilot:** Usa `.copilot/instructions.md`
3. **Cursor:** Aplica reglas de `.cursorrules`
4. **Consulta ejemplos:** `.docs/EJEMPLOS_CODIGO.md`

### Antes de Commit

- [ ] Sin `any` en TypeScript
- [ ] Props tipadas
- [ ] Loading/error states manejados
- [ ] Tests escritos
- [ ] Sin console.log
- [ ] ESLint sin warnings
- [ ] Prettier aplicado

---

## 🆘 Troubleshooting

### Docker no inicia

```bash
# Rebuild containers
docker compose down
docker compose build --no-cache
docker compose up -d
```

### Puerto 5173 ocupado

```bash
# Cambiar puerto en docker-compose.yml
ports:
  - "5174:5173"  # Usar 5174 en tu máquina
```

### Hot reload no funciona

```bash
# Verificar volumes en docker-compose.yml
volumes:
  - .:/app
  - /app/node_modules
```

### Problemas con node_modules

```bash
# Reinstalar dentro del container
docker compose exec frontend rm -rf node_modules
docker compose exec frontend npm install
```

---

## 📞 Soporte

### Documentación
- Lee `.claude/context.md` para contexto completo
- Consulta `.docs/SETUP_INICIAL.md` para setup paso a paso
- Revisa `.docs/CODING_STANDARDS.md` para estándares

### Issues
- Backend issues: Ver repo `bo-backend-api`
- Frontend issues: Crear issue en repo frontend

---

## 📝 Changelog

### 2025-11-04 - v1.0.0
- ✅ Estructura completa de plantillas creada
- ✅ Documentación para Claude, Copilot y Cursor
- ✅ Configuración Docker completa
- ✅ Ejemplos de código completos
- ✅ Estándares y convenciones definidas
- ✅ Guías de setup paso a paso

---

<div align="center">

**Plantillas Admin Web - bMOI**

React 18 + TypeScript 5 + Vite 5 + Docker

Listo para iniciar desarrollo desde cero

</div>
