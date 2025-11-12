# Architecture Decisions - bMOI Landing Page

This document explains the **architectural decisions** made for the bMOI Landing Page and the reasoning behind them.

## Table of Contents

1. [Framework Selection](#framework-selection)
2. [Styling Strategy](#styling-strategy)
3. [Responsive Design Architecture](#responsive-design-architecture)
4. [Dark Mode Architecture](#dark-mode-architecture)
5. [Content Management](#content-management)
6. [Performance Strategy](#performance-strategy)
7. [SEO Architecture](#seo-architecture)
8. [Deployment Strategy](#deployment-strategy)

---

## Framework Selection

### Decision: Astro 4

**Chosen:** Astro 4
**Alternatives Considered:** Next.js (Static Export), Nuxt 3 (Static), Gatsby, 11ty

### Comparison Matrix

| Framework | Performance | DX | Ecosystem | Learning Curve | Verdict |
|-----------|-------------|-----|-----------|----------------|---------|
| **Astro 4** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ **Selected** |
| Next.js | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ❌ Too heavy |
| Nuxt 3 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ❌ Vue-specific |
| Gatsby | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ❌ Slow builds |
| 11ty | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ❌ Less features |

### Why Astro 4?

#### ✅ Advantages

1. **Zero JavaScript by Default**
   ```astro
   ---
   // This component ships ZERO JavaScript
   ---
   <header class="site-header">
     <nav>...</nav>
   </header>
   ```

2. **Islands Architecture**
   ```astro
   ---
   import StaticComponent from './Static.astro';
   import InteractiveForm from './Form.tsx';
   ---

   <StaticComponent />  <!-- 0 KB JavaScript -->
   <InteractiveForm client:load />  <!-- Only this hydrates -->
   ```

3. **Framework Agnostic**
   - Use React for forms
   - Use Vue for charts
   - Use Svelte for animations
   - All in the same project!

4. **Built-in Optimizations**
   - Automatic image optimization
   - CSS bundling and minification
   - Scoped CSS support
   - Built-in TypeScript

5. **Perfect for Content Sites**
   - Content Collections with type safety
   - MDX support out of the box
   - Fast builds (incremental)
   - Excellent SEO

#### ❌ Trade-offs Accepted

- Smaller community than Next.js
- Fewer third-party integrations
- Less suitable for highly interactive apps (not our use case)

### Bundle Size Comparison

```
Landing Page (Home):
┌─────────────┬──────────────┬──────────┐
│ Framework   │ Initial Load │ Total JS │
├─────────────┼──────────────┼──────────┤
│ Astro       │ 15 KB        │ 25 KB    │
│ Next.js     │ 85 KB        │ 120 KB   │
│ Nuxt 3      │ 70 KB        │ 110 KB   │
│ Gatsby      │ 95 KB        │ 150 KB   │
└─────────────┴──────────────┴──────────┘
```

**Result:** Astro is **6x smaller** than Next.js!

---

## Styling Strategy

### Decision: Tailwind CSS 4

**Chosen:** Tailwind CSS 4
**Alternatives Considered:** CSS Modules, Styled Components, Vanilla CSS

### Why Tailwind CSS?

#### ✅ Advantages

1. **Utility-First Approach**
   ```html
   <!-- No CSS file needed -->
   <button class="bg-blue-600 text-white px-6 py-3 rounded-lg hover:bg-blue-700">
     Click me
   </button>
   ```

2. **No Unused CSS**
   - Tailwind purges unused classes
   - Production bundle: ~10-30 KB (vs 200+ KB without purge)

3. **Responsive Design Built-in**
   ```html
   <!-- Mobile-first responsive -->
   <div class="text-sm md:text-base lg:text-lg">
     Responsive text
   </div>
   ```

4. **Dark Mode Support**
   ```html
   <!-- Easy dark mode variants -->
   <div class="bg-white dark:bg-slate-900">
     Content
   </div>
   ```

5. **Customizable Design System**
   ```javascript
   // tailwind.config.mjs
   theme: {
     extend: {
       colors: {
         brand: {
           600: '#2563eb',  // Primary color
         },
       },
     },
   }
   ```

#### ❌ Trade-offs Accepted

- HTML can look verbose
- Learning curve for class names
- Need to configure purging carefully

### CSS Architecture

```
Global Styles (global.css)
├── @tailwind base        → Reset + base styles
├── @tailwind components  → Reusable component classes
├── @tailwind utilities   → Utility classes
└── Custom CSS            → Specific overrides
```

---

## Responsive Design Architecture

### Mobile-First Strategy

**Philosophy:** Design for mobile first, then enhance for larger screens.

```css
/* ✅ Mobile-first (recommended) */
.card {
  padding: 1rem;  /* Mobile default */
}

@media (min-width: 768px) {
  .card {
    padding: 2rem;  /* Tablet+ */
  }
}

/* ❌ Desktop-first (NOT recommended) */
.card {
  padding: 2rem;  /* Desktop default */
}

@media (max-width: 767px) {
  .card {
    padding: 1rem;  /* Mobile override */
  }
}
```

**Why mobile-first?**
- 📱 **Mobile traffic dominates** (60%+ of web traffic)
- 🚀 **Better performance** (load mobile styles first)
- 🎯 **Forces prioritization** (what's truly important?)

### Breakpoint System

```javascript
// tailwind.config.mjs
theme: {
  screens: {
    'sm': '640px',   // @media (min-width: 640px)
    'md': '768px',   // @media (min-width: 768px)
    'lg': '1024px',  // @media (min-width: 1024px)
    'xl': '1280px',  // @media (min-width: 1280px)
    '2xl': '1536px', // @media (min-width: 1536px)
  }
}
```

### Responsive Patterns

#### 1. **Grid System**

```html
<!-- Mobile: 1 column, Tablet: 2 columns, Desktop: 3 columns -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>
```

#### 2. **Typography Scale**

```html
<!-- Responsive font sizes -->
<h1 class="text-3xl md:text-4xl lg:text-5xl xl:text-6xl">
  Responsive Heading
</h1>

<p class="text-base md:text-lg lg:text-xl">
  Responsive paragraph
</p>
```

#### 3. **Spacing System**

```html
<!-- Responsive padding -->
<section class="py-12 md:py-16 lg:py-24">
  <!-- Responsive margin -->
  <div class="mb-8 md:mb-12 lg:mb-16">
    Content
  </div>
</section>
```

#### 4. **Container Strategy**

```html
<!-- Responsive container -->
<div class="container mx-auto px-4 sm:px-6 lg:px-8">
  <!-- Max width responsive -->
  <div class="max-w-7xl mx-auto">
    Content
  </div>
</div>
```

#### 5. **Navigation Patterns**

```html
<!-- Desktop: Horizontal menu, Mobile: Hamburger -->
<nav>
  <!-- Mobile hamburger (hidden on desktop) -->
  <button class="md:hidden">
    <MenuIcon />
  </button>

  <!-- Desktop menu (hidden on mobile) -->
  <ul class="hidden md:flex space-x-6">
    <li>Home</li>
    <li>Features</li>
    <li>Pricing</li>
  </ul>
</nav>
```

### Responsive Images

```astro
---
import { Image } from 'astro:assets';
import heroMobile from '@/assets/hero-mobile.jpg';
import heroDesktop from '@/assets/hero-desktop.jpg';
---

<!-- Different images for different breakpoints -->
<picture>
  <source media="(min-width: 768px)" srcset={heroDesktop.src} />
  <img src={heroMobile.src} alt="Hero" />
</picture>

<!-- Or use Astro Image component with sizes -->
<Image
  src={heroDesktop}
  alt="Hero"
  widths={[320, 640, 1024, 1280]}
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1280px"
/>
```

### Testing Responsive Design

**Breakpoints to test:**
- 📱 **Mobile**: 320px, 375px, 414px
- 📱 **Tablet**: 768px, 1024px
- 💻 **Desktop**: 1280px, 1440px, 1920px

**Tools:**
- Chrome DevTools (Device Toolbar)
- Firefox Responsive Design Mode
- BrowserStack (real devices)
- LambdaTest (cross-browser)

---

## Dark Mode Architecture

### Strategy: Class-Based with System Preference

**Approach:** Use `class` strategy with automatic system preference detection

```javascript
// tailwind.config.mjs
export default {
  darkMode: 'class',  // Use .dark class on <html>
  // ...
}
```

### Why Class-Based?

✅ **User control** - Can override system preference
✅ **No flash** - Can set class before render
✅ **Persistence** - Store user preference in localStorage
✅ **Smooth transitions** - Easier to animate

### Implementation Flow

```
1. Page Load
   ↓
2. Check localStorage ('theme')
   ↓
3. If no preference, check system preference
   ↓
4. Apply .dark class if needed
   ↓
5. User clicks toggle
   ↓
6. Toggle .dark class
   ↓
7. Save to localStorage
```

### Theme Detection Script

```html
<!-- In <head>, before any content renders -->
<script is:inline>
  // MUST be inline to prevent flash
  const theme = localStorage.getItem('theme');
  const systemDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

  if (theme === 'dark' || (!theme && systemDark)) {
    document.documentElement.classList.add('dark');
  } else {
    document.documentElement.classList.remove('dark');
  }
</script>
```

**Why inline script?**
- ⚡ Executes before page renders (no flash)
- 🚫 Blocks render until theme is set
- ✅ Only ~200 bytes

### CSS Variables for Theming

```css
/* global.css */
@layer base {
  :root {
    /* Light theme (default) */
    --color-background: 255 255 255;
    --color-foreground: 15 23 42;
    --color-primary: 37 99 235;
    --color-border: 226 232 240;
  }

  .dark {
    /* Dark theme */
    --color-background: 15 23 42;
    --color-foreground: 241 245 249;
    --color-primary: 96 165 250;
    --color-border: 51 65 85;
  }

  body {
    background-color: rgb(var(--color-background));
    color: rgb(var(--color-foreground));
  }
}
```

### Tailwind Dark Variants

```html
<!-- Background colors -->
<div class="bg-white dark:bg-slate-900">
  Content
</div>

<!-- Text colors -->
<p class="text-slate-900 dark:text-slate-50">
  Text
</p>

<!-- Borders -->
<div class="border border-slate-300 dark:border-slate-700">
  Content
</div>

<!-- Hover states -->
<button class="bg-blue-600 hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-600">
  Click me
</button>
```

### Theme Toggle Component

```typescript
// ThemeToggle.tsx (React island)
import { useEffect, useState } from 'react';

export default function ThemeToggle() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  useEffect(() => {
    const isDark = document.documentElement.classList.contains('dark');
    setTheme(isDark ? 'dark' : 'light');
  }, []);

  const toggleTheme = () => {
    const newTheme = theme === 'light' ? 'dark' : 'light';

    if (newTheme === 'dark') {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }

    localStorage.setItem('theme', newTheme);
    setTheme(newTheme);
  };

  return (
    <button
      onClick={toggleTheme}
      className="rounded-lg p-2 hover:bg-slate-100 dark:hover:bg-slate-800"
      aria-label={`Switch to ${theme === 'light' ? 'dark' : 'light'} theme`}
    >
      {theme === 'light' ? <MoonIcon /> : <SunIcon />}
    </button>
  );
}
```

### Color Palette Guidelines

**Light Theme:**
- Background: White (#FFFFFF)
- Surface: Light Gray (#F8FAFC)
- Primary: Blue 600 (#2563EB)
- Text Primary: Slate 900 (#0F172A)
- Text Secondary: Slate 600 (#475569)

**Dark Theme:**
- Background: Slate 900 (#0F172A)
- Surface: Slate 800 (#1E293B)
- Primary: Blue 400 (#60A5FA) - Lighter for contrast
- Text Primary: Slate 50 (#F8FAFC)
- Text Secondary: Slate 400 (#94A3B8)

### Contrast Ratios (WCAG 2.1 AA)

**Minimum requirements:**
- Normal text: 4.5:1
- Large text (18pt+): 3:1
- UI components: 3:1

**Testing tools:**
- Chrome DevTools (Lighthouse)
- WebAIM Contrast Checker
- Colorable.jxnblk.com

---

## Content Management

### Decision: File-Based with Content Collections

**Chosen:** Astro Content Collections
**Alternatives Considered:** Headless CMS (Contentful, Strapi), Git-based CMS (Netlify CMS)

### Why File-Based?

#### ✅ Advantages

1. **Version Control**
   - All content in Git
   - Full history
   - Easy rollbacks

2. **Type Safety**
   ```typescript
   // content/config.ts
   const blogCollection = defineCollection({
     schema: z.object({
       title: z.string(),
       publishedAt: z.date(),
       author: z.string(),
     }),
   });
   ```

3. **No Database Required**
   - Simpler deployment
   - No DB costs
   - Faster builds

4. **Developer-Friendly**
   - Write in Markdown/MDX
   - Use VSCode
   - Syntax highlighting

#### ❌ Trade-offs Accepted

- Non-technical users can't edit easily
- No GUI for content management
- Requires rebuild for content updates

### Content Structure

```
src/content/
├── config.ts           # Schema definitions
├── features/           # Feature descriptions
│   ├── multi-tenant.md
│   ├── security.md
│   └── analytics.md
├── blog/               # Blog posts
│   ├── post-1.mdx
│   └── post-2.mdx
└── legal/              # Legal pages
    ├── privacy.md
    └── terms.md
```

### Future: Add Headless CMS?

**When to consider:**
- Marketing team needs GUI
- Frequent content updates
- Non-technical editors

**Recommended:** Contentful, Strapi, or Sanity

---

## Performance Strategy

### Goal: Lighthouse Score 95+

**Target metrics:**
- Performance: 95+
- Accessibility: 100
- Best Practices: 100
- SEO: 100

### Optimization Techniques

#### 1. **Static Generation**
- Pre-render all pages at build time
- No server-side rendering overhead

#### 2. **Image Optimization**
```astro
---
import { Image } from 'astro:assets';
---

<Image
  src={heroImage}
  alt="Hero"
  width={1200}
  height={600}
  format="webp"  <!-- Modern format -->
  loading="lazy"  <!-- Lazy load -->
/>
```

#### 3. **CSS Optimization**
- Tailwind purges unused CSS
- Scoped CSS per component
- Critical CSS inlined

#### 4. **JavaScript Optimization**
- Zero JS by default
- Only hydrate interactive islands
- Code splitting automatic

#### 5. **Font Optimization**
```html
<!-- Preload critical fonts -->
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>

<!-- Use font-display: swap -->
<style>
  @font-face {
    font-family: 'Inter';
    font-display: swap;
    src: url('/fonts/inter.woff2') format('woff2');
  }
</style>
```

---

## SEO Architecture

### On-Page SEO

**Every page must have:**
- Unique title (50-60 chars)
- Meta description (120-160 chars)
- H1 heading (one per page)
- Open Graph tags
- Twitter Card tags
- Canonical URL

### Technical SEO

```astro
---
// BaseLayout.astro
const canonicalUrl = new URL(Astro.url.pathname, Astro.site);
---

<head>
  <!-- Canonical URL -->
  <link rel="canonical" href={canonicalUrl} />

  <!-- Open Graph -->
  <meta property="og:url" content={canonicalUrl} />
  <meta property="og:type" content="website" />

  <!-- Sitemap -->
  <link rel="sitemap" href="/sitemap-index.xml" />
</head>
```

---

## Deployment Strategy

### Decision: S3 + CloudFront

**Chosen:** AWS S3 + CloudFront
**Alternatives:** Netlify, Vercel, Cloudflare Pages

### Why S3 + CloudFront?

✅ **Cost-effective** ($0.50-2/month for most sites)
✅ **Scalable** (handle millions of requests)
✅ **Global CDN** (edge locations worldwide)
✅ **Custom domain** (easy setup)
✅ **HTTPS** (free with ACM)

### Build & Deploy Process

```bash
# 1. Build static site
npm run build
# Output: dist/ folder

# 2. Upload to S3
aws s3 sync dist/ s3://bmoi-landing --delete

# 3. Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890 \
  --paths "/*"
```

---

## Summary of Key Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| **Framework** | Astro 4 | Zero JS, Islands, Performance |
| **Styling** | Tailwind CSS | Utility-first, Responsive, Dark mode |
| **Responsive** | Mobile-first | Better UX, Performance |
| **Dark Mode** | Class-based | User control, No flash |
| **Content** | File-based | Version control, Type safety |
| **Deployment** | S3 + CloudFront | Cost-effective, Scalable |

---

**Architecture optimized for:**
- ⚡ Maximum performance
- 📱 Perfect mobile experience
- 🌓 Seamless dark mode
- 🔍 Excellent SEO
- 💰 Low cost

