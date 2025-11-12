# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**bMOI CPanel Multi-Tenant BO** is an enterprise control panel designed for managing multiple organizations (tenants) in an isolated and secure manner. Built with React 18, TypeScript 5, and Vite 5.

**Logo:** bMOI (simple, professional, no decorations)

**Key Characteristics:**
- Multi-tenant architecture with complete data isolation
- Plan-based resource limits (Free, Plan1, Plan2, Plan3)
- Role-based access control (RBAC)
- **Responsive design** - Mobile-first approach (320px+)
- Internationalization ready (i18n)
- Docker-ready development and production environments

**Current Status:** Phase 0 - Setup & Configuration (90% complete)

## 📚 Critical Documentation Ecosystem

This project has a comprehensive documentation system. **Read these files in this order when starting:**

1. **`.claude/context.md`** - Complete project context (MUST READ FIRST)
2. **`.docs/PROJECT_INSTRUCTIONS.md`** - Master document for AI assistants
3. **`.docs/CODING_STANDARDS.md`** - Detailed coding standards with examples
4. **`.claude/conventions.md`** - Mandatory coding conventions
5. **`.claude/session-state.md`** - Current development status
6. **This file (CLAUDE.md)** - Quick reference guide

**Other AI Configurations:**
- `.copilot/instructions.md` - Instructions for GitHub Copilot
- `.cursorrules` - Rules for Cursor AI
- `.docs/START.md` - Step-by-step guide for starting development with AI
- `.docs/ARQUITECTURA.md` - Architectural decisions
- `.docs/QUICKSTART.md` - 5-minute quick start

## 🐳 Development Environment: DOCKER (MANDATORY)

**⚠️ CRITICAL: This project MUST be developed using Docker.**

### Why Docker is Mandatory

1. **Consistency**: Same environment for all developers
2. **Onboarding**: New dev productive in 5 minutes
3. **No pollution**: Dependencies isolated from your system
4. **CI/CD Ready**: Same container in dev, staging, prod
5. **Multi-OS**: Works identically on Windows, Mac, Linux

### Docker Commands (Use These)

```bash
# Start development environment (with hot-reload)
docker compose --profile dev up -d

# View logs
docker compose logs -f frontend-dev

# Execute commands inside container
docker compose exec frontend-dev npm install [package]
docker compose exec frontend-dev npm run lint
docker compose exec frontend-dev npm run test

# Stop environment
docker compose --profile dev down

# Full stack (PostgreSQL + Redis + Frontend)
docker compose --profile fullstack up -d

# Production build testing
docker compose --profile prod up -d
```

### ❌ DON'T Run Commands Directly

```bash
# ❌ NEVER do this (runs on your machine):
npm install
npm run dev
npm run build

# ✅ ALWAYS do this (runs in Docker):
docker compose --profile dev up
docker compose exec frontend-dev npm install
```

See `DOCKER.md` and `TECHNICAL_ANALYSIS.md` for complete documentation.

## 📊 Key Technical Decisions

### HTTP Client: Axios ✅

**Decision:** Use **Axios** (not fetch, not ky, not ofetch)

**Why:**
- ✅ **Interceptors**: Critical for automatic `X-Tenant-ID` header injection
- ✅ **Battle-tested**: 49M weekly downloads, proven in production
- ✅ **Error handling**: Superior HTTP error handling
- ✅ **Retry logic**: Easy to implement exponential backoff
- ✅ **Code already written**: `src/services/api/client.ts` uses axios patterns

**Trade-off accepted:** 13KB bundle size (2.6% of 500KB target) is worth the DX benefits

**Installation required:**
```bash
docker compose exec frontend-dev npm install axios
docker compose exec frontend-dev npm install -D @types/axios
```

See `TECHNICAL_ANALYSIS.md` section 2 for detailed comparison with alternatives.

### State Management: Zustand + React Query ✅

- **Zustand**: Client state (UI, preferences) - 1KB bundle
- **React Query v5**: Server state (API data) - Perfect for multi-tenant caching
- **Context API**: Auth, Tenant, Theme - Built-in React

**Why this combo:** Separation of concerns, minimal boilerplate, excellent DX

### Component Library: Ant Design 5 + Tailwind CSS ✅

- **Ant Design**: Enterprise components (Table, Form, Modal)
- **Tailwind**: Custom layouts and spacing
- **CSS Modules**: Component-specific styles

**Why both:** Complementary, not competing. Fast development + flexibility.

### Build Tool: Vite 5 ✅

**Why not CRA:** Vite has <100ms HMR vs CRA's 5-10s. CRA is deprecated.

See `.docs/ARQUITECTURA.md` for complete architectural decisions.

## Development Commands (Inside Docker Container)

When you exec into the container, these commands are available:

### Core Development
```bash
npm run dev              # Start dev server (port 3000)
npm run build            # TypeScript compile + Vite build
npm run preview          # Preview production build
npm run lint             # Run ESLint
npm run type-check       # Verify TypeScript types
npm run format           # Format with Prettier
npm run validate         # Complete validation (lint + type-check + test)
```

### Testing
```bash
npm run test             # Run tests
npm run test:watch       # Tests in watch mode
npm run test:coverage    # Coverage report
npm run test:e2e         # End-to-end tests
```

### Generators (Planned)
```bash
npm run generate:component [name] [type]  # Generate component
npm run generate:page [name]              # Generate page
npm run generate:hook [name]              # Generate hook
npm run generate:service [name]           # Generate service
```

### AI Memory Helpers (Planned)
```bash
npm run ai:checkpoint    # Save memory checkpoint (use every 5 messages)
npm run ai:remind        # Recall project state
npm run ai:validate      # Validate AI conventions
```

### Docker Development
```bash
# Development with hot reload
docker compose --profile dev up

# Full stack with PostgreSQL + Redis
docker compose --profile fullstack up

# Production build
docker compose --profile prod up
```

See `DOCKER.md` and `DOCKER-QUICKSTART.md` for detailed Docker workflows.

## 🧠 AI Memory System (Anti-Forgetting)

This project implements a memory checkpoint system to prevent context loss in long conversations:

### Every 5 Messages - Memory Checkpoint
When working with Claude Code, every ~5 messages you should mentally verify:
- [ ] Project name: **bMOI CPanel Multi-Tenant BO**
- [ ] Logo: **bMOI**
- [ ] Conventions: **camelCase** (variables), **PascalCase** (components), **UPPER_SNAKE_CASE** (constants)
- [ ] **NEVER** use snake_case in JS/TS
- [ ] **NEVER** use `any` type without justification
- [ ] **ALWAYS** verify `currentTenant` before API calls
- [ ] **ALWAYS** include `tenant.id` in query keys
- [ ] Colors: Primary **#2563EB**, Secondary **#7C3AED**

### End of Session - Update State
Before ending a development session, update `.claude/session-state.md` with:
- Components created today
- Technical decisions made
- Problems encountered and solutions
- Next steps
- Current phase progress

### Context Recovery
If you lose context mid-conversation:
1. Read `.claude/context.md` immediately
2. Read `.claude/session-state.md` to know where you left off
3. Confirm you remember the mandatory conventions
4. Ask the user: "What were we working on?"

## Architecture & Key Patterns

### Multi-Tenant Architecture

**Critical**: Every API request MUST include the tenant context.

The system uses a centralized tenant context (`src/context/TenantContext.tsx`) that:
1. Manages the current tenant selection
2. Automatically adds `X-Tenant-ID` header to all API requests
3. Validates permissions and plan limits
4. Persists tenant selection in localStorage

**Key implementation details:**
- The `apiClient` singleton (`src/services/api/client.ts`) handles tenant headers automatically via interceptors
- Tenant context must be verified before making any API calls
- All query keys in React Query MUST include the tenant ID for proper cache isolation

**Example pattern:**
```typescript
const { currentTenant, hasPermission, isWithinLimits } = useTenant()

// Always verify tenant before operations
if (!currentTenant) return

// Query keys must include tenant ID
queryKey: ['users', currentTenant.id, filters]

// Check permissions
if (!hasPermission('users.create')) return

// Check plan limits
if (!isWithinLimits('users', currentUserCount)) {
  // Show upgrade prompt
}
```

### API Client Architecture

The `ApiClient` class (`src/services/api/client.ts`) is a sophisticated wrapper around axios with:
- **Automatic tenant header injection**: Uses interceptors to add `X-Tenant-ID` to every request
- **JWT authentication**: Manages bearer tokens with automatic refresh
- **Retry logic**: Exponential backoff for network errors and 5xx responses
- **Error formatting**: Consistent error handling across the app
- **Request/response logging**: Development-only console logging

**Important**: Always use the `apiClient` singleton, never create new axios instances.

### Environment Configuration

All environment variables are centralized in `src/config/env.ts` with:
- Type-safe access via named exports (`API_BASE_URL`, `TENANT_HEADER_KEY`, etc.)
- Validation on app startup
- Default values for development
- Naming convention: `VITE_*` for client-side vars, exported as `UPPER_SNAKE_CASE`

### Type System

All types are defined in `src/types/common.types.ts`:
- Interface naming: `I` + PascalCase (e.g., `ITenant`, `IUser`)
- Plan limits and permissions are strongly typed
- Shared constants like `DEFAULT_PLAN_LIMITS` and `ROLE_PERMISSIONS`
- Utility types for partial updates and entity creation

### Plan Limits System

Four plan tiers with enforced limits:
- **Free**: 10 users, 50 products, 1GB storage
- **Plan1**: 50 users, 500 products, 10GB storage
- **Plan2**: 200 users, 2000 products, 50GB storage
- **Plan3**: Unlimited users/products, 500GB storage

Limits are checked via `isWithinLimits()` from the tenant context before creating resources.

### Naming Conventions (MANDATORY)

From `.cursorrules` - these are strictly enforced:
- **Variables/functions**: camelCase (`getUserData`, `currentTenant`)
- **Components**: PascalCase (`UserProfile`, `TenantDashboard`)
- **Hooks**: `use` + camelCase (`useAuth`, `useTenant`)
- **Constants**: UPPER_SNAKE_CASE (`API_BASE_URL`, `MAX_RETRIES`)
- **Interfaces**: `I` + PascalCase (`IUser`, `ITenant`)
- **NEVER** use snake_case in JavaScript/TypeScript

### Import Order (MANDATORY)

Always follow this order:
1. React imports
2. External libraries
3. Internal components
4. Hooks
5. Services
6. Types
7. Styles

### Code Quality Rules

From `.cursorrules`:
- Component max size: 300 lines
- NO `any` type without justification
- NO `console.log` in production (use `ENABLE_DEV_LOGS` flag)
- Test coverage minimum: 80%
- All queries MUST include tenant.id in query keys

## Project Structure

```
src/
├── config/          # Environment and app configuration
├── context/         # React Context providers (TenantContext, etc.)
├── services/        # API client and service layer
│   └── api/
│       └── client.ts    # Singleton API client with interceptors
├── types/           # TypeScript type definitions
│   ├── common.types.ts  # Main type definitions
│   └── index.ts         # Type exports
└── (pending)
    ├── components/  # Atomic Design structure (atoms/molecules/organisms)
    ├── pages/       # Route pages
    ├── hooks/       # Custom hooks
    ├── store/       # Zustand stores
    └── utils/       # Utilities
```

## Key Files

**Read these first:**
- `README.md` - Comprehensive project documentation
- `.cursorrules` - Mandatory coding conventions
- `src/context/TenantContext.tsx` - Multi-tenant context implementation
- `src/services/api/client.ts` - API client with tenant interceptors
- `src/types/common.types.ts` - All type definitions

**Docker:**
- `docker-compose.yml` - Multi-profile setup (dev/prod/fullstack)
- `DOCKER.md` - Detailed Docker documentation
- `Makefile` - Common Docker commands

## Common Patterns

### Creating a New API Service

```typescript
// services/userService.ts
import { apiClient } from './api/client'
import type { IUser } from '@/types/common.types'

export const userService = {
  async getUsers(tenantId: string) {
    // apiClient automatically adds X-Tenant-ID header
    return apiClient.get<IUser[]>('/users')
  },

  async createUser(userData: CreateEntity<IUser>) {
    return apiClient.post<IUser>('/users', userData)
  }
}
```

### Using Tenant Context

```typescript
import { useTenant } from '@/context/TenantContext'

function MyComponent() {
  const {
    currentTenant,
    hasPermission,
    isWithinLimits
  } = useTenant()

  // Always check tenant exists
  if (!currentTenant) {
    return <SelectTenantPrompt />
  }

  // Check permissions
  const canCreate = hasPermission('users.create')

  // Check limits
  const canAddMore = isWithinLimits('users', currentCount)

  return (/* ... */)
}
```

### Conditional Rendering with Permissions

```typescript
import { RequirePermission } from '@/context/TenantContext'

function Dashboard() {
  return (
    <RequirePermission
      permission="dashboard.view"
      fallback={<AccessDenied />}
    >
      <DashboardContent />
    </RequirePermission>
  )
}
```

## Development Workflow

1. **Environment Setup**: Copy `.env.example` to `.env.local` and configure
2. **Install Dependencies**: `npm install`
3. **Start Dev Server**: `npm run dev` or `docker compose --profile dev up`
4. **Lint Before Commit**: `npm run lint`
5. **Type Check**: `tsc -b` (happens automatically during build)

## Testing

- Unit tests should cover > 80% of code
- All tests must include tenant context when testing multi-tenant features
- Mock `apiClient` using proper TypeScript types
- Test both success and error scenarios

## Important Notes

- **Never bypass tenant context**: All API calls must go through `apiClient` which adds tenant headers
- **Permission checks are client-side only**: Backend must also validate (security-in-depth)
- **Plan limits are soft limits**: UI should prevent exceeding limits, but backend enforces them
- **Development logs**: Use `ENABLE_DEV_LOGS` flag, never commit `console.log` statements
- **Component size**: Keep components under 300 lines, split larger ones

## Color Palette

From README.md design system:
- Primary: `#2563EB` (Blue-600) - Trust and professionalism
- Secondary: `#7C3AED` (Violet-600) - Innovation
- Success: `#059669` (Emerald-600)
- Warning: `#D97706` (Amber-600)
- Danger: `#DC2626` (Red-600)

## Development Phases

The project is organized in phases. Current status from `.claude/session-state.md`:

```
Phase 0 (Setup & Config):     ██████████ 90%  - Documentation, structure
Phase 1 (Auth & Layout):      ░░░░░░░░░░  0%  - Login, TenantSelector, MainLayout
Phase 2 (Dashboard):          ░░░░░░░░░░  0%  - Metrics per tenant
Phase 3 (User Management):    ░░░░░░░░░░  0%  - CRUD users multi-tenant
Phase 4 (Products & Orders):  ░░░░░░░░░░  0%  - Catalog and orders per tenant
```

**Next Immediate Steps (Phase 0 completion):**
1. Initialize Vite project structure
2. Install all dependencies
3. Configure Tailwind CSS
4. Create complete `src/` folder structure
5. Setup ESLint + Prettier + Husky

**Next Phase (Phase 1):**
1. Implement type definitions (`src/types/common.types.ts`)
2. Create API client with tenant interceptors
3. Implement TenantContext and AuthContext
4. Build LoginPage, TenantSelector, MainLayout
5. Create useAuth and useTenant hooks

## Code Templates

The project includes standardized templates in `.copilot/instructions.md` and `.docs/CODING_STANDARDS.md`:

### Component Template
```typescript
import React, { useState, useEffect } from 'react'
import { useTenant } from '@/hooks/useTenant'
import type { IComponentProps } from './Component.types'
import styles from './Component.module.css'

export const ComponentName: React.FC<IComponentProps> = ({ prop1 }) => {
  // 1. State
  const [loading, setLoading] = useState(false)

  // 2. Hooks
  const { currentTenant } = useTenant()

  // 3. Effects
  useEffect(() => {
    if (!currentTenant) return
    // Logic
  }, [currentTenant])

  // 4. Handlers
  const handleClick = () => { /* ... */ }

  // 5. Early returns
  if (!currentTenant) return <div>Select tenant</div>

  // 6. Render
  return (
    <div className={styles.container} data-tenant={currentTenant.slug}>
      {/* Content */}
    </div>
  )
}
```

### Hook Template
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

  return { state, setState, data, isLoading }
}
```

### Service Template
```typescript
import { apiClient } from '@/services/api/client'
import type { IResource } from '@/types'

export const resourceService = {
  getAll: async (tenantId: string, filters = {}) => {
    return apiClient.get<IResource[]>('/resources', { params: filters })
  },

  getById: async (tenantId: string, id: string) => {
    return apiClient.get<IResource>(`/resources/${id}`)
  },

  create: async (tenantId: string, data: Partial<IResource>) => {
    return apiClient.post<IResource>('/resources', data)
  },

  update: async (tenantId: string, id: string, data: Partial<IResource>) => {
    return apiClient.put<IResource>(`/resources/${id}`, data)
  },

  delete: async (tenantId: string, id: string) => {
    return apiClient.delete(`/resources/${id}`)
  },
}
```

## First Session Prompt Template

When starting a new development session, use this prompt (from `.docs/START.md`):

```
Hello! I'm working on the bMOI CPanel Multi-Tenant BO project.

STEP 1: Please read these documents COMPLETELY (they are critical):
- .claude/context.md
- .docs/PROJECT_INSTRUCTIONS.md
- .docs/CODING_STANDARDS.md
- .claude/session-state.md

STEP 2: When you finish reading, confirm you understood:
1. Project name and logo
2. Tech stack (React + TypeScript + Vite)
3. Color palette (Primary #2563EB, Secondary #7C3AED)
4. MANDATORY conventions:
   - Variables: camelCase
   - Components: PascalCase
   - Constants: UPPER_SNAKE_CASE
   - NEVER snake_case
   - NEVER any type
5. Multi-Tenant rules:
   - Always verify currentTenant
   - Query keys include tenant.id
   - Validate permissions with hasPermission()

STEP 3: Once you confirm you understood everything,
we'll begin with [current phase/task].

Ready to start?
```

## Troubleshooting AI Behavior

### If AI forgets conventions:
```
🧠 CONVENTION REMINDER:

This project uses STRICTLY:
- camelCase: variables and functions
- PascalCase: React components
- UPPER_SNAKE_CASE: constants
- NEVER snake_case in JS/TS
- NEVER any type

Please confirm you'll remember this.
```

### If AI creates code without tenant context:
```
⚠️ ERROR: Missing tenant validation

ALL code using API must:
1. const { currentTenant } = useTenant()
2. if (!currentTenant) return null
3. queryKey: ['resource', currentTenant.id]

Please correct the code above.
```

### If AI uses snake_case:
```
❌ STOP: snake_case detected

This project PROHIBITS snake_case in JavaScript/TypeScript.

CORRECT examples:
- userName (camelCase)
- UserProfile (PascalCase)
- API_BASE_URL (UPPER_SNAKE_CASE)

Please correct and DON'T use snake_case again.
```

## 🏗️ Architectural Improvements (Pending)

The project has solid architecture (8/10) but needs these improvements:

### High Priority (Phase 1)
1. **Error Boundaries** - Add `src/components/ErrorBoundary.tsx`
2. **Logger** - Create `src/utils/logger.ts` for debugging and production errors
3. **Testing Pyramid** - Document strategy in `.docs/ARQUITECTURA.md`

### Medium Priority (Phase 2)
4. **Feature Flags** - Add `src/utils/featureFlags.ts` for gradual rollouts
5. **Dependency Injection** - Improve `apiClient` testability with context
6. **Monitoring** - Integrate Sentry or similar service

### Low Priority (Phase 3+)
7. **API Versioning** - Strategy for `/api/v1` → `/api/v2` migration
8. **Optimistic Updates** - Document pattern for better UX

**Current Status:**
- ✅ SOLID Principles
- ✅ Separation of Concerns
- ✅ Atomic Design
- ✅ Multi-layer Caching
- ✅ Security Layers
- ⚠️ Missing: Error Boundaries, Logging, Feature Flags

See `TECHNICAL_ANALYSIS.md` section 3 for detailed analysis and code examples.

## Additional Resources

- Project documentation in `README.md` includes detailed feature descriptions
- **Responsive Design Guide** in `.docs/RESPONSIVE_DESIGN.md` - **MUST READ** for UI development
- Docker quickstart available in `DOCKER-QUICKSTART.md`
- `.cursorrules` contains mandatory coding standards for Cursor AI
- Type definitions in `src/types/common.types.ts` are the source of truth
- Complete step-by-step guide in `.docs/START.md`
- Architectural decisions documented in `.docs/ARQUITECTURA.md`
- Memory checkpoint system in `.claude/memory-checkpoints.md`
