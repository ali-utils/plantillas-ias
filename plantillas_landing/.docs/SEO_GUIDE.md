# SEO Guide - bMOI Landing Page

Complete SEO implementation guide for maximum visibility.

## On-Page SEO Checklist

### Meta Tags (Every Page)

```astro
---
const title = 'Your Page Title (50-60 characters)';
const description = 'Your compelling description that encourages clicks. Keep it between 120-160 characters for optimal display.';
const canonicalUrl = new URL(Astro.url.pathname, Astro.site);
const ogImage = '/images/og-image.jpg';  // 1200x630px
---

<head>
  <!-- Primary Meta Tags -->
  <title>{title} | bMOI</title>
  <meta name="title" content={title} />
  <meta name="description" content={description} />
  <link rel="canonical" href={canonicalUrl} />

  <!-- Open Graph / Facebook -->
  <meta property="og:type" content="website" />
  <meta property="og:url" content={canonicalUrl} />
  <meta property="og:title" content={title} />
  <meta property="og:description" content={description} />
  <meta property="og:image" content={ogImage} />

  <!-- Twitter -->
  <meta property="twitter:card" content="summary_large_image" />
  <meta property="twitter:url" content={canonicalUrl} />
  <meta property="twitter:title" content={title} />
  <meta property="twitter:description" content={description} />
  <meta property="twitter:image" content={ogImage} />
</head>
```

### Heading Hierarchy

```html
<!-- ✅ CORRECT: One H1 per page -->
<h1>Main Page Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>
<h2>Another Section</h2>

<!-- ❌ WRONG: Multiple H1s -->
<h1>First Title</h1>
<h1>Second Title</h1>  <!-- Don't do this -->
```

### Internal Linking

```html
<!-- Link related pages together -->
<p>Learn more about our <a href="/features">features</a> or check our <a href="/pricing">pricing</a>.</p>

<!-- Use descriptive anchor text -->
✅ <a href="/blog/multi-tenant-guide">read our complete multi-tenant guide</a>
❌ <a href="/blog/multi-tenant-guide">click here</a>
```

## Technical SEO

### Sitemap.xml

```typescript
// astro.config.mjs
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://bmoi.com',
  integrations: [sitemap()],
});

// Automatically generates sitemap at /sitemap-index.xml
```

### Robots.txt

```txt
# public/robots.txt
User-agent: *
Allow: /

# Disallow admin pages (if any)
Disallow: /admin/

# Sitemap
Sitemap: https://bmoi.com/sitemap-index.xml
```

### Structured Data (Schema.org)

**Organization Schema:**
```astro
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "bMOI",
  "url": "https://bmoi.com",
  "logo": "https://bmoi.com/images/logo.png",
  "description": "Multi-tenant platform for businesses",
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "+1-555-0123",
    "contactType": "Customer Service"
  },
  "sameAs": [
    "https://twitter.com/bmoi",
    "https://linkedin.com/company/bmoi",
    "https://github.com/bmoi"
  ]
}
</script>
```

**WebSite Schema:**
```astro
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "url": "https://bmoi.com",
  "name": "bMOI",
  "potentialAction": {
    "@type": "SearchAction",
    "target": "https://bmoi.com/search?q={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
</script>
```

**Article Schema (Blog Posts):**
```astro
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "{post.data.title}",
  "image": "{post.data.image}",
  "author": {
    "@type": "Person",
    "name": "{post.data.author}"
  },
  "datePublished": "{post.data.publishedAt.toISOString()}",
  "dateModified": "{post.data.updatedAt?.toISOString()}",
  "publisher": {
    "@type": "Organization",
    "name": "bMOI",
    "logo": {
      "@type": "ImageObject",
      "url": "https://bmoi.com/images/logo.png"
    }
  }
}
</script>
```

## Image Optimization for SEO

```astro
---
import { Image } from 'astro:assets';
import productImage from '@/assets/product.jpg';
---

<!-- ✅ Optimized image with descriptive alt text -->
<Image
  src={productImage}
  alt="bMOI multi-tenant dashboard showing analytics and user management"
  width={800}
  height={600}
  format="webp"
  loading="lazy"
/>

<!-- ✅ Always include descriptive alt text -->
<img src="chart.png" alt="Annual revenue growth chart showing 150% increase" />

<!-- ❌ Empty or generic alt text -->
<img src="image.png" alt="image" />  <!-- Bad -->
<img src="photo.jpg" alt="" />  <!-- Bad (unless decorative) -->
```

## Performance for SEO

### Core Web Vitals

**Target scores:**
- **LCP (Largest Contentful Paint)**: < 2.5s
- **FID (First Input Delay)**: < 100ms
- **CLS (Cumulative Layout Shift)**: < 0.1

**Optimization techniques:**
```html
<!-- Preload critical resources -->
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" as="image" href="/images/hero.jpg">

<!-- Use WebP format -->
<picture>
  <source srcset="hero.webp" type="image/webp">
  <img src="hero.jpg" alt="Hero image">
</picture>

<!-- Set image dimensions (prevent CLS) -->
<img src="logo.png" alt="Logo" width="200" height="50">
```

## Mobile SEO

### Mobile-Friendly Test

```html
<!-- Viewport meta tag (required) -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Mobile-optimized navigation -->
<nav>
  <!-- Hamburger menu on mobile, horizontal on desktop -->
  <button class="md:hidden">Menu</button>
  <ul class="hidden md:flex">
    <li><a href="/">Home</a></li>
  </ul>
</nav>

<!-- Touch-friendly buttons (min 44x44px) -->
<button class="px-6 py-3">Click me</button>  <!-- ✅ Touch-friendly -->
<button class="px-2 py-1">Too small</button>  <!-- ❌ Too small -->
```

## Content SEO Strategy

### Keyword Research

**Target keywords:**
- Primary: "multi-tenant platform", "SaaS multi-tenant"
- Secondary: "tenant management system", "multi-tenant database"
- Long-tail: "how to build multi-tenant application"

**Where to place keywords:**
- ✅ Page title
- ✅ H1 heading
- ✅ First 100 words
- ✅ Subheadings (H2, H3)
- ✅ Image alt text
- ✅ URL slug
- ❌ Don't keyword stuff

### Content Guidelines

**Blog post SEO checklist:**
- [ ] 1500+ words (in-depth content)
- [ ] Include target keyword in title
- [ ] Use target keyword naturally 3-5 times
- [ ] Include related keywords (LSI keywords)
- [ ] Add internal links to related posts
- [ ] Add external links to authoritative sources
- [ ] Include images with alt text
- [ ] Add table of contents for long posts
- [ ] Use short paragraphs (2-3 sentences)
- [ ] Include meta description
- [ ] Set focus keyword
- [ ] Add schema markup

## Local SEO (Optional)

```astro
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "bMOI Inc.",
  "image": "https://bmoi.com/images/office.jpg",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "123 Tech Street",
    "addressLocality": "San Francisco",
    "addressRegion": "CA",
    "postalCode": "94105",
    "addressCountry": "US"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 37.7749,
    "longitude": -122.4194
  },
  "openingHoursSpecification": {
    "@type": "OpeningHoursSpecification",
    "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
    "opens": "09:00",
    "closes": "18:00"
  },
  "telephone": "+1-555-0123"
}
</script>
```

## Monitoring & Analytics

### Google Search Console

**Setup:**
1. Add property (https://bmoi.com)
2. Verify ownership (HTML file or DNS)
3. Submit sitemap (https://bmoi.com/sitemap-index.xml)
4. Monitor:
   - Search queries
   - Click-through rates
   - Index coverage
   - Mobile usability

### Google Analytics 4

```html
<!-- In BaseLayout.astro <head> -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

### Key Metrics to Track

- **Organic traffic** (from search engines)
- **Bounce rate** (< 40% is good)
- **Average session duration** (> 2 minutes)
- **Pages per session** (> 2 is good)
- **Conversion rate** (sign-ups / visitors)

## SEO Tools

### Free Tools
- **Google Search Console** - Track search performance
- **Google PageSpeed Insights** - Check page speed
- **Lighthouse** - SEO audit in Chrome DevTools
- **Google Rich Results Test** - Validate schema markup

### Paid Tools (Optional)
- **Ahrefs** - Backlink analysis, keyword research
- **SEMrush** - Competitor analysis, rank tracking
- **Moz Pro** - SEO metrics, site audits

## Common SEO Mistakes to Avoid

```html
<!-- ❌ Duplicate content -->
Don't copy content from other sites

<!-- ❌ Missing meta descriptions -->
<head>
  <title>Page Title</title>
  <!-- Missing: <meta name="description"> -->
</head>

<!-- ❌ Broken links -->
<a href="/page-that-doesnt-exist">Link</a>  <!-- 404 error -->

<!-- ❌ Slow page speed -->
<!-- Large unoptimized images, too much JavaScript -->

<!-- ❌ Not mobile-friendly -->
<!-- Fixed-width layouts, tiny text on mobile -->

<!-- ❌ Thin content -->
<!-- Pages with < 300 words -->

<!-- ❌ Keyword stuffing -->
<p>Multi-tenant multi-tenant platform multi-tenant...</p>  <!-- Don't do this -->
```

## SEO Checklist for Launch

- [ ] All pages have unique titles and descriptions
- [ ] Sitemap.xml generated and submitted
- [ ] Robots.txt configured
- [ ] Schema.org markup added (Organization, WebSite)
- [ ] Open Graph tags on all pages
- [ ] Twitter Card tags on all pages
- [ ] All images have descriptive alt text
- [ ] Mobile-friendly (responsive design)
- [ ] Fast page speed (< 3s load time)
- [ ] HTTPS enabled
- [ ] Google Search Console set up
- [ ] Google Analytics installed
- [ ] Internal linking structure planned
- [ ] Blog content calendar created
- [ ] Backlink strategy defined

---

**Follow this guide for optimal SEO performance and search visibility!**
