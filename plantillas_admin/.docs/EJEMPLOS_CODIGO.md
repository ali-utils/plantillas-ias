# Ejemplos de Código - Admin Web bMOI

> **Templates y ejemplos completos listos para usar**

---

## 📋 Índice

1. [Layouts](#layouts)
2. [Pages](#pages)
3. [Components](#components)
4. [Hooks](#hooks)
5. [Services](#services)
6. [Store](#store)
7. [Forms](#forms)
8. [API Integration](#api-integration)

---

## 1. Layouts

### Dashboard Layout

```typescript
// src/layouts/DashboardLayout.tsx
import { Outlet } from 'react-router-dom';
import { Header } from '@/components/common/Header';
import { Sidebar } from '@/components/common/Sidebar';

export const DashboardLayout = () => {
  return (
    <div className="flex h-screen bg-background">
      {/* Sidebar */}
      <Sidebar />

      {/* Main Content */}
      <div className="flex flex-1 flex-col overflow-hidden">
        {/* Header */}
        <Header />

        {/* Page Content */}
        <main className="flex-1 overflow-y-auto p-6">
          <Outlet />
        </main>
      </div>
    </div>
  );
};
```

### Auth Layout

```typescript
// src/layouts/AuthLayout.tsx
import { Outlet } from 'react-router-dom';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';

export const AuthLayout = () => {
  return (
    <div className="flex min-h-screen items-center justify-center bg-gradient-to-br from-slate-50 to-slate-100 p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="space-y-1 text-center">
          <div className="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-full bg-primary">
            <span className="text-2xl font-bold text-primary-foreground">B</span>
          </div>
          <CardTitle className="text-2xl">bMOI Admin</CardTitle>
          <CardDescription>Panel de Control</CardDescription>
        </CardHeader>
        <CardContent>
          <Outlet />
        </CardContent>
      </Card>
    </div>
  );
};
```

---

## 2. Pages

### Login Page

```typescript
// src/pages/auth/LoginPage.tsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { useAuth } from '@/features/auth/hooks/useAuth';
import { toast } from 'sonner';
import { Loader2 } from 'lucide-react';

const loginSchema = z.object({
  email: z.string().email('Email inválido'),
  password: z.string().min(8, 'Mínimo 8 caracteres'),
});

type LoginFormData = z.infer<typeof loginSchema>;

export const LoginPage = () => {
  const navigate = useNavigate();
  const { login } = useAuth();
  const [isLoading, setIsLoading] = useState(false);

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = async (data: LoginFormData) => {
    setIsLoading(true);
    try {
      await login(data.email, data.password);
      toast.success('Inicio de sesión exitoso');
      navigate('/dashboard');
    } catch (error: any) {
      toast.error(error.message || 'Error al iniciar sesión');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <div className="space-y-2">
        <Label htmlFor="email">Email</Label>
        <Input
          id="email"
          type="email"
          placeholder="admin@bmoi.com"
          {...register('email')}
          disabled={isLoading}
        />
        {errors.email && (
          <p className="text-sm text-destructive">{errors.email.message}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="password">Contraseña</Label>
        <Input
          id="password"
          type="password"
          placeholder="••••••••"
          {...register('password')}
          disabled={isLoading}
        />
        {errors.password && (
          <p className="text-sm text-destructive">{errors.password.message}</p>
        )}
      </div>

      <Button type="submit" className="w-full" disabled={isLoading}>
        {isLoading && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
        Iniciar Sesión
      </Button>
    </form>
  );
};
```

### Dashboard Page

```typescript
// src/pages/dashboard/DashboardPage.tsx
import { useQuery } from '@tanstack/react-query';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { Users, Building, Package, TrendingUp } from 'lucide-react';
import { dashboardService } from '@/features/dashboard/services/dashboard.service';

export const DashboardPage = () => {
  const { data: stats, isLoading } = useQuery({
    queryKey: ['dashboard-stats'],
    queryFn: dashboardService.getStats,
  });

  if (isLoading) {
    return <DashboardSkeleton />;
  }

  return (
    <div className="space-y-6">
      <div>
        <h1 className="text-3xl font-bold">Dashboard</h1>
        <p className="text-muted-foreground">Resumen general del sistema</p>
      </div>

      {/* Stats Cards */}
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        <StatCard
          title="Total Usuarios"
          value={stats?.totalUsers || 0}
          icon={<Users className="h-4 w-4" />}
          trend="+12%"
        />
        <StatCard
          title="Tenants Activos"
          value={stats?.activeTenants || 0}
          icon={<Building className="h-4 w-4" />}
          trend="+5%"
        />
        <StatCard
          title="Artesanías"
          value={stats?.totalArtesanias || 0}
          icon={<Package className="h-4 w-4" />}
          trend="+18%"
        />
        <StatCard
          title="Transacciones"
          value={stats?.totalTransactions || 0}
          icon={<TrendingUp className="h-4 w-4" />}
          trend="+23%"
        />
      </div>

      {/* Charts */}
      <div className="grid gap-4 md:grid-cols-2">
        <Card>
          <CardHeader>
            <CardTitle>Usuarios por Mes</CardTitle>
          </CardHeader>
          <CardContent>
            {/* Chart component */}
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>Ingresos</CardTitle>
          </CardHeader>
          <CardContent>
            {/* Chart component */}
          </CardContent>
        </Card>
      </div>
    </div>
  );
};

// Stat Card Component
interface StatCardProps {
  title: string;
  value: number;
  icon: React.ReactNode;
  trend: string;
}

const StatCard = ({ title, value, icon, trend }: StatCardProps) => (
  <Card>
    <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
      <CardTitle className="text-sm font-medium">{title}</CardTitle>
      {icon}
    </CardHeader>
    <CardContent>
      <div className="text-2xl font-bold">{value.toLocaleString()}</div>
      <p className="text-xs text-muted-foreground">
        <span className="text-green-600">{trend}</span> vs mes anterior
      </p>
    </CardContent>
  </Card>
);

const DashboardSkeleton = () => (
  <div className="space-y-6">
    <Skeleton className="h-20 w-full" />
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
      {[...Array(4)].map((_, i) => (
        <Skeleton key={i} className="h-32 w-full" />
      ))}
    </div>
  </div>
);
```

### Tenants List Page

```typescript
// src/pages/tenants/TenantsListPage.tsx
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Link } from 'react-router-dom';
import { Plus, Search } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { TenantsTable } from '@/features/tenants/components/TenantsTable';
import { tenantsService } from '@/features/tenants/services/tenants.service';

export const TenantsListPage = () => {
  const [search, setSearch] = useState('');

  const { data: tenants, isLoading } = useQuery({
    queryKey: ['tenants', search],
    queryFn: () => tenantsService.getAll({ search }),
  });

  return (
    <div className="space-y-6">
      {/* Header */}
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-3xl font-bold">Tenants</h1>
          <p className="text-muted-foreground">Gestión de organizaciones</p>
        </div>
        <Button asChild>
          <Link to="/tenants/create">
            <Plus className="mr-2 h-4 w-4" />
            Nuevo Tenant
          </Link>
        </Button>
      </div>

      {/* Search */}
      <div className="relative w-full max-w-sm">
        <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
        <Input
          placeholder="Buscar tenants..."
          value={search}
          onChange={(e) => setSearch(e.target.value)}
          className="pl-9"
        />
      </div>

      {/* Table */}
      <TenantsTable tenants={tenants || []} isLoading={isLoading} />
    </div>
  );
};
```

---

## 3. Components

### Header Component

```typescript
// src/components/common/Header.tsx
import { Bell, Settings, User } from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar';
import { useAuth } from '@/features/auth/hooks/useAuth';

export const Header = () => {
  const { user, logout } = useAuth();

  return (
    <header className="flex h-16 items-center justify-between border-b bg-background px-6">
      {/* Left side - can add breadcrumbs */}
      <div className="flex items-center gap-4">
        {/* Breadcrumbs component */}
      </div>

      {/* Right side - actions */}
      <div className="flex items-center gap-2">
        {/* Notifications */}
        <Button variant="ghost" size="icon">
          <Bell className="h-5 w-5" />
        </Button>

        {/* Settings */}
        <Button variant="ghost" size="icon">
          <Settings className="h-5 w-5" />
        </Button>

        {/* User menu */}
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="relative h-10 w-10 rounded-full">
              <Avatar>
                <AvatarImage src={user?.avatar} alt={user?.name} />
                <AvatarFallback>
                  {user?.name?.charAt(0).toUpperCase() || 'A'}
                </AvatarFallback>
              </Avatar>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent className="w-56" align="end" forceMount>
            <DropdownMenuLabel className="font-normal">
              <div className="flex flex-col space-y-1">
                <p className="text-sm font-medium leading-none">{user?.name}</p>
                <p className="text-xs leading-none text-muted-foreground">
                  {user?.email}
                </p>
              </div>
            </DropdownMenuLabel>
            <DropdownMenuSeparator />
            <DropdownMenuItem>
              <User className="mr-2 h-4 w-4" />
              Perfil
            </DropdownMenuItem>
            <DropdownMenuItem>
              <Settings className="mr-2 h-4 w-4" />
              Configuración
            </DropdownMenuItem>
            <DropdownMenuSeparator />
            <DropdownMenuItem onClick={logout}>
              Cerrar Sesión
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      </div>
    </header>
  );
};
```

### Sidebar Component

```typescript
// src/components/common/Sidebar.tsx
import { Link, useLocation } from 'react-router-dom';
import { cn } from '@/lib/utils';
import {
  LayoutDashboard,
  Building,
  Users,
  Package,
  FileText,
  Settings,
} from 'lucide-react';

const menuItems = [
  {
    title: 'Dashboard',
    href: '/dashboard',
    icon: LayoutDashboard,
  },
  {
    title: 'Tenants',
    href: '/tenants',
    icon: Building,
  },
  {
    title: 'Usuarios',
    href: '/users',
    icon: Users,
  },
  {
    title: 'Artesanías',
    href: '/artesanias',
    icon: Package,
  },
  {
    title: 'Auditoría',
    href: '/audit',
    icon: FileText,
  },
  {
    title: 'Configuración',
    href: '/settings',
    icon: Settings,
  },
];

export const Sidebar = () => {
  const location = useLocation();

  return (
    <aside className="flex w-64 flex-col border-r bg-background">
      {/* Logo */}
      <div className="flex h-16 items-center border-b px-6">
        <Link to="/" className="flex items-center gap-2">
          <div className="flex h-8 w-8 items-center justify-center rounded-lg bg-primary">
            <span className="text-lg font-bold text-primary-foreground">B</span>
          </div>
          <span className="text-xl font-bold">bMOI Admin</span>
        </Link>
      </div>

      {/* Navigation */}
      <nav className="flex-1 space-y-1 p-4">
        {menuItems.map((item) => {
          const Icon = item.icon;
          const isActive = location.pathname === item.href ||
            location.pathname.startsWith(item.href + '/');

          return (
            <Link
              key={item.href}
              to={item.href}
              className={cn(
                'flex items-center gap-3 rounded-lg px-3 py-2 text-sm font-medium transition-colors',
                isActive
                  ? 'bg-primary text-primary-foreground'
                  : 'text-muted-foreground hover:bg-accent hover:text-accent-foreground'
              )}
            >
              <Icon className="h-5 w-5" />
              {item.title}
            </Link>
          );
        })}
      </nav>
    </aside>
  );
};
```

### Data Table Component

```typescript
// src/components/common/DataTable.tsx
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Skeleton } from '@/components/ui/skeleton';
import { flexRender, Table as TanstackTable } from '@tanstack/react-table';

interface DataTableProps<TData> {
  table: TanstackTable<TData>;
  isLoading?: boolean;
  columns: number;
}

export function DataTable<TData>({ table, isLoading, columns }: DataTableProps<TData>) {
  if (isLoading) {
    return (
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => (
                  <TableHead key={header.id}>
                    {header.isPlaceholder
                      ? null
                      : flexRender(header.column.columnDef.header, header.getContext())}
                  </TableHead>
                ))}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {[...Array(5)].map((_, i) => (
              <TableRow key={i}>
                {[...Array(columns)].map((_, j) => (
                  <TableCell key={j}>
                    <Skeleton className="h-6 w-full" />
                  </TableCell>
                ))}
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </div>
    );
  }

  return (
    <div className="rounded-md border">
      <Table>
        <TableHeader>
          {table.getHeaderGroups().map((headerGroup) => (
            <TableRow key={headerGroup.id}>
              {headerGroup.headers.map((header) => (
                <TableHead key={header.id}>
                  {header.isPlaceholder
                    ? null
                    : flexRender(header.column.columnDef.header, header.getContext())}
                </TableHead>
              ))}
            </TableRow>
          ))}
        </TableHeader>
        <TableBody>
          {table.getRowModel().rows?.length ? (
            table.getRowModel().rows.map((row) => (
              <TableRow key={row.id} data-state={row.getIsSelected() && 'selected'}>
                {row.getVisibleCells().map((cell) => (
                  <TableCell key={cell.id}>
                    {flexRender(cell.column.columnDef.cell, cell.getContext())}
                  </TableCell>
                ))}
              </TableRow>
            ))
          ) : (
            <TableRow>
              <TableCell colSpan={columns} className="h-24 text-center">
                No hay resultados.
              </TableCell>
            </TableRow>
          )}
        </TableBody>
      </Table>
    </div>
  );
}
```

---

## 4. Hooks

### useAuth Hook

```typescript
// src/features/auth/hooks/useAuth.ts
import { useAuthStore } from '../store/authStore';
import { authService } from '../services/auth.service';

export const useAuth = () => {
  const { user, isAuthenticated, setUser, setTokens, logout: logoutStore } = useAuthStore();

  const login = async (email: string, password: string) => {
    const response = await authService.login(email, password);
    setUser(response.user);
    setTokens(response.accessToken, response.refreshToken);
  };

  const logout = () => {
    authService.logout();
    logoutStore();
  };

  const checkAuth = async () => {
    try {
      const user = await authService.me();
      setUser(user);
      return true;
    } catch (error) {
      logout();
      return false;
    }
  };

  return {
    user,
    isAuthenticated,
    login,
    logout,
    checkAuth,
  };
};
```

### useDebounce Hook

```typescript
// src/hooks/useDebounce.ts
import { useEffect, useState } from 'react';

export function useDebounce<T>(value: T, delay: number = 500): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}
```

---

## 5. Services

### Auth Service

```typescript
// src/features/auth/services/auth.service.ts
import apiClient from '@/lib/api/client';
import { ENDPOINTS } from '@/lib/api/endpoints';
import type { User, LoginResponse } from '../types/auth.types';

export const authService = {
  async login(email: string, password: string): Promise<LoginResponse> {
    const response = await apiClient.post(ENDPOINTS.LOGIN, { email, password });
    return response.data;
  },

  async logout(): Promise<void> {
    await apiClient.post(ENDPOINTS.LOGOUT);
    localStorage.removeItem('access_token');
    localStorage.removeItem('refresh_token');
  },

  async me(): Promise<User> {
    const response = await apiClient.get(ENDPOINTS.ME);
    return response.data;
  },

  async refresh(refreshToken: string): Promise<{ accessToken: string }> {
    const response = await apiClient.post(ENDPOINTS.REFRESH, { refreshToken });
    return response.data;
  },
};
```

### Tenants Service

```typescript
// src/features/tenants/services/tenants.service.ts
import apiClient from '@/lib/api/client';
import { ENDPOINTS } from '@/lib/api/endpoints';
import type { Tenant, CreateTenantDto, UpdateTenantDto } from '../types/tenants.types';

export const tenantsService = {
  async getAll(filters?: { search?: string; plan?: string }): Promise<Tenant[]> {
    const response = await apiClient.get(ENDPOINTS.TENANTS, { params: filters });
    return response.data;
  },

  async getById(id: string): Promise<Tenant> {
    const response = await apiClient.get(ENDPOINTS.TENANT_BY_ID(id));
    return response.data;
  },

  async create(data: CreateTenantDto): Promise<Tenant> {
    const response = await apiClient.post(ENDPOINTS.TENANTS, data);
    return response.data;
  },

  async update(id: string, data: UpdateTenantDto): Promise<Tenant> {
    const response = await apiClient.patch(ENDPOINTS.TENANT_BY_ID(id), data);
    return response.data;
  },

  async delete(id: string): Promise<void> {
    await apiClient.delete(ENDPOINTS.TENANT_BY_ID(id));
  },
};
```

---

## 6. Store (Zustand)

### Auth Store

```typescript
// src/features/auth/store/authStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import type { User } from '../types/auth.types';

interface AuthState {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;

  setUser: (user: User | null) => void;
  setTokens: (accessToken: string, refreshToken: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      isAuthenticated: false,

      setUser: (user) =>
        set({
          user,
          isAuthenticated: !!user,
        }),

      setTokens: (accessToken, refreshToken) => {
        localStorage.setItem('access_token', accessToken);
        localStorage.setItem('refresh_token', refreshToken);
        set({ accessToken, refreshToken });
      },

      logout: () => {
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        set({
          user: null,
          accessToken: null,
          refreshToken: null,
          isAuthenticated: false,
        });
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);
```

---

## 7. Forms

### Tenant Form

```typescript
// src/features/tenants/components/TenantForm.tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

const tenantSchema = z.object({
  name: z.string().min(3, 'Mínimo 3 caracteres'),
  slug: z.string().min(3, 'Mínimo 3 caracteres').regex(/^[a-z0-9-]+$/, 'Solo letras minúsculas, números y guiones'),
  plan: z.enum(['free', 'plan1', 'plan2', 'plan3']),
  email: z.string().email('Email inválido'),
});

type TenantFormData = z.infer<typeof tenantSchema>;

interface TenantFormProps {
  defaultValues?: Partial<TenantFormData>;
  onSubmit: (data: TenantFormData) => Promise<void>;
  isLoading?: boolean;
}

export const TenantForm = ({ defaultValues, onSubmit, isLoading }: TenantFormProps) => {
  const {
    register,
    handleSubmit,
    formState: { errors },
    watch,
    setValue,
  } = useForm<TenantFormData>({
    resolver: zodResolver(tenantSchema),
    defaultValues,
  });

  // Auto-generate slug from name
  const name = watch('name');
  useEffect(() => {
    if (name && !defaultValues?.slug) {
      const slug = name.toLowerCase().replace(/[^a-z0-9]+/g, '-');
      setValue('slug', slug);
    }
  }, [name, setValue, defaultValues]);

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <div className="space-y-2">
        <Label htmlFor="name">Nombre</Label>
        <Input id="name" {...register('name')} disabled={isLoading} />
        {errors.name && <p className="text-sm text-destructive">{errors.name.message}</p>}
      </div>

      <div className="space-y-2">
        <Label htmlFor="slug">Slug</Label>
        <Input id="slug" {...register('slug')} disabled={isLoading} />
        {errors.slug && <p className="text-sm text-destructive">{errors.slug.message}</p>}
      </div>

      <div className="space-y-2">
        <Label htmlFor="email">Email</Label>
        <Input id="email" type="email" {...register('email')} disabled={isLoading} />
        {errors.email && <p className="text-sm text-destructive">{errors.email.message}</p>}
      </div>

      <div className="space-y-2">
        <Label htmlFor="plan">Plan</Label>
        <Select onValueChange={(value) => setValue('plan', value as any)} defaultValue={defaultValues?.plan}>
          <SelectTrigger>
            <SelectValue placeholder="Seleccionar plan" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="free">Free</SelectItem>
            <SelectItem value="plan1">Plan 1</SelectItem>
            <SelectItem value="plan2">Plan 2</SelectItem>
            <SelectItem value="plan3">Plan 3</SelectItem>
          </SelectContent>
        </Select>
        {errors.plan && <p className="text-sm text-destructive">{errors.plan.message}</p>}
      </div>

      <div className="flex justify-end gap-4">
        <Button type="button" variant="outline" disabled={isLoading}>
          Cancelar
        </Button>
        <Button type="submit" disabled={isLoading}>
          {isLoading ? 'Guardando...' : 'Guardar'}
        </Button>
      </div>
    </form>
  );
};
```

---

## 8. API Integration

### React Query Usage

```typescript
// src/features/tenants/hooks/useTenants.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { tenantsService } from '../services/tenants.service';
import type { CreateTenantDto, UpdateTenantDto } from '../types/tenants.types';
import { toast } from 'sonner';

export const useTenants = () => {
  const queryClient = useQueryClient();

  // Get all tenants
  const { data: tenants, isLoading, error } = useQuery({
    queryKey: ['tenants'],
    queryFn: tenantsService.getAll,
  });

  // Create tenant
  const createMutation = useMutation({
    mutationFn: (data: CreateTenantDto) => tenantsService.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tenants'] });
      toast.success('Tenant creado exitosamente');
    },
    onError: (error: any) => {
      toast.error(error.message || 'Error al crear tenant');
    },
  });

  // Update tenant
  const updateMutation = useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateTenantDto }) =>
      tenantsService.update(id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tenants'] });
      toast.success('Tenant actualizado exitosamente');
    },
    onError: (error: any) => {
      toast.error(error.message || 'Error al actualizar tenant');
    },
  });

  // Delete tenant
  const deleteMutation = useMutation({
    mutationFn: (id: string) => tenantsService.delete(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['tenants'] });
      toast.success('Tenant eliminado exitosamente');
    },
    onError: (error: any) => {
      toast.error(error.message || 'Error al eliminar tenant');
    },
  });

  return {
    tenants,
    isLoading,
    error,
    create: createMutation.mutate,
    update: updateMutation.mutate,
    delete: deleteMutation.mutate,
    isCreating: createMutation.isPending,
    isUpdating: updateMutation.isPending,
    isDeleting: deleteMutation.isPending,
  };
};
```

---

<div align="center">

**Ejemplos de Código Completos y Listos para Usar**

Copy-paste y adapta según tus necesidades

</div>
