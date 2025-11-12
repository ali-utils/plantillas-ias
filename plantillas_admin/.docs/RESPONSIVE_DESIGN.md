# Responsive Design Guide - bMOI Admin Panel

**Objetivo:** Asegurar que el panel de administración sea completamente funcional en todos los dispositivos para supervisión y gestión de la plataforma.

---

## 📱 Filosofía: Mobile-First para Supervisión

El Admin Panel debe ser **responsive** con enfoque especial en **supervisión móvil**:

```
Mobile (320px+) → Tablet (768px+) → Desktop (1024px+) → Wide (1440px+)
```

### ¿Por qué Responsive es Crítico para Admin?

- ✅ **Supervisión 24/7** - Admins revisan alertas y métricas desde móvil
- ✅ **Respuesta rápida** - Gestionar incidentes desde cualquier dispositivo
- ✅ **Dashboard ejecutivo** - Directivos ven KPIs desde tablet
- ✅ **Trabajo remoto** - 60%+ de supervisión ocurre fuera de oficina
- ✅ **Auditoría móvil** - Revisar logs y eventos desde dispositivo móvil

---

## 🎯 Breakpoints de Tailwind CSS

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    screens: {
      'sm': '640px',   // Móviles grandes / Tablets pequeñas
      'md': '768px',   // Tablets
      'lg': '1024px',  // Laptops
      'xl': '1280px',  // Desktops
      '2xl': '1536px', // Pantallas grandes (dashboards múltiples)
    },
  },
}
```

### Uso en Componentes Admin

```typescript
// ✅ Mobile-first: Estilos base para supervisión móvil
<div className="
  w-full              // Móvil: ancho completo
  px-4                // Móvil: padding cómodo para lectura
  md:w-2/3            // Tablet: 66% de ancho
  md:px-6             // Tablet: más espacio
  lg:w-1/2            // Desktop: contenido centrado
  xl:w-1/3            // Wide: columna lateral para múltiples dashboards
">
  Contenido
</div>
```

---

## 🏗️ Layouts Responsive para Admin

### 1. Admin Navigation / Sidebar

**Desktop:** Sidebar fijo con menú completo de administración
**Mobile:** Hamburger menu con overlay + quick actions sticky

```typescript
// components/Layout/AdminLayout.tsx
import { useState } from 'react';
import { Bell, AlertTriangle, Users, Settings } from 'lucide-react';

export const AdminLayout = ({ children }) => {
  const [sidebarOpen, setSidebarOpen] = useState(false);
  const [notifications, setNotifications] = useState(3);

  return (
    <div className="flex h-screen overflow-hidden bg-slate-100">
      {/* Sidebar Desktop */}
      <aside className="
        hidden             // Oculto en móvil
        lg:flex            // Visible en desktop
        lg:w-64            // 256px de ancho
        lg:flex-col
        bg-slate-900
        text-white
      ">
        <div className="px-6 py-4 bg-slate-800">
          <h2 className="text-xl font-bold">bMOI Admin</h2>
          <p className="text-xs text-slate-400">Platform Management</p>
        </div>

        <nav className="flex-1 px-4 py-6 space-y-2">
          {/* Admin Menu Items */}
          <a href="/admin/dashboard" className="block px-4 py-2 rounded hover:bg-slate-800">
            Dashboard
          </a>
          <a href="/admin/tenants" className="block px-4 py-2 rounded hover:bg-slate-800">
            Tenants
          </a>
          <a href="/admin/users" className="block px-4 py-2 rounded hover:bg-slate-800">
            All Users
          </a>
          <a href="/admin/audit" className="block px-4 py-2 rounded hover:bg-slate-800">
            Audit Logs
          </a>
          <a href="/admin/plans" className="block px-4 py-2 rounded hover:bg-slate-800">
            Plans & Billing
          </a>
          <a href="/admin/settings" className="block px-4 py-2 rounded hover:bg-slate-800">
            System Settings
          </a>
        </nav>
      </aside>

      {/* Sidebar Mobile (Overlay) */}
      {sidebarOpen && (
        <div className="fixed inset-0 z-50 lg:hidden">
          <div
            className="fixed inset-0 bg-black/50"
            onClick={() => setSidebarOpen(false)}
          />

          <aside className="
            fixed left-0 top-0 bottom-0
            w-72 bg-slate-900 text-white
            transform transition-transform
            overflow-y-auto
          ">
            <div className="px-6 py-4 bg-slate-800">
              <h2 className="text-xl font-bold">bMOI Admin</h2>
              <button
                onClick={() => setSidebarOpen(false)}
                className="absolute top-4 right-4 text-white"
              >
                ✕
              </button>
            </div>

            <nav className="p-4 space-y-2">
              {/* Same menu items as desktop */}
            </nav>
          </aside>
        </div>
      )}

      {/* Main Content */}
      <main className="flex-1 overflow-y-auto">
        {/* Header Mobile - Sticky con quick actions */}
        <header className="
          sticky top-0 z-40
          bg-white shadow-sm
          px-4 py-3
          lg:px-6 lg:py-4
        ">
          <div className="flex items-center justify-between">
            {/* Left: Menu + Title */}
            <div className="flex items-center gap-3">
              <button
                onClick={() => setSidebarOpen(true)}
                className="lg:hidden text-slate-700"
              >
                <svg className="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h16" />
                </svg>
              </button>

              <h1 className="
                text-lg font-semibold text-slate-900
                lg:text-xl
              ">
                Platform Dashboard
              </h1>
            </div>

            {/* Right: Quick Actions */}
            <div className="flex items-center gap-2">
              {/* Notifications - Always visible */}
              <button className="
                relative p-2 text-slate-600 hover:text-slate-900
                hover:bg-slate-100 rounded-lg
              ">
                <Bell className="w-5 h-5" />
                {notifications > 0 && (
                  <span className="
                    absolute -top-1 -right-1
                    w-5 h-5 bg-red-500 text-white
                    text-xs font-bold rounded-full
                    flex items-center justify-center
                  ">
                    {notifications}
                  </span>
                )}
              </button>

              {/* Alerts - Desktop only */}
              <button className="
                hidden md:flex
                items-center gap-2 px-3 py-2
                text-amber-700 bg-amber-50
                hover:bg-amber-100 rounded-lg
              ">
                <AlertTriangle className="w-4 h-4" />
                <span className="text-sm font-medium">2 Alerts</span>
              </button>

              {/* User menu */}
              <button className="
                flex items-center gap-2
                px-2 py-1 rounded-lg
                hover:bg-slate-100
              ">
                <div className="w-8 h-8 bg-blue-500 rounded-full flex items-center justify-center">
                  <span className="text-white text-sm font-bold">AD</span>
                </div>
                <span className="hidden sm:block text-sm font-medium text-slate-700">
                  Admin
                </span>
              </button>
            </div>
          </div>
        </header>

        {/* Page Content */}
        <div className="
          p-4                // Móvil: padding compacto
          md:p-6             // Tablet: más espacio
          lg:p-8             // Desktop: espacioso
        ">
          {children}
        </div>
      </main>
    </div>
  );
};
```

### 2. Admin Dashboard - Métricas Globales

```typescript
// pages/AdminDashboard.tsx
export const AdminDashboard = () => {
  return (
    <div className="space-y-4 md:space-y-6">
      {/* KPI Cards - Global Platform Metrics */}
      <div className="
        grid gap-3 md:gap-4
        grid-cols-2        // Móvil: 2 columnas (prioritizar métricas clave)
        lg:grid-cols-4     // Desktop: 4 columnas
        xl:grid-cols-6     // Wide: 6 columnas
      ">
        <KPICard
          title="Active Tenants"
          value={247}
          change="+12%"
          icon={<Users className="w-5 h-5" />}
          color="blue"
        />
        <KPICard
          title="Total Users"
          value="15.2K"
          change="+8%"
          icon={<Users className="w-5 h-5" />}
          color="green"
        />
        <KPICard
          title="Revenue (MRR)"
          value="$45.7K"
          change="+15%"
          icon={<DollarSign className="w-5 h-5" />}
          color="emerald"
        />
        <KPICard
          title="Alerts"
          value={3}
          change="-2"
          icon={<AlertTriangle className="w-5 h-5" />}
          color="amber"
        />
        <KPICard
          title="Uptime"
          value="99.97%"
          change="+0.02%"
          icon={<Activity className="w-5 h-5" />}
          color="green"
        />
        <KPICard
          title="Avg Response"
          value="142ms"
          change="-8ms"
          icon={<Zap className="w-5 h-5" />}
          color="blue"
        />
      </div>

      {/* Charts - Platform-wide Analytics */}
      <div className="
        grid gap-4
        grid-cols-1        // Móvil: charts apilados (mejor visualización)
        xl:grid-cols-3     // Desktop: 3 columnas
      ">
        <div className="xl:col-span-2 bg-white rounded-lg shadow p-4 md:p-6">
          <h3 className="text-base md:text-lg font-semibold mb-4">
            Tenant Growth Over Time
          </h3>
          <ResponsiveChart data={tenantGrowthData} height={250} />
        </div>

        <div className="bg-white rounded-lg shadow p-4 md:p-6">
          <h3 className="text-base md:text-lg font-semibold mb-4">
            Plans Distribution
          </h3>
          <ResponsivePieChart data={plansData} />
        </div>
      </div>

      {/* Recent Activity - Critical for mobile monitoring */}
      <div className="
        grid gap-4
        grid-cols-1        // Móvil: lista completa
        lg:grid-cols-2     // Desktop: 2 columnas
      ">
        <div className="bg-white rounded-lg shadow p-4 md:p-6">
          <h3 className="text-base md:text-lg font-semibold mb-4">
            Recent Tenants
          </h3>
          <RecentTenantsList />
        </div>

        <div className="bg-white rounded-lg shadow p-4 md:p-6">
          <h3 className="text-base md:text-lg font-semibold mb-4">
            Critical Alerts
          </h3>
          <AlertsList />
        </div>
      </div>
    </div>
  );
};
```

### 3. Admin Tables - Tenants List (Responsive)

**Desktop:** Tabla completa con todas las métricas
**Tablet:** Tabla con columnas esenciales
**Mobile:** Cards con información crítica + quick actions

```typescript
// components/TenantsTable.tsx
import { useMediaQuery } from '@/hooks/useMediaQuery';

export const TenantsTable = ({ tenants }) => {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 768px) and (max-width: 1024px)');

  if (isMobile) {
    // Vista móvil: Cards compactas con quick actions
    return (
      <div className="space-y-3">
        {tenants.map(tenant => (
          <div key={tenant.id} className="
            bg-white rounded-lg shadow p-4
            border-l-4
            {tenant.status === 'active' ? 'border-green-500' :
             tenant.status === 'suspended' ? 'border-red-500' :
             'border-gray-500'}
          ">
            {/* Header: Name + Status Badge */}
            <div className="flex justify-between items-start mb-2">
              <div>
                <h4 className="font-semibold text-slate-900">
                  {tenant.name}
                </h4>
                <p className="text-xs text-slate-500">
                  {tenant.slug}
                </p>
              </div>
              <span className={`
                px-2 py-1 rounded-full text-xs font-medium
                ${tenant.status === 'active' ? 'bg-green-100 text-green-700' :
                  tenant.status === 'suspended' ? 'bg-red-100 text-red-700' :
                  'bg-gray-100 text-gray-700'}
              `}>
                {tenant.status}
              </span>
            </div>

            {/* Metrics Grid */}
            <div className="grid grid-cols-3 gap-2 mb-3">
              <div className="text-center">
                <p className="text-xs text-slate-500">Users</p>
                <p className="text-sm font-semibold">{tenant.users}</p>
              </div>
              <div className="text-center">
                <p className="text-xs text-slate-500">Plan</p>
                <p className="text-sm font-semibold">{tenant.plan}</p>
              </div>
              <div className="text-center">
                <p className="text-xs text-slate-500">MRR</p>
                <p className="text-sm font-semibold">${tenant.mrr}</p>
              </div>
            </div>

            {/* Quick Actions */}
            <div className="flex gap-2">
              <button className="flex-1 px-3 py-1.5 text-xs font-medium text-blue-700 bg-blue-50 rounded-lg hover:bg-blue-100">
                View
              </button>
              <button className="flex-1 px-3 py-1.5 text-xs font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200">
                Edit
              </button>
              <button className="px-3 py-1.5 text-xs font-medium text-slate-700 bg-slate-100 rounded-lg hover:bg-slate-200">
                ⋮
              </button>
            </div>
          </div>
        ))}
      </div>
    );
  }

  // Vista desktop/tablet: Tabla con columnas adaptadas
  return (
    <div className="bg-white rounded-lg shadow overflow-hidden">
      <div className="overflow-x-auto">
        <table className="min-w-full divide-y divide-slate-200">
          <thead className="bg-slate-50">
            <tr>
              <th className="px-4 py-3 text-left text-xs font-medium text-slate-500 uppercase">
                Tenant
              </th>
              <th className="px-4 py-3 text-left text-xs font-medium text-slate-500 uppercase">
                Plan
              </th>
              {!isTablet && (
                <th className="px-4 py-3 text-left text-xs font-medium text-slate-500 uppercase">
                  Users
                </th>
              )}
              {!isTablet && (
                <th className="px-4 py-3 text-left text-xs font-medium text-slate-500 uppercase">
                  MRR
                </th>
              )}
              <th className="px-4 py-3 text-left text-xs font-medium text-slate-500 uppercase">
                Status
              </th>
              <th className="px-4 py-3 text-right text-xs font-medium text-slate-500 uppercase">
                Actions
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-slate-200">
            {tenants.map(tenant => (
              <tr key={tenant.id} className="hover:bg-slate-50">
                <td className="px-4 py-3">
                  <div>
                    <div className="text-sm font-medium text-slate-900">
                      {tenant.name}
                    </div>
                    <div className="text-xs text-slate-500">
                      {tenant.slug}
                    </div>
                  </div>
                </td>
                <td className="px-4 py-3 text-sm text-slate-900">
                  {tenant.plan}
                </td>
                {!isTablet && (
                  <td className="px-4 py-3 text-sm text-slate-900">
                    {tenant.users}
                  </td>
                )}
                {!isTablet && (
                  <td className="px-4 py-3 text-sm text-slate-900">
                    ${tenant.mrr}
                  </td>
                )}
                <td className="px-4 py-3">
                  <span className={`
                    px-2 py-1 rounded-full text-xs font-medium
                    ${tenant.status === 'active' ? 'bg-green-100 text-green-700' :
                      tenant.status === 'suspended' ? 'bg-red-100 text-red-700' :
                      'bg-gray-100 text-gray-700'}
                  `}>
                    {tenant.status}
                  </span>
                </td>
                <td className="px-4 py-3 text-right text-sm font-medium">
                  <button className="text-blue-600 hover:text-blue-900 mr-3">
                    View
                  </button>
                  <button className="text-slate-600 hover:text-slate-900">
                    Edit
                  </button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};
```

### 4. Admin Forms - Crear/Editar Tenant

```typescript
// components/TenantForm.tsx
export const TenantForm = ({ tenant, onSubmit }) => {
  return (
    <form className="
      max-w-full
      lg:max-w-3xl
      mx-auto
      bg-white rounded-lg shadow
      p-4 md:p-6
    ">
      <h2 className="text-xl md:text-2xl font-bold mb-6">
        {tenant ? 'Edit Tenant' : 'Create New Tenant'}
      </h2>

      {/* Grid responsive para formulario */}
      <div className="
        grid gap-4 md:gap-6
        grid-cols-1        // Móvil: 1 columna
        md:grid-cols-2     // Tablet+: 2 columnas
      ">
        {/* Tenant Name - Full width */}
        <div className="md:col-span-2">
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Tenant Name *
          </label>
          <input
            type="text"
            required
            className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            placeholder="Acme Corporation"
          />
        </div>

        {/* Slug */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Slug *
          </label>
          <input
            type="text"
            required
            className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            placeholder="acme-corp"
          />
        </div>

        {/* Plan */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Plan *
          </label>
          <select className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500">
            <option value="free">Free</option>
            <option value="plan1">Plan 1</option>
            <option value="plan2">Plan 2</option>
            <option value="plan3">Plan 3</option>
          </select>
        </div>

        {/* Admin Email */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Admin Email *
          </label>
          <input
            type="email"
            required
            className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            placeholder="admin@acme.com"
          />
        </div>

        {/* Phone */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Phone
          </label>
          <input
            type="tel"
            className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            placeholder="+1 234 567 8900"
          />
        </div>

        {/* Max Users */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Max Users
          </label>
          <input
            type="number"
            className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            placeholder="50"
          />
        </div>

        {/* Max Storage (GB) */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Max Storage (GB)
          </label>
          <input
            type="number"
            className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            placeholder="10"
          />
        </div>

        {/* Status */}
        <div className="md:col-span-2">
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Status
          </label>
          <div className="flex gap-4">
            <label className="flex items-center">
              <input type="radio" name="status" value="active" className="mr-2" />
              Active
            </label>
            <label className="flex items-center">
              <input type="radio" name="status" value="suspended" className="mr-2" />
              Suspended
            </label>
            <label className="flex items-center">
              <input type="radio" name="status" value="inactive" className="mr-2" />
              Inactive
            </label>
          </div>
        </div>

        {/* Notes - Full width */}
        <div className="md:col-span-2">
          <label className="block text-sm font-medium text-slate-700 mb-2">
            Internal Notes
          </label>
          <textarea
            rows={3}
            className="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-blue-500"
            placeholder="Internal notes about this tenant..."
          />
        </div>
      </div>

      {/* Action Buttons - Responsive */}
      <div className="
        mt-6 pt-6 border-t border-slate-200
        flex flex-col sm:flex-row
        gap-3
      ">
        <button
          type="submit"
          className="
            flex-1 sm:flex-none
            px-6 py-2.5
            bg-blue-600 text-white
            rounded-lg font-medium
            hover:bg-blue-700
            focus:ring-2 focus:ring-blue-500 focus:ring-offset-2
          "
        >
          {tenant ? 'Update Tenant' : 'Create Tenant'}
        </button>
        <button
          type="button"
          className="
            flex-1 sm:flex-none
            px-6 py-2.5
            bg-slate-200 text-slate-700
            rounded-lg font-medium
            hover:bg-slate-300
          "
        >
          Cancel
        </button>
        {tenant && (
          <button
            type="button"
            className="
              sm:ml-auto
              px-6 py-2.5
              bg-red-100 text-red-700
              rounded-lg font-medium
              hover:bg-red-200
            "
          >
            Delete Tenant
          </button>
        )}
      </div>
    </form>
  );
};
```

---

## 🪝 Custom Hook: useMediaQuery

```typescript
// hooks/useMediaQuery.ts
import { useState, useEffect } from 'react';

export const useMediaQuery = (query: string): boolean => {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);

    if (media.matches !== matches) {
      setMatches(media.matches);
    }

    const listener = () => setMatches(media.matches);
    media.addEventListener('change', listener);

    return () => media.removeEventListener('change', listener);
  }, [matches, query]);

  return matches;
};

// Presets útiles para Admin Panel
export const useBreakpoints = () => {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 768px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1024px)');
  const isWide = useMediaQuery('(min-width: 1536px)');

  return { isMobile, isTablet, isDesktop, isWide };
};
```

---

## 📊 Charts Responsive para Admin

```typescript
// components/ResponsiveChart.tsx
import { LineChart, Line, BarChart, Bar, ResponsiveContainer, XAxis, YAxis, Tooltip } from 'recharts';
import { useMediaQuery } from '@/hooks/useMediaQuery';

export const ResponsiveChart = ({ data, type = 'line', height }) => {
  const isMobile = useMediaQuery('(max-width: 768px)');

  const chartHeight = height || (isMobile ? 200 : 300);
  const strokeWidth = isMobile ? 2 : 3;
  const fontSize = isMobile ? 10 : 12;

  const ChartComponent = type === 'line' ? LineChart : BarChart;
  const DataComponent = type === 'line' ? Line : Bar;

  return (
    <ResponsiveContainer width="100%" height={chartHeight}>
      <ChartComponent data={data}>
        <XAxis
          dataKey="name"
          tick={{ fontSize }}
          angle={isMobile ? -45 : 0}
          textAnchor={isMobile ? 'end' : 'middle'}
          height={isMobile ? 60 : 30}
        />
        <YAxis tick={{ fontSize }} width={isMobile ? 40 : 60} />
        <Tooltip />
        <DataComponent
          type={type === 'line' ? 'monotone' : undefined}
          dataKey="value"
          stroke="#2563EB"
          fill="#2563EB"
          strokeWidth={strokeWidth}
        />
      </ChartComponent>
    </ResponsiveContainer>
  );
};
```

---

## 🎯 Testing Responsive para Admin Panel

### Breakpoints Críticos para Admin

1. **Mobile Small:** 375x667 (iPhone SE) - Supervisión básica
2. **Mobile Large:** 414x896 (iPhone 11 Pro Max) - Supervisión completa
3. **Tablet:** 768x1024 (iPad) - Dashboard ejecutivo
4. **Desktop:** 1366x768 (Laptop común) - Gestión completa
5. **Wide:** 1920x1080 (Full HD) - Múltiples dashboards

### Checklist de Testing Admin Panel

#### Funcionalidad Crítica (Mobile)
- [ ] Ver lista de tenants con métricas clave
- [ ] Ver dashboard con KPIs globales
- [ ] Recibir y ver alertas/notificaciones
- [ ] Cambiar status de tenant (active/suspended)
- [ ] Ver audit logs recientes
- [ ] Acceder a configuración de sistema

#### Layout (Todos los tamaños)
- [ ] Sidebar se convierte en hamburger en móvil
- [ ] Header sticky con quick actions visible
- [ ] KPI cards apilan correctamente (2 cols → 4 cols → 6 cols)
- [ ] Tablas se convierten en cards en móvil
- [ ] Forms tienen campos apilados en móvil
- [ ] Charts son legibles en todos los tamaños

#### Performance
- [ ] Tablas grandes usan virtualización en móvil
- [ ] Charts usan debounce en resize
- [ ] Lazy loading de secciones pesadas
- [ ] Service Worker para offline monitoring (crítico)

### Herramientas de Testing

```bash
# Chrome DevTools
F12 → Ctrl+Shift+M (Device Toolbar)
# Probar: iPhone SE, iPad, Desktop 1080p

# Lighthouse CI
npm run lighthouse:mobile
npm run lighthouse:desktop

# Real Device Testing
# iOS: Safari + Xcode Remote Debugging
# Android: Chrome + USB Debugging

# Accessibility Testing
npm run test:a11y
# Verificar contraste, touch targets, ARIA labels
```

---

## 🚨 Errores Comunes a Evitar en Admin Panel

### ❌ DON'T

```typescript
// ❌ Tablas sin vista móvil
<table>
  {/* 10 columnas → scroll horizontal horrible en móvil */}
</table>

// ❌ Alerts invisibles en móvil
<div className="hidden md:block">
  <AlertBanner critical />
</div>

// ❌ Charts sin height definido
<ResponsiveContainer width="100%">
  {/* Height indefinido → se colapsa en móvil */}
</ResponsiveContainer>

// ❌ Métricas sin priorización
<div className="grid-cols-6">
  {/* 6 KPIs en móvil → ilegible */}
</div>
```

### ✅ DO

```typescript
// ✅ Tablas con vista móvil alternativa
const isMobile = useMediaQuery('(max-width: 768px)');
return isMobile ? <CardView /> : <TableView />;

// ✅ Alerts siempre visibles
<div className="block">
  <AlertBanner critical responsive />
</div>

// ✅ Charts con height responsive
<ResponsiveContainer width="100%" height={isMobile ? 200 : 300}>

// ✅ Métricas priorizadas
<div className="grid grid-cols-2 lg:grid-cols-4 xl:grid-cols-6">
  {/* 2 en móvil, 4 en desktop, 6 en wide */}
</div>
```

---

## 📝 Checklist Final de Implementación

### Alta Prioridad (Supervisión Crítica)
- [ ] Dashboard con KPIs globales responsive
- [ ] Lista de tenants con filtros mobile-friendly
- [ ] Alerts/notifications sticky en header móvil
- [ ] Audit logs con vista móvil (cards)
- [ ] Quick actions accesibles desde móvil

### Media Prioridad
- [ ] Forms de crear/editar tenant responsive
- [ ] Charts con leyendas adaptadas a tamaño
- [ ] Configuración de planes responsive
- [ ] User management global con búsqueda móvil

### Baja Prioridad
- [ ] Advanced filters con modal en móvil
- [ ] Export data con opciones simplificadas móvil
- [ ] System settings con tabs responsive

---

## 🎓 Recursos Adicionales

- [Tailwind Responsive Design](https://tailwindcss.com/docs/responsive-design)
- [Recharts Responsive Container](https://recharts.org/en-US/api/ResponsiveContainer)
- [Admin Dashboard Best Practices (Smashing Magazine)](https://www.smashingmagazine.com/2015/04/designing-the-perfect-dashboard/)

---

**Última actualización:** 2025-01-04
**Versión:** 1.0.0
**Adaptado para:** Admin Panel (Platform Management)
