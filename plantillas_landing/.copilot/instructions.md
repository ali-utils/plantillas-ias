# GitHub Copilot Instructions - bMOI Landing Page

## Project Context

You are working on the **bMOI Landing Page**, a static marketing website built with **Astro 4 + TypeScript 5 + Tailwind CSS**.

**Purpose:** Attract visitors, showcase bMOI platform features, convert visitors to users
**Deployment:** S3 + CloudFront (static hosting)
**Type:** Static Site Generation (SSG), NOT a web application

---

## Core Technologies

- **Framework:** Astro 4 (static site generator)
- **Language:** TypeScript 5
- **Styling:** Tailwind CSS 4
- **Content:** Markdown/MDX with Content Collections
- **Deployment:** AWS S3 + CloudFront (or Netlify/Vercel)
- **Package Manager:** npm
- **Node Version:** 20.x

---

## Mandatory Conventions

### File Naming

```typescript
✅ CORRECT:
components/Header.astro          // PascalCase for components
components/Hero.astro
components/ui/Button.astro
pages/index.astro                // kebab-case for pages
pages/pricing.astro
pages/privacy-policy.astro
content/blog/getting-started.md  // kebab-case for content
utils/format-date.ts             // kebab-case for utilities
types/content-types.ts           // kebab-case for type files

❌ INCORRECT:
components/header.astro          // Should be PascalCase
components/hero-section.astro    // Should be Hero.astro
pages/Privacy-Policy.astro       // Should be kebab-case
content/blog/Getting-Started.md  // Should be kebab-case
utils/formatDate.ts              // Should be kebab-case
```

### Code Naming

```typescript
✅ CORRECT:
// Variables & functions: camelCase
const siteName = 'bMOI';
const apiBaseUrl = 'https://api.bmoi.com';
function formatDate(date: Date): string {}
async function fetchBlogPosts(): Promise<Post[]> {}

// Constants: UPPER_SNAKE_CASE
const API_URL = 'https://api.bmoi.com';
const MAX_RETRIES = 3;
const SITE_NAME = 'bMOI';

// Classes & types: PascalCase
interface BlogPost {}
class ApiClient {}
type PlanType = 'free' | 'plan1' | 'plan2' | 'plan3';

❌ INCORRECT:
const SiteName = 'bMOI';           // Should be camelCase
const api_base_url = 'https://...'; // Never snake_case
function FormatDate() {}           // Should be camelCase
interface blogPost {}              // Should be PascalCase
```

---

## Astro-Specific Patterns

### Component Structure

Always use this structure for Astro components:

```astro
---
// 1. Imports (framework, external, internal)
import { Image } from 'astro:assets';
import BaseLayout from '@/layouts/BaseLayout.astro';
import Button from '@/components/ui/Button.astro';

// 2. Type definitions
interface Props {
  title: string;
  description?: string;
  showCta?: boolean;
}

// 3. Props destructuring
const { title, description, showCta = true } = Astro.props;

// 4. Logic
const currentYear = new Date().getFullYear();
---

<!-- 5. Template -->
<BaseLayout title={title}>
  <section class="container mx-auto px-4 py-12">
    <h1 class="text-4xl font-bold">{title}</h1>
    {description && <p class="text-lg mt-4">{description}</p>}
    {showCta && <Button>Get Started</Button>}
  </section>
</BaseLayout>

<!-- 6. Scoped styles (if needed) -->
<style>
  /* Component-specific styles */
</style>
```

### Islands Architecture

Use React/Vue islands only for interactive components:

```astro
---
// Static component (preferred for performance)
import Header from '@/components/Header.astro';

// Interactive island (only when needed)
import ContactForm from '@/components/ContactForm.tsx';
---

<Header />

<!-- Static by default -->
<section class="hero">...</section>

<!-- Interactive island (hydrates client-side) -->
<ContactForm client:load />
```

**Client directives:**
- `client:load` - Hydrate immediately on page load
- `client:idle` - Hydrate when browser is idle
- `client:visible` - Hydrate when element is visible
- `client:media` - Hydrate on media query match

---

## Common Patterns

### 1. Page Layout

```astro
---
import BaseLayout from '@/layouts/BaseLayout.astro';
import Header from '@/components/Header.astro';
import Hero from '@/components/Hero.astro';
import Features from '@/components/Features.astro';
import CallToAction from '@/components/CallToAction.astro';
import Footer from '@/components/Footer.astro';
---

<BaseLayout
  title="bMOI - Multi-Tenant Management Platform"
  description="Powerful multi-tenant platform for business management"
>
  <Header />
  <main>
    <Hero />
    <Features />
    <CallToAction />
  </main>
  <Footer />
</BaseLayout>
```

### 2. Dynamic Content (Blog Posts)

```astro
---
// src/pages/blog/[slug].astro
import { getCollection } from 'astro:content';
import BlogLayout from '@/layouts/BlogLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<BlogLayout title={post.data.title} publishedAt={post.data.publishedAt}>
  <Content />
</BlogLayout>
```

### 3. Content Collections

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const blogCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    publishedAt: z.date(),
    author: z.string(),
    tags: z.array(z.string()).optional(),
  }),
});

export const collections = {
  blog: blogCollection,
};
```

### 4. SEO Meta Tags

```astro
---
interface Props {
  title: string;
  description: string;
  ogImage?: string;
}

const { title, description, ogImage = '/images/og-default.jpg' } = Astro.props;
const canonicalUrl = new URL(Astro.url.pathname, Astro.site);
---

<head>
  <!-- Basic Meta -->
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{title} | bMOI</title>
  <meta name="description" content={description} />

  <!-- Canonical URL -->
  <link rel="canonical" href={canonicalUrl} />

  <!-- Open Graph -->
  <meta property="og:title" content={title} />
  <meta property="og:description" content={description} />
  <meta property="og:image" content={ogImage} />
  <meta property="og:url" content={canonicalUrl} />
  <meta property="og:type" content="website" />

  <!-- Twitter Card -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content={title} />
  <meta name="twitter:description" content={description} />
  <meta name="twitter:image" content={ogImage} />
</head>
```

### 5. Tailwind Component Classes

```astro
---
interface Props {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
}

const { variant = 'primary', size = 'md' } = Astro.props;

const baseClasses = 'inline-flex items-center justify-center font-semibold rounded-lg transition-colors';

const variantClasses = {
  primary: 'bg-blue-600 text-white hover:bg-blue-700',
  secondary: 'bg-gray-600 text-white hover:bg-gray-700',
  outline: 'border-2 border-blue-600 text-blue-600 hover:bg-blue-50',
};

const sizeClasses = {
  sm: 'px-4 py-2 text-sm',
  md: 'px-6 py-3 text-base',
  lg: 'px-8 py-4 text-lg',
};

const classes = `${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]}`;
---

<button class={classes}>
  <slot />
</button>
```

---

## Imports & Path Aliases

Always use `@/` alias for absolute imports:

```typescript
✅ CORRECT:
import Header from '@/components/Header.astro';
import { formatDate } from '@/utils/format-date';
import { API_URL } from '@/config/site-config';
import type { BlogPost } from '@/types/content-types';

❌ INCORRECT:
import Header from '../../../components/Header.astro';
import { formatDate } from '../../utils/format-date';
```

**Import order:**
1. Astro/Framework imports
2. External libraries
3. Internal components
4. Utilities/helpers
5. Types
6. Styles

---

## Styling Guidelines

### Tailwind Utility Classes

```astro
✅ CORRECT:
<div class="container mx-auto px-4 py-12">
  <h1 class="text-4xl font-bold text-gray-900 mb-4">Title</h1>
  <p class="text-lg text-gray-600 leading-relaxed">Description</p>
  <button class="bg-blue-600 text-white px-6 py-3 rounded-lg hover:bg-blue-700">
    Get Started
  </button>
</div>

❌ AVOID (custom classes unless reusable):
<div class="custom-container">
  <h1 class="custom-title">Title</h1>
</div>
```

### Responsive Design

```astro
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  <!-- Mobile: 1 column, Tablet: 2 columns, Desktop: 3 columns -->
</div>

<h1 class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl">
  <!-- Responsive font sizes -->
</h1>
```

---

## Performance Best Practices

### 1. Image Optimization

```astro
---
import { Image } from 'astro:assets';
import heroImage from '@/assets/hero.jpg';
---

<!-- ✅ Use Astro Image component (optimized automatically) -->
<Image
  src={heroImage}
  alt="Hero banner"
  width={1200}
  height={600}
  loading="lazy"
  format="webp"
/>

<!-- ❌ Don't use plain <img> for local images -->
<img src="/images/hero.jpg" alt="Hero" />
```

### 2. Lazy Loading

```astro
<!-- Load critical content immediately -->
<Hero client:load />

<!-- Defer non-critical content -->
<Testimonials client:idle />

<!-- Load when visible in viewport -->
<LatestBlogPosts client:visible />
```

### 3. Minimize JavaScript

```astro
<!-- ✅ Static Astro component (0 JS) -->
<Features />

<!-- ❌ Avoid unnecessary islands -->
<Features client:load />  <!-- Only if truly interactive! -->
```

---

## TypeScript Patterns

### Type-Safe Props

```typescript
interface Props {
  title: string;                    // Required
  description?: string;             // Optional
  items: Array<{ id: string; name: string }>; // Complex type
  variant?: 'primary' | 'secondary'; // Union type
  onSubmit?: (data: FormData) => void; // Function type
}

const {
  title,
  description,
  items,
  variant = 'primary',  // Default value
  onSubmit
} = Astro.props;
```

### Content Collection Types

```typescript
import { getCollection, type CollectionEntry } from 'astro:content';

type BlogPost = CollectionEntry<'blog'>;

const posts: BlogPost[] = await getCollection('blog');
const sortedPosts = posts.sort((a, b) =>
  b.data.publishedAt.getTime() - a.data.publishedAt.getTime()
);
```

---

## Common Mistakes to Avoid

### ❌ DON'T

```typescript
// ❌ Don't use snake_case in TypeScript
const api_url = 'https://...';
const user_name = 'John';

// ❌ Don't use any type
function fetchData(): any {}

// ❌ Don't use console.log (use proper logging)
console.log('Debug data:', data);

// ❌ Don't create server-side routes (this is SSG only)
export const GET = (context) => {};  // Wrong! No SSR routes

// ❌ Don't hydrate everything
<Header client:load />  // Static component doesn't need hydration
```

### ✅ DO

```typescript
// ✅ Use camelCase for variables
const apiUrl = 'https://...';
const userName = 'John';

// ✅ Use proper types
function fetchData(): Promise<BlogPost[]> {}

// ✅ Only hydrate interactive components
<ContactForm client:load />  // Needs interactivity
<Header />  // Static, no hydration needed

// ✅ Use environment variables for config
const apiUrl = import.meta.env.PUBLIC_API_URL;
```

---

## Environment Variables

```typescript
// Access environment variables
const apiUrl = import.meta.env.PUBLIC_API_URL;
const gaId = import.meta.env.PUBLIC_GA_ID;

// Type-safe env (src/env.d.ts)
interface ImportMetaEnv {
  readonly PUBLIC_API_URL: string;
  readonly PUBLIC_APP_URL: string;
  readonly PUBLIC_SITE_URL: string;
  readonly PUBLIC_GA_ID: string;
  readonly PUBLIC_GTM_ID: string;
}
```

**Important:** Only `PUBLIC_*` variables are accessible in client-side code!

---

## Testing & Quality

### Lighthouse Targets

- **Performance:** 95+
- **Accessibility:** 100
- **Best Practices:** 100
- **SEO:** 100

### Accessibility Checklist

```astro
<!-- ✅ Semantic HTML -->
<header>
  <nav aria-label="Main navigation">
    <ul>...</ul>
  </nav>
</header>

<!-- ✅ Alt text for images -->
<Image src={image} alt="Descriptive text" />

<!-- ✅ ARIA labels for icon buttons -->
<button aria-label="Close menu">
  <IconClose />
</button>

<!-- ✅ Keyboard navigation support -->
<a href="/features" class="focus:ring-2 focus:ring-blue-500">
  Features
</a>
```

---

## Quick Reference

### Common Commands

```bash
# Development
npm run dev              # Start dev server
npm run build            # Build for production
npm run preview          # Preview production build

# Docker
docker compose up -d     # Start in Docker
docker compose logs -f   # View logs

# Utilities
npm run astro check      # Type check
npm run format           # Format with Prettier
```

### Project Structure

```
src/
├── components/          # Reusable UI components (PascalCase)
├── content/             # Markdown/MDX content (kebab-case)
├── layouts/             # Page layouts (PascalCase)
├── pages/               # Routes (kebab-case)
├── styles/              # Global styles
├── utils/               # Utility functions (kebab-case)
└── types/               # TypeScript types (kebab-case)
```

---

## Integration with bMOI Ecosystem

### Sign-Up Form → Backend API

```typescript
// POST to backend API
const response = await fetch(`${API_URL}/auth/register`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    email: formData.email,
    password: formData.password,
    companyName: formData.companyName,
    plan: formData.plan,
  }),
});

if (response.ok) {
  // Redirect to admin panel
  window.location.href = `${APP_URL}/dashboard?new=true`;
}
```

---

## Remember

1. **Astro 4** - Static Site Generator, not Next.js
2. **TypeScript** - Full type safety, no `any`
3. **Tailwind CSS** - Utility-first styling
4. **Islands Architecture** - Minimal JavaScript
5. **Performance first** - Lazy load, optimize images
6. **SEO critical** - Meta tags, schema markup, sitemap
7. **Mobile-first** - Responsive design mandatory
8. **Accessibility** - WCAG 2.1 AA compliance

---

**For questions or clarifications, refer to:**
- `.claude/context.md` - Complete project context
- `.claude/conventions.md` - Detailed naming conventions
- `.docs/CODING_STANDARDS.md` - Coding standards with examples
