# 🏗️ Arquitectura Frontend - Admin Web

> **Decisiones arquitectónicas y patrones de diseño para el panel administrativo de bMOI**

---

## 📋 Índice

1. [Visión General](#visión-general)
2. [Stack Tecnológico](#stack-tecnológico)
3. [Decisiones Arquitectónicas](#decisiones-arquitectónicas)
4. [Patrones de Diseño](#patrones-de-diseño)
5. [Estructura de Carpetas](#estructura-de-carpetas)
6. [Flujo de Datos](#flujo-de-datos)
7. [Gestión de Estado](#gestión-de-estado)
8. [Comunicación con Backend](#comunicación-con-backend)
9. [Autenticación y Seguridad](#autenticación-y-seguridad)
10. [Performance y Optimización](#performance-y-optimización)
11. [Testing Strategy](#testing-strategy)
12. [Deployment](#deployment)

---

## 🎯 Visión General

### Propósito

El **Admin Web** es el panel de administración para el ecosistema bMOI, permitiendo a los tenants gestionar:
- Usuarios y permisos (RBAC)
- Productos, categorías y órdenes
- Dashboard con métricas y reportes
- Configuración del tenant
- Auditoría y logs

### Características Clave

- ✅ **Multi-tenant**: Aislamiento completo por tenant
- ✅ **SPA (Single Page Application)**: Navegación fluida sin recargas
- ✅ **Type-Safe**: TypeScript end-to-end con validaciones
- ✅ **Responsive**: Mobile-first design con Tailwind CSS
- ✅ **Performante**: Code splitting, lazy loading, caching
- ✅ **Accesible**: WCAG 2.1 AA compliance
- ✅ **Testeable**: Cobertura > 80%
- ✅ **Deployable**: Docker + Nginx optimizado

---

## 🛠️ Stack Tecnológico

### Core Framework

#### React 18 ✅

**Decisión:** Usar **React 18** con Concurrent Features

**Por qué:**
- ✅ **Más popular y maduro**: Mayor comunidad, más recursos
- ✅ **Concurrent Rendering**: Mejor UX con transiciones suaves
- ✅ **Suspense for Data Fetching**: Loading states elegantes
- ✅ **Server Components (futuro)**: Preparado para SSR/RSC
- ✅ **Ecosistema rico**: Miles de librerías compatibles

**Alternativas descartadas:**
- ❌ **Vue 3**: Menos adopción corporativa, menos TypeScript-first
- ❌ **Angular**: Demasiado opinionado, curva de aprendizaje alta
- ❌ **Svelte**: Ecosistema más pequeño, menos empresarial

**Trade-offs aceptados:**
- Slightly más bundle size que Svelte
- Requiere librerías externas para routing, state, forms

---

### Build Tool

#### Vite 5 ✅

**Decisión:** Usar **Vite** (no Webpack, no Create React App)

**Por qué:**
- ✅ **Dev server ultra-rápido**: ESM nativo, HMR instantáneo
- ✅ **Build optimizado**: Rollup bajo el capó
- ✅ **TypeScript out-of-the-box**: Sin configuración extra
- ✅ **CSS Modules nativo**: Sin setup adicional
- ✅ **Plugins simples**: API clara y documentada

**Comparación:**

| Feature | Vite | Webpack | Create React App |
|---------|------|---------|------------------|
| Dev Start | < 1s | ~10s | ~15s |
| HMR | < 50ms | ~500ms | ~1s |
| Build Time | ~30s | ~2min | ~3min |
| Config | Simple | Complejo | Sin config |
| TS Support | Nativo | Plugin | Incluido |

**Configuración clave:**

```typescript
// vite.config.ts
export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': ['@radix-ui/react-dialog', '@radix-ui/react-dropdown-menu'],
          'form-vendor': ['react-hook-form', 'zod'],
          'query-vendor': ['@tanstack/react-query', 'axios'],
        },
      },
    },
    chunkSizeWarningLimit: 1000,
  },
});
```

---

### Routing

#### React Router 6 ✅

**Decisión:** Usar **React Router 6** (con Data APIs)

**Por qué:**
- ✅ **Loaders y Actions**: Data fetching declarativo
- ✅ **Suspense support**: Loading states automáticos
- ✅ **Nested Routes**: Layouts jerárquicos naturales
- ✅ **Type-safe**: Tipos inferidos con TypeScript

**Estructura de rutas:**

```typescript
// src/router/index.tsx
export const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    errorElement: <ErrorBoundary />,
    children: [
      {
        path: 'login',
        element: <LoginPage />,
      },
      {
        path: 'dashboard',
        element: <DashboardLayout />,
        loader: protectedLoader,
        children: [
          { index: true, element: <DashboardPage /> },
          { path: 'users', element: <UsersPage />, loader: usersLoader },
          { path: 'products', element: <ProductsPage />, loader: productsLoader },
          { path: 'orders', element: <OrdersPage />, loader: ordersLoader },
        ],
      },
    ],
  },
]);
```

**Protección de rutas:**

```typescript
// src/router/loaders.ts
export const protectedLoader: LoaderFunction = async () => {
  const { isAuthenticated } = useAuthStore.getState();

  if (!isAuthenticated) {
    return redirect('/login');
  }

  return null;
};
```

---

### State Management

#### Zustand 4 ✅

**Decisión:** Usar **Zustand** (no Redux, no Context API pura)

**Por qué:**
- ✅ **Minimalista**: 1KB gzipped vs 3KB Redux
- ✅ **Sin boilerplate**: Sin actions, reducers, dispatch
- ✅ **Type-safe**: TypeScript first-class
- ✅ **DevTools**: Redux DevTools compatible
- ✅ **Persistence**: Built-in con `persist` middleware

**Comparación:**

| Feature | Zustand | Redux | Context API |
|---------|---------|-------|-------------|
| Bundle Size | 1KB | 3KB | 0KB (nativo) |
| Boilerplate | Mínimo | Alto | Medio |
| TypeScript | Excelente | Bueno | Regular |
| DevTools | ✅ | ✅ | ❌ |
| Persistence | ✅ Built-in | Plugin | Manual |
| Performance | ✅ Selector-based | ✅ | ⚠️ Re-renders |

**Cuándo usar cada store:**

1. **Auth Store (Zustand)**: Estado global persistente
2. **UI Store (Zustand)**: Sidebar, theme, modals
3. **Server State (React Query)**: Datos del backend
4. **Component State (useState)**: Estado local UI

**Ejemplo Auth Store:**

```typescript
// src/stores/auth.store.ts
interface AuthState {
  user: User | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  selectedTenantId: string | null;
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
          selectedTenantId: null
        });
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        selectedTenantId: state.selectedTenantId
      }),
    }
  )
);
```

---

### Server State

#### React Query (TanStack Query) 5 ✅

**Decisión:** Usar **React Query** para server state

**Por qué:**
- ✅ **Caching automático**: Menos requests al backend
- ✅ **Background refetch**: Datos siempre frescos
- ✅ **Optimistic updates**: UX instantáneo
- ✅ **Error/Retry logic**: Manejo robusto de errores
- ✅ **DevTools**: Visualización del cache

**Patterns implementados:**

1. **Query Keys estandarizados:**

```typescript
// src/lib/query-keys.ts
export const queryKeys = {
  users: {
    all: ['users'] as const,
    lists: () => [...queryKeys.users.all, 'list'] as const,
    list: (filters: string) => [...queryKeys.users.lists(), { filters }] as const,
    details: () => [...queryKeys.users.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.users.details(), id] as const,
  },
  products: {
    all: ['products'] as const,
    lists: () => [...queryKeys.products.all, 'list'] as const,
    list: (filters: string) => [...queryKeys.products.lists(), { filters }] as const,
    details: () => [...queryKeys.products.all, 'detail'] as const,
    detail: (id: string) => [...queryKeys.products.details(), id] as const,
  },
};
```

2. **Custom Hooks para queries:**

```typescript
// src/hooks/queries/use-users.ts
export const useUsers = (filters?: UserFilters) => {
  return useQuery({
    queryKey: queryKeys.users.list(JSON.stringify(filters)),
    queryFn: () => usersApi.getAll(filters),
    staleTime: 5 * 60 * 1000, // 5 minutos
    gcTime: 10 * 60 * 1000, // 10 minutos
  });
};

export const useUser = (id: string) => {
  return useQuery({
    queryKey: queryKeys.users.detail(id),
    queryFn: () => usersApi.getById(id),
    enabled: !!id,
  });
};
```

3. **Mutations con optimistic updates:**

```typescript
// src/hooks/mutations/use-create-user.ts
export const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: usersApi.create,
    onMutate: async (newUser) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey: queryKeys.users.lists() });

      // Snapshot previous value
      const previousUsers = queryClient.getQueryData(queryKeys.users.lists());

      // Optimistically update
      queryClient.setQueryData(queryKeys.users.lists(), (old: User[]) => [
        ...old,
        { id: 'temp-id', ...newUser },
      ]);

      return { previousUsers };
    },
    onError: (err, newUser, context) => {
      // Rollback on error
      queryClient.setQueryData(queryKeys.users.lists(), context?.previousUsers);
    },
    onSuccess: () => {
      // Invalidate to refetch real data
      queryClient.invalidateQueries({ queryKey: queryKeys.users.lists() });
    },
  });
};
```

---

### Forms & Validation

#### React Hook Form + Zod ✅

**Decisión:** **React Hook Form** + **Zod** (no Formik)

**Por qué:**
- ✅ **Performance**: Uncontrolled forms, menos re-renders
- ✅ **Type-safe**: Zod schemas generan tipos TypeScript
- ✅ **Schema validation**: Un solo schema para frontend y backend
- ✅ **Built-in errors**: Manejo de errores integrado
- ✅ **Bundle size**: 8KB vs 15KB Formik

**Comparación:**

| Feature | RHF + Zod | Formik + Yup |
|---------|-----------|--------------|
| Bundle Size | 8KB + 12KB | 15KB + 18KB |
| Re-renders | Mínimos | Muchos |
| TypeScript | Excelente | Bueno |
| Performance | ⚡ Rápido | ⚠️ Medio |
| API | Hooks | HOC/Render Props |

**Pattern reusable:**

```typescript
// src/lib/form-utils.ts
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';
import { z } from 'zod';

export const createForm = <T extends z.ZodType>(schema: T) => {
  type FormValues = z.infer<T>;

  return {
    useFormWithSchema: (defaultValues?: Partial<FormValues>) => {
      return useForm<FormValues>({
        resolver: zodResolver(schema),
        defaultValues: defaultValues as any,
      });
    },
    schema,
  };
};

// Uso:
const { schema, useFormWithSchema } = createForm(
  z.object({
    email: z.string().email('Email inválido'),
    password: z.string().min(8, 'Mínimo 8 caracteres'),
  })
);

const MyComponent = () => {
  const form = useFormWithSchema();
  // ...
};
```

---

### UI Components

#### Shadcn/UI + Tailwind CSS ✅

**Decisión:** **Shadcn/UI** (no MUI, no Ant Design)

**Por qué:**
- ✅ **Copy-paste approach**: Código en tu proyecto, no node_modules
- ✅ **Totalmente customizable**: Modificas el código directamente
- ✅ **Tailwind-first**: No CSS-in-JS runtime overhead
- ✅ **Accessible**: Radix UI bajo el capó (WCAG AA)
- ✅ **Type-safe**: TypeScript completo

**Comparación:**

| Feature | Shadcn/UI | MUI | Ant Design |
|---------|-----------|-----|------------|
| Approach | Copy-paste | npm package | npm package |
| Customization | 🟢 Total | 🟡 Theme | 🟡 Theme |
| Bundle Size | 🟢 Solo lo que usas | 🔴 ~300KB | 🔴 ~500KB |
| TypeScript | ✅ | ✅ | ⚠️ |
| Accessibility | ✅ Radix | ✅ | ⚠️ |
| Learning Curve | Bajo | Medio | Medio |

**Componentes base incluidos:**

```bash
npx shadcn-ui@latest init
npx shadcn-ui@latest add button
npx shadcn-ui@latest add input
npx shadcn-ui@latest add form
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add table
npx shadcn-ui@latest add card
npx shadcn-ui@latest add tabs
npx shadcn-ui@latest add toast
npx shadcn-ui@latest add select
npx shadcn-ui@latest add checkbox
npx shadcn-ui@latest add radio-group
npx shadcn-ui@latest add switch
npx shadcn-ui@latest add label
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add avatar
npx shadcn-ui@latest add skeleton
```

---

## 🏛️ Decisiones Arquitectónicas

### 1. Component-Based Architecture

**Pattern:** Atomic Design (modificado)

```
atoms/       → Button, Input, Label, Badge
molecules/   → FormField, SearchBar, UserAvatar
organisms/   → Sidebar, Header, DataTable, UserForm
templates/   → DashboardLayout, AuthLayout
pages/       → DashboardPage, UsersPage
```

**Reglas:**
- Atoms: No lógica de negocio, solo presentación
- Molecules: Agrupación de atoms, lógica simple
- Organisms: Lógica compleja, conectados a stores/queries
- Templates: Layouts reutilizables
- Pages: Enrutables, componen templates y organisms

### 2. Feature-Based Structure (Opcional para apps grandes)

Si la app crece mucho (>50 componentes), migrar a:

```
src/
├── features/
│   ├── users/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types/
│   │   └── index.ts
│   ├── products/
│   └── orders/
└── shared/
    ├── components/
    ├── hooks/
    └── utils/
```

### 3. Separation of Concerns

**Separar claramente:**
1. **Presentational Components**: Solo UI, reciben props
2. **Container Components**: Conectan a store/queries
3. **Custom Hooks**: Lógica reutilizable
4. **Services**: API calls, lógica de negocio
5. **Utils**: Funciones puras, helpers

**Ejemplo:**

```typescript
// ❌ MAL - Todo mezclado
const UsersPage = () => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    axios.get('/api/users').then(res => setUsers(res.data));
  }, []);

  return (
    <div>
      {users.map(user => (
        <div key={user.id}>
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
};

// ✅ BIEN - Separado
// Presentational
const UserCard = ({ user }: { user: User }) => (
  <Card>
    <CardHeader>
      <CardTitle>{user.name}</CardTitle>
    </CardHeader>
    <CardContent>
      <p>{user.email}</p>
    </CardContent>
  </Card>
);

// Custom Hook
const useUsers = () => {
  return useQuery({
    queryKey: queryKeys.users.all,
    queryFn: usersApi.getAll,
  });
};

// Container
const UsersPage = () => {
  const { data: users, isLoading } = useUsers();

  if (isLoading) return <Skeleton />;

  return (
    <div className="grid grid-cols-3 gap-4">
      {users?.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
};
```

---

## 🎨 Patrones de Diseño

### 1. Compound Components Pattern

**Para componentes complejos con subcomponentes:**

```typescript
// src/components/organisms/DataTable/index.tsx
interface DataTableProps<T> {
  data: T[];
  children: React.ReactNode;
}

interface DataTableContextValue<T> {
  data: T[];
}

const DataTableContext = createContext<DataTableContextValue<any> | null>(null);

export const DataTable = <T,>({ data, children }: DataTableProps<T>) => {
  return (
    <DataTableContext.Provider value={{ data }}>
      <table className="w-full">{children}</table>
    </DataTableContext.Provider>
  );
};

DataTable.Header = ({ children }: { children: React.ReactNode }) => (
  <thead className="bg-gray-50">{children}</thead>
);

DataTable.Body = ({ children }: { children: React.ReactNode }) => (
  <tbody>{children}</tbody>
);

DataTable.Row = ({ children }: { children: React.ReactNode }) => (
  <tr className="border-b">{children}</tr>
);

// Uso:
<DataTable data={users}>
  <DataTable.Header>
    <DataTable.Row>
      <th>Nombre</th>
      <th>Email</th>
    </DataTable.Row>
  </DataTable.Header>
  <DataTable.Body>
    {users.map(user => (
      <DataTable.Row key={user.id}>
        <td>{user.name}</td>
        <td>{user.email}</td>
      </DataTable.Row>
    ))}
  </DataTable.Body>
</DataTable>
```

### 2. Render Props Pattern

**Para lógica reutilizable con UI flexible:**

```typescript
interface PermissionGateProps {
  permission: string;
  children: (hasPermission: boolean) => React.ReactNode;
  fallback?: React.ReactNode;
}

export const PermissionGate = ({ permission, children, fallback }: PermissionGateProps) => {
  const { hasPermission } = usePermissions();
  const allowed = hasPermission(permission);

  if (!allowed && fallback) return <>{fallback}</>;

  return <>{children(allowed)}</>;
};

// Uso:
<PermissionGate permission="users.delete">
  {(hasPermission) => (
    <Button
      variant="destructive"
      disabled={!hasPermission}
    >
      Eliminar Usuario
    </Button>
  )}
</PermissionGate>
```

### 3. Higher-Order Components (HOC)

**Para autenticación y autorización:**

```typescript
// src/hoc/with-auth.tsx
export const withAuth = <P extends object>(
  Component: React.ComponentType<P>
) => {
  return (props: P) => {
    const { isAuthenticated } = useAuthStore();
    const navigate = useNavigate();

    useEffect(() => {
      if (!isAuthenticated) {
        navigate('/login');
      }
    }, [isAuthenticated, navigate]);

    if (!isAuthenticated) {
      return <div>Redirecting...</div>;
    }

    return <Component {...props} />;
  };
};

// Uso:
export default withAuth(DashboardPage);
```

### 4. Custom Hooks Pattern

**Para lógica compartida:**

```typescript
// src/hooks/use-pagination.ts
export const usePagination = (totalItems: number, itemsPerPage: number = 10) => {
  const [currentPage, setCurrentPage] = useState(1);

  const totalPages = Math.ceil(totalItems / itemsPerPage);
  const startIndex = (currentPage - 1) * itemsPerPage;
  const endIndex = startIndex + itemsPerPage;

  const goToPage = (page: number) => {
    setCurrentPage(Math.max(1, Math.min(page, totalPages)));
  };

  const nextPage = () => goToPage(currentPage + 1);
  const prevPage = () => goToPage(currentPage - 1);

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

// Uso:
const UsersPage = () => {
  const { data: users } = useUsers();
  const pagination = usePagination(users?.length || 0, 10);

  const visibleUsers = users?.slice(
    pagination.startIndex,
    pagination.endIndex
  );

  return (
    <>
      <UsersList users={visibleUsers} />
      <Pagination {...pagination} />
    </>
  );
};
```

---

## 📁 Estructura de Carpetas

### Estructura Recomendada

```
src/
├── assets/                      # Imágenes, fonts, etc.
│   ├── images/
│   └── fonts/
│
├── components/                  # Componentes reutilizables
│   ├── ui/                      # Shadcn/UI components (copy-paste)
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── dialog.tsx
│   │   └── ...
│   │
│   ├── atoms/                   # Componentes básicos
│   │   ├── Logo.tsx
│   │   ├── Spinner.tsx
│   │   └── Avatar.tsx
│   │
│   ├── molecules/               # Agrupaciones simples
│   │   ├── FormField.tsx
│   │   ├── SearchBar.tsx
│   │   └── UserAvatar.tsx
│   │
│   └── organisms/               # Componentes complejos
│       ├── Sidebar.tsx
│       ├── Header.tsx
│       ├── DataTable/
│       │   ├── index.tsx
│       │   ├── DataTableHeader.tsx
│       │   ├── DataTableBody.tsx
│       │   └── DataTableRow.tsx
│       └── UserForm.tsx
│
├── pages/                       # Páginas enrutables
│   ├── auth/
│   │   ├── LoginPage.tsx
│   │   └── ForgotPasswordPage.tsx
│   │
│   ├── dashboard/
│   │   └── DashboardPage.tsx
│   │
│   ├── users/
│   │   ├── UsersPage.tsx
│   │   ├── UserDetailPage.tsx
│   │   └── CreateUserPage.tsx
│   │
│   ├── products/
│   └── orders/
│
├── layouts/                     # Layouts reutilizables
│   ├── RootLayout.tsx
│   ├── AuthLayout.tsx
│   └── DashboardLayout.tsx
│
├── hooks/                       # Custom hooks
│   ├── queries/                 # React Query hooks
│   │   ├── use-users.ts
│   │   ├── use-products.ts
│   │   └── use-orders.ts
│   │
│   ├── mutations/               # React Query mutations
│   │   ├── use-create-user.ts
│   │   ├── use-update-user.ts
│   │   └── use-delete-user.ts
│   │
│   └── use-pagination.ts
│       use-debounce.ts
│       use-permissions.ts
│
├── services/                    # API services
│   ├── api-client.ts            # Axios configurado
│   ├── auth.service.ts
│   ├── users.service.ts
│   ├── products.service.ts
│   └── orders.service.ts
│
├── stores/                      # Zustand stores
│   ├── auth.store.ts
│   └── ui.store.ts
│
├── types/                       # TypeScript types
│   ├── api.types.ts
│   ├── user.types.ts
│   ├── product.types.ts
│   └── order.types.ts
│
├── lib/                         # Utilities y helpers
│   ├── query-keys.ts            # React Query keys
│   ├── form-utils.ts
│   ├── date-utils.ts
│   ├── string-utils.ts
│   └── validators.ts
│
├── router/                      # React Router config
│   ├── index.tsx
│   ├── routes.tsx
│   └── loaders.ts
│
├── hoc/                         # Higher-Order Components
│   ├── with-auth.tsx
│   └── with-permission.tsx
│
├── constants/                   # Constantes
│   ├── permissions.ts
│   ├── roles.ts
│   └── routes.ts
│
├── styles/                      # Estilos globales
│   └── globals.css
│
├── App.tsx                      # Componente raíz
├── main.tsx                     # Entry point
└── vite-env.d.ts                # Vite types
```

---

## 🔄 Flujo de Datos

### Arquitectura de Datos

```
┌─────────────────────────────────────────────────────────────────┐
│                          USER INTERACTION                        │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                         COMPONENT                                │
│  ┌─────────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │   UI Events     │  │  useState()  │  │  useForm()       │  │
│  │  (onClick, etc) │  │  (local UI)  │  │  (form state)    │  │
│  └─────────────────┘  └──────────────┘  └──────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                      STATE MANAGEMENT                            │
│  ┌────────────────────┐         ┌──────────────────────────┐   │
│  │   Zustand Stores   │         │    React Query Cache     │   │
│  │  • Auth Store      │         │  • Server State Cache    │   │
│  │  • UI Store        │         │  • Query Invalidation    │   │
│  │  (Global UI State) │         │  • Optimistic Updates    │   │
│  └────────────────────┘         └──────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                        API LAYER                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    Axios Client                           │  │
│  │  • JWT Interceptor (Authorization)                        │  │
│  │  • Tenant Interceptor (X-Tenant-ID)                       │  │
│  │  • Error Handling                                         │  │
│  │  • Refresh Token Logic                                    │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    API GATEWAY (Backend)                         │
│  • Authentication & Authorization                                │
│  • Multi-tenant Isolation                                        │
│  • Rate Limiting                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Flujo de Autenticación

```
1. User Login
   ↓
2. POST /api/v1/auth/login { email, password }
   ↓
3. Backend valida credenciales
   ↓
4. Backend retorna { accessToken, refreshToken, user }
   ↓
5. Frontend guarda tokens en localStorage
   ↓
6. Frontend actualiza Auth Store (Zustand)
   ↓
7. Redirect a /dashboard
   ↓
8. Todas las requests incluyen:
   - Authorization: Bearer {accessToken}
   - X-Tenant-ID: {selectedTenantId}
```

### Flujo de Request Protegida

```
1. Component monta
   ↓
2. useQuery ejecuta API call
   ↓
3. Axios Request Interceptor:
   - Agrega Authorization header
   - Agrega X-Tenant-ID header
   ↓
4. Backend valida:
   - JWT válido?
   - User tiene acceso al tenant?
   - User tiene permiso?
   ↓
5. Backend retorna data
   ↓
6. React Query actualiza cache
   ↓
7. Component re-renderiza con data
```

### Flujo de Error 401 (Token Expirado)

```
1. Backend retorna 401 Unauthorized
   ↓
2. Axios Response Interceptor detecta 401
   ↓
3. Intenta refresh token:
   POST /api/v1/auth/refresh { refreshToken }
   ↓
4a. Si success:
    - Guarda nuevo accessToken
    - Retry request original
    ↓
4b. Si falla:
    - Logout user
    - Clear storage
    - Redirect a /login
```

---

## 🔐 Autenticación y Seguridad

### 1. JWT Storage

**Decisión:** Usar **localStorage** (no cookies)

**Por qué:**
- ✅ Más simple para SPA
- ✅ Compatible con React Native (futuro mobile app)
- ✅ No requiere configurar CORS credentials

**Trade-offs aceptados:**
- ⚠️ Vulnerable a XSS (mitigado con Content Security Policy)
- ⚠️ No httpOnly (pero backend valida siempre)

**Implementación:**

```typescript
// src/services/auth.service.ts
export const authService = {
  login: async (email: string, password: string) => {
    const response = await apiClient.post<LoginResponse>('/auth/login', {
      email,
      password,
    });

    const { accessToken, refreshToken, user } = response.data;

    // Guardar tokens
    localStorage.setItem('access_token', accessToken);
    localStorage.setItem('refresh_token', refreshToken);

    // Actualizar store
    useAuthStore.getState().setUser(user);
    useAuthStore.getState().setTokens(accessToken, refreshToken);

    return response.data;
  },

  logout: () => {
    localStorage.clear();
    useAuthStore.getState().logout();
  },

  refreshToken: async () => {
    const refreshToken = localStorage.getItem('refresh_token');

    if (!refreshToken) {
      throw new Error('No refresh token');
    }

    const response = await apiClient.post<RefreshResponse>('/auth/refresh', {
      refreshToken,
    });

    const { accessToken: newAccessToken } = response.data;

    localStorage.setItem('access_token', newAccessToken);
    useAuthStore.getState().setTokens(newAccessToken, refreshToken);

    return newAccessToken;
  },
};
```

### 2. Axios Interceptors

**Request Interceptor** (agregar JWT y Tenant ID):

```typescript
// src/services/api-client.ts
apiClient.interceptors.request.use(
  (config) => {
    // JWT Token
    const token = localStorage.getItem('access_token');
    if (token && config.headers) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    // Tenant ID
    const tenantId = localStorage.getItem('selected_tenant_id');
    if (tenantId && config.headers) {
      config.headers['X-Tenant-ID'] = tenantId;
    }

    return config;
  },
  (error) => Promise.reject(error)
);
```

**Response Interceptor** (handle 401 y refresh):

```typescript
apiClient.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;

    // Si es 401 y no hemos retried aún
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      try {
        // Intentar refresh token
        const newAccessToken = await authService.refreshToken();

        // Actualizar header y retry
        originalRequest.headers.Authorization = `Bearer ${newAccessToken}`;
        return apiClient(originalRequest);
      } catch (refreshError) {
        // Refresh falló, logout
        authService.logout();
        window.location.href = '/login';
        return Promise.reject(refreshError);
      }
    }

    return Promise.reject(error);
  }
);
```

### 3. Protected Routes

```typescript
// src/router/loaders.ts
export const protectedLoader: LoaderFunction = async () => {
  const { isAuthenticated } = useAuthStore.getState();

  if (!isAuthenticated) {
    return redirect('/login');
  }

  return null;
};

// Con permisos:
export const protectedLoaderWithPermission = (permission: string): LoaderFunction => {
  return async () => {
    const { isAuthenticated, user } = useAuthStore.getState();

    if (!isAuthenticated) {
      return redirect('/login');
    }

    if (!hasPermission(user, permission)) {
      return redirect('/unauthorized');
    }

    return null;
  };
};

// Uso:
{
  path: 'users',
  element: <UsersPage />,
  loader: protectedLoaderWithPermission('users.read'),
}
```

### 4. Permission Checks en Componentes

```typescript
// src/hooks/use-permissions.ts
export const usePermissions = () => {
  const { user } = useAuthStore();

  const hasPermission = (permission: string): boolean => {
    if (!user) return false;

    // Super admin tiene todos los permisos
    if (user.role === 'SUPER_ADMIN') return true;

    // Check user permissions
    return user.permissions.includes(permission);
  };

  const hasAnyPermission = (permissions: string[]): boolean => {
    return permissions.some(hasPermission);
  };

  const hasAllPermissions = (permissions: string[]): boolean => {
    return permissions.every(hasPermission);
  };

  return {
    hasPermission,
    hasAnyPermission,
    hasAllPermissions,
  };
};

// Uso en componente:
const UsersPage = () => {
  const { hasPermission } = usePermissions();

  return (
    <div>
      <h1>Usuarios</h1>

      {hasPermission('users.create') && (
        <Button onClick={handleCreate}>
          Crear Usuario
        </Button>
      )}

      <UsersList />
    </div>
  );
};
```

### 5. Content Security Policy (CSP)

**En nginx.conf:**

```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:; connect-src 'self' https://api.bmoi.com;" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "DENY" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

---

## ⚡ Performance y Optimización

### 1. Code Splitting

**Lazy Loading de rutas:**

```typescript
// src/router/index.tsx
import { lazy } from 'react';

const DashboardPage = lazy(() => import('@/pages/dashboard/DashboardPage'));
const UsersPage = lazy(() => import('@/pages/users/UsersPage'));
const ProductsPage = lazy(() => import('@/pages/products/ProductsPage'));

export const router = createBrowserRouter([
  {
    path: 'dashboard',
    element: (
      <Suspense fallback={<PageLoader />}>
        <DashboardLayout />
      </Suspense>
    ),
    children: [
      {
        index: true,
        element: (
          <Suspense fallback={<Skeleton />}>
            <DashboardPage />
          </Suspense>
        ),
      },
      // ...
    ],
  },
]);
```

### 2. Bundle Size Optimization

**Manual chunks en Vite:**

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': [
            '@radix-ui/react-dialog',
            '@radix-ui/react-dropdown-menu',
            '@radix-ui/react-select',
          ],
          'form-vendor': ['react-hook-form', 'zod', '@hookform/resolvers'],
          'query-vendor': ['@tanstack/react-query', 'axios'],
          'state-vendor': ['zustand'],
        },
      },
    },
  },
});
```

**Resultado esperado:**
```
dist/
├── index.html (2KB)
├── assets/
│   ├── index-abc123.js (50KB) ← App code
│   ├── react-vendor-def456.js (150KB) ← React
│   ├── ui-vendor-ghi789.js (80KB) ← Radix UI
│   ├── form-vendor-jkl012.js (40KB) ← Forms
│   ├── query-vendor-mno345.js (30KB) ← React Query
│   └── state-vendor-pqr678.js (5KB) ← Zustand
```

### 3. React Query Optimizations

**Stale time y cache time:**

```typescript
// src/lib/query-client.ts
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutos
      gcTime: 10 * 60 * 1000, // 10 minutos (antes cacheTime)
      retry: 3,
      refetchOnWindowFocus: false,
    },
  },
});
```

**Prefetching:**

```typescript
// Prefetch en hover
const UsersTable = () => {
  const queryClient = useQueryClient();

  const handleMouseEnter = (userId: string) => {
    queryClient.prefetchQuery({
      queryKey: queryKeys.users.detail(userId),
      queryFn: () => usersApi.getById(userId),
    });
  };

  return (
    <table>
      {users.map(user => (
        <tr
          key={user.id}
          onMouseEnter={() => handleMouseEnter(user.id)}
        >
          <td>{user.name}</td>
        </tr>
      ))}
    </table>
  );
};
```

### 4. Image Optimization

**Lazy loading de imágenes:**

```typescript
const ProductImage = ({ src, alt }: { src: string; alt: string }) => {
  return (
    <img
      src={src}
      alt={alt}
      loading="lazy"
      decoding="async"
      className="w-full h-auto"
    />
  );
};
```

**Responsive images:**

```tsx
<picture>
  <source
    srcSet="/images/product-small.webp"
    media="(max-width: 640px)"
    type="image/webp"
  />
  <source
    srcSet="/images/product-medium.webp"
    media="(max-width: 1024px)"
    type="image/webp"
  />
  <img
    src="/images/product-large.webp"
    alt="Product"
    loading="lazy"
  />
</picture>
```

### 5. Memoization

**React.memo para componentes costosos:**

```typescript
export const UserCard = React.memo(({ user }: { user: User }) => {
  return (
    <Card>
      <CardHeader>
        <CardTitle>{user.name}</CardTitle>
      </CardHeader>
      <CardContent>
        <p>{user.email}</p>
      </CardContent>
    </Card>
  );
}, (prevProps, nextProps) => {
  // Solo re-render si el user cambió
  return prevProps.user.id === nextProps.user.id;
});
```

**useMemo para cálculos costosos:**

```typescript
const ProductsPage = () => {
  const { data: products } = useProducts();

  const sortedProducts = useMemo(() => {
    return products?.sort((a, b) => b.createdAt - a.createdAt);
  }, [products]);

  return <ProductsList products={sortedProducts} />;
};
```

---

## 🧪 Testing Strategy

### Test Pyramid

```
       ┌─────────────────┐
       │   E2E Tests     │ ← 10% (Critical flows)
       │   (Playwright)  │
       ├─────────────────┤
       │ Integration     │ ← 30% (Components + API)
       │   (Vitest+RTL)  │
       ├─────────────────┤
       │  Unit Tests     │ ← 60% (Utils, hooks, logic)
       │    (Vitest)     │
       └─────────────────┘
```

### Tools

- **Vitest**: Test runner (más rápido que Jest)
- **React Testing Library**: Component testing
- **MSW (Mock Service Worker)**: API mocking
- **Playwright**: E2E testing

### Example Unit Test

```typescript
// src/lib/__tests__/date-utils.test.ts
import { describe, it, expect } from 'vitest';
import { formatDate } from '../date-utils';

describe('formatDate', () => {
  it('should format date correctly', () => {
    const date = new Date('2024-01-15T10:30:00');
    expect(formatDate(date)).toBe('15/01/2024');
  });

  it('should handle invalid dates', () => {
    expect(formatDate(null)).toBe('-');
  });
});
```

### Example Component Test

```typescript
// src/components/organisms/__tests__/UserForm.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { UserForm } from '../UserForm';

describe('UserForm', () => {
  it('should submit form with valid data', async () => {
    const onSubmit = vi.fn();
    const user = userEvent.setup();

    render(<UserForm onSubmit={onSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'test@example.com');
    await user.type(screen.getByLabelText(/nombre/i), 'Test User');
    await user.click(screen.getByRole('button', { name: /guardar/i }));

    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'test@example.com',
        name: 'Test User',
      });
    });
  });

  it('should show validation errors', async () => {
    const user = userEvent.setup();

    render(<UserForm onSubmit={vi.fn()} />);

    await user.click(screen.getByRole('button', { name: /guardar/i }));

    expect(await screen.findByText(/email es requerido/i)).toBeInTheDocument();
  });
});
```

---

## 🚀 Deployment

### Docker Multi-Stage Build

```dockerfile
# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Nginx Configuration

```nginx
server {
  listen 80;
  server_name localhost;
  root /usr/share/nginx/html;
  index index.html;

  # Gzip
  gzip on;
  gzip_types text/plain text/css application/json application/javascript text/xml application/xml+rss text/javascript;

  # SPA routing
  location / {
    try_files $uri $uri/ /index.html;
  }

  # Cache static assets
  location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
  }

  # Security headers
  add_header X-Frame-Options "DENY" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header X-XSS-Protection "1; mode=block" always;
}
```

---

## 📚 Conclusiones

### Principios Clave

1. **Type Safety First**: TypeScript estricto en todo
2. **Performance Matters**: Code splitting, lazy loading, caching
3. **User Experience**: Optimistic updates, loading states, error handling
4. **Security**: JWT, RBAC, CSP, input validation
5. **Testability**: Cobertura >80%, testing pyramid
6. **Maintainability**: Código limpio, patterns consistentes
7. **Scalability**: Feature-based structure para apps grandes

### Métricas de Éxito

- **Bundle Size**: < 500KB inicial, < 2MB total
- **First Contentful Paint**: < 1.5s
- **Time to Interactive**: < 3s
- **Test Coverage**: > 80%
- **Lighthouse Score**: > 90

---

<div align="center">

**Admin Web Multi-Tenant con React + TypeScript**

Type-safe | Performante | Escalable | Seguro

</div>
