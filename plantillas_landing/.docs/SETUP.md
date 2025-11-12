# Setup Guide - bMOI Landing Page

This guide will help you set up the bMOI Landing Page project from scratch in **under 30 minutes**.

## Prerequisites

Before starting, ensure you have these tools installed:

### Required Software

| Tool | Minimum Version | Purpose |
|------|----------------|---------|
| **Node.js** | 20.x | JavaScript runtime |
| **npm** | 10.x | Package manager |
| **Git** | 2.x | Version control |
| **VS Code** | Latest | Code editor (recommended) |

### Verify Installation

```bash
# Check Node.js version
node --version
# Should output: v20.x.x or higher

# Check npm version
npm --version
# Should output: 10.x.x or higher

# Check Git version
git --version
# Should output: git version 2.x.x
```

### Optional (Recommended)

- **Docker Desktop** - For containerized development
- **Postman/Insomnia** - For API testing

---

## Step 1: Initialize Astro Project

### Create New Project

```bash
# Create Astro project with TypeScript
npm create astro@latest bmoi-landing-page

# When prompted, choose:
# - How would you like to start your new project? → Empty
# - Do you plan to write TypeScript? → Yes
# - How strict should TypeScript be? → Strict
# - Install dependencies? → Yes
# - Initialize a new git repository? → Yes
```

### Navigate to Project

```bash
cd bmoi-landing-page
```

---

## Step 2: Install Dependencies

### Core Dependencies

```bash
# Tailwind CSS (styling)
npx astro add tailwind

# When prompted:
# - Install Tailwind CSS? → Yes

# React (for interactive islands)
npx astro add react

# When prompted:
# - Install React? → Yes
```

### Additional Packages

```bash
npm install --save \
  @astrojs/mdx \
  @astrojs/sitemap \
  @astrojs/image \
  date-fns \
  clsx \
  tailwind-merge
```

### Dev Dependencies

```bash
npm install --save-dev \
  @types/node \
  prettier \
  prettier-plugin-astro \
  prettier-plugin-tailwindcss
```

---

## Step 3: Configure Astro

### Update `astro.config.mjs`

```javascript
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';
import react from '@astrojs/react';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://bmoi.com',
  integrations: [
    tailwind({
      // Allow writing nested CSS
      applyBaseStyles: false,
    }),
    react(),
    mdx(),
    sitemap(),
  ],
  image: {
    domains: ['images.unsplash.com'],
  },
  vite: {
    resolve: {
      alias: {
        '@': '/src',
      },
    },
  },
});
```

---

## Step 4: Configure Tailwind CSS

### Update `tailwind.config.mjs`

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}'],
  darkMode: 'class', // Enable dark mode with class strategy
  theme: {
    extend: {
      colors: {
        // Brand colors
        brand: {
          50: '#eff6ff',
          100: '#dbeafe',
          200: '#bfdbfe',
          300: '#93c5fd',
          400: '#60a5fa',
          500: '#3b82f6',
          600: '#2563eb', // Primary
          700: '#1d4ed8',
          800: '#1e40af',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
      container: {
        center: true,
        padding: {
          DEFAULT: '1rem',
          sm: '2rem',
          lg: '4rem',
          xl: '5rem',
          '2xl': '6rem',
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
};
```

### Install Tailwind Plugins

```bash
npm install --save @tailwindcss/typography @tailwindcss/forms
```

---

## Step 5: Configure TypeScript

### Update `tsconfig.json`

```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "jsx": "react-jsx",
    "jsxImportSource": "react"
  }
}
```

---

## Step 6: Create Folder Structure

```bash
# Create directories
mkdir -p src/{components,components/ui,layouts,pages,styles,utils,content,content/features,content/blog,content/legal}

# Create public directories
mkdir -p public/{images,icons,fonts}
```

**Final structure:**

```
src/
├── components/
│   └── ui/
├── content/
│   ├── features/
│   ├── blog/
│   └── legal/
├── layouts/
├── pages/
├── styles/
└── utils/
```

---

## Step 7: Setup Global Styles

### Create `src/styles/global.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  /* Custom base styles */
  :root {
    --color-background: 255 255 255;
    --color-foreground: 15 23 42;
    --color-primary: 37 99 235;
    --color-secondary: 100 116 139;
  }

  .dark {
    --color-background: 15 23 42;
    --color-foreground: 241 245 249;
    --color-primary: 96 165 250;
    --color-secondary: 148 163 184;
  }

  * {
    @apply border-border;
  }

  body {
    @apply bg-white text-slate-900 dark:bg-slate-900 dark:text-slate-50;
  }

  /* Smooth scrolling */
  html {
    scroll-behavior: smooth;
  }

  /* Font rendering */
  body {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
}

@layer components {
  /* Container */
  .container {
    @apply mx-auto px-4 sm:px-6 lg:px-8;
  }

  /* Button variants */
  .btn-primary {
    @apply inline-flex items-center justify-center rounded-lg bg-brand-600 px-6 py-3 text-sm font-semibold text-white shadow-sm hover:bg-brand-700 focus:outline-none focus:ring-2 focus:ring-brand-500 focus:ring-offset-2 transition-colors;
  }

  .btn-secondary {
    @apply inline-flex items-center justify-center rounded-lg border border-slate-300 bg-white px-6 py-3 text-sm font-semibold text-slate-900 shadow-sm hover:bg-slate-50 focus:outline-none focus:ring-2 focus:ring-brand-500 focus:ring-offset-2 transition-colors dark:border-slate-700 dark:bg-slate-800 dark:text-slate-50 dark:hover:bg-slate-700;
  }
}

@layer utilities {
  /* Text gradient */
  .text-gradient {
    @apply bg-gradient-to-r from-brand-600 to-purple-600 bg-clip-text text-transparent;
  }
}
```

---

## Step 8: Configure Content Collections

### Create `src/content/config.ts`

```typescript
import { defineCollection, z } from 'astro:content';

const featuresCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    icon: z.string().optional(),
    order: z.number().optional(),
  }),
});

const blogCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    publishedAt: z.date(),
    updatedAt: z.date().optional(),
    author: z.string(),
    authorImage: z.string().optional(),
    image: z.string().optional(),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
  }),
});

const legalCollection = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    lastUpdated: z.date(),
  }),
});

export const collections = {
  features: featuresCollection,
  blog: blogCollection,
  legal: legalCollection,
};
```

---

## Step 9: Create Base Layout

### Create `src/layouts/BaseLayout.astro`

```astro
---
import '@/styles/global.css';

interface Props {
  title: string;
  description: string;
  image?: string;
}

const { title, description, image = '/images/og-default.jpg' } = Astro.props;
const canonicalUrl = new URL(Astro.url.pathname, Astro.site);
---

<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="generator" content={Astro.generator} />

    <!-- SEO Meta Tags -->
    <title>{title} | bMOI</title>
    <meta name="description" content={description} />
    <link rel="canonical" href={canonicalUrl} />

    <!-- Open Graph -->
    <meta property="og:title" content={title} />
    <meta property="og:description" content={description} />
    <meta property="og:image" content={image} />
    <meta property="og:url" content={canonicalUrl} />
    <meta property="og:type" content="website" />

    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:title" content={title} />
    <meta name="twitter:description" content={description} />
    <meta name="twitter:image" content={image} />

    <!-- Theme Script (prevent flash) -->
    <script is:inline>
      const theme = localStorage.getItem('theme');
      const systemDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

      if (theme === 'dark' || (!theme && systemDark)) {
        document.documentElement.classList.add('dark');
      }
    </script>
  </head>
  <body>
    <slot />
  </body>
</html>
```

---

## Step 10: Create Theme Toggle Component

### Create `src/components/ThemeToggle.tsx`

```typescript
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
      className="rounded-lg p-2 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors"
      aria-label="Toggle theme"
    >
      {theme === 'light' ? (
        <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" />
        </svg>
      ) : (
        <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
        </svg>
      )}
    </button>
  );
}
```

---

## Step 11: Create First Page

### Create `src/pages/index.astro`

```astro
---
import BaseLayout from '@/layouts/BaseLayout.astro';
---

<BaseLayout
  title="Plataforma Multi-Tenant para Empresas"
  description="bMOI es la plataforma completa para gestionar tu negocio con múltiples tenants"
>
  <main class="container mx-auto px-4 py-12">
    <div class="text-center">
      <h1 class="text-4xl font-bold text-gradient mb-4">
        Bienvenido a bMOI
      </h1>
      <p class="text-xl text-slate-600 dark:text-slate-400 mb-8">
        Tu plataforma multi-tenant lista para crecer
      </p>
      <div class="flex gap-4 justify-center">
        <a href="/features" class="btn-primary">
          Explorar Características
        </a>
        <a href="/pricing" class="btn-secondary">
          Ver Precios
        </a>
      </div>
    </div>
  </main>
</BaseLayout>
```

---

## Step 12: Configure Environment Variables

### Create `.env.example`

```bash
# API URLs
PUBLIC_API_URL=https://api.bmoi.com
PUBLIC_APP_URL=https://app.bmoi.com
PUBLIC_SITE_URL=https://bmoi.com

# Analytics
PUBLIC_GA_ID=G-XXXXXXXXXX
PUBLIC_GTM_ID=GTM-XXXXXXX

# Optional: Contact form service
PUBLIC_FORMSPREE_ID=xxxxx
```

### Create `.env` (local)

```bash
cp .env.example .env

# Edit .env with your local values
```

---

## Step 13: Configure Prettier

### Create `.prettierrc.mjs`

```javascript
export default {
  semi: true,
  singleQuote: true,
  tabWidth: 2,
  trailingComma: 'es5',
  printWidth: 100,
  plugins: ['prettier-plugin-astro', 'prettier-plugin-tailwindcss'],
  overrides: [
    {
      files: '*.astro',
      options: {
        parser: 'astro',
      },
    },
  ],
};
```

---

## Step 14: Update Package Scripts

### Update `package.json`

```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro check && astro build",
    "preview": "astro preview",
    "astro": "astro",
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

---

## Step 15: Run Development Server

```bash
# Start development server
npm run dev

# Open browser
# Visit: http://localhost:4321
```

You should see your landing page running! 🎉

---

## Step 16: Docker Setup (Optional)

### Create `Dockerfile.dev`

```dockerfile
FROM node:20-alpine

WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source
COPY . .

# Expose port
EXPOSE 4321

# Start dev server
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

### Create `docker-compose.yml`

```yaml
version: '3.8'

services:
  landing-dev:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: bmoi-landing-dev
    ports:
      - "4321:4321"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
    command: npm run dev -- --host 0.0.0.0
```

### Run with Docker

```bash
# Start
docker compose up -d

# View logs
docker compose logs -f

# Stop
docker compose down
```

---

## Troubleshooting

### Issue: Port 4321 already in use

```bash
# Find and kill process using port 4321
# macOS/Linux:
lsof -ti:4321 | xargs kill -9

# Windows:
netstat -ano | findstr :4321
taskkill /PID <PID> /F
```

### Issue: Module not found errors

```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Issue: TypeScript errors

```bash
# Run type check
npm run astro check

# Sync content collection types
npm run astro sync
```

---

## Next Steps

After completing setup:

1. ✅ **Read [ARQUITECTURA.md](ARQUITECTURA.md)** - Understand architecture decisions
2. ✅ **Read [CODING_STANDARDS.md](CODING_STANDARDS.md)** - Learn coding standards
3. ✅ **Read [EJEMPLOS_CODIGO.md](EJEMPLOS_CODIGO.md)** - See code examples
4. ✅ **Start building components** - Create Header, Footer, Hero, etc.
5. ✅ **Add content** - Write blog posts, feature descriptions

---

## Useful Commands

```bash
# Development
npm run dev              # Start dev server
npm run build            # Build for production
npm run preview          # Preview production build

# Code quality
npm run format           # Format code with Prettier
npm run astro check      # Type check

# Content
npm run astro sync       # Sync content collection types

# Docker
docker compose up -d     # Start in Docker
docker compose logs -f   # View logs
docker compose down      # Stop Docker
```

---

**Setup complete! 🚀**

You now have a fully configured Astro landing page with:
- ✅ TypeScript
- ✅ Tailwind CSS
- ✅ Dark mode support
- ✅ Content collections
- ✅ SEO optimizations
- ✅ Responsive design ready
