# Responsive Design Guide - bMOI CPanel Multi-Tenant

**Objetivo:** Asegurar que el panel de control sea completamente funcional en todos los dispositivos (desktop, tablet, mobile).

---

## 📱 Filosofía: Mobile-First

El panel de control debe ser **responsive** y seguir un enfoque **mobile-first**:

```
Mobile (320px+) → Tablet (768px+) → Desktop (1024px+) → Wide (1440px+)
```

### ¿Por qué Mobile-First?

- ✅ **Acceso remoto** - Administradores revisan métricas desde móvil
- ✅ **Mejor performance** - Carga estilos móviles primero (más rápido)
- ✅ **Priorización** - Fuerza a decidir qué es realmente importante
- ✅ **Usuarios modernos** - 40%+ de usuarios B2B usan tablet/móvil

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
      '2xl': '1536px', // Pantallas grandes
    },
  },
}
```

### Uso en Componentes

```typescript
// ✅ Mobile-first: Estilos base para móvil, luego agregar para desktop
<div className="
  w-full              // Móvil: ancho completo
  px-4                // Móvil: padding horizontal 1rem
  md:w-1/2            // Tablet: 50% de ancho
  md:px-6             // Tablet: padding 1.5rem
  lg:w-1/3            // Desktop: 33% de ancho
  lg:px-8             // Desktop: padding 2rem
">
  Contenido
</div>
```

---

## 🏗️ Layouts Responsive

### 1. Navigation / Sidebar

**Desktop:** Sidebar fijo a la izquierda
**Mobile:** Hamburger menu con overlay

```typescript
// components/Layout/MainLayout.tsx
import { useState } from 'react';

export const MainLayout = ({ children }) => {
  const [sidebarOpen, setSidebarOpen] = useState(false);

  return (
    <div className="flex h-screen overflow-hidden">
      {/* Sidebar Desktop */}
      <aside className="
        hidden             // Oculto en móvil
        lg:flex            // Visible en desktop
        lg:w-64            // 256px de ancho
        lg:flex-col
        bg-slate-900
        text-white
      ">
        <nav className="flex-1 px-4 py-6">
          {/* Menú items */}
        </nav>
      </aside>

      {/* Sidebar Mobile (Overlay) */}
      {sidebarOpen && (
        <div className="
          fixed inset-0 z-50
          lg:hidden          // Solo móvil
        ">
          {/* Overlay oscuro */}
          <div
            className="fixed inset-0 bg-black/50"
            onClick={() => setSidebarOpen(false)}
          />

          {/* Sidebar con animación */}
          <aside className="
            fixed left-0 top-0 bottom-0
            w-64 bg-slate-900
            transform transition-transform
          ">
            <nav className="p-4">
              {/* Menú items */}
            </nav>
          </aside>
        </div>
      )}

      {/* Main Content */}
      <main className="
        flex-1              // Toma el espacio restante
        overflow-y-auto     // Scroll vertical
        bg-slate-50
      ">
        {/* Header Mobile */}
        <header className="
          sticky top-0 z-40
          lg:hidden          // Solo móvil
          bg-white shadow
          p-4
        ">
          <button
            onClick={() => setSidebarOpen(true)}
            className="text-slate-700"
          >
            <MenuIcon />
          </button>
        </header>

        {/* Page Content */}
        <div className="
          p-4                // Móvil: padding 1rem
          md:p-6             // Tablet: padding 1.5rem
          lg:p-8             // Desktop: padding 2rem
        ">
          {children}
        </div>
      </main>
    </div>
  );
};
```

### 2. Dashboard Grids

```typescript
// pages/Dashboard.tsx
export const Dashboard = () => {
  return (
    <div className="grid gap-4 md:gap-6">
      {/* KPI Cards - Responsive Grid */}
      <div className="
        grid gap-4
        grid-cols-1        // Móvil: 1 columna
        sm:grid-cols-2     // Móvil grande: 2 columnas
        lg:grid-cols-4     // Desktop: 4 columnas
      ">
        <KPICard title="Total Users" value={1234} />
        <KPICard title="Revenue" value="$45,678" />
        <KPICard title="Orders" value={567} />
        <KPICard title="Products" value={89} />
      </div>

      {/* Charts - Responsive Layout */}
      <div className="
        grid gap-4
        grid-cols-1        // Móvil: 1 columna (charts apilados)
        lg:grid-cols-2     // Desktop: 2 columnas
      ">
        <div className="bg-white rounded-lg shadow p-6">
          <h3 className="text-lg font-semibold mb-4">Sales Over Time</h3>
          <ResponsiveChart data={salesData} />
        </div>

        <div className="bg-white rounded-lg shadow p-6">
          <h3 className="text-lg font-semibold mb-4">Top Products</h3>
          <ResponsiveChart data={productsData} />
        </div>
      </div>
    </div>
  );
};
```

### 3. Data Tables

**Desktop:** Tabla completa con todas las columnas
**Mobile:** Cards con información esencial

```typescript
// components/ProductsTable.tsx
import { useMediaQuery } from '@/hooks/useMediaQuery';

export const ProductsTable = ({ products }) => {
  const isMobile = useMediaQuery('(max-width: 768px)');

  if (isMobile) {
    // Vista móvil: Cards
    return (
      <div className="space-y-4">
        {products.map(product => (
          <div key={product.id} className="
            bg-white rounded-lg shadow p-4
            border-l-4 border-blue-500
          ">
            <div className="flex justify-between items-start">
              <div>
                <h4 className="font-semibold text-slate-900">
                  {product.name}
                </h4>
                <p className="text-sm text-slate-500">
                  SKU: {product.sku}
                </p>
              </div>
              <span className="text-lg font-bold text-slate-900">
                ${product.price}
              </span>
            </div>
            <div className="mt-2 flex gap-2">
              <button className="btn-sm btn-primary">Edit</button>
              <button className="btn-sm btn-danger">Delete</button>
            </div>
          </div>
        ))}
      </div>
    );
  }

  // Vista desktop: Tabla completa
  return (
    <div className="bg-white rounded-lg shadow overflow-hidden">
      <table className="min-w-full divide-y divide-slate-200">
        <thead className="bg-slate-50">
          <tr>
            <th className="px-6 py-3 text-left text-xs font-medium text-slate-500 uppercase">
              Product
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-slate-500 uppercase">
              SKU
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-slate-500 uppercase">
              Price
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-slate-500 uppercase">
              Stock
            </th>
            <th className="px-6 py-3 text-left text-xs font-medium text-slate-500 uppercase">
              Actions
            </th>
          </tr>
        </thead>
        <tbody className="bg-white divide-y divide-slate-200">
          {products.map(product => (
            <tr key={product.id}>
              <td className="px-6 py-4 whitespace-nowrap">
                {product.name}
              </td>
              <td className="px-6 py-4 whitespace-nowrap">
                {product.sku}
              </td>
              <td className="px-6 py-4 whitespace-nowrap">
                ${product.price}
              </td>
              <td className="px-6 py-4 whitespace-nowrap">
                {product.stock}
              </td>
              <td className="px-6 py-4 whitespace-nowrap">
                <button className="btn-sm btn-primary mr-2">Edit</button>
                <button className="btn-sm btn-danger">Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
};
```

### 4. Forms

```typescript
// components/ProductForm.tsx
export const ProductForm = () => {
  return (
    <form className="
      max-w-full          // Móvil: ancho completo
      lg:max-w-2xl        // Desktop: máximo 672px
      mx-auto
    ">
      <div className="
        grid gap-4
        grid-cols-1        // Móvil: 1 columna
        md:grid-cols-2     // Tablet+: 2 columnas
      ">
        {/* Nombre - Ocupa 2 columnas en desktop */}
        <div className="md:col-span-2">
          <label className="block text-sm font-medium text-slate-700 mb-1">
            Product Name
          </label>
          <input
            type="text"
            className="w-full px-3 py-2 border rounded-lg"
          />
        </div>

        {/* SKU */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">
            SKU
          </label>
          <input
            type="text"
            className="w-full px-3 py-2 border rounded-lg"
          />
        </div>

        {/* Price */}
        <div>
          <label className="block text-sm font-medium text-slate-700 mb-1">
            Price
          </label>
          <input
            type="number"
            className="w-full px-3 py-2 border rounded-lg"
          />
        </div>

        {/* Description - Ocupa 2 columnas */}
        <div className="md:col-span-2">
          <label className="block text-sm font-medium text-slate-700 mb-1">
            Description
          </label>
          <textarea
            rows={4}
            className="w-full px-3 py-2 border rounded-lg"
          />
        </div>
      </div>

      {/* Buttons - Stack en móvil, inline en desktop */}
      <div className="
        mt-6
        flex flex-col         // Móvil: columna (apilados)
        sm:flex-row           // Desktop: fila (lado a lado)
        gap-3
      ">
        <button type="submit" className="
          btn btn-primary
          w-full sm:w-auto    // Móvil: ancho completo, Desktop: auto
        ">
          Save Product
        </button>
        <button type="button" className="
          btn btn-secondary
          w-full sm:w-auto
        ">
          Cancel
        </button>
      </div>
    </form>
  );
};
```

---

## 🎨 Componentes Responsive

### Typography Responsive

```typescript
// Headings que escalan con viewport
<h1 className="
  text-2xl           // Móvil: 1.5rem
  md:text-3xl        // Tablet: 1.875rem
  lg:text-4xl        // Desktop: 2.25rem
  font-bold
">
  Dashboard Title
</h1>

<p className="
  text-sm            // Móvil: 0.875rem
  md:text-base       // Tablet+: 1rem
  text-slate-600
">
  Description text
</p>
```

### Containers Responsive

```typescript
// Container con padding responsive
<div className="
  container          // Centra y limita ancho
  mx-auto
  px-4               // Móvil: 1rem padding
  sm:px-6            // Móvil grande: 1.5rem
  lg:px-8            // Desktop: 2rem
">
  Content
</div>
```

### Spacing Responsive

```typescript
// Espaciado que crece con viewport
<div className="
  space-y-4          // Móvil: 1rem entre elementos
  md:space-y-6       // Tablet: 1.5rem
  lg:space-y-8       // Desktop: 2rem
">
  <ComponentA />
  <ComponentB />
</div>
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

// Uso:
const isMobile = useMediaQuery('(max-width: 768px)');
const isTablet = useMediaQuery('(min-width: 768px) and (max-width: 1024px)');
const isDesktop = useMediaQuery('(min-width: 1024px)');
```

---

## 📊 Charts Responsive

```typescript
// components/ResponsiveChart.tsx
import { LineChart, Line, ResponsiveContainer } from 'recharts';
import { useMediaQuery } from '@/hooks/useMediaQuery';

export const ResponsiveChart = ({ data }) => {
  const isMobile = useMediaQuery('(max-width: 768px)');

  return (
    <ResponsiveContainer width="100%" height={isMobile ? 200 : 300}>
      <LineChart data={data}>
        <Line
          type="monotone"
          dataKey="value"
          stroke="#2563EB"
          strokeWidth={isMobile ? 2 : 3}  // Línea más delgada en móvil
        />
        {/* Mostrar menos labels en móvil */}
        <XAxis
          dataKey="name"
          tick={{ fontSize: isMobile ? 10 : 12 }}
        />
      </LineChart>
    </ResponsiveContainer>
  );
};
```

---

## 🎯 Testing Responsive

### Breakpoints a Testear

1. **Mobile Small:** 375x667 (iPhone SE)
2. **Mobile Large:** 414x896 (iPhone 11 Pro Max)
3. **Tablet:** 768x1024 (iPad)
4. **Desktop:** 1366x768 (Laptop común)
5. **Desktop Large:** 1920x1080 (Full HD)

### Checklist de Testing

- [ ] Sidebar se convierte en hamburger menu en móvil
- [ ] Grids de dashboard apilan correctamente
- [ ] Tablas se convierten en cards en móvil
- [ ] Formularios tienen campos apilados en móvil
- [ ] Botones ocupan ancho completo en móvil
- [ ] Typography es legible en todos los tamaños
- [ ] Charts se ajustan al ancho disponible
- [ ] No hay overflow horizontal en ningún dispositivo
- [ ] Touch targets son >= 44x44px en móvil
- [ ] Spacing es apropiado en cada breakpoint

### Herramientas de Testing

```bash
# Chrome DevTools
# - Abrir DevTools (F12)
# - Toggle Device Toolbar (Ctrl+Shift+M)
# - Probar diferentes dispositivos

# Firefox Responsive Design Mode
# - Ctrl+Shift+M
# - Probar diferentes resoluciones

# Testing real
# - iOS: Safari + Remote debugging
# - Android: Chrome + Remote debugging
```

---

## 🚨 Errores Comunes a Evitar

### ❌ DON'T

```typescript
// ❌ Ancho fijo en píxeles
<div style={{ width: '1200px' }}>
  // Se rompe en móvil
</div>

// ❌ Usar solo desktop en media queries
<div className="hidden xl:block">
  // Invisible en tablet y móvil
</div>

// ❌ Texto muy pequeño en móvil
<p className="text-xs">
  // Ilegible en móvil (< 16px)
</p>

// ❌ Touch targets pequeños
<button className="w-6 h-6">
  // Muy pequeño para tocar en móvil (debe ser >=44px)
</button>

// ❌ Overflow horizontal
<div className="flex">
  <div className="w-96">A</div>
  <div className="w-96">B</div>
  <div className="w-96">C</div>
  // Se sale del viewport en móvil
</div>
```

### ✅ DO

```typescript
// ✅ Ancho relativo
<div className="w-full max-w-7xl mx-auto">
  // Se adapta al viewport
</div>

// ✅ Mobile-first approach
<div className="block lg:hidden">
  // Visible en móvil, oculto en desktop
</div>

// ✅ Texto legible
<p className="text-sm md:text-base">
  // 14px móvil, 16px desktop
</p>

// ✅ Touch targets apropiados
<button className="w-12 h-12 md:w-10 md:h-10">
  // 48px móvil, 40px desktop
</button>

// ✅ Grid responsive
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  // 1 col móvil, 2 tablet, 3 desktop
</div>
```

---

## 📝 Checklist de Implementación

### Por Componente

- [ ] Layout funciona en móvil, tablet y desktop
- [ ] Sidebar/Navigation responsive (hamburger en móvil)
- [ ] Grids/Cards usan `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`
- [ ] Tablas se convierten en cards en móvil
- [ ] Formularios apilan campos en móvil
- [ ] Botones son `w-full sm:w-auto`
- [ ] Typography escala: `text-sm md:text-base lg:text-lg`
- [ ] Spacing responsive: `space-y-4 md:space-y-6 lg:space-y-8`
- [ ] Charts usan `ResponsiveContainer`
- [ ] Imágenes usan `max-w-full h-auto`

### Global

- [ ] Breakpoints de Tailwind configurados
- [ ] `useMediaQuery` hook implementado
- [ ] CSS custom para casos especiales minimizado
- [ ] Testing en 5 tamaños de viewport
- [ ] No overflow horizontal en ningún tamaño
- [ ] Touch targets >= 44x44px
- [ ] Performance optimizada (lazy loading de componentes grandes)

---

## 🎓 Recursos

- [Tailwind Responsive Design](https://tailwindcss.com/docs/responsive-design)
- [MDN: Responsive Images](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images)
- [Google Web Fundamentals: Responsive Web Design](https://developers.google.com/web/fundamentals/design-and-ux/responsive)

---

**Última actualización:** 2025-01-04
**Versión:** 1.0.0
