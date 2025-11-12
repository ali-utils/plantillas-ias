# CLAUDE.md - Frontend

This file provides guidance to Claude Code (claude.ai/code) when working with the **frontend** code in this repository.

## Project Overview

**bMOI Admin Web** is a Single Page Application (SPA) built with React 18 + TypeScript that serves as the admin panel for the bMOI multi-tenant ecosystem. It provides a complete interface for managing users, products, orders, and tenant configuration.

**Framework:** React 18 + TypeScript 5 (strict)
**Build Tool:** Vite 5
**Router:** React Router 6
**State:** Zustand (client) + React Query (server)
**UI:** Tailwind CSS + Shadcn/UI
**Testing:** Vitest + React Testing Library

**Current Status:** Phase 0 - Frontend Setup (0% complete)

## 📚 Critical Documentation Ecosystem

This frontend project has a comprehensive documentation system. **Read these files in this order when starting:**

1. **`.claude/context.md`** - Complete frontend context (MUST READ FIRST)
2. **`.claude/conventions.md`** - Mandatory naming conventions
3. **`.docs/CODING_STANDARDS.md`** - Detailed React/TS coding standards
4. **`.claude/session-state.md`** - Current development status
5. **This file (CLAUDE.md)** - Quick reference guide

**Other AI Configurations:**
- `.copilot/instructions.md` - Instructions for GitHub Copilot
- `.cursorrules` - Rules for Cursor AI
- `.docs/SETUP_INICIAL.md` - Step-by-step setup guide
- `.docs/ARQUITECTURA.md` - Architectural decisions
- `.docs/EJEMPLOS_CODIGO.md` - Complete code examples

**Backend (Separate Repository):**
- The backend is developed separately in repository `bo-backend-api`
- Communication via REST API with `Authorization` and `X-Tenant-ID` headers

## 🐳 Development Environment: DOCKER (MANDATORY)

**⚠️ CRITICAL: This frontend MUST be developed using Docker.**

### Why Docker is Mandatory

1. **Consistency**: Same environment in dev, staging, prod
2. **Onboarding**: New dev productive in 5 minutes
3. **No pollution**: Dependencies isolated from your system
4. **Nginx configured**: Production-ready web server
5. **CI/CD Ready**: Same container everywhere

### Docker Commands (Use These)

```bash
# Start frontend environment
docker compose up -d

# View logs
docker compose logs -f frontend

# Execute commands inside container
docker compose exec frontend npm install [package]
docker compose exec frontend npm run test
docker compose exec frontend npm run build

# Stop environment
docker compose down

# Full stack (with backend integration)
docker compose --profile full up -d
```

### ❌ DON'T Run Commands Directly

```bash
# ❌ NEVER do this (runs on your machine):
npm install
npm run dev
npm run build

# ✅ ALWAYS do this (runs in Docker):
docker compose up -d
docker compose exec frontend npm install
```

## 📊 Key Technical Decisions

### Frontend Framework: React 18 ✅

**Decision:** Use **React 18** (not Vue, not Angular)

**Why:**
- ✅ **Most popular**: Largest community, more resources
- ✅ **Concurrent Features**: Better UX with smooth transitions
- ✅ **Suspense for Data**: Elegant loading states
- ✅ **TypeScript first-class**: Excellent type support
- ✅ **Rich ecosystem**: Thousands of compatible libraries

**Trade-off accepted:** Slightly larger bundle than Svelte, but worth it for the ecosystem

### Build Tool: Vite 5 ✅

**Decision:** Use **Vite** (not Webpack, not Create React App)

**Why:**
- ✅ **Ultra-fast dev server**: < 1s start, < 50ms HMR
- ✅ **Optimized build**: Rollup under the hood
- ✅ **TypeScript out-of-the-box**: No extra config
- ✅ **Simple config**: Clean and documented API

**How it works:**
```typescript
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### State Management: Zustand + React Query ✅

**Decision:** **Zustand** for client state, **React Query** for server state

**Why:**
- ✅ Zustand: Minimal (1KB), no boilerplate, TypeScript-first
- ✅ React Query: Automatic caching, background refetch, optimistic updates
- ✅ Clear separation: Client state vs Server state

**Alternatives discarded:**
- ❌ Redux: Too much boilerplate
- ❌ Context API alone: Performance issues with frequent updates

### UI: Shadcn/UI + Tailwind ✅

**Decision:** **Shadcn/UI** (not MUI, not Ant Design)

**Why:**
- ✅ **Copy-paste approach**: Code in your project, not node_modules
- ✅ **Fully customizable**: Modify code directly
- ✅ **Tailwind-first**: No CSS-in-JS runtime overhead
- ✅ **Accessible**: Radix UI under the hood (WCAG AA)

## Development Commands (Inside Docker Container)

When you exec into the container, these commands are available:

### Core Development
```bash
npm run dev              # Start dev server with HMR
npm run build            # Build for production
npm run preview          # Preview production build
npm run lint             # Run ESLint
npm run format           # Format with Prettier
npm run test             # Run unit tests
npm run test:watch       # Run tests in watch mode
npm run test:coverage    # Generate coverage report
```

## 🧠 AI Memory System (Anti-Forgetting)

This project implements a memory checkpoint system to prevent context loss:

### Every 5 Messages - Memory Checkpoint
When working with Claude Code, every ~5 messages you should mentally verify:
- [ ] Project name: **bMOI Admin Web** (React SPA)
- [ ] Framework: **React 18 + TypeScript 5 + Vite**
- [ ] State: **Zustand** (client) + **React Query** (server)
- [ ] UI: **Tailwind CSS + Shadcn/UI**
- [ ] Multi-tenant: **X-Tenant-ID header MANDATORY**
- [ ] Conventions: **PascalCase** (components), **camelCase** (variables), **kebab-case** (files)
- [ ] **NEVER** use `any` type
- [ ] **ALWAYS** tipar props con interfaces
- [ ] **ALWAYS** include X-Tenant-ID in requests
- [ ] **ALWAYS** validate with Zod
- [ ] **ALWAYS** handle loading/error states

### End of Session - Update State
Before ending a development session, update `.claude/session-state.md` with:
- Components created today
- Features implemented
- Technical decisions made
- Problems encountered and solutions
- Next steps
- Current phase progress

## Architecture & Key Patterns

### Multi-Tenant Architecture

**Critical**: Every API request MUST include tenant context.

**Flow:**
```
1. User logs in
2. Backend returns accessible tenants list
3. User selects a tenant
4. Frontend saves tenant_id in localStorage
5. All requests include X-Tenant-ID header (interceptor)
6. Backend validates access and applies RLS
```

**Implementation:**
```typescript
// Axios interceptor (MANDATORY)
apiClient.interceptors.request.use((config) => {
  const tenantId = localStorage.getItem('selected_tenant_id');
  if (tenantId && config.headers) {
    config.headers['X-Tenant-ID'] = tenantId;
  }
  return config;
});
```

### Naming Conventions (MANDATORY)

From `.claude/conventions.md` - these are strictly enforced:
- **Files:**
  - Components: PascalCase (`UserCard.tsx`)
  - Hooks: kebab-case (`use-users.ts`)
  - Services: kebab-case + `.service` (`users.service.ts`)
  - Utils: kebab-case (`date-utils.ts`)
- **Code:**
  - Components: PascalCase (`UserCard`)
  - Variables/Functions: camelCase (`userName`, `handleSubmit`)
  - Constants: UPPER_SNAKE_CASE (`MAX_USERS`, `API_URL`)
  - Interfaces: PascalCase NO "I" prefix (`UserCardProps`)
- **NEVER** use snake_case in TypeScript

### Component Structure

Every React component should follow this structure:
```typescript
// UserCard.tsx
import React from 'react';
import { Button } from '@/components/ui/button';
import type { User } from '@/types/user.types';

// 1. Props interface
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}

// 2. Component
export const UserCard = ({ user, onEdit }: UserCardProps) => {
  // 3. Handlers outside JSX
  const handleEdit = () => {
    onEdit(user);
  };

  // 4. Return JSX
  return (
    <div className="rounded-lg border p-4">
      <h3>{user.name}</h3>
      <Button onClick={handleEdit}>Edit</Button>
    </div>
  );
};
```

## Common Patterns

### Creating a New Page

```typescript
// src/pages/users/UsersPage.tsx
import React from 'react';
import { useUsers } from '@/hooks/queries/use-users';
import { UserCard } from '@/components/organisms/UserCard';
import { Skeleton } from '@/components/ui/skeleton';

export const UsersPage = () => {
  const { data: users, isLoading, error } = useUsers();

  if (isLoading) return <Skeleton />;
  if (error) return <div>Error: {error.message}</div>;
  if (!users) return <div>No users found</div>;

  return (
    <div className="grid grid-cols-3 gap-4">
      {users.map(user => (
        <UserCard key={user.id} user={user} onEdit={handleEdit} />
      ))}
    </div>
  );
};
```

### Creating a Service

```typescript
// src/services/users.service.ts
import { apiClient } from './api-client';
import type { User, CreateUserDto } from '@/types/user.types';

export const usersService = {
  async getAll(): Promise<User[]> {
    const response = await apiClient.get<User[]>('/users');
    return response.data;
  },

  async create(data: CreateUserDto): Promise<User> {
    const response = await apiClient.post<User>('/users', data);
    return response.data;
  },
};
```

### Creating a Query Hook

```typescript
// src/hooks/queries/use-users.ts
import { useQuery } from '@tanstack/react-query';
import { usersService } from '@/services/users.service';

export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: usersService.getAll,
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
};
```

### Creating a Form

```typescript
// src/components/forms/UserForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().min(3),
  email: z.string().email(),
});

type CreateUserFormData = z.infer<typeof createUserSchema>;

export const UserForm = ({ onSubmit }: { onSubmit: (data: CreateUserFormData) => void }) => {
  const form = useForm<CreateUserFormData>({
    resolver: zodResolver(createUserSchema),
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <input {...form.register('name')} />
      {form.formState.errors.name && <span>{form.formState.errors.name.message}</span>}
      <button type="submit">Submit</button>
    </form>
  );
};
```

## Development Phases

The project is organized in phases. Current status from `.claude/session-state.md`:

```
Phase 0 (Setup Frontend):    ░░░░░░░░░░  0%  - Vite + Docker + Config
Phase 1 (Auth & Layout):     ░░░░░░░░░░  0%  - Login, JWT, Dashboard
Phase 2 (Core Modules):      ░░░░░░░░░░  0%  - Users, Products, Orders
Phase 3 (Advanced Features): ░░░░░░░░░░  0%  - Reports, Export, Charts
Phase 4 (Testing & Optim):   ░░░░░░░░░░  0%  - Tests, Performance, SEO
```

**Next Immediate Steps (Phase 0 completion):**
1. Initialize Vite project with React + TypeScript
2. Create docker-compose.yml (frontend + nginx)
3. Install dependencies (React Router, React Query, Zustand, etc.)
4. Configure Tailwind CSS + Shadcn/UI
5. Configure ESLint + Prettier
6. Create folder structure
7. Verify hot-reload works in Docker

## Important Notes

- **Never bypass tenant isolation**: All services must include X-Tenant-ID header
- **Permission checks in both places**: Frontend (UX) + Backend (security)
- **Form validation**: Always use Zod schemas
- **Development logs**: Remove console.log before commit
- **Test coverage**: Minimum 80%

## Troubleshooting AI Behavior

### If AI forgets conventions:
```
🧠 CONVENTION REMINDER - FRONTEND:

This project uses STRICTLY:
- Components: PascalCase (UserCard.tsx)
- Hooks: kebab-case + "use" (use-users.ts)
- Variables: camelCase (userName)
- Constants: UPPER_SNAKE_CASE (MAX_USERS)
- NEVER snake_case in TypeScript
- NEVER any type

Multi-tenant MANDATORY:
- X-Tenant-ID in ALL requests (interceptor handles it)
- Verify tenant selected in pages

Please confirm you'll remember this.
```

### If AI creates code without types:
```
⚠️ ERROR: Missing types

ALL components MUST:
1. Have interface Props defined
2. Type the component correctly
3. NEVER use "any"

Example:
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}

export const UserCard = ({ user, onEdit }: UserCardProps) => {
  // ...
};

Please correct the code above.
```

## Additional Resources

### Core Documentation
- Frontend architecture in `.docs/ARQUITECTURA.md`
- **Responsive Design Guide** in `.docs/RESPONSIVE_DESIGN.md` - **MUST READ** for UI development
- Docker quickstart in `.docs/SETUP_INICIAL.md`
- Complete code examples in `.docs/EJEMPLOS_CODIGO.md`
- Configuration files in `.docs/CONFIGURACIONES.md`
- `.cursorrules` contains mandatory coding standards for Cursor AI

### 🎯 bMOI-Specific Documentation (CRITICAL)
- **`.docs/BMOI_STACK_EVALUATION.md`** - Stack tecnológico evaluado para bMOI Admin (KYC, moderación, disputas, dashboards)
- **`.docs/BMOI_KYC_WORKFLOWS.md`** - Sistema completo de KYC (estados, componentes, servicios, ejemplos de código)
- **`.docs/BMOI_SECURITY_BEST_PRACTICES.md`** - MFA, RBAC/ABAC, auditoría, buenas prácticas obligatorias

**⚠️ IMPORTANTE:** Lee los documentos bMOI-specific ANTES de implementar cualquier feature de KYC, moderación, disputas o seguridad avanzada.

---

<div align="center">

**Frontend Multi-Tenant con React + TypeScript**

Type-safe | Performante | Escalable | Docker-first

</div>
