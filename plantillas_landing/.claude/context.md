# Landing Page Context - bMOI Project

## Project Identity

**Project Name:** bMOI Landing Page
**Type:** Static Marketing Website
**Repository:** bmoi-landing-page
**Part of:** bMOI Multi-Tenant Ecosystem
**Status:** Template/Plantilla (Starting from Zero)

## Core Purpose

The bMOI Landing Page is a **static, informative marketing website** designed to:
- Present the bMOI platform and its features
- Convert visitors into users (sign-up funnel)
- Provide SEO-optimized content for discoverability
- Support **light/dark theme** for better user experience
- Be **fully responsive** (mobile-first design)
- Be deployed to S3/CDN for maximum performance and cost-efficiency

**This is NOT:**
- A web application with authentication
- A dashboard or control panel
- A server-rendered dynamic site
- Part of the multi-tenant system (it's public-facing)

## Technology Stack

### Core Framework: Astro 4 ✅

**Decision:** Use **Astro 4** (not Next.js, not Nuxt, not plain HTML)

**Why Astro:**
- ✅ **Zero JavaScript by default** - Ships 0 JS unless you need it
- ✅ **Perfect for static sites** - Built specifically for content-focused sites
- ✅ **Islands Architecture** - Interactive components only where needed
- ✅ **Multi-framework support** - Can use React, Vue, Svelte in same project
- ✅ **Built-in optimizations** - Image optimization, CSS minification, etc.
- ✅ **SSG native** - Static Site Generation is the default
- ✅ **Markdown/MDX support** - Perfect for content management
- ✅ **TypeScript native** - Full type safety
- ✅ **SEO friendly** - Perfect meta tags, sitemap, RSS support

**Alternatives discarded:**
- ❌ Next.js: Too heavy for static site, more oriented to apps
- ❌ Nuxt: Great but Vue-specific, Astro is more flexible
- ❌ Gatsby: Older technology, slower builds
- ❌ Plain HTML: No modern tooling, hard to maintain

### Styling: Tailwind CSS 4 ✅

**Why Tailwind:**
- ✅ Utility-first approach
- ✅ No unused CSS in production
- ✅ Fast prototyping
- ✅ Consistent design system
- ✅ Great with component frameworks

### TypeScript 5 ✅

**Why TypeScript:**
- ✅ Type safety in components
- ✅ Better IDE support
- ✅ Catch errors at build time
- ✅ Self-documenting code

### Animation: Framer Motion / GSAP (optional) ✅

**Why:**
- ✅ Smooth scroll animations
- ✅ Eye-catching hero sections
- ✅ Modern landing page feel

### Forms: React Hook Form + Zod (for sign-up) ✅

**Why:**
- ✅ Validation with Zod schemas
- ✅ Type-safe forms
- ✅ Best performance

## Architecture Decisions

### 1. Static Site Generation (SSG) ✅

**Decision:** Build entire site as static HTML at build time

**Benefits:**
- ⚡ **Fastest possible load times** - Pre-rendered HTML
- 💰 **Cheapest hosting** - Just S3 + CloudFront
- 🔒 **Maximum security** - No server to hack
- 📈 **Perfect SEO** - Search engines love static HTML
- 🌍 **Global CDN** - Deploy to edge locations worldwide

**Trade-off accepted:** Content updates require rebuild/redeploy (acceptable for marketing site)

### 2. Deployment: S3 + CloudFront ✅

**Primary deployment:**
```
Build → S3 Bucket → CloudFront CDN → Route 53 DNS
```

**Alternative platforms supported:**
- Netlify (easiest, free tier generous)
- Vercel (great DX, auto-deployments)
- Cloudflare Pages (fastest global CDN)
- AWS Amplify (if using AWS ecosystem)

**Why S3 + CloudFront:**
- ✅ **$0.50/month** for low-traffic sites
- ✅ **Infinite scalability** - Handle viral traffic
- ✅ **Global CDN** - Low latency worldwide
- ✅ **99.99% uptime** - AWS reliability
- ✅ **HTTPS included** - Free SSL with ACM

### 3. Content Strategy: File-Based ✅

**Decision:** Content in Markdown/MDX files, not CMS

**Structure:**
```
src/
├── content/
│   ├── features/
│   │   ├── multi-tenant.md
│   │   ├── security.md
│   │   └── analytics.md
│   ├── pricing/
│   │   └── plans.yaml
│   └── blog/
│       ├── launch-announcement.md
│       └── best-practices.md
```

**Why file-based:**
- ✅ Version control for content
- ✅ No CMS infrastructure needed
- ✅ Markdown is readable and portable
- ✅ TypeScript validation via Zod schemas
- ✅ Easy for developers to update

**Future:** Can add headless CMS (Contentful, Strapi) if marketing team needs GUI

### 4. Theming & Responsive Design ✅

**Decision:** Support light/dark theme + mobile-first responsive design

**Theme System:**
- **Light theme** (default) - White background, dark text
- **Dark theme** - Dark background, light text
- **System preference detection** - Auto-detect user's OS preference
- **Manual toggle** - User can override system preference
- **Persistence** - Theme choice stored in localStorage

**Implementation:**
```html
<!-- Theme detection script (prevents flash) -->
<script is:inline>
  const theme = localStorage.getItem('theme');
  const systemDark = window.matchMedia('(prefers-color-scheme: dark)').matches;

  if (theme === 'dark' || (!theme && systemDark)) {
    document.documentElement.classList.add('dark');
  }
</script>
```

**Tailwind Dark Mode:**
```html
<!-- Easy dark mode variants -->
<div class="bg-white dark:bg-slate-900">
  <p class="text-slate-900 dark:text-slate-50">Content</p>
</div>
```

**Responsive Design Strategy:**
- **Mobile-first approach** - Design for mobile, enhance for desktop
- **Breakpoints:** sm (640px), md (768px), lg (1024px), xl (1280px), 2xl (1536px)
- **Tailwind utilities:** Responsive classes (e.g., `md:grid-cols-2 lg:grid-cols-3`)
- **Touch-friendly:** Minimum 44x44px for interactive elements
- **Performance:** Optimized images with responsive srcset

**Why important:**
- ✅ **Better UX** - Users can choose preferred theme
- ✅ **Accessibility** - Dark mode reduces eye strain
- ✅ **Modern expectation** - Users expect theme options
- ✅ **Mobile traffic** - 60%+ of web traffic is mobile
- ✅ **SEO** - Mobile-friendly is a ranking factor

### 5. Internationalization (i18n) - Optional Phase 2 ✅

**Initial:** English only
**Future:** Spanish, Portuguese (Latin America focus)

**Implementation with Astro i18n:**
```typescript
// astro.config.mjs
export default defineConfig({
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'es', 'pt'],
  }
});
```

### 6. Analytics & Tracking ✅

**Essential tracking:**
- Google Analytics 4 (GA4)
- Facebook Pixel (for ads)
- Google Tag Manager (centralized tags)

**Privacy-first alternative:**
- Plausible Analytics (GDPR compliant, lightweight)

### 7. Performance Budget ✅

**Targets (Lighthouse scores):**
- Performance: **95+**
- Accessibility: **100**
- Best Practices: **100**
- SEO: **100**

**Specific metrics:**
- First Contentful Paint (FCP): < 1.2s
- Largest Contentful Paint (LCP): < 2.5s
- Time to Interactive (TTI): < 3.0s
- Total page size: < 200KB (before images)
- Images: WebP format, lazy-loaded

## Folder Structure

```
plantillas_landing/
├── .claude/                    # AI Assistant context
│   ├── context.md             # This file
│   ├── conventions.md         # Naming conventions
│   └── session-state.md       # Development state
├── .copilot/
│   └── instructions.md        # GitHub Copilot instructions
├── .cursorrules               # Cursor AI rules
├── .docs/                     # Documentation
│   ├── README.md              # Project overview
│   ├── SETUP.md               # Setup guide
│   ├── ARQUITECTURA.md        # Architecture decisions
│   ├── DEPLOYMENT.md          # Deployment guide (S3, Netlify, Vercel)
│   ├── CODING_STANDARDS.md    # Coding standards
│   ├── EJEMPLOS_CODIGO.md     # Code examples
│   └── SEO_GUIDE.md           # SEO best practices
├── public/                    # Static assets (copied as-is)
│   ├── favicon.svg
│   ├── robots.txt
│   ├── sitemap.xml
│   └── images/
├── src/
│   ├── components/            # Reusable UI components
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── Hero.astro
│   │   ├── Features.astro
│   │   ├── Pricing.astro
│   │   ├── Testimonials.astro
│   │   ├── CTA.astro
│   │   └── ui/                # Base UI components
│   ├── content/               # Markdown/MDX content
│   │   ├── config.ts          # Content collections config
│   │   ├── features/
│   │   ├── blog/
│   │   └── legal/
│   ├── layouts/               # Page layouts
│   │   └── BaseLayout.astro
│   ├── pages/                 # Routes (file-based routing)
│   │   ├── index.astro        # Home page
│   │   ├── features.astro
│   │   ├── pricing.astro
│   │   ├── about.astro
│   │   ├── contact.astro
│   │   └── blog/
│   │       └── [slug].astro
│   ├── styles/
│   │   └── global.css         # Global styles + Tailwind
│   └── env.d.ts
├── .env.example               # Environment variables template
├── .gitignore
├── .dockerignore
├── astro.config.mjs           # Astro configuration
├── tailwind.config.mjs        # Tailwind configuration
├── tsconfig.json              # TypeScript configuration
├── package.json
├── Dockerfile                 # Production build
├── docker-compose.yml         # Local development
├── nginx.conf                 # Nginx config for serving static files
├── CLAUDE.md                  # Quick reference for Claude Code
└── README_PLANTILLAS.md       # Complete template guide
```

## Development Workflow

### Local Development (With Docker - RECOMMENDED)

```bash
# Start development server with hot-reload
docker compose up -d

# View at http://localhost:4321
# Changes auto-reload
```

### Local Development (Without Docker)

```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Build & Deploy

```bash
# Build static site
npm run build
# Output: dist/ folder with static HTML

# Deploy to S3
aws s3 sync dist/ s3://bmoi-landing-bucket --delete
aws cloudfront create-invalidation --distribution-id XXX --paths "/*"

# Or deploy to Netlify
netlify deploy --prod --dir=dist

# Or deploy to Vercel
vercel --prod
```

## Key Pages & Sections

### 1. Home Page (/)
- **Hero Section** - Main value proposition, CTA
- **Features Overview** - 3-4 key features with icons
- **How It Works** - 3-step process
- **Social Proof** - Testimonials or logos
- **Pricing Teaser** - Basic plans preview
- **Final CTA** - "Get Started" button

### 2. Features Page (/features)
- **Feature Grid** - Detailed feature cards
- **Use Cases** - Industry-specific examples
- **Integrations** - Third-party integrations
- **Demo Video** - Product walkthrough

### 3. Pricing Page (/pricing)
- **Plan Comparison Table** - Free, Plan1, Plan2, Plan3
- **FAQ** - Common pricing questions
- **CTA** - "Start Free Trial"

### 4. About Page (/about)
- **Mission & Vision** - Why bMOI exists
- **Team** - Founders/key team members
- **Company Values** - What we believe

### 5. Contact Page (/contact)
- **Contact Form** - Sales inquiries
- **Support Links** - Link to support portal
- **Social Media** - Links to social profiles

### 6. Blog (/blog)
- **Article Listing** - Latest posts
- **Categories/Tags** - Content organization
- **Individual Posts** - MDX-powered articles

### 7. Legal Pages
- **/privacy** - Privacy policy
- **/terms** - Terms of service
- **/cookies** - Cookie policy

## Integration with bMOI Ecosystem

### Sign-Up Flow

Landing page → Sign Up Form → Backend API → Admin Panel

**Flow:**
1. User clicks "Get Started" on landing page
2. Form captures: email, password, company name, plan selection
3. POST to `https://api.bmoi.com/auth/register`
4. Backend creates tenant + admin user
5. Redirect to `https://app.bmoi.com/dashboard?new=true`

**Form data:**
```typescript
interface SignUpData {
  email: string;
  password: string;
  companyName: string;
  plan: 'free' | 'plan1' | 'plan2' | 'plan3';
  acceptTerms: boolean;
}
```

### API Endpoints Used

Landing page only calls these public endpoints:
- `POST /auth/register` - Create new tenant account
- `POST /contact/submit` - Contact form submissions
- `GET /public/plans` - Fetch pricing plans (optional, can be hardcoded)

### Analytics & Tracking

**User journey tracking:**
- Landing page view → Feature exploration → Pricing view → Sign up
- UTM parameters: `?utm_source=google&utm_campaign=launch`
- Conversion tracking: Sign-up button clicks, form submissions

## Environment Variables

```bash
# .env
PUBLIC_API_URL=https://api.bmoi.com
PUBLIC_APP_URL=https://app.bmoi.com
PUBLIC_SITE_URL=https://bmoi.com

# Analytics
PUBLIC_GA_ID=G-XXXXXXXXXX
PUBLIC_GTM_ID=GTM-XXXXXXX

# Optional: Contact form service (FormSpree, SendGrid, etc.)
PUBLIC_FORMSPREE_ID=xxxxx
SENDGRID_API_KEY=SG.xxxxx
```

## Docker Configuration

### Development (docker-compose.yml)

```yaml
services:
  landing-dev:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "4321:4321"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
```

### Production (Dockerfile)

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```

## SEO Strategy

### On-Page SEO
- **Title tags** - Unique per page, < 60 characters
- **Meta descriptions** - Compelling, < 160 characters
- **Header hierarchy** - Proper H1, H2, H3 structure
- **Alt text** - All images have descriptive alt text
- **Internal linking** - Related pages linked
- **Canonical URLs** - Avoid duplicate content

### Technical SEO
- **Sitemap.xml** - Auto-generated by Astro
- **Robots.txt** - Allow all crawling
- **Schema.org markup** - Organization, WebSite, Article
- **Open Graph tags** - Social media previews
- **Twitter Cards** - Twitter-specific previews
- **Page speed** - < 3s load time
- **Mobile-friendly** - Responsive design
- **HTTPS** - SSL certificate required

### Content SEO
- **Target keywords** - "multi-tenant SaaS", "business management platform"
- **Blog content** - 1-2 articles per week
- **Long-form content** - 1500+ word guides
- **Internal linking** - Blog ↔ Product pages

## Performance Optimizations

### Images
- **Format:** WebP with JPEG fallback
- **Lazy loading:** All below-the-fold images
- **Responsive images:** srcset for different screen sizes
- **CDN:** CloudFront for image delivery
- **Compression:** TinyPNG or similar

### JavaScript
- **Islands Architecture:** Only hydrate interactive components
- **Code splitting:** Automatic with Astro
- **Defer non-critical JS:** Analytics, chat widgets
- **No jQuery:** Modern vanilla JS only

### CSS
- **Critical CSS:** Inlined in `<head>`
- **Unused CSS removed:** By Tailwind purge
- **Minification:** Automatic in production
- **CSS-in-JS avoided:** Use Tailwind classes

### Fonts
- **System fonts preferred:** Or subset Google Fonts
- **Preload critical fonts:** `<link rel="preload">`
- **Font display swap:** `font-display: swap`

## Security Considerations

### Content Security Policy (CSP)

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               script-src 'self' https://www.googletagmanager.com;
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;">
```

### Other Headers
- **X-Frame-Options:** DENY (prevent clickjacking)
- **X-Content-Type-Options:** nosniff
- **Referrer-Policy:** strict-origin-when-cross-origin
- **Permissions-Policy:** Restrict unnecessary browser features

### Form Security
- **CORS:** Proper CORS headers on API
- **Rate limiting:** On backend API
- **Input validation:** Client + server-side
- **HTTPS only:** Redirect HTTP to HTTPS

## Accessibility (a11y)

### WCAG 2.1 Level AA Compliance

**Must-haves:**
- ✅ **Keyboard navigation** - All interactive elements focusable
- ✅ **Color contrast** - 4.5:1 minimum for text
- ✅ **Alt text** - All images have meaningful descriptions
- ✅ **ARIA labels** - For icon buttons, complex widgets
- ✅ **Focus indicators** - Visible focus states
- ✅ **Semantic HTML** - Proper heading levels, landmarks
- ✅ **Skip links** - "Skip to main content"
- ✅ **Form labels** - All inputs have labels

### Testing Tools
- Lighthouse accessibility audit
- axe DevTools browser extension
- WAVE Web Accessibility Evaluation Tool

## Monitoring & Analytics

### Performance Monitoring
- **Google PageSpeed Insights** - Weekly checks
- **Lighthouse CI** - Automated in GitHub Actions
- **WebPageTest** - Detailed performance analysis

### Uptime Monitoring
- **Uptime Robot** - Free, 5-minute intervals
- **Pingdom** - More detailed monitoring
- **CloudWatch Alarms** - If using AWS

### Error Tracking
- **Sentry** - JavaScript error tracking (optional for static site)
- **Google Tag Manager** - Custom error events

## AI Assistant Guidelines

### When Working on This Project

**ALWAYS remember:**
- This is a **static landing page**, not a web application
- Framework is **Astro 4** (not Next.js, not React SPA)
- Deployment target is **S3 + CloudFront** (or Netlify/Vercel)
- Content is in **Markdown/MDX files**, not database
- JavaScript should be **minimal** (Islands Architecture)
- **Performance is critical** - aim for 95+ Lighthouse score
- **SEO is critical** - proper meta tags, schema markup
- **Mobile-first** - responsive design mandatory

**NEVER:**
- Create server-side API routes (Astro SSR not used)
- Use heavy JavaScript frameworks (React SPA, Vue SPA)
- Implement authentication (that's in admin panel)
- Use server-side rendering (SSR) - we use SSG only
- Ignore performance (lazy load images, minimize JS)
- Forget meta tags (SEO is crucial)

### Preferred AI Workflow

1. **Start task** → Read `.claude/context.md` (this file)
2. **Check conventions** → Read `.claude/conventions.md`
3. **Understand current state** → Read `.claude/session-state.md`
4. **Implement** → Follow coding standards in `.docs/CODING_STANDARDS.md`
5. **Test** → Check examples in `.docs/EJEMPLOS_CODIGO.md`
6. **Update state** → Update `.claude/session-state.md` before ending

## Next Steps (From Zero to Production)

### Phase 0: Setup (Day 1)
- [ ] Initialize Astro project: `npm create astro@latest`
- [ ] Configure Tailwind CSS
- [ ] Setup TypeScript
- [ ] Create folder structure
- [ ] Configure Docker (Dockerfile, docker-compose.yml)
- [ ] Create .env.example

### Phase 1: Core Pages (Days 2-5)
- [ ] Create BaseLayout with Header/Footer
- [ ] Implement Home page with Hero section
- [ ] Create Features page
- [ ] Create Pricing page
- [ ] Create About page
- [ ] Create Contact page with form

### Phase 2: Content & Blog (Days 6-8)
- [ ] Setup content collections
- [ ] Create blog layout
- [ ] Write initial blog posts (3-5 articles)
- [ ] Implement blog listing page
- [ ] Create legal pages (Privacy, Terms)

### Phase 3: SEO & Performance (Days 9-10)
- [ ] Add meta tags to all pages
- [ ] Implement Schema.org markup
- [ ] Generate sitemap.xml
- [ ] Optimize images (WebP, lazy loading)
- [ ] Add Open Graph tags
- [ ] Configure robots.txt

### Phase 4: Integrations (Days 11-12)
- [ ] Integrate Google Analytics 4
- [ ] Add Google Tag Manager
- [ ] Implement sign-up form with API integration
- [ ] Add contact form (FormSpree or custom)
- [ ] Configure social media sharing

### Phase 5: Deployment (Days 13-14)
- [ ] Setup S3 bucket + CloudFront
- [ ] Configure custom domain
- [ ] Setup SSL certificate (ACM)
- [ ] Create deployment script
- [ ] Configure CI/CD (GitHub Actions)
- [ ] Test production deployment

### Phase 6: Optimization (Day 15)
- [ ] Run Lighthouse audit (target 95+)
- [ ] Optimize bundle size
- [ ] Test on mobile devices
- [ ] Cross-browser testing
- [ ] Performance monitoring setup

## Additional Resources

### Documentation Files (`.docs/` folder)

- **`.docs/README.md`** - Project overview with responsive design and dark mode
- **`.docs/SETUP.md`** - Complete step-by-step setup guide (includes theme toggle setup)
- **`.docs/ARQUITECTURA.md`** - Architecture decisions (why Astro, responsive strategy, dark mode implementation)
- **`.docs/DEPLOYMENT.md`** - Deployment guides (S3, Netlify, Vercel, Cloudflare Pages)
- **`.docs/CODING_STANDARDS.md`** - Coding standards with responsive and dark mode examples
- **`.docs/EJEMPLOS_CODIGO.md`** - Complete code examples (includes ThemeToggle component, responsive components)
- **`.docs/SEO_GUIDE.md`** - SEO best practices and implementation guide

### Other Important Files

- **`CLAUDE.md`** - Quick reference guide for Claude Code
- **`README_PLANTILLAS.md`** - Complete template usage guide
- **`.claude/conventions.md`** - Mandatory naming conventions
- **`.claude/session-state.md`** - Development state tracking

## Related Documentation

- **Backend API:** See `bo-backend-api/CLAUDE.md` for API endpoints
- **Admin Panel:** See `plantillas_admin/.claude/context.md` for web app integration
- **Mobile App:** See `plantillas_mobile/.claude/context.md` for mobile sign-up flow

---

**Last Updated:** 2025-01-04
**Template Version:** 1.0.0
**Status:** Initial Creation
