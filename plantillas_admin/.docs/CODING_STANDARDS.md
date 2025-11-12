# 📏 Estándares de Código - Admin Web

> **Guía completa de convenciones, patrones y mejores prácticas para el frontend de bMOI**

---

## 📋 Índice

1. [Convenciones de Nomenclatura](#convenciones-de-nomenclatura)
2. [Estructura de Archivos](#estructura-de-archivos)
3. [Componentes](#componentes)
4. [Hooks](#hooks)
5. [Services y API](#services-y-api)
6. [Types e Interfaces](#types-e-interfaces)
7. [State Management](#state-management)
8. [Forms y Validación](#forms-y-validación)
9. [Estilos y CSS](#estilos-y-css)
10. [Testing](#testing)
11. [Performance](#performance)
12. [ESLint y Prettier](#eslint-y-prettier)
13. [Git y Commits](#git-y-commits)
14. [Code Review](#code-review)

---

## 🏷️ Convenciones de Nomenclatura

### Archivos y Carpetas

```typescript
// ✅ BIEN - PascalCase para componentes React
UserCard.tsx
DashboardLayout.tsx
CreateProductForm.tsx

// ✅ BIEN - kebab-case para archivos de utilidades
date-utils.ts
string-utils.ts
api-client.ts

// ✅ BIEN - kebab-case para hooks
use-users.ts
use-pagination.ts
use-debounce.ts

// ✅ BIEN - kebab-case para services
auth.service.ts
users.service.ts
products.service.ts

// ❌ MAL - snake_case
user_card.tsx           // ❌
dashboard_layout.tsx    // ❌

// ❌ MAL - camelCase para archivos
userCard.tsx            // ❌
dashboardLayout.tsx     // ❌
```

### Componentes

```typescript
// ✅ BIEN - PascalCase para componentes
export const UserCard = () => { };
export const DashboardLayout = () => { };
export const CreateProductModal = () => { };

// ❌ MAL - camelCase o snake_case
export const userCard = () => { };         // ❌
export const user_card = () => { };        // ❌
```

### Variables y Funciones

```typescript
// ✅ BIEN - camelCase
const userName = 'John';
const isAuthenticated = true;
const handleSubmit = () => { };
const fetchUsers = async () => { };

// ❌ MAL - snake_case o PascalCase
const user_name = 'John';                  // ❌
const UserName = 'John';                   // ❌
const handle_submit = () => { };           // ❌
```

### Constantes

```typescript
// ✅ BIEN - UPPER_SNAKE_CASE para constantes verdaderas
const MAX_USERS_PER_PAGE = 10;
const API_BASE_URL = 'https://api.bmoi.com';
const DEFAULT_TIMEOUT = 5000;

// ✅ BIEN - camelCase para valores que no cambian pero no son configuración
const colors = {
  primary: '#007bff',
  secondary: '#6c757d',
} as const;

// ❌ MAL - PascalCase o camelCase para constantes de configuración
const maxUsersPerPage = 10;               // ❌
const ApiBaseUrl = 'https://api.bmoi.com'; // ❌
```

### Types e Interfaces

```typescript
// ✅ BIEN - PascalCase, interfaces sin prefijo "I"
interface User {
  id: string;
  name: string;
}

type UserFilters = {
  search?: string;
  role?: string;
};

// ✅ BIEN - Prefijo para Props
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}

// ❌ MAL - Prefijo "I" en interfaces (estilo C#)
interface IUser {                          // ❌
  id: string;
}

// ❌ MAL - snake_case o camelCase
interface user {                           // ❌
  id: string;
}
```

### Enums

```typescript
// ✅ BIEN - PascalCase para nombre, UPPER_SNAKE_CASE para valores
enum UserRole {
  ADMIN = 'ADMIN',
  USER = 'USER',
  GUEST = 'GUEST',
}

enum OrderStatus {
  PENDING = 'PENDING',
  CONFIRMED = 'CONFIRMED',
  SHIPPED = 'SHIPPED',
  DELIVERED = 'DELIVERED',
}

// ⚠️ MEJOR - Usar const objects con "as const" en lugar de enums
const UserRole = {
  ADMIN: 'ADMIN',
  USER: 'USER',
  GUEST: 'GUEST',
} as const;

type UserRole = typeof UserRole[keyof typeof UserRole];
```

### Carpetas

```typescript
// ✅ BIEN - kebab-case para carpetas
components/
hooks/
services/
utils/
api-client/

// ❌ MAL - PascalCase o snake_case
Components/        // ❌
api_client/        // ❌
```

---

## 📁 Estructura de Archivos

### Estructura de Componente

```
UserCard/
├── index.tsx                 # Componente principal
├── UserCard.test.tsx         # Tests
├── UserCard.stories.tsx      # Storybook (opcional)
└── types.ts                  # Types específicos (si es complejo)
```

**index.tsx:**

```typescript
// ✅ BIEN - Export default al final
import React from 'react';
import { Card, CardHeader, CardContent } from '@/components/ui/card';
import type { UserCardProps } from './types';

export const UserCard = ({ user, onEdit }: UserCardProps) => {
  return (
    <Card>
      <CardHeader>
        <h3>{user.name}</h3>
      </CardHeader>
      <CardContent>
        <p>{user.email}</p>
        <button onClick={() => onEdit(user)}>Edit</button>
      </CardContent>
    </Card>
  );
};

// ❌ MAL - Export default inline
export default function UserCard() { }
```

### Orden de Imports

```typescript
// ✅ BIEN - Orden correcto
// 1. React y librerías externas
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { useQuery } from '@tanstack/react-query';

// 2. Componentes internos
import { Button } from '@/components/ui/button';
import { UserCard } from '@/components/organisms/UserCard';

// 3. Hooks
import { useAuth } from '@/hooks/use-auth';
import { useUsers } from '@/hooks/queries/use-users';

// 4. Services y utils
import { usersService } from '@/services/users.service';
import { formatDate } from '@/lib/date-utils';

// 5. Types
import type { User, UserFilters } from '@/types/user.types';

// 6. Styles (si hay CSS modules)
import styles from './UsersPage.module.css';

// ❌ MAL - Sin orden
import { formatDate } from '@/lib/date-utils';
import React from 'react';
import type { User } from '@/types/user.types';
import { useAuth } from '@/hooks/use-auth';
```

---

## 🧩 Componentes

### Componente Funcional

```typescript
// ✅ BIEN - Componente funcional con TypeScript
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
  onDelete?: (userId: string) => void;
}

export const UserCard = ({ user, onEdit, onDelete }: UserCardProps) => {
  const [isExpanded, setIsExpanded] = useState(false);

  const handleEdit = () => {
    onEdit(user);
  };

  const handleDelete = () => {
    if (onDelete) {
      onDelete(user.id);
    }
  };

  return (
    <Card>
      <CardHeader>
        <CardTitle>{user.name}</CardTitle>
      </CardHeader>
      <CardContent>
        <p>{user.email}</p>
        {isExpanded && (
          <div>
            <p>Role: {user.role}</p>
            <p>Created: {formatDate(user.createdAt)}</p>
          </div>
        )}
        <div className="flex gap-2">
          <Button onClick={handleEdit}>Edit</Button>
          {onDelete && (
            <Button variant="destructive" onClick={handleDelete}>
              Delete
            </Button>
          )}
        </div>
      </CardContent>
    </Card>
  );
};

// ❌ MAL - Sin tipos
export const UserCard = ({ user, onEdit }) => {  // ❌
  // ...
};

// ❌ MAL - Funciones arrow inline en JSX
<Button onClick={() => onEdit(user)}>Edit</Button>  // ❌ (re-crea función en cada render)
```

### Props Destructuring

```typescript
// ✅ BIEN - Destructuring en parámetros
export const UserCard = ({ user, onEdit }: UserCardProps) => {
  return <div>{user.name}</div>;
};

// ✅ BIEN - Destructuring con valores por defecto
export const UserCard = ({
  user,
  onEdit,
  showActions = true,
  variant = 'default',
}: UserCardProps) => {
  return <div>{user.name}</div>;
};

// ❌ MAL - Sin destructuring
export const UserCard = (props: UserCardProps) => {  // ❌
  return <div>{props.user.name}</div>;
};
```

### Componentes Presentacionales vs Contenedores

```typescript
// ✅ BIEN - Componente Presentacional (solo UI, sin lógica de negocio)
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}

export const UserCard = ({ user, onEdit }: UserCardProps) => {
  return (
    <Card>
      <CardHeader>{user.name}</CardHeader>
      <CardContent>{user.email}</CardContent>
      <CardFooter>
        <Button onClick={() => onEdit(user)}>Edit</Button>
      </CardFooter>
    </Card>
  );
};

// ✅ BIEN - Componente Contenedor (con lógica, conecta con stores/queries)
export const UsersPage = () => {
  const { data: users, isLoading } = useUsers();
  const { mutate: updateUser } = useUpdateUser();

  const handleEdit = (user: User) => {
    // Lógica de negocio
    updateUser(user);
  };

  if (isLoading) return <Skeleton />;

  return (
    <div className="grid grid-cols-3 gap-4">
      {users?.map(user => (
        <UserCard key={user.id} user={user} onEdit={handleEdit} />
      ))}
    </div>
  );
};

// ❌ MAL - Todo mezclado
export const UserCard = ({ userId }: { userId: string }) => {  // ❌
  const { data: user } = useUser(userId);  // ❌ API call en componente presentacional
  const { mutate: updateUser } = useUpdateUser();  // ❌

  if (!user) return null;

  return (
    <Card>
      <CardHeader>{user.name}</CardHeader>
      <Button onClick={() => updateUser(user)}>Edit</Button>
    </Card>
  );
};
```

### Conditional Rendering

```typescript
// ✅ BIEN - Operador ternario para if-else
{isLoading ? <Skeleton /> : <UsersList users={users} />}

// ✅ BIEN - && para if simple (cuando el false no renderiza nada)
{hasPermission && <Button>Create</Button>}

// ✅ BIEN - Early return para casos complejos
export const UsersPage = () => {
  const { data: users, isLoading, error } = useUsers();

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;
  if (!users || users.length === 0) return <EmptyState />;

  return <UsersList users={users} />;
};

// ❌ MAL - Ternarios anidados
{isLoading ? (
  <Skeleton />
) : error ? (
  <ErrorMessage />
) : users.length === 0 ? (  // ❌ Muy difícil de leer
  <EmptyState />
) : (
  <UsersList />
)}

// ❌ MAL - Usar && con números (puede renderizar 0)
{users.length && <UsersList />}  // ❌ Si users.length es 0, renderiza "0"

// ✅ MEJOR
{users.length > 0 && <UsersList />}
```

### Event Handlers

```typescript
// ✅ BIEN - Función declarada fuera del JSX
export const UserCard = ({ user, onEdit }: UserCardProps) => {
  const handleEdit = () => {
    onEdit(user);
  };

  const handleDelete = () => {
    if (window.confirm('Are you sure?')) {
      deleteUser(user.id);
    }
  };

  return (
    <div>
      <Button onClick={handleEdit}>Edit</Button>
      <Button onClick={handleDelete}>Delete</Button>
    </div>
  );
};

// ❌ MAL - Arrow function inline (re-crea en cada render)
<Button onClick={() => onEdit(user)}>Edit</Button>  // ❌

// ❌ MAL - Invocar función directamente (ejecuta en render)
<Button onClick={onEdit(user)}>Edit</Button>  // ❌ Se ejecuta inmediatamente

// ✅ EXCEPCIÓN - Inline está OK si no hay parámetros y es simple
<Button onClick={onClose}>Close</Button>  // ✅ OK
```

### Children Pattern

```typescript
// ✅ BIEN - Usar children para composición
interface CardProps {
  children: React.ReactNode;
  className?: string;
}

export const Card = ({ children, className }: CardProps) => {
  return (
    <div className={cn('rounded-lg border bg-card', className)}>
      {children}
    </div>
  );
};

// Uso:
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>
    <p>Content here</p>
  </CardContent>
</Card>

// ✅ BIEN - Render props para lógica reutilizable
interface DataFetcherProps<T> {
  url: string;
  children: (data: T | null, isLoading: boolean) => React.ReactNode;
}

export const DataFetcher = <T,>({ url, children }: DataFetcherProps<T>) => {
  const { data, isLoading } = useQuery<T>({
    queryKey: [url],
    queryFn: () => fetch(url).then(res => res.json()),
  });

  return <>{children(data, isLoading)}</>;
};

// Uso:
<DataFetcher<User[]> url="/api/users">
  {(users, isLoading) => (
    isLoading ? <Skeleton /> : <UsersList users={users} />
  )}
</DataFetcher>
```

---

## 🪝 Hooks

### Custom Hooks Naming

```typescript
// ✅ BIEN - Prefijo "use"
export const useAuth = () => { };
export const useUsers = () => { };
export const usePagination = () => { };
export const useDebounce = () => { };

// ❌ MAL - Sin prefijo "use"
export const auth = () => { };         // ❌
export const getUsers = () => { };     // ❌
```

### Custom Hook Structure

```typescript
// ✅ BIEN - Custom hook completo
interface UsePaginationOptions {
  totalItems: number;
  itemsPerPage?: number;
}

interface UsePaginationReturn {
  currentPage: number;
  totalPages: number;
  startIndex: number;
  endIndex: number;
  goToPage: (page: number) => void;
  nextPage: () => void;
  prevPage: () => void;
  hasNextPage: boolean;
  hasPrevPage: boolean;
}

export const usePagination = ({
  totalItems,
  itemsPerPage = 10,
}: UsePaginationOptions): UsePaginationReturn => {
  const [currentPage, setCurrentPage] = useState(1);

  const totalPages = Math.ceil(totalItems / itemsPerPage);
  const startIndex = (currentPage - 1) * itemsPerPage;
  const endIndex = Math.min(startIndex + itemsPerPage, totalItems);

  const goToPage = useCallback((page: number) => {
    setCurrentPage(Math.max(1, Math.min(page, totalPages)));
  }, [totalPages]);

  const nextPage = useCallback(() => {
    goToPage(currentPage + 1);
  }, [currentPage, goToPage]);

  const prevPage = useCallback(() => {
    goToPage(currentPage - 1);
  }, [currentPage, goToPage]);

  return {
    currentPage,
    totalPages,
    startIndex,
    endIndex,
    goToPage,
    nextPage,
    prevPage,
    hasNextPage: currentPage < totalPages,
    hasPrevPage: currentPage > 1,
  };
};

// ❌ MAL - Sin tipos
export const usePagination = (totalItems, itemsPerPage = 10) => {  // ❌
  // ...
};
```

### React Query Hooks

```typescript
// ✅ BIEN - Hook de query con tipos y opciones
export const useUsers = (filters?: UserFilters) => {
  return useQuery({
    queryKey: ['users', filters],
    queryFn: () => usersService.getAll(filters),
    staleTime: 5 * 60 * 1000,
    gcTime: 10 * 60 * 1000,
    enabled: !!filters, // Solo ejecutar si hay filtros
  });
};

export const useUser = (id: string) => {
  return useQuery({
    queryKey: ['users', id],
    queryFn: () => usersService.getById(id),
    enabled: !!id,
  });
};

// ✅ BIEN - Hook de mutation con callbacks
export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: usersService.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      toast.success('User created successfully');
    },
    onError: (error: Error) => {
      toast.error(`Failed to create user: ${error.message}`);
    },
  });
};

// ❌ MAL - Sin tipos, sin opciones
export const useUsers = () => {  // ❌
  return useQuery(['users'], () => fetch('/api/users'));
};
```

### useEffect Best Practices

```typescript
// ✅ BIEN - useEffect con cleanup
export const useWebSocket = (url: string) => {
  useEffect(() => {
    const ws = new WebSocket(url);

    ws.onmessage = (event) => {
      console.log('Message:', event.data);
    };

    // Cleanup function
    return () => {
      ws.close();
    };
  }, [url]);
};

// ✅ BIEN - useEffect con dependencias correctas
export const UserProfile = ({ userId }: { userId: string }) => {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    let cancelled = false;

    const fetchUser = async () => {
      const data = await usersService.getById(userId);
      if (!cancelled) {
        setUser(data);
      }
    };

    fetchUser();

    return () => {
      cancelled = true;
    };
  }, [userId]); // ✅ userId en dependencias

  return <div>{user?.name}</div>;
};

// ❌ MAL - useEffect sin dependencias correctas
useEffect(() => {
  fetchUser(userId);
}, []); // ❌ Falta userId en dependencias

// ❌ MAL - useEffect sin cleanup
useEffect(() => {
  const interval = setInterval(() => {
    console.log('Tick');
  }, 1000);
  // ❌ Falta cleanup (se acumulan intervals)
}, []);
```

### useMemo y useCallback

```typescript
// ✅ BIEN - useMemo para cálculos costosos
export const ProductsPage = () => {
  const { data: products } = useProducts();

  const sortedProducts = useMemo(() => {
    return products?.sort((a, b) => b.price - a.price);
  }, [products]);

  const totalRevenue = useMemo(() => {
    return products?.reduce((sum, p) => sum + p.price, 0) || 0;
  }, [products]);

  return <ProductsList products={sortedProducts} />;
};

// ✅ BIEN - useCallback para funciones pasadas como props
export const UsersPage = () => {
  const { mutate: updateUser } = useUpdateUser();

  const handleEdit = useCallback((user: User) => {
    updateUser(user);
  }, [updateUser]);

  return (
    <div>
      {users.map(user => (
        <UserCard key={user.id} user={user} onEdit={handleEdit} />
      ))}
    </div>
  );
};

// ❌ MAL - useMemo innecesario (cálculos triviales)
const double = useMemo(() => count * 2, [count]);  // ❌ No necesita useMemo

// ❌ MAL - useCallback sin dependencias correctas
const handleEdit = useCallback((user: User) => {
  updateUser(user);
}, []); // ❌ Falta updateUser en dependencias
```

---

## 🌐 Services y API

### Service Structure

```typescript
// ✅ BIEN - Service con tipos y manejo de errores
// src/services/users.service.ts
import { apiClient } from './api-client';
import type { User, CreateUserDto, UpdateUserDto, UserFilters } from '@/types/user.types';

export const usersService = {
  async getAll(filters?: UserFilters): Promise<User[]> {
    const response = await apiClient.get<User[]>('/users', {
      params: filters,
    });
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

// ❌ MAL - Sin tipos, sin estructura
export const getUsers = async () => {  // ❌
  return fetch('/api/users').then(res => res.json());
};
```

### API Client Configuration

```typescript
// ✅ BIEN - Axios configurado con interceptors
// src/services/api-client.ts
import axios from 'axios';
import { authService } from './auth.service';

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:3000/api/v1',
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor - agregar JWT y Tenant ID
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access_token');
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    const tenantId = localStorage.getItem('selected_tenant_id');
    if (tenantId && config.headers) {
      config.headers['X-Tenant-ID'] = tenantId;
    }

    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor - refresh token en 401
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        const newAccessToken = await authService.refreshToken();
        originalRequest.headers.Authorization = `Bearer ${newAccessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        authService.logout();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);
```

---

## 📝 Types e Interfaces

### Interfaces vs Types

```typescript
// ✅ BIEN - Interfaces para objetos que pueden extenderse
interface User {
  id: string;
  name: string;
  email: string;
}

interface Admin extends User {
  permissions: string[];
}

// ✅ BIEN - Types para unions, intersections, utilities
type UserRole = 'ADMIN' | 'USER' | 'GUEST';
type UserWithRole = User & { role: UserRole };
type Optional<T> = { [K in keyof T]?: T[K] };

// ✅ BIEN - Types para props
type UserCardProps = {
  user: User;
  onEdit: (user: User) => void;
};
```

### Evitar "any"

```typescript
// ❌ MAL - Usar "any"
const handleSubmit = (data: any) => {  // ❌
  console.log(data);
};

// ✅ BIEN - Usar tipos específicos
interface FormData {
  email: string;
  password: string;
}

const handleSubmit = (data: FormData) => {
  console.log(data);
};

// ✅ BIEN - Usar "unknown" cuando realmente no sabes el tipo
const handleError = (error: unknown) => {
  if (error instanceof Error) {
    console.error(error.message);
  }
};
```

### Utility Types

```typescript
// ✅ BIEN - Usar utility types de TypeScript
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Omit password para respuestas
type UserResponse = Omit<User, 'password'>;

// Pick para DTOs
type CreateUserDto = Pick<User, 'name' | 'email' | 'password'>;

// Partial para updates
type UpdateUserDto = Partial<Pick<User, 'name' | 'email'>>;

// Readonly para immutability
type ReadonlyUser = Readonly<User>;
```

---

## 🗂️ State Management

### Zustand Store

```typescript
// ✅ BIEN - Zustand store con tipos y persist
// src/stores/auth.store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import type { User } from '@/types/user.types';

interface AuthState {
  user: User | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  selectedTenantId: string | null;

  // Actions
  setUser: (user: User | null) => void;
  setTokens: (accessToken: string, refreshToken: string) => void;
  setSelectedTenant: (tenantId: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,
      isAuthenticated: false,
      selectedTenantId: null,

      setUser: (user) => set({ user, isAuthenticated: !!user }),

      setTokens: (accessToken, refreshToken) => {
        localStorage.setItem('access_token', accessToken);
        localStorage.setItem('refresh_token', refreshToken);
        set({ accessToken });
      },

      setSelectedTenant: (tenantId) => {
        localStorage.setItem('selected_tenant_id', tenantId);
        set({ selectedTenantId: tenantId });
      },

      logout: () => {
        localStorage.clear();
        set({
          user: null,
          accessToken: null,
          isAuthenticated: false,
          selectedTenantId: null,
        });
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        selectedTenantId: state.selectedTenantId,
      }),
    }
  )
);
```

### Store Usage

```typescript
// ✅ BIEN - Seleccionar solo lo necesario
export const Header = () => {
  const user = useAuthStore((state) => state.user);
  const logout = useAuthStore((state) => state.logout);

  return (
    <header>
      <span>{user?.name}</span>
      <button onClick={logout}>Logout</button>
    </header>
  );
};

// ❌ MAL - Seleccionar todo el store (causa re-renders innecesarios)
export const Header = () => {
  const authStore = useAuthStore();  // ❌

  return (
    <header>
      <span>{authStore.user?.name}</span>
    </header>
  );
};
```

---

## 📋 Forms y Validación

### React Hook Form + Zod

```typescript
// ✅ BIEN - Schema Zod reutilizable
// src/schemas/user.schema.ts
import { z } from 'zod';

export const createUserSchema = z.object({
  name: z.string().min(3, 'Name must be at least 3 characters'),
  email: z.string().email('Invalid email address'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number'),
  role: z.enum(['ADMIN', 'USER', 'GUEST']),
});

export type CreateUserFormData = z.infer<typeof createUserSchema>;

// ✅ BIEN - Usar schema en formulario
// src/components/forms/CreateUserForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { createUserSchema, type CreateUserFormData } from '@/schemas/user.schema';

export const CreateUserForm = ({ onSubmit }: { onSubmit: (data: CreateUserFormData) => void }) => {
  const form = useForm<CreateUserFormData>({
    resolver: zodResolver(createUserSchema),
    defaultValues: {
      name: '',
      email: '',
      password: '',
      role: 'USER',
    },
  });

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <div>
        <label>Name</label>
        <input {...form.register('name')} />
        {form.formState.errors.name && (
          <span>{form.formState.errors.name.message}</span>
        )}
      </div>

      <div>
        <label>Email</label>
        <input {...form.register('email')} type="email" />
        {form.formState.errors.email && (
          <span>{form.formState.errors.email.message}</span>
        )}
      </div>

      <button type="submit" disabled={form.formState.isSubmitting}>
        {form.formState.isSubmitting ? 'Creating...' : 'Create User'}
      </button>
    </form>
  );
};
```

---

## 🎨 Estilos y CSS

### Tailwind CSS

```typescript
// ✅ BIEN - Tailwind classes ordenadas y legibles
<div className="flex flex-col gap-4 rounded-lg border bg-white p-6 shadow-sm">
  <h2 className="text-xl font-semibold">Title</h2>
  <p className="text-gray-600">Description</p>
</div>

// ✅ BIEN - Usar cn() helper para conditional classes
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

// ❌ MAL - Classes inline muy largas
<div className="flex flex-col gap-4 rounded-lg border bg-white p-6 shadow-sm hover:shadow-md transition-shadow duration-200 cursor-pointer active:scale-95">
  {/* ❌ Muy difícil de leer */}
</div>

// ✅ MEJOR - Extraer a constante o componente
const cardClasses = cn(
  'flex flex-col gap-4 rounded-lg border bg-white p-6 shadow-sm',
  'hover:shadow-md transition-shadow duration-200',
  'cursor-pointer active:scale-95'
);

<div className={cardClasses}>
  ...
</div>
```

### CSS Modules (opcional)

```typescript
// ✅ BIEN - CSS Modules para estilos complejos
// UserCard.module.css
.card {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  padding: 1.5rem;
  border-radius: 0.5rem;
  background: white;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.card:hover {
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

// UserCard.tsx
import styles from './UserCard.module.css';

export const UserCard = ({ user }: UserCardProps) => {
  return (
    <div className={styles.card}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
};
```

---

## 🧪 Testing

### Component Testing

```typescript
// ✅ BIEN - Test de componente completo
// src/components/__tests__/UserCard.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserCard } from '../UserCard';
import type { User } from '@/types/user.types';

describe('UserCard', () => {
  const mockUser: User = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    role: 'USER',
    createdAt: new Date('2024-01-01'),
  };

  it('should render user information', () => {
    render(<UserCard user={mockUser} onEdit={vi.fn()} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('should call onEdit when edit button is clicked', async () => {
    const onEdit = vi.fn();
    const user = userEvent.setup();

    render(<UserCard user={mockUser} onEdit={onEdit} />);

    const editButton = screen.getByRole('button', { name: /edit/i });
    await user.click(editButton);

    expect(onEdit).toHaveBeenCalledWith(mockUser);
    expect(onEdit).toHaveBeenCalledTimes(1);
  });

  it('should not render delete button when onDelete is not provided', () => {
    render(<UserCard user={mockUser} onEdit={vi.fn()} />);

    expect(screen.queryByRole('button', { name: /delete/i })).not.toBeInTheDocument();
  });
});
```

### Hook Testing

```typescript
// ✅ BIEN - Test de custom hook
// src/hooks/__tests__/use-pagination.test.ts
import { describe, it, expect } from 'vitest';
import { renderHook, act } from '@testing-library/react';
import { usePagination } from '../use-pagination';

describe('usePagination', () => {
  it('should initialize with correct values', () => {
    const { result } = renderHook(() => usePagination({
      totalItems: 100,
      itemsPerPage: 10,
    }));

    expect(result.current.currentPage).toBe(1);
    expect(result.current.totalPages).toBe(10);
    expect(result.current.startIndex).toBe(0);
    expect(result.current.endIndex).toBe(10);
  });

  it('should go to next page', () => {
    const { result } = renderHook(() => usePagination({
      totalItems: 100,
      itemsPerPage: 10,
    }));

    act(() => {
      result.current.nextPage();
    });

    expect(result.current.currentPage).toBe(2);
    expect(result.current.startIndex).toBe(10);
  });

  it('should not go beyond last page', () => {
    const { result } = renderHook(() => usePagination({
      totalItems: 100,
      itemsPerPage: 10,
    }));

    act(() => {
      result.current.goToPage(15); // Intenta ir más allá
    });

    expect(result.current.currentPage).toBe(10); // Se queda en la última
  });
});
```

---

## 🔧 ESLint y Prettier

### ESLint Configuration

```json
// ✅ BIEN - .eslintrc.cjs
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
    'prettier',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs'],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh', '@typescript-eslint'],
  rules: {
    // React
    'react/react-in-jsx-scope': 'off',
    'react/prop-types': 'off',
    'react-refresh/only-export-components': [
      'warn',
      { allowConstantExport: true },
    ],

    // TypeScript
    '@typescript-eslint/no-explicit-any': 'error',
    '@typescript-eslint/no-unused-vars': ['error', {
      argsIgnorePattern: '^_',
      varsIgnorePattern: '^_',
    }],
    '@typescript-eslint/consistent-type-imports': 'warn',

    // General
    'no-console': ['warn', { allow: ['warn', 'error'] }],
    'prefer-const': 'error',
    'no-var': 'error',
  },
};
```

### Prettier Configuration

```json
// ✅ BIEN - .prettierrc
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

---

## 📝 Git y Commits

### Commit Messages

```bash
# ✅ BIEN - Conventional Commits
feat: add user creation form
fix: resolve login redirect issue
chore: update dependencies
docs: add API documentation
style: format code with prettier
refactor: extract user card component
test: add tests for user service
perf: optimize image loading

# ✅ BIEN - Con scope
feat(auth): implement JWT refresh token
fix(users): correct email validation
test(api): add integration tests for orders

# ❌ MAL - Mensajes vagos
git commit -m "fixes"
git commit -m "update"
git commit -m "wip"
git commit -m "changes"
```

### Branch Naming

```bash
# ✅ BIEN
feature/user-authentication
fix/login-redirect-bug
chore/update-dependencies
refactor/user-card-component

# ❌ MAL
dev
my-branch
test
```

---

## 👀 Code Review

### Checklist de Code Review

**Funcionalidad:**
- [ ] El código hace lo que se supone que debe hacer
- [ ] Se han probado los edge cases
- [ ] No hay bugs obvios

**Calidad:**
- [ ] El código sigue las convenciones del proyecto
- [ ] Los nombres son descriptivos y claros
- [ ] No hay código duplicado
- [ ] No hay código comentado sin razón

**TypeScript:**
- [ ] No se usa `any` (excepto casos justificados)
- [ ] Los tipos están correctamente definidos
- [ ] Se usan utility types cuando corresponde

**Performance:**
- [ ] No hay operaciones costosas en render
- [ ] Se usa `useMemo`/`useCallback` cuando es necesario
- [ ] Las imágenes están optimizadas

**Testing:**
- [ ] Hay tests para la nueva funcionalidad
- [ ] Los tests son claros y mantenibles
- [ ] La cobertura es adecuada

**Seguridad:**
- [ ] No hay datos sensibles en el código
- [ ] La validación de inputs es correcta
- [ ] Los permisos están verificados

---

## 📚 Resumen de Convenciones

### Nomenclatura Rápida

| Elemento | Convención | Ejemplo |
|----------|-----------|---------|
| **Componentes** | PascalCase | `UserCard.tsx` |
| **Hooks** | camelCase + prefijo "use" | `use-users.ts` |
| **Services** | kebab-case + sufijo ".service" | `users.service.ts` |
| **Utils** | kebab-case | `date-utils.ts` |
| **Types** | PascalCase | `User`, `UserFilters` |
| **Interfaces** | PascalCase (sin "I") | `User`, `UserCardProps` |
| **Variables** | camelCase | `userName`, `isAuthenticated` |
| **Constantes** | UPPER_SNAKE_CASE | `MAX_USERS`, `API_URL` |
| **Enums** | PascalCase + UPPER_SNAKE_CASE | `UserRole.ADMIN` |
| **Carpetas** | kebab-case | `components/`, `hooks/` |

### Orden de Imports

1. React y librerías externas
2. Componentes internos
3. Hooks
4. Services y utils
5. Types
6. Styles

### Reglas de Oro

1. **NUNCA usar `any`** - Usar tipos específicos o `unknown`
2. **SIEMPRE tipar props** - Interfaces para componentes
3. **SIEMPRE cleanup en useEffect** - Evitar memory leaks
4. **SIEMPRE validar permisos** - Frontend y backend
5. **NUNCA inline functions en JSX** - Declarar fuera
6. **SIEMPRE usar keys en listas** - Unique y stable
7. **NUNCA console.log en producción** - Usar logger
8. **SIEMPRE handle loading y error states** - UX

---

<div align="center">

**Código Limpio, Type-Safe y Mantenible**

Consistencia | Calidad | Performance

</div>
