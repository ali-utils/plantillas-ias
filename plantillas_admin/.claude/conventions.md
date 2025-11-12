# 📏 Conventions - Admin Web Frontend

> **Convenciones obligatorias para mantener consistencia en el código**

---

## 🚨 REGLAS CRÍTICAS (NUNCA IGNORAR)

### 1. TypeScript Strict Mode

```typescript
// ❌ PROHIBIDO - Usar "any"
const handleSubmit = (data: any) => { };  // ❌ NUNCA

// ✅ OBLIGATORIO - Tipos específicos
interface FormData {
  email: string;
  password: string;
}
const handleSubmit = (data: FormData) => { };  // ✅ SIEMPRE
```

### 2. Props Typing

```typescript
// ❌ PROHIBIDO - Props sin tipos
export const UserCard = ({ user, onEdit }) => { };  // ❌

// ✅ OBLIGATORIO - Interface Props
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}
export const UserCard = ({ user, onEdit }: UserCardProps) => { };  // ✅
```

### 3. Multi-Tenant Header

```typescript
// ❌ PROHIBIDO - Request sin X-Tenant-ID
axios.get('/api/users');  // ❌ Falta tenant header

// ✅ OBLIGATORIO - Interceptor configurado
apiClient.interceptors.request.use((config) => {
  const tenantId = localStorage.getItem('selected_tenant_id');
  if (tenantId && config.headers) {
    config.headers['X-Tenant-ID'] = tenantId;  // ✅ SIEMPRE
  }
  return config;
});
```

### 4. Validación con Zod

```typescript
// ❌ PROHIBIDO - Validación manual
if (email.includes('@')) { }  // ❌ No confiable

// ✅ OBLIGATORIO - Schema Zod
const schema = z.object({
  email: z.string().email('Email inválido'),
  password: z.string().min(8),
});
```

### 5. Error Handling

```typescript
// ❌ PROHIBIDO - No manejar errores
const { data } = useUsers();  // ❌ ¿Qué pasa si falla?

// ✅ OBLIGATORIO - Loading + Error states
const { data, isLoading, error } = useUsers();

if (isLoading) return <Skeleton />;
if (error) return <ErrorMessage error={error} />;
if (!data) return <EmptyState />;

return <UsersList users={data} />;  // ✅
```

---

## 📁 Nomenclatura de Archivos

### Componentes React

```bash
# ✅ CORRECTO - PascalCase
UserCard.tsx
DashboardLayout.tsx
CreateProductModal.tsx
LoginPage.tsx

# ❌ INCORRECTO
userCard.tsx          # ❌ camelCase
user_card.tsx         # ❌ snake_case
user-card.tsx         # ❌ kebab-case
```

### Custom Hooks

```bash
# ✅ CORRECTO - kebab-case con prefijo "use"
use-auth.ts
use-users.ts
use-pagination.ts
use-debounce.ts

# ❌ INCORRECTO
useAuth.ts            # ❌ camelCase en archivo
use_auth.ts           # ❌ snake_case
auth.hook.ts          # ❌ Sufijo incorrecto
```

### Services

```bash
# ✅ CORRECTO - kebab-case + sufijo ".service"
users.service.ts
auth.service.ts
products.service.ts
api-client.ts         # Excepción: archivo config

# ❌ INCORRECTO
usersService.ts       # ❌ camelCase
users_service.ts      # ❌ snake_case
users.ts              # ❌ Sin sufijo .service
```

### Utilities

```bash
# ✅ CORRECTO - kebab-case + sufijo "-utils"
date-utils.ts
string-utils.ts
validators.ts         # OK sin sufijo si es obvio

# ❌ INCORRECTO
dateUtils.ts          # ❌ camelCase
date_utils.ts         # ❌ snake_case
```

### Types

```bash
# ✅ CORRECTO - kebab-case + sufijo ".types"
user.types.ts
product.types.ts
api.types.ts

# ❌ INCORRECTO
userTypes.ts          # ❌ camelCase
user_types.ts         # ❌ snake_case
types.ts              # ❌ Demasiado genérico
```

### Tests

```bash
# ✅ CORRECTO - Mismo nombre + ".test"
UserCard.test.tsx
use-pagination.test.ts
date-utils.test.ts

# También OK - ".spec"
UserCard.spec.tsx

# ❌ INCORRECTO
UserCard.tests.tsx    # ❌ Plural
test-UserCard.tsx     # ❌ Prefijo incorrecto
```

### Carpetas

```bash
# ✅ CORRECTO - kebab-case, singular o plural según contexto
components/
hooks/
services/
api-client/

# ❌ INCORRECTO
Components/           # ❌ PascalCase
my_hooks/             # ❌ snake_case
```

---

## 🏷️ Nomenclatura de Código

### Componentes

```typescript
// ✅ CORRECTO - PascalCase
export const UserCard = () => { };
export const DashboardLayout = () => { };
export const CreateProductModal = () => { };

// ❌ INCORRECTO
export const userCard = () => { };        // ❌ camelCase
export const user_card = () => { };       // ❌ snake_case
export default function UserCard() { };   // ⚠️ Evitar default export inline
```

### Variables y Funciones

```typescript
// ✅ CORRECTO - camelCase
const userName = 'John';
const isAuthenticated = true;
const currentTenant = getTenant();

const handleSubmit = () => { };
const fetchUsers = async () => { };

// ❌ INCORRECTO
const UserName = 'John';              // ❌ PascalCase
const user_name = 'John';             // ❌ snake_case
const is_authenticated = true;        // ❌ snake_case
```

### Constantes

```typescript
// ✅ CORRECTO - UPPER_SNAKE_CASE para constantes de configuración
const MAX_USERS_PER_PAGE = 10;
const API_BASE_URL = 'https://api.bmoi.com';
const DEFAULT_TIMEOUT = 5000;
const JWT_STORAGE_KEY = 'access_token';

// ✅ CORRECTO - camelCase para objetos inmutables
const colors = {
  primary: '#007bff',
  secondary: '#6c757d',
} as const;

const routes = {
  home: '/',
  login: '/login',
  dashboard: '/dashboard',
} as const;

// ❌ INCORRECTO
const maxUsersPerPage = 10;           // ❌ camelCase para config
const ApiBaseUrl = 'https://...';     // ❌ PascalCase
const api_base_url = 'https://...';   // ❌ snake_case
```

### Types e Interfaces

```typescript
// ✅ CORRECTO - PascalCase, SIN prefijo "I"
interface User {
  id: string;
  name: string;
}

type UserRole = 'ADMIN' | 'USER' | 'GUEST';
type UserFilters = Pick<User, 'name' | 'email'>;

// ✅ CORRECTO - Sufijo "Props" para props de componentes
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
}

// ✅ CORRECTO - Sufijo "Dto" para Data Transfer Objects
interface CreateUserDto {
  name: string;
  email: string;
  password: string;
}

// ❌ INCORRECTO
interface IUser { }                   // ❌ Prefijo "I" (estilo C#)
interface userProps { }               // ❌ camelCase
interface user_props { }              // ❌ snake_case
type user = { };                      // ❌ minúscula
```

### Enums (o const objects)

```typescript
// ✅ MEJOR - Const object con "as const" (preferido sobre enums)
const UserRole = {
  ADMIN: 'ADMIN',
  USER: 'USER',
  GUEST: 'GUEST',
} as const;

type UserRole = typeof UserRole[keyof typeof UserRole];

// ✅ TAMBIÉN OK - Enum tradicional
enum OrderStatus {
  PENDING = 'PENDING',
  CONFIRMED = 'CONFIRMED',
  SHIPPED = 'SHIPPED',
  DELIVERED = 'DELIVERED',
}

// ❌ INCORRECTO
const userRole = {                    // ❌ camelCase
  admin: 'admin',                     // ❌ valores en minúscula
};

enum order_status { }                 // ❌ snake_case
```

### Hooks

```typescript
// ✅ CORRECTO - Prefijo "use" + camelCase
export const useAuth = () => { };
export const useUsers = () => { };
export const usePagination = () => { };
export const useDebounce = () => { };

// ❌ INCORRECTO
export const auth = () => { };            // ❌ Sin prefijo "use"
export const getUsers = () => { };        // ❌ Prefijo incorrecto
export const use_pagination = () => { };  // ❌ snake_case
export const UsePagination = () => { };   // ❌ PascalCase
```

---

## 📂 Estructura de Componente

### Componente Simple (Single File)

```typescript
// ✅ CORRECTO - UserCard.tsx
import React from 'react';
import { Card, CardHeader, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import type { User } from '@/types/user.types';

interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
  onDelete?: (userId: string) => void;
}

export const UserCard = ({ user, onEdit, onDelete }: UserCardProps) => {
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
        <h3>{user.name}</h3>
      </CardHeader>
      <CardContent>
        <p>{user.email}</p>
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
```

### Componente Complejo (Carpeta con múltiples archivos)

```
DataTable/
├── index.tsx              # Componente principal
├── DataTable.test.tsx     # Tests
├── DataTableHeader.tsx    # Subcomponente
├── DataTableBody.tsx      # Subcomponente
├── DataTableRow.tsx       # Subcomponente
├── types.ts               # Types específicos
└── utils.ts               # Utils específicos
```

```typescript
// ✅ CORRECTO - DataTable/index.tsx
import React from 'react';
import { DataTableHeader } from './DataTableHeader';
import { DataTableBody } from './DataTableBody';
import type { DataTableProps } from './types';

export const DataTable = <T,>({ data, columns }: DataTableProps<T>) => {
  return (
    <table className="w-full">
      <DataTableHeader columns={columns} />
      <DataTableBody data={data} columns={columns} />
    </table>
  );
};

// Re-export subcomponents if needed
export { DataTableHeader } from './DataTableHeader';
export { DataTableBody } from './DataTableBody';
export type * from './types';
```

---

## 📦 Orden de Imports

### Template Obligatorio

```typescript
// 1️⃣ React y librerías de React (PRIMERO)
import React, { useState, useEffect, useMemo } from 'react';
import { useNavigate, useParams } from 'react-router-dom';

// 2️⃣ Librerías externas (terceros)
import { useQuery, useMutation } from '@tanstack/react-query';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// 3️⃣ Componentes UI internos
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card, CardHeader, CardContent } from '@/components/ui/card';

// 4️⃣ Componentes de negocio internos
import { UserCard } from '@/components/organisms/UserCard';
import { DashboardLayout } from '@/layouts/DashboardLayout';

// 5️⃣ Hooks personalizados
import { useAuth } from '@/hooks/use-auth';
import { useUsers } from '@/hooks/queries/use-users';
import { usePagination } from '@/hooks/use-pagination';

// 6️⃣ Services y API
import { usersService } from '@/services/users.service';
import { apiClient } from '@/services/api-client';

// 7️⃣ Stores
import { useAuthStore } from '@/stores/auth.store';
import { useUIStore } from '@/stores/ui.store';

// 8️⃣ Utils y helpers
import { formatDate } from '@/lib/date-utils';
import { cn } from '@/lib/utils';

// 9️⃣ Types e interfaces
import type { User, UserFilters } from '@/types/user.types';
import type { ApiResponse } from '@/types/api.types';

// 🔟 Constants
import { MAX_USERS_PER_PAGE, USER_ROLES } from '@/constants/users';

// 1️⃣1️⃣ Styles (ÚLTIMO)
import styles from './UsersPage.module.css';
```

### Reglas de Imports

```typescript
// ✅ CORRECTO - Named imports
import { useState, useEffect } from 'react';
import { Button, Input } from '@/components/ui';

// ✅ CORRECTO - Type imports separados (TypeScript 4.5+)
import type { User } from '@/types/user.types';
import type { FC, ReactNode } from 'react';

// ✅ CORRECTO - Alias path (@/)
import { Button } from '@/components/ui/button';
import { useAuth } from '@/hooks/use-auth';

// ❌ INCORRECTO - Paths relativos largos
import { Button } from '../../../components/ui/button';  // ❌
import { useAuth } from '../../hooks/use-auth';          // ❌

// ❌ INCORRECTO - Mezclar named y default imports sin orden
import React from 'react';
import Button from '@/components/ui/button';   // ❌ Inconsistente
import { Input } from '@/components/ui/input'; // ✅
```

---

## 🎨 Estructura de Componente React

### Template de Componente Completo

```typescript
// UserCard.tsx
import React, { useState, useCallback, useMemo } from 'react';
import { Card, CardHeader, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { formatDate } from '@/lib/date-utils';
import type { User } from '@/types/user.types';

// 1️⃣ Props interface (SIEMPRE al inicio)
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
  onDelete?: (userId: string) => void;
  showActions?: boolean;
}

// 2️⃣ Componente principal
export const UserCard = ({
  user,
  onEdit,
  onDelete,
  showActions = true,
}: UserCardProps) => {
  // 3️⃣ Hooks de estado (useState, useReducer)
  const [isExpanded, setIsExpanded] = useState(false);

  // 4️⃣ Hooks de contexto y stores
  const { hasPermission } = usePermissions();

  // 5️⃣ useMemo (cálculos derivados)
  const formattedDate = useMemo(
    () => formatDate(user.createdAt),
    [user.createdAt]
  );

  // 6️⃣ useCallback (funciones memorizadas)
  const handleEdit = useCallback(() => {
    onEdit(user);
  }, [user, onEdit]);

  const handleDelete = useCallback(() => {
    if (onDelete && window.confirm('Are you sure?')) {
      onDelete(user.id);
    }
  }, [user.id, onDelete]);

  // 7️⃣ useEffect (side effects)
  useEffect(() => {
    console.log('User card mounted:', user.id);

    return () => {
      console.log('User card unmounted:', user.id);
    };
  }, [user.id]);

  // 8️⃣ Conditional render (early returns)
  if (!user) {
    return null;
  }

  // 9️⃣ JSX principal
  return (
    <Card>
      <CardHeader>
        <h3 className="text-xl font-semibold">{user.name}</h3>
      </CardHeader>
      <CardContent>
        <p className="text-gray-600">{user.email}</p>
        <p className="text-sm text-gray-400">
          Created: {formattedDate}
        </p>

        {showActions && hasPermission('users.edit') && (
          <div className="mt-4 flex gap-2">
            <Button onClick={handleEdit}>Edit</Button>
            {onDelete && hasPermission('users.delete') && (
              <Button variant="destructive" onClick={handleDelete}>
                Delete
              </Button>
            )}
          </div>
        )}
      </CardContent>
    </Card>
  );
};
```

---

## 🪝 Custom Hooks

### Template de Hook Completo

```typescript
// use-pagination.ts
import { useState, useCallback, useMemo } from 'react';

// 1️⃣ Types del hook
interface UsePaginationOptions {
  totalItems: number;
  itemsPerPage?: number;
  initialPage?: number;
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

// 2️⃣ Hook principal (prefijo "use" OBLIGATORIO)
export const usePagination = ({
  totalItems,
  itemsPerPage = 10,
  initialPage = 1,
}: UsePaginationOptions): UsePaginationReturn => {
  // 3️⃣ State
  const [currentPage, setCurrentPage] = useState(initialPage);

  // 4️⃣ Valores derivados con useMemo
  const totalPages = useMemo(
    () => Math.ceil(totalItems / itemsPerPage),
    [totalItems, itemsPerPage]
  );

  const startIndex = useMemo(
    () => (currentPage - 1) * itemsPerPage,
    [currentPage, itemsPerPage]
  );

  const endIndex = useMemo(
    () => Math.min(startIndex + itemsPerPage, totalItems),
    [startIndex, itemsPerPage, totalItems]
  );

  // 5️⃣ Funciones con useCallback
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

  // 6️⃣ Return del hook
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
```

---

## 🌐 Services

### Template de Service

```typescript
// users.service.ts
import { apiClient } from './api-client';
import type {
  User,
  CreateUserDto,
  UpdateUserDto,
  UserFilters,
  PaginatedResponse,
} from '@/types/user.types';

// 1️⃣ Service como objeto con métodos (NO clase)
export const usersService = {
  /**
   * Get all users with optional filters
   */
  async getAll(filters?: UserFilters): Promise<User[]> {
    const response = await apiClient.get<User[]>('/users', {
      params: filters,
    });
    return response.data;
  },

  /**
   * Get paginated users
   */
  async getPaginated(
    page: number = 1,
    limit: number = 10,
    filters?: UserFilters
  ): Promise<PaginatedResponse<User>> {
    const response = await apiClient.get<PaginatedResponse<User>>('/users', {
      params: {
        page,
        limit,
        ...filters,
      },
    });
    return response.data;
  },

  /**
   * Get user by ID
   */
  async getById(id: string): Promise<User> {
    const response = await apiClient.get<User>(`/users/${id}`);
    return response.data;
  },

  /**
   * Create new user
   */
  async create(data: CreateUserDto): Promise<User> {
    const response = await apiClient.post<User>('/users', data);
    return response.data;
  },

  /**
   * Update user
   */
  async update(id: string, data: UpdateUserDto): Promise<User> {
    const response = await apiClient.patch<User>(`/users/${id}`, data);
    return response.data;
  },

  /**
   * Delete user
   */
  async delete(id: string): Promise<void> {
    await apiClient.delete(`/users/${id}`);
  },

  /**
   * Bulk delete users
   */
  async bulkDelete(ids: string[]): Promise<void> {
    await apiClient.post('/users/bulk-delete', { ids });
  },
};
```

---

## 📝 Comentarios

### Cuándo comentar

```typescript
// ✅ CORRECTO - Comentar lógica compleja
// Calculate the compound interest using the formula: A = P(1 + r/n)^(nt)
const finalAmount = principal * Math.pow(1 + rate / periods, periods * time);

// ✅ CORRECTO - Explicar "por qué" no "qué"
// We need to debounce to avoid hitting rate limits
const debouncedSearch = useDebounce(searchTerm, 500);

// ✅ CORRECTO - Warnings y TODOs
// TODO: Implement pagination when backend supports it
// FIXME: This causes memory leak, needs cleanup
// HACK: Temporary workaround until API v2 is ready

// ❌ INCORRECTO - Comentar lo obvio
// Increment counter
setCounter(counter + 1);  // ❌ Obvio

// Set user name
setUserName(name);  // ❌ Obvio
```

### JSDoc para funciones públicas

```typescript
// ✅ CORRECTO - JSDoc para hooks y utils públicos
/**
 * Debounces a value by a specified delay
 *
 * @param value - The value to debounce
 * @param delay - Delay in milliseconds (default: 500)
 * @returns The debounced value
 *
 * @example
 * ```tsx
 * const debouncedSearch = useDebounce(searchTerm, 300);
 * ```
 */
export const useDebounce = <T>(value: T, delay: number = 500): T => {
  // ...
};
```

---

## ✅ Checklist de Convenciones

### Antes de cada commit:

- [ ] **Nomenclatura:**
  - [ ] Componentes en PascalCase
  - [ ] Hooks con prefijo "use" en camelCase
  - [ ] Services en kebab-case + ".service"
  - [ ] Variables en camelCase
  - [ ] Constantes en UPPER_SNAKE_CASE

- [ ] **TypeScript:**
  - [ ] Sin `any` (usar tipos específicos)
  - [ ] Props tipadas con interface
  - [ ] Return types explícitos en funciones

- [ ] **Imports:**
  - [ ] Orden correcto (React → externos → internos → types → styles)
  - [ ] Usando alias `@/` (no paths relativos)
  - [ ] Type imports con `import type`

- [ ] **Componentes:**
  - [ ] Props interface definida
  - [ ] Loading states manejados
  - [ ] Error states manejados
  - [ ] Keys en listas

- [ ] **Multi-tenant:**
  - [ ] X-Tenant-ID en requests
  - [ ] Verificación de tenant en páginas

- [ ] **Performance:**
  - [ ] No inline functions en JSX
  - [ ] useMemo para cálculos costosos
  - [ ] useCallback para funciones en deps

- [ ] **Testing:**
  - [ ] Tests escritos para nueva funcionalidad
  - [ ] Tests pasando

- [ ] **Limpieza:**
  - [ ] Sin console.log
  - [ ] Sin código comentado
  - [ ] Sin imports no usados

---

<div align="center">

**Convenciones Estrictas = Código Consistente**

Estas reglas NO son opcionales

</div>
