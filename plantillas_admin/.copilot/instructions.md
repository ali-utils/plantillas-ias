# GitHub Copilot Instructions - Admin Web Frontend

> **Instrucciones para GitHub Copilot al trabajar en el Admin Web de bMOI**

---

## 🎯 Proyecto Overview

Estás trabajando en el **bMOI Admin Web**, una aplicación SPA (Single Page Application) construida con:
- **React 18** + **TypeScript 5**
- **Vite 5** (build tool)
- **Tailwind CSS** + **Shadcn/UI** (styling)
- **React Router 6** (navegación)
- **Zustand** (client state) + **React Query** (server state)
- **React Hook Form** + **Zod** (forms y validación)

**Propósito:** Panel de administración multi-tenant para gestionar usuarios, productos, órdenes y configuración del tenant.

---

## 📏 Convenciones de Nomenclatura (OBLIGATORIAS)

### Archivos

```
# ✅ CORRECTO
UserCard.tsx              # Componentes en PascalCase
use-users.ts              # Hooks en kebab-case con prefijo "use"
users.service.ts          # Services en kebab-case + ".service"
date-utils.ts             # Utils en kebab-case
user.types.ts             # Types en kebab-case + ".types"

# ❌ INCORRECTO
userCard.tsx              # ❌ camelCase
user_card.tsx             # ❌ snake_case
UsersService.ts           # ❌ PascalCase para services
```

### Código

```typescript
// ✅ CORRECTO
export const UserCard = () => { };         // Componentes: PascalCase
const userName = 'John';                   // Variables: camelCase
const MAX_USERS = 100;                     // Constantes: UPPER_SNAKE_CASE
interface UserCardProps { }                // Interfaces: PascalCase SIN prefijo "I"
export const useAuth = () => { };          // Hooks: prefijo "use" + camelCase

// ❌ INCORRECTO
export const userCard = () => { };         // ❌
const UserName = 'John';                   // ❌
const max_users = 100;                     // ❌
interface IUserCardProps { }               // ❌ No usar prefijo "I"
export const auth = () => { };             // ❌ Falta prefijo "use"
```

---

## 🧩 Estructura de Componentes

### Siempre usar este template:

```typescript
import React from 'react';
import { Button } from '@/components/ui/button';
import type { User } from '@/types/user.types';

// 1. Props interface primero
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
  onDelete?: (userId: string) => void;
}

// 2. Componente funcional con tipos
export const UserCard = ({ user, onEdit, onDelete }: UserCardProps) => {
  // 3. Handlers declarados fuera del JSX
  const handleEdit = () => {
    onEdit(user);
  };

  const handleDelete = () => {
    if (onDelete) {
      onDelete(user.id);
    }
  };

  // 4. Return con JSX
  return (
    <div className="rounded-lg border p-4">
      <h3 className="text-lg font-semibold">{user.name}</h3>
      <p className="text-gray-600">{user.email}</p>
      <div className="mt-4 flex gap-2">
        <Button onClick={handleEdit}>Edit</Button>
        {onDelete && (
          <Button variant="destructive" onClick={handleDelete}>
            Delete
          </Button>
        )}
      </div>
    </div>
  );
};
```

### Reglas:
- **SIEMPRE** definir interface Props
- **NUNCA** usar `any` (usar tipos específicos o `unknown`)
- **NUNCA** inline functions en JSX (declarar fuera)
- **SIEMPRE** manejar loading y error states
- **SIEMPRE** usar path alias `@/` (no paths relativos)

---

## 🪝 Custom Hooks

### Template:

```typescript
import { useState, useCallback, useMemo } from 'react';

// 1. Types del hook
interface UsePaginationOptions {
  totalItems: number;
  itemsPerPage?: number;
}

interface UsePaginationReturn {
  currentPage: number;
  totalPages: number;
  goToPage: (page: number) => void;
  nextPage: () => void;
  prevPage: () => void;
}

// 2. Hook (prefijo "use" OBLIGATORIO)
export const usePagination = ({
  totalItems,
  itemsPerPage = 10,
}: UsePaginationOptions): UsePaginationReturn => {
  const [currentPage, setCurrentPage] = useState(1);

  const totalPages = useMemo(
    () => Math.ceil(totalItems / itemsPerPage),
    [totalItems, itemsPerPage]
  );

  const goToPage = useCallback(
    (page: number) => {
      setCurrentPage(Math.max(1, Math.min(page, totalPages)));
    },
    [totalPages]
  );

  const nextPage = useCallback(() => {
    goToPage(currentPage + 1);
  }, [currentPage, goToPage]);

  const prevPage = useCallback(() => {
    goToPage(currentPage - 1);
  }, [currentPage, goToPage]);

  return {
    currentPage,
    totalPages,
    goToPage,
    nextPage,
    prevPage,
  };
};
```

---

## 🌐 Services (API Calls)

### Template:

```typescript
import { apiClient } from './api-client';
import type { User, CreateUserDto, UpdateUserDto } from '@/types/user.types';

export const usersService = {
  async getAll(): Promise<User[]> {
    const response = await apiClient.get<User[]>('/users');
    return response.data;
  },

  async getById(id: string): Promise<User> {
    const response = await apiClient.get<User>(`/users/${id}`);
    return response.data;
  },

  async create(data: CreateUserDto): Promise<User> {
    const response = await apiClient.post<User>('/users', data);
    return response.data;
  },

  async update(id: string, data: UpdateUserDto): Promise<User> {
    const response = await apiClient.patch<User>(`/users/${id}`, data);
    return response.data;
  },

  async delete(id: string): Promise<void> {
    await apiClient.delete(`/users/${id}`);
  },
};
```

**Nota:** `apiClient` ya tiene interceptors configurados para JWT y X-Tenant-ID.

---

## 🗂️ React Query Hooks

### Template para Queries:

```typescript
import { useQuery } from '@tanstack/react-query';
import { usersService } from '@/services/users.service';
import type { User } from '@/types/user.types';

export const useUsers = () => {
  return useQuery({
    queryKey: ['users'],
    queryFn: usersService.getAll,
    staleTime: 5 * 60 * 1000,  // 5 minutos
  });
};

export const useUser = (id: string) => {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => usersService.getById(id),
    enabled: !!id,  // Solo ejecutar si hay ID
  });
};
```

### Template para Mutations:

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { usersService } from '@/services/users.service';
import type { CreateUserDto } from '@/types/user.types';

export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: usersService.create,
    onSuccess: () => {
      // Invalidar cache para refetch
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
};
```

---

## 📝 Forms con React Hook Form + Zod

### Template:

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';

// 1. Schema Zod
const createUserSchema = z.object({
  name: z.string().min(3, 'Name must be at least 3 characters'),
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

type CreateUserFormData = z.infer<typeof createUserSchema>;

// 2. Componente de formulario
interface UserFormProps {
  onSubmit: (data: CreateUserFormData) => void;
}

export const UserForm = ({ onSubmit }: UserFormProps) => {
  const form = useForm<CreateUserFormData>({
    resolver: zodResolver(createUserSchema),
    defaultValues: {
      name: '',
      email: '',
      password: '',
    },
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
      <div>
        <label>Name</label>
        <Input {...form.register('name')} />
        {form.formState.errors.name && (
          <span className="text-sm text-red-500">
            {form.formState.errors.name.message}
          </span>
        )}
      </div>

      <div>
        <label>Email</label>
        <Input type="email" {...form.register('email')} />
        {form.formState.errors.email && (
          <span className="text-sm text-red-500">
            {form.formState.errors.email.message}
          </span>
        )}
      </div>

      <div>
        <label>Password</label>
        <Input type="password" {...form.register('password')} />
        {form.formState.errors.password && (
          <span className="text-sm text-red-500">
            {form.formState.errors.password.message}
          </span>
        )}
      </div>

      <Button type="submit" disabled={form.formState.isSubmitting}>
        {form.formState.isSubmitting ? 'Creating...' : 'Create User'}
      </Button>
    </form>
  );
};
```

---

## 🎨 Tailwind CSS

### Usar clases de Tailwind (no CSS inline):

```typescript
// ✅ CORRECTO
<div className="flex flex-col gap-4 rounded-lg border bg-white p-6 shadow-sm">
  <h2 className="text-xl font-semibold">Title</h2>
  <p className="text-gray-600">Description</p>
</div>

// ❌ INCORRECTO
<div style={{ display: 'flex', flexDirection: 'column' }}>
  {/* ❌ No usar inline styles */}
</div>
```

### Usar helper `cn()` para classes condicionales:

```typescript
import { cn } from '@/lib/utils';

<Button
  className={cn(
    'rounded-md px-4 py-2',
    isPrimary && 'bg-blue-500 text-white',
    isDisabled && 'opacity-50 cursor-not-allowed'
  )}
>
  Click me
</Button>
```

---

## 🔒 Multi-Tenant (CRÍTICO)

### TODAS las requests deben incluir X-Tenant-ID:

El interceptor ya lo hace automáticamente:

```typescript
// api-client.ts (ya configurado)
apiClient.interceptors.request.use((config) => {
  // JWT Token
  const token = localStorage.getItem('access_token');
  if (token && config.headers) {
    config.headers.Authorization = `Bearer ${token}`;
  }

  // Tenant ID (OBLIGATORIO)
  const tenantId = localStorage.getItem('selected_tenant_id');
  if (tenantId && config.headers) {
    config.headers['X-Tenant-ID'] = tenantId;
  }

  return config;
});
```

### En componentes, verificar tenant:

```typescript
const UsersPage = () => {
  const selectedTenantId = useAuthStore((state) => state.selectedTenantId);

  if (!selectedTenantId) {
    return <div>Please select a tenant</div>;
  }

  const { data: users, isLoading, error } = useUsers();

  // ...
};
```

---

## 🔐 Verificación de Permisos

### Usar hook usePermissions:

```typescript
import { usePermissions } from '@/hooks/use-permissions';

const UsersPage = () => {
  const { hasPermission } = usePermissions();

  return (
    <div>
      <h1>Users</h1>

      {hasPermission('users.create') && (
        <Button onClick={handleCreate}>
          Create User
        </Button>
      )}

      {hasPermission('users.delete') && (
        <Button variant="destructive" onClick={handleDelete}>
          Delete User
        </Button>
      )}
    </div>
  );
};
```

---

## 📦 Orden de Imports (ESTRICTO)

```typescript
// 1. React y librerías de React
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

// 2. Librerías externas
import { useQuery } from '@tanstack/react-query';
import { useForm } from 'react-hook-form';

// 3. Componentes UI
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';

// 4. Componentes internos
import { UserCard } from '@/components/organisms/UserCard';

// 5. Hooks
import { useAuth } from '@/hooks/use-auth';
import { useUsers } from '@/hooks/queries/use-users';

// 6. Services
import { usersService } from '@/services/users.service';

// 7. Stores
import { useAuthStore } from '@/stores/auth.store';

// 8. Utils
import { formatDate } from '@/lib/date-utils';
import { cn } from '@/lib/utils';

// 9. Types
import type { User } from '@/types/user.types';

// 10. Styles (último)
import styles from './Component.module.css';
```

---

## 🚨 Errores Comunes a Evitar

### ❌ NO hacer:

```typescript
// 1. Inline functions en JSX (performance)
<Button onClick={() => handleEdit(user)}>Edit</Button>  // ❌

// 2. Usar "any"
const handleSubmit = (data: any) => { };  // ❌

// 3. No manejar loading/error states
const { data } = useUsers();  // ❌ ¿Y si está cargando o falla?
return <div>{data.map(...)}</div>;

// 4. Paths relativos
import { Button } from '../../../components/ui/button';  // ❌

// 5. No tipar props
export const UserCard = ({ user, onEdit }) => { };  // ❌
```

### ✅ SÍ hacer:

```typescript
// 1. Declarar handlers fuera
const handleEdit = () => handleEditUser(user);
<Button onClick={handleEdit}>Edit</Button>  // ✅

// 2. Tipos específicos
const handleSubmit = (data: CreateUserFormData) => { };  // ✅

// 3. Manejar todos los estados
const { data, isLoading, error } = useUsers();
if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage />;
return <div>{data?.map(...)}</div>;  // ✅

// 4. Usar alias @/
import { Button } from '@/components/ui/button';  // ✅

// 5. Tipar props
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}
export const UserCard = ({ user, onEdit }: UserCardProps) => { };  // ✅
```

---

## 🧪 Testing

### Usar Vitest + React Testing Library:

```typescript
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserCard } from '../UserCard';

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
  };

  it('should render user information', () => {
    render(<UserCard user={mockUser} onEdit={vi.fn()} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('should call onEdit when button clicked', async () => {
    const onEdit = vi.fn();
    const user = userEvent.setup();

    render(<UserCard user={mockUser} onEdit={onEdit} />);

    await user.click(screen.getByRole('button', { name: /edit/i }));

    expect(onEdit).toHaveBeenCalledWith(mockUser);
  });
});
```

---

## 📚 Recursos

### Documentación del Proyecto
- Context completo: `.claude/context.md`
- Convenciones: `.claude/conventions.md`
- Coding Standards: `.docs/CODING_STANDARDS.md`
- Ejemplos de código: `.docs/EJEMPLOS_CODIGO.md`
- Setup inicial: `.docs/SETUP_INICIAL.md`

### Documentación Externa
- React: https://react.dev/
- TypeScript: https://www.typescriptlang.org/
- Vite: https://vitejs.dev/
- React Router: https://reactrouter.com/
- TanStack Query: https://tanstack.com/query/latest
- Zustand: https://zustand-demo.pmnd.rs/
- Tailwind CSS: https://tailwindcss.com/
- Shadcn/UI: https://ui.shadcn.com/

---

## 🎯 Resumen Rápido

**Cuando generes código:**

1. ✅ **SIEMPRE tipar** props, variables, funciones
2. ✅ **NUNCA usar `any`** → usar tipos específicos
3. ✅ **Componentes en PascalCase**, variables en camelCase
4. ✅ **Usar alias `@/`** para imports
5. ✅ **Declarar handlers fuera** del JSX
6. ✅ **Manejar loading/error states**
7. ✅ **Incluir X-Tenant-ID** en todas las requests (interceptor lo hace)
8. ✅ **Validar con Zod** en formularios
9. ✅ **Tests para nueva funcionalidad**
10. ✅ **Tailwind CSS** para estilos

---

<div align="center">

**GitHub Copilot configurado para bMOI Admin Web**

Sigue estas convenciones estrictamente

</div>
