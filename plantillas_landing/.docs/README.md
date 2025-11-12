# bMOI Landing Page - Project Overview

## 🎯 Project Purpose

The bMOI Landing Page is a **static, high-performance marketing website** designed to:

- **Attract visitors** through SEO-optimized content
- **Convert visitors to users** with compelling CTAs
- **Showcase platform features** with beautiful, responsive design
- **Provide fast load times** through static generation
- **Support light/dark themes** for better user experience

## 🏗️ Architecture

### Static Site Generation (SSG)

The landing page uses **Astro 4** for static site generation:

```
Source Files (Astro/MDX) → Build Process → Static HTML/CSS/JS → CDN (S3/CloudFront)
```

**Benefits:**
- ⚡ **Fastest possible load times** (pre-rendered HTML)
- 💰 **Cheapest hosting** (static files on S3)
- 🔒 **Maximum security** (no server to hack)
- 📈 **Perfect SEO** (search engines love static HTML)
- 🌍 **Global CDN** (edge locations worldwide)

### Islands Architecture

Astro's Islands Architecture = **Zero JavaScript by default**

```
Static Components (No JS)    Interactive Islands (Hydrated)
┌─────────────────────┐      ┌──────────────────────┐
│ Header              │      │ ContactForm          │
│ Footer              │      │ (client:load)        │
│ Hero Section        │      │                      │
│ Features Grid       │      │ ThemeToggle          │
│ Testimonials        │      │ (client:idle)        │
└─────────────────────┘      └──────────────────────┘
```

Only interactive components ship JavaScript!

## 🎨 Design System

### Responsive Design Strategy

**Mobile-First Approach:**

```css
/* Mobile: 320px - 639px (default) */
.container { padding: 1rem; }

/* Tablet: 640px - 1023px */
@media (min-width: 640px) {
  .container { padding: 2rem; }
}

/* Desktop: 1024px+ */
@media (min-width: 1024px) {
  .container { padding: 4rem; }
}
```

**Tailwind CSS Breakpoints:**
- `sm`: 640px (tablet portrait)
- `md`: 768px (tablet landscape)
- `lg`: 1024px (desktop)
- `xl`: 1280px (large desktop)
- `2xl`: 1536px (extra large)

### Theme System (Light/Dark Mode)

**Two-theme support:**

1. **Light Theme** (default)
   - Background: White/Light Gray
   - Text: Dark Gray/Black
   - Primary: Blue 600
   - Perfect for daytime reading

2. **Dark Theme**
   - Background: Dark Gray/Black
   - Text: Light Gray/White
   - Primary: Blue 400 (lighter for contrast)
   - Reduces eye strain at night

**Implementation Strategy:**
```html
<!-- System preference detection -->
<script>
  if (localStorage.theme === 'dark' ||
     (!('theme' in localStorage) &&
      window.matchMedia('(prefers-color-scheme: dark)').matches)) {
    document.documentElement.classList.add('dark')
  }
</script>
```

**CSS Variables for Theming:**
```css
:root {
  --color-background: 255 255 255;
  --color-foreground: 0 0 0;
  --color-primary: 37 99 235;
}

.dark {
  --color-background: 15 23 42;
  --color-foreground: 241 245 249;
  --color-primary: 96 165 250;
}
```

## 🛠️ Technology Stack

### Core Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| **Astro** | 4.0+ | Static site generator |
| **TypeScript** | 5.0+ | Type safety |
| **Tailwind CSS** | 4.0+ | Utility-first styling |
| **MDX** | 3.0+ | Markdown with components |
| **React** | 18.0+ | Interactive islands (optional) |

### Why Astro?

✅ **Zero JavaScript by default** - Only ship what you need
✅ **Islands Architecture** - Partial hydration
✅ **Framework agnostic** - Use React, Vue, Svelte together
✅ **Built-in optimizations** - Image optimization, CSS minification
✅ **Perfect for content sites** - Marketing pages, blogs, docs

**Comparison with alternatives:**

| Framework | Build Speed | Bundle Size | Learning Curve |
|-----------|-------------|-------------|----------------|
| **Astro** | ⚡⚡⚡ | 🟢 Tiny | 🟢 Easy |
| Next.js Static | ⚡⚡ | 🟡 Medium | 🟡 Medium |
| Gatsby | ⚡ | 🔴 Large | 🔴 Hard |

### Why Tailwind CSS?

✅ **Utility-first** - Rapid prototyping
✅ **No unused CSS** - Automatic purging
✅ **Responsive design** - Mobile-first utilities
✅ **Dark mode built-in** - `dark:` variant
✅ **Customizable** - Design tokens in config

## 📦 Project Structure

```
plantillas_landing/
├── public/                     # Static assets (copied as-is)
│   ├── favicon.svg
│   ├── robots.txt
│   ├── sitemap.xml
│   └── images/
│       ├── logo.svg
│       └── og-image.jpg
│
├── src/
│   ├── components/             # Reusable UI components
│   │   ├── Header.astro        # Site header with nav
│   │   ├── Footer.astro        # Site footer
│   │   ├── Hero.astro          # Hero section
│   │   ├── Features.astro      # Features grid
│   │   ├── Pricing.astro       # Pricing table
│   │   ├── Testimonials.astro  # Customer testimonials
│   │   ├── CallToAction.astro  # CTA sections
│   │   ├── ThemeToggle.tsx     # Light/dark toggle (React island)
│   │   └── ui/                 # Base UI components
│   │       ├── Button.astro
│   │       ├── Card.astro
│   │       ├── Badge.astro
│   │       └── Input.astro
│   │
│   ├── content/                # Markdown/MDX content
│   │   ├── config.ts           # Content collections config
│   │   ├── features/           # Feature descriptions
│   │   │   ├── multi-tenant.md
│   │   │   ├── security.md
│   │   │   └── analytics.md
│   │   ├── blog/               # Blog posts
│   │   │   ├── getting-started.mdx
│   │   │   └── best-practices.mdx
│   │   └── legal/              # Legal pages
│   │       ├── privacy.md
│   │       ├── terms.md
│   │       └── cookies.md
│   │
│   ├── layouts/                # Page layouts
│   │   ├── BaseLayout.astro    # Base HTML structure
│   │   ├── BlogLayout.astro    # Blog post layout
│   │   └── LegalLayout.astro   # Legal pages layout
│   │
│   ├── pages/                  # Routes (file-based routing)
│   │   ├── index.astro         # Home page (/)
│   │   ├── features.astro      # Features page (/features)
│   │   ├── pricing.astro       # Pricing page (/pricing)
│   │   ├── about.astro         # About page (/about)
│   │   ├── contact.astro       # Contact page (/contact)
│   │   ├── blog/
│   │   │   ├── index.astro     # Blog listing (/blog)
│   │   │   └── [slug].astro    # Blog post (/blog/post-name)
│   │   └── legal/
│   │       ├── privacy.astro   # Privacy policy
│   │       └── terms.astro     # Terms of service
│   │
│   ├── styles/
│   │   └── global.css          # Global styles + Tailwind imports
│   │
│   ├── utils/                  # Utility functions
│   │   ├── format-date.ts
│   │   ├── seo-helpers.ts
│   │   └── theme-helpers.ts
│   │
│   └── env.d.ts                # TypeScript environment types
│
├── .claude/                    # AI assistant context
├── .copilot/                   # GitHub Copilot instructions
├── .cursorrules                # Cursor AI rules
├── .docs/                      # Documentation
├── astro.config.mjs            # Astro configuration
├── tailwind.config.mjs         # Tailwind CSS configuration
├── tsconfig.json               # TypeScript configuration
├── package.json                # Dependencies
└── docker-compose.yml          # Docker development
```

## 🎯 Key Features

### 1. Home Page (/)

**Sections:**
- **Hero** - Main value proposition with CTA
- **Features Overview** - 3-4 key features with icons
- **How It Works** - 3-step process
- **Social Proof** - Testimonials or client logos
- **Pricing Teaser** - Quick pricing overview
- **Final CTA** - "Get Started" button

### 2. Features Page (/features)

**Sections:**
- **Feature Grid** - Detailed feature cards
- **Use Cases** - Industry-specific examples
- **Integrations** - Third-party integrations
- **Demo Video** - Product walkthrough

### 3. Pricing Page (/pricing)

**Sections:**
- **Plan Comparison Table** - Free, Plan1, Plan2, Plan3
- **FAQ** - Common pricing questions
- **Feature Comparison** - Detailed feature matrix
- **CTA** - "Start Free Trial"

### 4. Blog (/blog)

**Sections:**
- **Post Listing** - Latest articles
- **Categories/Tags** - Content organization
- **Search** - Find articles
- **Newsletter Signup** - Email capture

### 5. Contact Page (/contact)

**Sections:**
- **Contact Form** - Sales inquiries
- **Support Links** - Link to support portal
- **Social Media** - Links to social profiles
- **Office Locations** - Physical addresses (optional)

## 🚀 Performance Targets

### Lighthouse Scores

| Metric | Target | Why |
|--------|--------|-----|
| **Performance** | 95+ | Fast load times = better UX + SEO |
| **Accessibility** | 100 | WCAG 2.1 AA compliance |
| **Best Practices** | 100 | Security, HTTPS, no console errors |
| **SEO** | 100 | Proper meta tags, schema markup |

### Core Web Vitals

| Metric | Target | Description |
|--------|--------|-------------|
| **LCP** | < 2.5s | Largest Contentful Paint |
| **FID** | < 100ms | First Input Delay |
| **CLS** | < 0.1 | Cumulative Layout Shift |

### Bundle Size Targets

- **Initial HTML**: < 50KB
- **CSS**: < 30KB (after purge)
- **JavaScript**: < 50KB (only for islands)
- **Total Page**: < 200KB (before images)

## 🔍 SEO Strategy

### On-Page SEO

✅ **Title Tags** - Unique per page, 50-60 characters
✅ **Meta Descriptions** - Compelling, 120-160 characters
✅ **Header Hierarchy** - Proper H1, H2, H3 structure
✅ **Alt Text** - Descriptive for all images
✅ **Internal Linking** - Related pages linked
✅ **Canonical URLs** - Avoid duplicate content

### Technical SEO

✅ **Sitemap.xml** - Auto-generated
✅ **Robots.txt** - Allow all crawling
✅ **Schema.org Markup** - Organization, WebSite, Article
✅ **Open Graph Tags** - Social media previews
✅ **Page Speed** - < 3s load time
✅ **Mobile-Friendly** - Responsive design
✅ **HTTPS** - SSL certificate

### Content SEO

✅ **Target Keywords** - "multi-tenant SaaS", "business platform"
✅ **Blog Content** - 1-2 articles per week
✅ **Long-Form Content** - 1500+ word guides
✅ **Internal Linking** - Blog ↔ Product pages

## 🔐 Security

### Headers

```nginx
# Security headers (nginx.conf)
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
add_header Referrer-Policy "strict-origin-when-cross-origin";
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()";
```

### Content Security Policy (CSP)

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               script-src 'self' https://www.googletagmanager.com;
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;">
```

## 📱 Responsive Design Best Practices

### Breakpoint Strategy

```javascript
// tailwind.config.mjs
export default {
  theme: {
    screens: {
      'sm': '640px',   // Tablet portrait
      'md': '768px',   // Tablet landscape
      'lg': '1024px',  // Desktop
      'xl': '1280px',  // Large desktop
      '2xl': '1536px', // Extra large
    }
  }
}
```

### Mobile-First CSS

```html
<!-- Mobile: 1 column (default) -->
<div class="grid grid-cols-1">

<!-- Tablet: 2 columns -->
<div class="grid grid-cols-1 md:grid-cols-2">

<!-- Desktop: 3 columns -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
```

### Typography Scale

```css
/* Mobile */
h1 { font-size: 2rem; }    /* 32px */
h2 { font-size: 1.5rem; }  /* 24px */
p { font-size: 1rem; }     /* 16px */

/* Desktop */
@media (min-width: 1024px) {
  h1 { font-size: 3rem; }   /* 48px */
  h2 { font-size: 2rem; }   /* 32px */
  p { font-size: 1.125rem; } /* 18px */
}
```

## 🌓 Dark Mode Implementation

### Strategy

1. **System Preference Detection** - Respect OS setting
2. **User Override** - Allow manual toggle
3. **Persistence** - Remember user choice
4. **No Flash** - Prevent theme flash on load

### Implementation

```typescript
// src/utils/theme-helpers.ts
export function initTheme() {
  const theme = localStorage.getItem('theme');
  const systemDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

  if (theme === 'dark' || (!theme && systemDark)) {
    document.documentElement.classList.add('dark');
  } else {
    document.documentElement.classList.remove('dark');
  }
}

export function toggleTheme() {
  const isDark = document.documentElement.classList.contains('dark');

  if (isDark) {
    document.documentElement.classList.remove('dark');
    localStorage.setItem('theme', 'light');
  } else {
    document.documentElement.classList.add('dark');
    localStorage.setItem('theme', 'dark');
  }
}
```

## 📊 Analytics & Monitoring

### Google Analytics 4

```html
<!-- Global site tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

### Events to Track

- Page views
- CTA button clicks
- Form submissions
- Outbound links
- Scroll depth
- Time on page

## 🚢 Deployment

### S3 + CloudFront (Primary)

```bash
# Build
npm run build

# Deploy to S3
aws s3 sync dist/ s3://bmoi-landing-bucket --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation --distribution-id XXX --paths "/*"
```

### Alternative Platforms

- **Netlify** - Easiest, generous free tier
- **Vercel** - Great DX, auto-deployments
- **Cloudflare Pages** - Fastest CDN
- **AWS Amplify** - If using AWS ecosystem

## 📚 Related Documentation

- **[SETUP.md](SETUP.md)** - Step-by-step setup guide
- **[ARQUITECTURA.md](ARQUITECTURA.md)** - Architecture decisions
- **[DEPLOYMENT.md](DEPLOYMENT.md)** - Deployment guides
- **[CODING_STANDARDS.md](CODING_STANDARDS.md)** - Coding standards
- **[EJEMPLOS_CODIGO.md](EJEMPLOS_CODIGO.md)** - Code examples
- **[SEO_GUIDE.md](SEO_GUIDE.md)** - SEO best practices

---

**Version:** 1.0.0
**Last Updated:** 2025-01-04
