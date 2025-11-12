# 📄 README_PLANTILLAS.md - Landing Page Template Guide

## Overview

This document is the **complete guide** for using the bMOI Landing Page templates. These are **instruction templates** (not working code) designed to guide developers and AI assistants in building the static marketing website from scratch.

**Template Type:** Instruction/Documentation templates
**Purpose:** Guide for creating Astro 4 static landing page
**Target Deployment:** AWS S3 + CloudFront (or Netlify/Vercel)

---

## 🎯 What Are These Templates?

These templates provide **comprehensive instructions** for building the bMOI landing page with:

✅ **Complete documentation** for AI assistants (Claude Code, GitHub Copilot, Cursor)
✅ **Architecture decisions** explained (why Astro vs Next.js, etc.)
✅ **Naming conventions** enforced across the project
✅ **Code examples** for common patterns
✅ **SEO best practices** built-in
✅ **Performance optimization** strategies
✅ **Docker configuration** for consistent development
✅ **Deployment guides** for S3, Netlify, Vercel

---

## 📁 Template Structure

```
plantillas_landing/
├── .claude/                        # AI Assistant Context (Claude Code)
│   ├── context.md                  # Complete project context & architecture
│   ├── conventions.md              # Mandatory naming conventions
│   └── session-state.md            # Development state tracking
│
├── .copilot/                       # GitHub Copilot Instructions
│   └── instructions.md             # Copilot-specific guidance
│
├── .cursorrules                    # Cursor AI Rules
│
├── .docs/                          # Comprehensive Documentation
│   ├── README.md                   # Project overview
│   ├── SETUP.md                    # Step-by-step setup guide
│   ├── ARQUITECTURA.md             # Architecture decisions (Astro vs Next.js, etc.)
│   ├── DEPLOYMENT.md               # Deployment guide (S3, Netlify, Vercel)
│   ├── CODING_STANDARDS.md         # Coding standards with examples
│   ├── EJEMPLOS_CODIGO.md          # Complete code examples
│   └── SEO_GUIDE.md                # SEO best practices
│
├── CLAUDE.md                       # Quick reference for Claude Code
├── README.md                       # Standard README
├── README_PLANTILLAS.md            # This file (template guide)
├── .env.example                    # Environment variables template
├── .gitignore                      # Git ignore rules
├── .dockerignore                   # Docker ignore rules
├── Dockerfile                      # Production Docker build
├── Dockerfile.dev                  # Development Docker build
├── docker-compose.yml              # Docker development environment
└── nginx.conf                      # Nginx config for serving static files
```

---

## 📚 Documentation Files Explained

### `.claude/` Folder - AI Assistant Context

#### `context.md` (10KB)
**Purpose:** Complete project context for Claude Code

**Contains:**
- Project identity and purpose
- Technology stack decisions (Astro 4, Tailwind CSS)
- Architecture decisions explained (why Astro vs Next.js)
- Deployment strategy (S3 + CloudFront)
- Folder structure breakdown
- Development workflow
- Key pages & sections
- Integration with bMOI ecosystem
- SEO strategy
- Performance optimizations
- Security considerations
- Accessibility guidelines
- Monitoring & analytics
- Development phases breakdown
- Next steps from zero to production

**When to read:** First thing when starting any work on this project

#### `conventions.md` (14KB)
**Purpose:** Mandatory naming conventions

**Contains:**
- File naming conventions (PascalCase, kebab-case)
- Code naming conventions (camelCase, UPPER_SNAKE_CASE)
- Folder structure conventions
- Import path conventions (@/ alias)
- Environment variable naming
- Git commit conventions (Conventional Commits)
- Branch naming conventions
- Astro-specific conventions
- TypeScript type definitions
- Common pitfalls to avoid

**When to read:** Before writing any code

#### `session-state.md` (5KB)
**Purpose:** Track development progress

**Contains:**
- Current project status
- Phase progress (0-6)
- Recent changes log
- Active development session info
- Technical decisions made
- Problems encountered & solutions
- Files created/modified
- Next steps

**When to update:** At the end of each development session

### `.copilot/` Folder - GitHub Copilot Instructions

#### `instructions.md` (9KB)
**Purpose:** Guide GitHub Copilot when writing code

**Contains:**
- Project context summary
- Core technologies
- Mandatory conventions
- Astro-specific patterns
- Common code patterns (layouts, dynamic content, SEO)
- Import guidelines
- Styling guidelines (Tailwind)
- Performance best practices
- TypeScript patterns
- Common mistakes to avoid
- Quick reference

**When used:** Automatically by GitHub Copilot when writing code

### `.cursorrules` File - Cursor AI Rules

**Purpose:** Configure Cursor AI behavior for this project

**Contains:**
- Project identity
- Core principles
- Mandatory file naming
- Code standards (TypeScript, Astro, Tailwind)
- Import conventions
- Performance rules
- SEO requirements
- Accessibility requirements
- Content collections patterns
- Environment variables
- Common mistakes to avoid
- Testing standards
- Integration with bMOI ecosystem

**When used:** Automatically by Cursor AI when working in this project

### `.docs/` Folder - Comprehensive Documentation

#### `README.md`
Project overview, tech stack, quick links

#### `SETUP.md`
Complete step-by-step setup guide:
- Prerequisites installation
- Project initialization from scratch
- Astro configuration
- Tailwind CSS setup
- TypeScript configuration
- Docker setup
- First component creation
- First page creation
- Development server start

#### `ARQUITECTURA.md`
Architecture decisions explained:
- Why Astro 4 vs Next.js vs Nuxt
- Why Tailwind CSS vs other solutions
- Why static site (SSG) vs SSR
- Why S3 + CloudFront deployment
- Why file-based content vs CMS
- Islands Architecture explained
- Content Collections strategy
- Performance budget decisions
- Trade-offs and alternatives

#### `DEPLOYMENT.md`
Deployment guides:
- AWS S3 + CloudFront (detailed steps)
- Netlify deployment
- Vercel deployment
- Cloudflare Pages
- Custom domain setup
- SSL certificate setup
- CI/CD with GitHub Actions
- Deployment troubleshooting

#### `CODING_STANDARDS.md`
Coding standards with examples:
- Astro component structure
- TypeScript conventions
- Tailwind CSS patterns
- File organization
- Import ordering
- Error handling
- Testing patterns
- Accessibility standards
- SEO implementation

#### `EJEMPLOS_CODIGO.md`
Complete code examples:
- Basic Astro component
- Page with SEO
- Dynamic routes (blog posts)
- Content collections setup
- Form with validation
- Hero section
- Pricing table
- Testimonials carousel
- Contact form
- Newsletter signup

#### `SEO_GUIDE.md`
SEO best practices:
- Meta tags implementation
- Schema.org markup
- Sitemap generation
- Robots.txt configuration
- Open Graph tags
- Twitter Cards
- Canonical URLs
- Performance optimization for SEO
- Content strategy
- Keyword research
- Internal linking

---

## 🚀 How to Use These Templates

### For Developers (Manual Setup)

1. **Read the documentation** in this order:
   - `README.md` - Get overview
   - `.docs/SETUP.md` - Follow step-by-step setup
   - `.docs/ARQUITECTURA.md` - Understand decisions
   - `.docs/CODING_STANDARDS.md` - Learn conventions

2. **Initialize your project:**
   ```bash
   npm create astro@latest bmoi-landing-page
   cd bmoi-landing-page
   ```

3. **Configure Tailwind CSS:**
   ```bash
   npx astro add tailwind
   ```

4. **Follow the examples** in `.docs/EJEMPLOS_CODIGO.md`

5. **Reference conventions** in `.claude/conventions.md` while coding

### For AI Assistants (Claude Code, Copilot, Cursor)

1. **Claude Code:**
   - Reads `.claude/context.md` first
   - Follows `.claude/conventions.md` for naming
   - Updates `.claude/session-state.md` after each session
   - Uses `CLAUDE.md` for quick reference

2. **GitHub Copilot:**
   - Automatically reads `.copilot/instructions.md`
   - Suggests code following the patterns defined
   - Respects conventions and best practices

3. **Cursor AI:**
   - Automatically applies `.cursorrules`
   - Enforces naming conventions
   - Suggests code following project standards

### For Teams

1. **Onboarding new developers:**
   - Share `README.md` for overview
   - Have them read `.docs/SETUP.md`
   - Review `.docs/ARQUITECTURA.md` together
   - Pair program using `.docs/EJEMPLOS_CODIGO.md`

2. **Code reviews:**
   - Check against `.claude/conventions.md`
   - Verify SEO implementation with `.docs/SEO_GUIDE.md`
   - Test performance against targets in `.docs/ARQUITECTURA.md`

3. **AI-assisted development:**
   - Ensure AI reads context files before starting
   - Update `.claude/session-state.md` regularly
   - Use AI memory checkpoints to prevent context loss

---

## 🎯 Key Decisions Explained

### Why Astro 4?

**Decision:** Use Astro 4 (not Next.js, not Nuxt, not plain HTML)

**Reasons:**
- ✅ **Zero JavaScript by default** - Ships 0 JS unless you need it
- ✅ **Perfect for static sites** - Built specifically for content-focused sites
- ✅ **Islands Architecture** - Interactive components only where needed
- ✅ **SEO friendly** - Perfect meta tags, sitemap, RSS support
- ✅ **Fastest builds** - Faster than Next.js static export

**Trade-offs:**
- ❌ Less suitable for highly interactive apps (but that's not our use case)
- ❌ Smaller ecosystem than Next.js (but growing rapidly)

### Why Static Site Generation (SSG)?

**Decision:** Build entire site as static HTML at build time

**Reasons:**
- ⚡ **Fastest possible load times** - Pre-rendered HTML
- 💰 **Cheapest hosting** - Just S3 + CloudFront ($0.50/month)
- 🔒 **Maximum security** - No server to hack
- 📈 **Perfect SEO** - Search engines love static HTML

**Trade-offs:**
- ❌ Content updates require rebuild/redeploy (acceptable for marketing site)
- ❌ No real-time data (but we don't need it)

### Why Tailwind CSS?

**Decision:** Use Tailwind CSS 4 (not custom CSS, not CSS-in-JS)

**Reasons:**
- ✅ **Utility-first** - Fast prototyping
- ✅ **No unused CSS** - Automatic purging
- ✅ **Consistent design system** - Design tokens built-in
- ✅ **Great with Astro** - Works perfectly together

### Why Docker?

**Decision:** Docker-based development (optional but recommended)

**Reasons:**
- ✅ **Consistent environment** - Same setup for all developers
- ✅ **Easy onboarding** - New dev productive in 5 minutes
- ✅ **CI/CD ready** - Same container in dev, staging, prod

---

## 🧠 AI Assistant Usage Guidelines

### Best Practices

1. **Always read context first**
   ```
   Before starting any task, read:
   1. .claude/context.md
   2. .claude/conventions.md
   3. .claude/session-state.md
   ```

2. **Follow naming conventions strictly**
   - Files: PascalCase (components), kebab-case (pages)
   - Code: camelCase (variables), UPPER_SNAKE_CASE (constants)
   - NEVER use snake_case in TypeScript

3. **Update session state**
   - Update `.claude/session-state.md` after each session
   - Log decisions made, problems solved, next steps

4. **Use memory checkpoints**
   - Every ~5 messages, verify you remember key facts
   - Project name, framework, conventions, etc.

### AI Memory Checkpoints

Every 5 messages, AI should verify:
- [ ] Project: **bMOI Landing Page**
- [ ] Framework: **Astro 4** (NOT Next.js)
- [ ] Type: **Static Site (SSG)**, NOT SSR
- [ ] Styling: **Tailwind CSS**
- [ ] Deployment: **S3 + CloudFront**
- [ ] Conventions: **PascalCase** (components), **kebab-case** (pages), **camelCase** (variables)
- [ ] **NEVER** use snake_case in TypeScript
- [ ] **ALWAYS** use @/ for imports
- [ ] **JavaScript minimal** - Islands Architecture
- [ ] **Performance target:** 95+ Lighthouse score
- [ ] **SEO critical:** Meta tags mandatory

---

## 📊 Development Phases

### Phase 0: Setup (0% Complete)
- Initialize Astro project
- Configure Tailwind CSS
- Setup Docker
- Create folder structure
- Configure TypeScript

### Phase 1: Core Pages (0% Complete)
- Create base layout (Header/Footer)
- Implement Home page (Hero, Features, CTA)
- Create Features page
- Build Pricing page
- Implement About page
- Create Contact page

### Phase 2: Content Management (0% Complete)
- Setup Content Collections
- Create blog layout
- Write initial blog posts
- Create legal pages (Privacy, Terms)

### Phase 3: SEO & Performance (0% Complete)
- Add meta tags to all pages
- Implement Schema.org markup
- Generate sitemap.xml
- Optimize images (WebP, lazy loading)
- Configure robots.txt

### Phase 4: Integrations (0% Complete)
- Integrate Google Analytics 4
- Add Google Tag Manager
- Implement sign-up form with API
- Add contact form
- Configure social sharing

### Phase 5: Deployment (0% Complete)
- Setup S3 + CloudFront
- Configure custom domain + SSL
- Create deployment script
- Configure CI/CD (GitHub Actions)
- Test production deployment

### Phase 6: Testing & Optimization (0% Complete)
- Run Lighthouse audits (95+ target)
- Cross-browser testing
- Mobile device testing
- Accessibility audit (WCAG 2.1 AA)
- Performance monitoring setup

---

## 🔗 Integration with bMOI Ecosystem

The landing page integrates with the bMOI ecosystem:

```
Landing Page → Backend API → Admin Panel
     ↓             ↓              ↓
 Sign-Up Form   Creates Tenant  Dashboard
```

**API Endpoints Used:**
- `POST /auth/register` - Create new tenant account
- `POST /contact/submit` - Contact form submissions

**Environment Variables:**
```bash
PUBLIC_API_URL=https://api.bmoi.com
PUBLIC_APP_URL=https://app.bmoi.com
PUBLIC_SITE_URL=https://bmoi.com
```

---

## 🎨 Code Examples Quick Reference

### Basic Astro Component

```astro
---
interface Props {
  title: string;
}

const { title } = Astro.props;
---

<div class="container mx-auto">
  <h1>{title}</h1>
</div>
```

### Page with SEO

```astro
---
import BaseLayout from '@/layouts/BaseLayout.astro';

const title = 'Page Title';
const description = 'Page description';
---

<BaseLayout {title} {description}>
  <main>Content</main>
</BaseLayout>
```

### Image Optimization

```astro
---
import { Image } from 'astro:assets';
import hero from '@/assets/hero.jpg';
---

<Image src={hero} alt="Hero" width={1200} height={600} loading="lazy" />
```

---

## 📞 Support & Resources

### Documentation
- **Astro Docs:** https://docs.astro.build/
- **Tailwind Docs:** https://tailwindcss.com/docs
- **TypeScript Docs:** https://www.typescriptlang.org/docs

### Related Projects
- **Backend API:** `bo-backend-api/CLAUDE.md`
- **Admin Panel:** `plantillas_admin/.claude/context.md`
- **Mobile App:** `plantillas_mobile/.claude/context.md`

### Getting Help
If you encounter issues:
1. Check `.docs/` folder for detailed guides
2. Review `.claude/context.md` for architecture decisions
3. Consult `.docs/EJEMPLOS_CODIGO.md` for code examples
4. Check `.claude/session-state.md` for recent changes

---

## ✅ Template Checklist

Before starting development, ensure:
- [ ] Read `.claude/context.md` completely
- [ ] Understood `.claude/conventions.md`
- [ ] Reviewed `.docs/ARQUITECTURA.md`
- [ ] Checked `.docs/SETUP.md` for setup steps
- [ ] Configured your AI assistant (Claude/Copilot/Cursor)

During development:
- [ ] Follow naming conventions strictly
- [ ] Use @/ alias for imports
- [ ] Write TypeScript types (no `any`)
- [ ] Add SEO meta tags to pages
- [ ] Optimize images (WebP, lazy loading)
- [ ] Test Lighthouse scores (95+ target)

After development:
- [ ] Update `.claude/session-state.md`
- [ ] Document technical decisions
- [ ] Run build and check for errors
- [ ] Test on mobile devices
- [ ] Verify SEO implementation

---

**Template Version:** 1.0.0
**Last Updated:** 2025-01-04
**Status:** Initial Creation

<div align="center">

**Complete instruction templates for building the bMOI Landing Page with Astro 4**

</div>
