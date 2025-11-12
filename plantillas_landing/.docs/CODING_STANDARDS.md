# Coding Standards - bMOI Landing Page

Standards for writing consistent, maintainable code.

## File Naming

```
✅ CORRECT:
components/Header.astro           # PascalCase for components
pages/index.astro                 # kebab-case for pages
content/features/multi-tenant.md  # kebab-case for content
utils/format-date.ts              # kebab-case for utilities

❌ INCORRECT:
components/header.astro           # Should be PascalCase
pages/Index.astro                 # Should be kebab-case
content/features/Multi-Tenant.md  # Should be kebab-case
```

## Astro Component Structure

```astro
---
// 1. Imports (framework → external → internal → types → styles)
import { Image } from 'astro:assets';
import BaseLayout from '@/layouts/BaseLayout.astro';
import Button from '@/components/ui/Button.astro';
import type { Props } from './types';

// 2. Type definitions
interface Props {
  title: string;
  description?: string;
  showCta?: boolean;
}

// 3. Props destructuring
const {
  title,
  description,
  showCta = true,
} = Astro.props;

// 4. Logic (keep minimal)
const currentYear = new Date().getFullYear();
const formattedTitle = title.toUpperCase();
---

<!-- 5. Template (semantic HTML) -->
<section class="container mx-auto px-4 py-12">
  <h1 class="text-4xl font-bold">{formattedTitle}</h1>
  {description && <p class="mt-4">{description}</p>}
  {showCta && <Button href="/contact">Contact Us</Button>}
</section>

<!-- 6. Scoped styles (if needed) -->
<style>
  section {
    /* Component-specific styles */
  }
</style>
```

## TypeScript Standards

```typescript
// ✅ Always type function parameters and return types
function formatDate(date: Date): string {
  return date.toLocaleDateString();
}

// ✅ Use interfaces for objects
interface BlogPost {
  title: string;
  slug: string;
  publishedAt: Date;
}

// ✅ Use type for unions/primitives
type Theme = 'light' | 'dark';
type Status = 'idle' | 'loading' | 'success' | 'error';

// ❌ Never use any
function getData(): any {}  // WRONG

// ✅ Use unknown or specific type
function getData(): unknown {}
function getData(): BlogPost[] {}
```

## Tailwind CSS Standards

```html
<!-- ✅ Use utility classes -->
<div class="flex items-center justify-between p-4 bg-white rounded-lg shadow-md">

<!-- ✅ Responsive design -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

<!-- ✅ Dark mode variants -->
<div class="bg-white dark:bg-slate-900 text-slate-900 dark:text-slate-50">

<!-- ❌ Avoid inline styles -->
<div style="background: white; padding: 16px;">  <!-- WRONG -->

<!-- ✅ Custom classes in global.css if reused -->
<button class="btn-primary">Click</button>
```

## Responsive Design Patterns

```html
<!-- ✅ Mobile-first approach -->
<h1 class="text-3xl md:text-4xl lg:text-5xl">
  <!-- Mobile: 3xl, Tablet: 4xl, Desktop: 5xl -->
</h1>

<!-- ✅ Container with responsive padding -->
<div class="container mx-auto px-4 sm:px-6 lg:px-8">

<!-- ✅ Responsive grid -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">

<!-- ✅ Show/hide based on screen size -->
<div class="hidden md:block">Desktop only</div>
<div class="block md:hidden">Mobile only</div>
```

## Dark Mode Patterns

```html
<!-- ✅ Background/foreground -->
<div class="bg-white dark:bg-slate-900">
  <p class="text-slate-900 dark:text-slate-50">Text</p>
</div>

<!-- ✅ Borders -->
<div class="border border-slate-300 dark:border-slate-700">

<!-- ✅ Hover states -->
<button class="bg-blue-600 hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-600">

<!-- ✅ Use CSS variables for complex theming -->
<div style="background-color: rgb(var(--color-background))">
```

## SEO Best Practices

```astro
---
// ✅ Every page must have unique meta tags
const title = 'Page Title';
const description = 'Page description 120-160 characters';
const canonicalUrl = new URL(Astro.url.pathname, Astro.site);
---

<head>
  <!-- ✅ Essential meta tags -->
  <title>{title} | bMOI</title>
  <meta name="description" content={description} />
  <link rel="canonical" href={canonicalUrl} />

  <!-- ✅ Open Graph -->
  <meta property="og:title" content={title} />
  <meta property="og:description" content={description} />
  <meta property="og:image" content="/images/og-image.jpg" />
  <meta property="og:url" content={canonicalUrl} />

  <!-- ✅ Structured data -->
  <script type="application/ld+json">
    {JSON.stringify({
      "@context": "https://schema.org",
      "@type": "WebPage",
      "name": title,
      "description": description,
    })}
  </script>
</head>
```

## Accessibility Standards

```html
<!-- ✅ Semantic HTML -->
<header>
  <nav aria-label="Main navigation">
    <ul>
      <li><a href="/">Home</a></li>
    </ul>
  </nav>
</header>

<!-- ✅ Alt text for images -->
<img src="logo.png" alt="bMOI company logo" />

<!-- ✅ ARIA labels for icon buttons -->
<button aria-label="Close menu">
  <XIcon />
</button>

<!-- ✅ Focus states -->
<a href="/contact" class="focus:outline-none focus:ring-2 focus:ring-blue-500">

<!-- ✅ Proper heading hierarchy -->
<h1>Main Page Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>
```

## Performance Best Practices

```astro
---
// ✅ Use Astro Image component
import { Image } from 'astro:assets';
import heroImage from '@/assets/hero.jpg';
---

<!-- ✅ Optimized images -->
<Image
  src={heroImage}
  alt="Hero"
  width={1200}
  height={600}
  format="webp"
  loading="lazy"
/>

<!-- ✅ Preload critical resources -->
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>

<!-- ✅ Only hydrate interactive components -->
<ContactForm client:load />  <!-- Needs interactivity -->
<Header />  <!-- Static, no hydration -->
```

## Common Mistakes to Avoid

```typescript
// ❌ Don't use console.log in production
console.log('Debug data:', data);

// ✅ Use proper logging or remove
if (import.meta.env.DEV) {
  console.log('Debug data:', data);
}

// ❌ Don't hardcode URLs
const apiUrl = 'https://api.bmoi.com';

// ✅ Use environment variables
const apiUrl = import.meta.env.PUBLIC_API_URL;

// ❌ Don't ignore TypeScript errors
// @ts-ignore
const result = someFunction();

// ✅ Fix the type issue
const result: ReturnType<typeof someFunction> = someFunction();
```

## Git Commit Standards

```bash
# ✅ Use Conventional Commits
feat: add dark mode toggle
fix: correct mobile navigation alignment
docs: update README with setup instructions
style: format code with Prettier
refactor: simplify hero component
perf: optimize image loading
test: add tests for theme toggle

# ❌ Bad commits
Added stuff
Fixed bug
Updated files
```

---

**Follow these standards for consistent, maintainable code!**
