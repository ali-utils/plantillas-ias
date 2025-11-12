# Naming Conventions - bMOI Landing Page

## Overview

This document defines **mandatory naming conventions** for the bMOI Landing Page project. These conventions ensure consistency, readability, and maintainability across the codebase.

**Framework:** Astro 4 + TypeScript 5 + Tailwind CSS
**Project Type:** Static Landing Page (SSG)

---

## File Naming Conventions

### Astro Component Files

Use **PascalCase** with `.astro` extension:

```
✅ CORRECT:
Header.astro
Hero.astro
Features.astro
PricingCard.astro
TestimonialSlider.astro
CallToAction.astro

❌ INCORRECT:
header.astro          // Should be PascalCase
Hero.tsx              // Should be .astro not .tsx
pricing-card.astro    // Should be PascalCase not kebab-case
testimonial_slider.astro  // Never use snake_case
```

### React/Vue Components (Islands)

Use **PascalCase** with framework extension when using islands:

```
✅ CORRECT:
ContactForm.tsx       // React island
NewsletterSignup.tsx  // React island
AnimatedCounter.vue   // Vue island

❌ INCORRECT:
contact-form.tsx
newsletter_signup.tsx
animatedCounter.tsx   // Not camelCase, use PascalCase
```

### Page Files

Use **kebab-case** with `.astro` extension:

```
✅ CORRECT:
src/pages/index.astro
src/pages/features.astro
src/pages/pricing.astro
src/pages/about.astro
src/pages/contact.astro
src/pages/blog/[slug].astro
src/pages/privacy-policy.astro
src/pages/terms-of-service.astro

❌ INCORRECT:
src/pages/Index.astro        // Don't capitalize page files
src/pages/PrivacyPolicy.astro // Use kebab-case
src/pages/privacy_policy.astro // Never snake_case
src/pages/features.html      // Use .astro extension
```

### Layout Files

Use **PascalCase** with `.astro` extension:

```
✅ CORRECT:
src/layouts/BaseLayout.astro
src/layouts/BlogLayout.astro
src/layouts/LegalLayout.astro

❌ INCORRECT:
src/layouts/base-layout.astro   // Should be PascalCase
src/layouts/base_layout.astro   // Never snake_case
```

### Content Files (Markdown/MDX)

Use **kebab-case** with `.md` or `.mdx` extension:

```
✅ CORRECT:
src/content/features/multi-tenant-architecture.md
src/content/blog/getting-started-with-bmoi.mdx
src/content/legal/privacy-policy.md

❌ INCORRECT:
src/content/features/Multi-Tenant-Architecture.md  // Don't capitalize
src/content/blog/getting_started.md               // Never snake_case
src/content/legal/PrivacyPolicy.md                // Use kebab-case
```

### TypeScript/JavaScript Files

Use **kebab-case** for utility files, **PascalCase** for classes/types:

```
✅ CORRECT:
src/utils/format-date.ts
src/utils/api-client.ts
src/types/content-types.ts
src/lib/seo-helpers.ts
src/config/site-config.ts

❌ INCORRECT:
src/utils/formatDate.ts       // Should be kebab-case
src/utils/format_date.ts      // Never snake_case
src/types/ContentTypes.ts     // File should be kebab-case
```

### CSS Files

Use **kebab-case** with `.css` extension:

```
✅ CORRECT:
src/styles/global.css
src/styles/typography.css
src/styles/animations.css

❌ INCORRECT:
src/styles/Global.css
src/styles/typography_styles.css
```

### Configuration Files

Use **standard names** (conventional names for each tool):

```
✅ CORRECT:
astro.config.mjs        // Astro config
tailwind.config.mjs     // Tailwind config
tsconfig.json           // TypeScript config
package.json            // NPM config
.env.example            // Environment template
.gitignore              // Git ignore
.dockerignore           // Docker ignore
docker-compose.yml      // Docker Compose (kebab-case is standard)

❌ INCORRECT:
astro-config.mjs
tailwind.config.ts      // Use .mjs for configs
tsConfig.json
packageJson.json
```

---

## Code Naming Conventions

### Variables

Use **camelCase** for all variables:

```typescript
✅ CORRECT:
const siteName = 'bMOI';
const apiBaseUrl = 'https://api.bmoi.com';
const userEmail = 'user@example.com';
let isLoading = false;
let currentPage = 1;

❌ INCORRECT:
const SiteName = 'bMOI';           // Don't use PascalCase for variables
const api_base_url = 'https://...'; // Never snake_case
const APIBaseURL = 'https://...';  // Too many capitals
let is_loading = false;            // Never snake_case
```

### Constants

Use **UPPER_SNAKE_CASE** for true constants:

```typescript
✅ CORRECT:
const API_URL = 'https://api.bmoi.com';
const MAX_RETRIES = 3;
const DEFAULT_LOCALE = 'en';
const COMPANY_NAME = 'bMOI';
const ITEMS_PER_PAGE = 10;

❌ INCORRECT:
const apiUrl = 'https://...';      // Use UPPER_SNAKE_CASE for constants
const maxRetries = 3;              // Not enough emphasis
const defaultLocale = 'en';        // Should be emphasized
const company_name = 'bMOI';       // Use UPPER_SNAKE_CASE
```

**Note:** Only use UPPER_SNAKE_CASE for values that truly never change (API URLs, config values). For computed values or let variables, use camelCase.

### Functions

Use **camelCase** for all functions:

```typescript
✅ CORRECT:
function formatDate(date: Date): string {}
function calculatePrice(quantity: number): number {}
async function fetchBlogPosts(): Promise<Post[]> {}
const handleSubmit = (data: FormData) => {};

❌ INCORRECT:
function FormatDate(date: Date) {}        // Don't use PascalCase
function format_date(date: Date) {}       // Never snake_case
function FetchBlogPosts() {}              // Don't use PascalCase
const HandleSubmit = () => {};            // Don't use PascalCase
```

### Classes & Types

Use **PascalCase** for classes, types, interfaces, enums:

```typescript
✅ CORRECT:
class ApiClient {}
class FormValidator {}
interface BlogPost {}
interface UserData {}
type PlanType = 'free' | 'plan1' | 'plan2' | 'plan3';
type ResponseData<T> = { data: T; error?: string };
enum PlanTier {
  Free = 'free',
  Plan1 = 'plan1',
  Plan2 = 'plan2',
  Plan3 = 'plan3',
}

❌ INCORRECT:
class apiClient {}                // Should be PascalCase
class form_validator {}           // Never snake_case
interface blogPost {}             // Should be PascalCase
type planType = 'free' | 'plan1'; // Should be PascalCase
enum plan_tier {}                 // Never snake_case
```

### Astro Component Props

Use **camelCase** for prop names:

```astro
✅ CORRECT:
---
interface Props {
  title: string;
  description?: string;
  showCta?: boolean;
  ctaText?: string;
  imageSrc?: string;
  imageAlt?: string;
}

const { title, description, showCta = true, ctaText, imageSrc, imageAlt } = Astro.props;
---

❌ INCORRECT:
interface Props {
  Title: string;              // Should be camelCase
  show_cta?: boolean;         // Never snake_case
  CTAText?: string;           // Too many capitals, use ctaText
  image_src?: string;         // Never snake_case
}
```

### CSS Classes (Tailwind)

Use **Tailwind utility classes** directly (kebab-case built-in):

```astro
✅ CORRECT:
<div class="bg-blue-600 text-white p-4 rounded-lg shadow-md">
<button class="btn-primary hover:bg-blue-700 focus:ring-2">
<section class="container mx-auto px-4 py-12">

<!-- Custom classes in global.css -->
.btn-primary {
  @apply bg-blue-600 text-white px-6 py-3 rounded-lg;
}

❌ INCORRECT:
<div class="bgBlue600 textWhite p4">  // Don't invent class names
<div class="bg_blue_600">             // Tailwind uses kebab-case
<button class="btnPrimary">           // Use btn-primary
```

### Component Naming in JSX/Astro

Use **PascalCase** for component references:

```astro
✅ CORRECT:
---
import Header from '@/components/Header.astro';
import Hero from '@/components/Hero.astro';
import Features from '@/components/Features.astro';
import PricingCard from '@/components/PricingCard.astro';
---

<Header />
<Hero />
<Features />
<PricingCard plan="free" />

❌ INCORRECT:
import header from './Header.astro';  // Should be PascalCase
<header />                            // Conflicts with HTML <header>
<pricing-card plan="free" />         // Use PascalCase
```

---

## Folder Structure Conventions

### Component Organization

```
✅ CORRECT:
src/
├── components/
│   ├── Header.astro           # PascalCase for components
│   ├── Footer.astro
│   ├── Hero.astro
│   ├── Features.astro
│   ├── Pricing.astro
│   ├── Testimonials.astro
│   ├── CallToAction.astro
│   └── ui/                    # Lowercase for folders
│       ├── Button.astro
│       ├── Card.astro
│       ├── Badge.astro
│       └── Input.tsx          # React island for interactivity

❌ INCORRECT:
src/
├── components/
│   ├── header.astro           # Should be PascalCase
│   ├── Footer.tsx             # Use .astro unless needs React
│   ├── hero-section.astro     # Should be PascalCase: Hero.astro
│   ├── pricing_card.astro     # Never snake_case
│   └── UI/                    # Folder should be lowercase
```

### Page Organization

```
✅ CORRECT:
src/
├── pages/
│   ├── index.astro            # Home page (kebab-case)
│   ├── features.astro
│   ├── pricing.astro
│   ├── about.astro
│   ├── contact.astro
│   ├── blog/
│   │   ├── index.astro        # Blog listing
│   │   └── [slug].astro       # Dynamic route
│   └── legal/
│       ├── privacy-policy.astro
│       ├── terms-of-service.astro
│       └── cookie-policy.astro

❌ INCORRECT:
src/
├── pages/
│   ├── Index.astro            # Don't capitalize
│   ├── Features.astro         # Use lowercase
│   ├── privacy_policy.astro   # Never snake_case
│   └── Blog/                  # Folder should be lowercase
```

### Content Organization

```
✅ CORRECT:
src/
├── content/
│   ├── config.ts              # Content collections config
│   ├── features/
│   │   ├── multi-tenant-architecture.md
│   │   ├── security-features.md
│   │   └── analytics-dashboard.md
│   ├── blog/
│   │   ├── getting-started-with-bmoi.mdx
│   │   ├── best-practices-for-multi-tenancy.mdx
│   │   └── product-launch-announcement.md
│   └── legal/
│       ├── privacy-policy.md
│       ├── terms-of-service.md
│       └── cookie-policy.md

❌ INCORRECT:
src/
├── content/
│   ├── Features/              # Folder should be lowercase
│   ├── blog/
│   │   ├── Getting-Started.md # Don't capitalize
│   │   └── best_practices.md  # Never snake_case
```

---

## Import Path Conventions

### Absolute Imports (Preferred)

Use `@/` alias for absolute imports:

```typescript
✅ CORRECT:
import Header from '@/components/Header.astro';
import { formatDate } from '@/utils/format-date';
import { SITE_CONFIG } from '@/config/site-config';
import type { BlogPost } from '@/types/content-types';

❌ INCORRECT:
import Header from '../../../components/Header.astro';  // Too many ../
import { formatDate } from '../../utils/format-date';  // Use @/ alias
```

### Relative Imports

Only use relative imports for files in the same folder:

```typescript
✅ CORRECT:
// Inside src/components/Hero.astro
import Button from './ui/Button.astro';  // Same folder hierarchy

❌ INCORRECT:
import Button from '../components/ui/Button.astro';  // Use @/ instead
```

### Import Ordering

Group imports in this order:

```typescript
✅ CORRECT:
// 1. Astro/Framework imports
import { defineCollection, z } from 'astro:content';
import { Image } from 'astro:assets';

// 2. External libraries
import { format } from 'date-fns';

// 3. Internal components
import Header from '@/components/Header.astro';
import Footer from '@/components/Footer.astro';

// 4. Utilities/helpers
import { formatDate } from '@/utils/format-date';

// 5. Types
import type { BlogPost } from '@/types/content-types';

// 6. Styles (last)
import '@/styles/global.css';

❌ INCORRECT:
// Mixed order, hard to read
import type { BlogPost } from '@/types/content-types';
import '@/styles/global.css';
import { formatDate } from '@/utils/format-date';
import Header from '@/components/Header.astro';
import { format } from 'date-fns';
```

---

## Environment Variables

Use **UPPER_SNAKE_CASE** with `PUBLIC_` prefix for client-side variables:

```bash
✅ CORRECT:
# Client-side (accessible in browser)
PUBLIC_API_URL=https://api.bmoi.com
PUBLIC_APP_URL=https://app.bmoi.com
PUBLIC_SITE_URL=https://bmoi.com
PUBLIC_GA_ID=G-XXXXXXXXXX
PUBLIC_GTM_ID=GTM-XXXXXXX

# Server-side only (not prefixed with PUBLIC_)
SENDGRID_API_KEY=SG.xxxxx
FORMSPREE_ID=xxxxx

❌ INCORRECT:
public_api_url=https://api.bmoi.com   # Use UPPER_SNAKE_CASE
PublicApiUrl=https://api.bmoi.com     # Use UPPER_SNAKE_CASE
API_URL=https://api.bmoi.com          # Must prefix with PUBLIC_
```

**Important:** In Astro, only `PUBLIC_*` variables are accessible in client-side code.

---

## Git Commit Conventions

Use **Conventional Commits** format:

```bash
✅ CORRECT:
feat: add hero section with CTA button
fix: correct pricing table layout on mobile
docs: update README with deployment instructions
style: format code with Prettier
refactor: simplify contact form validation
perf: optimize image loading with lazy loading
test: add unit tests for date formatting
chore: update dependencies to latest versions

❌ INCORRECT:
Added hero section                    # Missing type prefix
Fixed bug                             # Not descriptive
Updated files                         # Too vague
FEAT: Add hero section                # Don't capitalize type
```

**Format:** `type(scope): description`

**Types:**
- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation changes
- `style` - Code formatting (not visual style)
- `refactor` - Code refactoring
- `perf` - Performance improvements
- `test` - Adding/updating tests
- `chore` - Maintenance tasks

---

## Branch Naming

Use **kebab-case** with type prefix:

```bash
✅ CORRECT:
feature/add-blog-section
feature/implement-contact-form
fix/hero-button-alignment
fix/mobile-navigation-menu
refactor/simplify-pricing-component
docs/update-deployment-guide

❌ INCORRECT:
feature_add_blog             # Never snake_case
AddBlogSection               # Don't use PascalCase
hero-button-fix              # Missing type prefix
```

---

## Astro-Specific Conventions

### Frontmatter Section

Always put imports and logic in the frontmatter (between `---`):

```astro
✅ CORRECT:
---
import Header from '@/components/Header.astro';
import { formatDate } from '@/utils/format-date';

interface Props {
  title: string;
}

const { title } = Astro.props;
const currentYear = new Date().getFullYear();
---

<html>
  <head><title>{title}</title></head>
  <body>
    <Header />
    <p>© {currentYear}</p>
  </body>
</html>

❌ INCORRECT:
<html>
  <head><title>{title}</title></head>  <!-- Props not defined -->
  <body>
    <script>
      // Don't put logic here, use frontmatter
      const currentYear = new Date().getFullYear();
    </script>
  </body>
</html>
```

### Component Slot Names

Use **camelCase** for named slots:

```astro
✅ CORRECT:
---
// BaseLayout.astro
---
<html>
  <head>
    <slot name="headContent" />
  </head>
  <body>
    <slot name="mainContent" />
    <slot name="footerContent" />
  </body>
</html>

<!-- Usage -->
<BaseLayout>
  <Fragment slot="headContent">
    <meta name="description" content="..." />
  </Fragment>
  <Fragment slot="mainContent">
    <h1>Content</h1>
  </Fragment>
</BaseLayout>

❌ INCORRECT:
<slot name="head-content" />    # Use camelCase not kebab-case
<slot name="MainContent" />     # Use camelCase not PascalCase
<slot name="footer_content" />  # Never snake_case
```

---

## TypeScript Type Definitions

### Interface vs Type

Use **Interface** for object shapes, **Type** for unions/primitives:

```typescript
✅ CORRECT:
// Interface for objects
interface BlogPost {
  title: string;
  slug: string;
  publishedAt: Date;
}

interface Props {
  title: string;
  description?: string;
}

// Type for unions/aliases
type PlanType = 'free' | 'plan1' | 'plan2' | 'plan3';
type Status = 'idle' | 'loading' | 'success' | 'error';
type ResponseData<T> = { data: T; error?: string };

❌ INCORRECT:
// Don't use type for simple objects (prefer interface)
type BlogPost = {
  title: string;
  slug: string;
};

// Don't use interface for unions (use type)
interface PlanType extends String {}  // Wrong approach
```

---

## Common Pitfalls to Avoid

### 1. Don't Mix Naming Conventions

```typescript
❌ AVOID:
const api_url = 'https://api.bmoi.com';  // snake_case
const SiteName = 'bMOI';                 // PascalCase for variable
function FormatDate() {}                 // PascalCase for function

✅ CORRECT:
const apiUrl = 'https://api.bmoi.com';   // camelCase
const siteName = 'bMOI';                 // camelCase
function formatDate() {}                 // camelCase
```

### 2. Don't Use Abbreviations Unless Standard

```typescript
❌ AVOID:
const usrNm = 'John';        // Unclear abbreviation
const btnTxt = 'Click me';   // Unclear abbreviation
const tmpData = [];          // What is tmp?

✅ CORRECT:
const userName = 'John';
const buttonText = 'Click me';
const temporaryData = [];

✅ ACCEPTABLE (standard abbreviations):
const apiUrl = 'https://...';    // API is standard
const htmlContent = '<div>';     // HTML is standard
const cssClass = 'text-blue';    // CSS is standard
const urlPath = '/pricing';      // URL is standard
```

### 3. Don't Use Single-Letter Variables (Except Loops)

```typescript
❌ AVOID:
const d = new Date();
const u = getUserData();
const r = await fetch(url);

✅ CORRECT:
const currentDate = new Date();
const userData = getUserData();
const response = await fetch(url);

✅ ACCEPTABLE (loops/iterations):
for (let i = 0; i < items.length; i++) {}
items.map((item, index) => {});
```

---

## AI Assistant Reminders

When working with this codebase, ALWAYS:

1. **Files:** PascalCase for components, kebab-case for pages/content
2. **Variables/Functions:** camelCase
3. **Constants:** UPPER_SNAKE_CASE
4. **Classes/Types:** PascalCase
5. **NEVER use snake_case** in TypeScript code
6. **Use @/ alias** for imports instead of ../../../
7. **Group imports** by type (framework → external → internal → types → styles)
8. **Commit messages:** Follow Conventional Commits (feat:, fix:, etc.)

If you're unsure, check this file or ask before implementing.

---

**Last Updated:** 2025-01-04
**Convention Version:** 1.0.0
