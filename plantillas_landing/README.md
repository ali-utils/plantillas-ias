# 🌐 bMOI Landing Page

> Static marketing website built with Astro 4 - Fast, SEO-optimized, and deployed to S3

---

## 🚀 Quick Start

```bash
# 1. Install dependencies
npm install

# 2. Start development server
npm run dev

# 3. Open browser
# Visit http://localhost:4321
```

---

## 📋 Prerequisites

- **Node.js 20+** - [Install](https://nodejs.org/)
- **npm 10+** - Comes with Node.js
- **Docker** (optional) - [Install](https://docs.docker.com/get-docker/)

```bash
# Verify installation
node --version  # v20.x.x
npm --version   # 10.x.x
```

---

## 🐳 Docker (Recommended)

Docker provides consistent development environment across all machines.

```bash
# Start development server
docker compose up -d

# View logs
docker compose logs -f landing-dev

# Stop server
docker compose down
```

---

## 📦 Available Commands

```bash
# Development
npm run dev              # Start dev server (http://localhost:4321)
npm run build            # Build for production (output: dist/)
npm run preview          # Preview production build

# Code Quality
npm run astro check      # Type check
npm run format           # Format with Prettier
npm run lint             # Lint code

# Content
npm run astro sync       # Sync content collections types
```

---

## 📁 Project Structure

```
plantillas_landing/
├── .claude/              # AI context files
├── .copilot/             # GitHub Copilot instructions
├── .docs/                # Comprehensive documentation
├── public/               # Static assets (robots.txt, favicon, etc.)
├── src/
│   ├── components/       # Reusable UI components
│   ├── content/          # Markdown/MDX content
│   ├── layouts/          # Page layouts
│   ├── pages/            # Routes (file-based routing)
│   ├── styles/           # Global CSS
│   └── utils/            # Helper functions
├── astro.config.mjs      # Astro configuration
├── tailwind.config.mjs   # Tailwind CSS configuration
├── docker-compose.yml    # Docker development
└── package.json
```

---

## 🛠️ Tech Stack

- **Astro 4** - Static site generator
- **TypeScript 5** - Type safety
- **Tailwind CSS 4** - Utility-first styling
- **Markdown/MDX** - Content management
- **Docker** - Consistent development environment

---

## 🌍 Deployment

### AWS S3 + CloudFront

```bash
# Build static site
npm run build

# Deploy to S3
aws s3 sync dist/ s3://bmoi-landing-bucket --delete
aws cloudfront create-invalidation --distribution-id XXX --paths "/*"
```

### Netlify

```bash
npm run build
netlify deploy --prod --dir=dist
```

### Vercel

```bash
vercel --prod
```

See [`.docs/DEPLOYMENT.md`](.docs/DEPLOYMENT.md) for detailed deployment instructions.

---

## 📚 Documentation

- **📖 [README_PLANTILLAS.md](README_PLANTILLAS.md)** - Complete template guide
- **🔧 [.docs/SETUP.md](.docs/SETUP.md)** - Step-by-step setup guide
- **🏗️ [.docs/ARQUITECTURA.md](.docs/ARQUITECTURA.md)** - Architecture decisions
- **☁️ [.docs/DEPLOYMENT.md](.docs/DEPLOYMENT.md)** - Deployment guide
- **📝 [.docs/CODING_STANDARDS.md](.docs/CODING_STANDARDS.md)** - Coding standards
- **💡 [.docs/EJEMPLOS_CODIGO.md](.docs/EJEMPLOS_CODIGO.md)** - Code examples
- **🔍 [.docs/SEO_GUIDE.md](.docs/SEO_GUIDE.md)** - SEO best practices

---

## 🎯 Performance Targets

- **Lighthouse Performance:** 95+
- **Lighthouse Accessibility:** 100
- **Lighthouse SEO:** 100
- **First Contentful Paint:** < 1.2s
- **Largest Contentful Paint:** < 2.5s

---

## 🔗 Related Projects

- **Backend API:** `bo-backend-api/` - REST API with NestJS
- **Admin Panel:** `plantillas_admin/` - React admin dashboard
- **Mobile App:** `plantillas_mobile/` - Flutter mobile app

---

## 📄 License

This is a template for the bMOI project.

---

<div align="center">

**Built with ⚡ Astro - Ship less JavaScript, load faster**

</div>
