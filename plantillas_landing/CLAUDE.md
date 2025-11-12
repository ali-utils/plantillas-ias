# CLAUDE.md - Landing Page

This file provides guidance to Claude Code (claude.ai/code) when working on the **bMOI Landing Page**.

## Project Overview

**bMOI Landing Page** is a static marketing website built with Astro 4 that showcases the bMOI platform, attracts visitors, and converts them to users.

**Framework:** Astro 4 + TypeScript 5 + Tailwind CSS
**Deployment:** AWS S3 + CloudFront (static hosting)
**Type:** Static Site Generation (SSG) - NOT a web application

**Current Status:** Template/Plantilla (Starting from Zero - 0% complete)

## 📚 Critical Documentation

Read these files in order when starting:

1. **`.claude/context.md`** - Complete project context (MUST READ FIRST)
2. **`.claude/conventions.md`** - Mandatory naming conventions
3. **`.claude/session-state.md`** - Current development status
4. **This file (CLAUDE.md)** - Quick reference
5. **`.docs/SETUP.md`** - Step-by-step setup guide
6. **`.docs/ARQUITECTURA.md`** - Architecture decisions

## 🎯 Key Points

### This Project IS
- ✅ Static marketing website (SSG)
- ✅ Astro 4 framework
- ✅ Markdown/MDX for content
- ✅ **Fully responsive** (mobile-first design)
- ✅ **Light/Dark theme support** (with system preference detection)
- ✅ Deployed to S3 + CloudFront
- ✅ Zero JavaScript by default
- ✅ Performance-critical (95+ Lighthouse)
- ✅ SEO-critical (meta tags, schema markup)

### This Project is NOT
- ❌ A web application with authentication
- ❌ Server-side rendered (no SSR)
- ❌ A React SPA
- ❌ Part of the multi-tenant system
- ❌ A dashboard or control panel

## 🛠️ Tech Stack

- **Framework:** Astro 4
- **Language:** TypeScript 5
- **Styling:** Tailwind CSS 4
- **Content:** Markdown/MDX with Content Collections
- **Package Manager:** npm
- **Node:** 20.x

## 📝 Naming Conventions (MANDATORY)

### Files
- Components: `PascalCase.astro` (e.g., `Header.astro`, `Hero.astro`)
- Pages: `kebab-case.astro` (e.g., `index.astro`, `pricing.astro`)
- Content: `kebab-case.md` (e.g., `getting-started.md`)
- Utilities: `kebab-case.ts` (e.g., `format-date.ts`)

### Code
- Variables/Functions: `camelCase` (e.g., `const apiUrl`, `function formatDate()`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `const API_URL`, `const MAX_RETRIES`)
- Classes/Types: `PascalCase` (e.g., `interface BlogPost`, `class ApiClient`)
- **NEVER use snake_case** in TypeScript

## 🏗️ Development Commands

```bash
# Local development
npm install              # Install dependencies
npm run dev              # Start dev server (http://localhost:4321)
npm run build            # Build for production
npm run preview          # Preview production build

# Docker (recommended)
docker compose up -d     # Start development environment
docker compose logs -f   # View logs
docker compose down      # Stop environment

# Code quality
npm run astro check      # Type check
npm run format           # Format with Prettier
```

## 🎨 Code Patterns

### Astro Component

```astro
---
import Header from '@/components/Header.astro';
import type { Props } from '@/types';

interface Props {
  title: string;
  description?: string;
}

const { title, description } = Astro.props;
---

<div class="container mx-auto px-4 py-12">
  <h1 class="text-4xl font-bold">{title}</h1>
  {description && <p class="text-lg mt-4">{description}</p>}
</div>
```

### Page with SEO

```astro
---
import BaseLayout from '@/layouts/BaseLayout.astro';

const title = 'Page Title';
const description = 'Page description 120-160 characters';
---

<BaseLayout title={title} description={description}>
  <main>
    <!-- Content -->
  </main>
</BaseLayout>
```

### Islands Architecture (Interactive Components)

```astro
---
// Static components (0 JS)
import Header from '@/components/Header.astro';

// Interactive islands (hydrated client-side)
import ContactForm from '@/components/ContactForm.tsx';
---

<Header />  <!-- Static, no hydration -->
<ContactForm client:load />  <!-- Interactive, hydrates on load -->
```

## 🎨 Responsive Design & Dark Mode

### Responsive Design (Mobile-First)

```html
<!-- Grid: Mobile 1 col, Tablet 2 cols, Desktop 3 cols -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  <!-- Content -->
</div>

<!-- Typography: Responsive font sizes -->
<h1 class="text-3xl md:text-4xl lg:text-5xl">
  Responsive Heading
</h1>

<!-- Container with responsive padding -->
<div class="container mx-auto px-4 sm:px-6 lg:px-8">
  <!-- Content -->
</div>
```

### Dark Mode Support

```html
<!-- Background and text colors -->
<div class="bg-white dark:bg-slate-900">
  <p class="text-slate-900 dark:text-slate-50">Content</p>
</div>

<!-- Borders and hover states -->
<button class="border border-slate-300 dark:border-slate-700 hover:bg-slate-50 dark:hover:bg-slate-800">
  Button
</button>
```

**Theme Toggle Component** (React island):
- Automatically detects system preference
- Allows manual override
- Persists choice in localStorage
- Prevents flash on page load

See `.docs/EJEMPLOS_CODIGO.md` for complete ThemeToggle implementation.

## 🚀 Performance & SEO

### Performance Targets
- Lighthouse Performance: **95+**
- First Contentful Paint: **< 1.2s**
- Largest Contentful Paint: **< 2.5s**

### SEO Requirements
Every page MUST have:
- Title tag (< 60 characters)
- Meta description (120-160 characters)
- Open Graph tags
- Twitter Card tags
- Canonical URL
- Schema.org markup

### Image Optimization

```astro
---
import { Image } from 'astro:assets';
import heroImage from '@/assets/hero.jpg';
---

<Image
  src={heroImage}
  alt="Descriptive text"
  width={1200}
  height={600}
  loading="lazy"
  format="webp"
/>
```

## 📂 Folder Structure

```
src/
├── components/       # Reusable UI (PascalCase.astro)
├── content/          # Markdown/MDX (kebab-case.md)
├── layouts/          # Page layouts (PascalCase.astro)
├── pages/            # Routes (kebab-case.astro)
├── styles/           # Global CSS
├── utils/            # Utilities (kebab-case.ts)
└── types/            # TypeScript types
```

## 🔗 Integration with bMOI Ecosystem

### Sign-Up Form → Backend API

```typescript
const response = await fetch(`${API_URL}/auth/register`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    email: formData.email,
    password: formData.password,
    companyName: formData.companyName,
    plan: formData.plan,
  }),
});

if (response.ok) {
  window.location.href = `${APP_URL}/dashboard?new=true`;
}
```

## 🧠 AI Memory Checkpoints

Every ~5 messages, verify you remember:
- [ ] Project: **bMOI Landing Page**
- [ ] Framework: **Astro 4** (NOT Next.js, NOT React SPA)
- [ ] Type: **Static Site (SSG)**, NOT SSR
- [ ] Styling: **Tailwind CSS** (utility-first)
- [ ] **Responsive design** - Mobile-first approach (use `md:`, `lg:` breakpoints)
- [ ] **Dark mode** - Support light/dark theme with `dark:` variants
- [ ] Deployment: **S3 + CloudFront**
- [ ] Conventions: **kebab-case** (files), **PascalCase** (components), **camelCase** (variables)
- [ ] **NEVER** use snake_case in TypeScript
- [ ] **ALWAYS** use `@/` for imports
- [ ] **MINIMAL JavaScript** - Islands Architecture
- [ ] **Performance first** - 95+ Lighthouse score
- [ ] **SEO critical** - Meta tags mandatory

## 🚨 Common Mistakes to Avoid

### ❌ DON'T

```typescript
// ❌ Don't use snake_case
const api_url = 'https://...';

// ❌ Don't use any type
function fetchData(): any {}

// ❌ Don't create SSR routes (this is SSG only)
export const GET = () => {};

// ❌ Don't hydrate static components
<Header client:load />

// ❌ Don't use relative imports across folders
import Header from '../../../components/Header.astro';
```

### ✅ DO

```typescript
// ✅ Use camelCase
const apiUrl = 'https://...';

// ✅ Type everything
function fetchData(): Promise<BlogPost[]> {}

// ✅ Use static generation
export async function getStaticPaths() {}

// ✅ Only hydrate interactive components
<ContactForm client:load />

// ✅ Use @/ alias
import Header from '@/components/Header.astro';
```

## 📊 Development Phases

```
Phase 0 (Setup):        ░░░░░░░░░░  0%  - Astro + Docker + Tailwind
Phase 1 (Core Pages):   ░░░░░░░░░░  0%  - Home, Features, Pricing
Phase 2 (Content):      ░░░░░░░░░░  0%  - Blog, Legal pages
Phase 3 (SEO):          ░░░░░░░░░░  0%  - Meta tags, Sitemap
Phase 4 (Integrations): ░░░░░░░░░░  0%  - Analytics, Forms
Phase 5 (Deployment):   ░░░░░░░░░░  0%  - S3, CloudFront, CI/CD
Phase 6 (Testing):      ░░░░░░░░░░  0%  - Lighthouse, Cross-browser
```

## 🔍 Troubleshooting AI Behavior

### If AI forgets conventions:

```
🧠 CONVENTION REMINDER - LANDING PAGE:

This project uses STRICTLY:
- Files: PascalCase (components), kebab-case (pages/content)
- Code: camelCase (variables), UPPER_SNAKE_CASE (constants), PascalCase (types)
- NEVER snake_case in TypeScript
- ALWAYS use @/ for imports

Framework: Astro 4 (NOT Next.js)
Type: Static Site (SSG), NOT SSR
JavaScript: Minimal (Islands Architecture)
Performance: 95+ Lighthouse score target
SEO: Meta tags mandatory on every page

Please confirm you'll remember this.
```

### If AI creates SSR code:

```
⚠️ ERROR: This is a static site (SSG), NOT SSR

This project does NOT support:
- Server-side rendering
- API routes in Astro
- Dynamic server-side logic

Use:
- getStaticPaths() for dynamic routes
- Markdown/MDX for content
- Client-side fetch for API calls

Please correct the code above.
```

## 📚 Additional Resources

- **Complete context:** `.claude/context.md`
- **Conventions:** `.claude/conventions.md`
- **Current state:** `.claude/session-state.md`
- **Setup guide:** `.docs/SETUP.md`
- **Architecture:** `.docs/ARQUITECTURA.md`
- **Code examples:** `.docs/EJEMPLOS_CODIGO.md`
- **SEO guide:** `.docs/SEO_GUIDE.md`

## 🔗 Related Projects

- Backend API: `bo-backend-api/CLAUDE.md`
- Admin Panel: `plantillas_admin/.claude/context.md`
- Mobile App: `plantillas_mobile/.claude/context.md`

---

<div align="center">

**Static Landing Page with Astro 4**

Fast | SEO-Optimized | Zero JS by Default

</div>
