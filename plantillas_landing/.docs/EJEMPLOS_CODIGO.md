# Code Examples - bMOI Landing Page

Complete, copy-paste ready code examples.

## Complete Components

### 1. Header with Navigation

```astro
---
// src/components/Header.astro
import ThemeToggle from '@/components/ThemeToggle';
---

<header class="sticky top-0 z-50 bg-white dark:bg-slate-900 border-b border-slate-200 dark:border-slate-800">
  <nav class="container mx-auto px-4 py-4">
    <div class="flex items-center justify-between">
      <!-- Logo -->
      <a href="/" class="text-2xl font-bold text-brand-600">
        bMOI
      </a>

      <!-- Desktop Navigation -->
      <ul class="hidden md:flex items-center space-x-8">
        <li><a href="/features" class="hover:text-brand-600">Features</a></li>
        <li><a href="/pricing" class="hover:text-brand-600">Pricing</a></li>
        <li><a href="/about" class="hover:text-brand-600">About</a></li>
        <li><a href="/blog" class="hover:text-brand-600">Blog</a></li>
      </ul>

      <!-- CTA + Theme Toggle -->
      <div class="flex items-center gap-4">
        <ThemeToggle client:load />
        <a href="/contact" class="btn-primary">
          Get Started
        </a>
      </div>

      <!-- Mobile menu button -->
      <button class="md:hidden" aria-label="Open menu">
        <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
        </svg>
      </button>
    </div>
  </nav>
</header>
```

### 2. Hero Section with Dark Mode

```astro
---
// src/components/Hero.astro
import { Image } from 'astro:assets';
import heroImage from '@/assets/hero.jpg';
---

<section class="bg-gradient-to-b from-slate-50 to-white dark:from-slate-900 dark:to-slate-800">
  <div class="container mx-auto px-4 py-20 lg:py-32">
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-12 items-center">
      <!-- Content -->
      <div>
        <h1 class="text-4xl md:text-5xl lg:text-6xl font-bold mb-6">
          <span class="text-gradient">Multi-Tenant Platform</span>
          <br />
          for Modern Businesses
        </h1>
        <p class="text-xl text-slate-600 dark:text-slate-400 mb-8">
          Manage multiple tenants with ease. Built for scale, designed for simplicity.
        </p>
        <div class="flex flex-col sm:flex-row gap-4">
          <a href="/pricing" class="btn-primary text-center">
            Start Free Trial
          </a>
          <a href="/demo" class="btn-secondary text-center">
            Watch Demo
          </a>
        </div>
        <!-- Social proof -->
        <div class="mt-12">
          <p class="text-sm text-slate-600 dark:text-slate-400 mb-4">
            Trusted by 500+ companies
          </p>
          <div class="flex gap-8 opacity-60">
            <!-- Company logos here -->
          </div>
        </div>
      </div>

      <!-- Image -->
      <div class="relative">
        <Image
          src={heroImage}
          alt="Platform dashboard"
          width={600}
          height={400}
          class="rounded-lg shadow-2xl"
        />
        <!-- Floating stats card -->
        <div class="absolute -bottom-6 -left-6 bg-white dark:bg-slate-800 p-6 rounded-lg shadow-xl">
          <div class="text-3xl font-bold text-brand-600">99.9%</div>
          <div class="text-sm text-slate-600 dark:text-slate-400">Uptime</div>
        </div>
      </div>
    </div>
  </div>
</section>
```

### 3. Features Grid (Responsive)

```astro
---
// src/components/Features.astro
const features = [
  {
    title: 'Multi-Tenant',
    description: 'Complete tenant isolation at database level with RLS',
    icon: '🏢',
  },
  {
    title: 'Scalable',
    description: 'Built to handle millions of requests per day',
    icon: '📈',
  },
  {
    title: 'Secure',
    description: 'Enterprise-grade security with JWT + RLS',
    icon: '🔒',
  },
  {
    title: 'Fast',
    description: 'Lightning-fast performance with optimized queries',
    icon: '⚡',
  },
  {
    title: 'Analytics',
    description: 'Real-time analytics and reporting dashboard',
    icon: '📊',
  },
  {
    title: 'Support',
    description: '24/7 customer support for all plans',
    icon: '💬',
  },
];
---

<section class="py-20 bg-white dark:bg-slate-900">
  <div class="container mx-auto px-4">
    <div class="text-center mb-16">
      <h2 class="text-3xl md:text-4xl font-bold mb-4">
        Everything you need to succeed
      </h2>
      <p class="text-xl text-slate-600 dark:text-slate-400">
        Powerful features for modern businesses
      </p>
    </div>

    <!-- Responsive grid -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
      {features.map((feature) => (
        <div class="bg-slate-50 dark:bg-slate-800 p-6 rounded-lg hover:shadow-lg transition-shadow">
          <div class="text-4xl mb-4">{feature.icon}</div>
          <h3 class="text-xl font-semibold mb-2">{feature.title}</h3>
          <p class="text-slate-600 dark:text-slate-400">{feature.description}</p>
        </div>
      ))}
    </div>
  </div>
</section>
```

### 4. Pricing Table (Responsive)

```astro
---
// src/components/Pricing.astro
const plans = [
  {
    name: 'Free',
    price: '$0',
    period: 'forever',
    features: ['10 users', '50 products', '1GB storage', 'Email support'],
    cta: 'Start Free',
    featured: false,
  },
  {
    name: 'Pro',
    price: '$49',
    period: '/month',
    features: ['50 users', '500 products', '10GB storage', 'Priority support', 'Advanced analytics'],
    cta: 'Start Trial',
    featured: true,
  },
  {
    name: 'Enterprise',
    price: '$199',
    period: '/month',
    features: ['Unlimited users', 'Unlimited products', '100GB storage', '24/7 support', 'Custom features'],
    cta: 'Contact Sales',
    featured: false,
  },
];
---

<section class="py-20 bg-slate-50 dark:bg-slate-800">
  <div class="container mx-auto px-4">
    <div class="text-center mb-16">
      <h2 class="text-3xl md:text-4xl font-bold mb-4">
        Simple, transparent pricing
      </h2>
      <p class="text-xl text-slate-600 dark:text-slate-400">
        Choose the perfect plan for your business
      </p>
    </div>

    <!-- Responsive pricing grid -->
    <div class="grid grid-cols-1 md:grid-cols-3 gap-8 max-w-6xl mx-auto">
      {plans.map((plan) => (
        <div class={`bg-white dark:bg-slate-900 p-8 rounded-lg ${plan.featured ? 'ring-2 ring-brand-600 scale-105' : ''}`}>
          {plan.featured && (
            <span class="bg-brand-600 text-white text-xs px-3 py-1 rounded-full">
              Most Popular
            </span>
          )}
          <h3 class="text-2xl font-bold mt-4">{plan.name}</h3>
          <div class="mt-4">
            <span class="text-4xl font-bold">{plan.price}</span>
            <span class="text-slate-600 dark:text-slate-400">{plan.period}</span>
          </div>
          <ul class="mt-8 space-y-4">
            {plan.features.map((feature) => (
              <li class="flex items-center gap-2">
                <svg class="w-5 h-5 text-green-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                </svg>
                <span>{feature}</span>
              </li>
            ))}
          </ul>
          <a
            href="/signup"
            class={`block mt-8 text-center py-3 rounded-lg ${plan.featured ? 'btn-primary' : 'btn-secondary'}`}
          >
            {plan.cta}
          </a>
        </div>
      ))}
    </div>
  </div>
</section>
```

### 5. Theme Toggle (React Island)

```typescript
// src/components/ThemeToggle.tsx
import { useEffect, useState } from 'react';

export default function ThemeToggle() {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
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

  // Prevent hydration mismatch
  if (!mounted) {
    return (
      <div className="w-10 h-10 rounded-lg bg-slate-100 dark:bg-slate-800 animate-pulse" />
    );
  }

  return (
    <button
      onClick={toggleTheme}
      className="rounded-lg p-2 hover:bg-slate-100 dark:hover:bg-slate-800 transition-colors"
      aria-label={`Switch to ${theme === 'light' ? 'dark' : 'light'} theme`}
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

### 6. Contact Form (with validation)

```typescript
// src/components/ContactForm.tsx
import { useState } from 'react';

interface FormData {
  name: string;
  email: string;
  message: string;
}

export default function ContactForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    email: '',
    message: '',
  });
  const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setStatus('loading');

    try {
      const response = await fetch('https://api.bmoi.com/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData),
      });

      if (response.ok) {
        setStatus('success');
        setFormData({ name: '', email: '', message: '' });
      } else {
        setStatus('error');
      }
    } catch (error) {
      setStatus('error');
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label htmlFor="name" className="block text-sm font-medium mb-2">
          Name
        </label>
        <input
          type="text"
          id="name"
          required
          value={formData.name}
          onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          className="w-full px-4 py-2 border border-slate-300 dark:border-slate-700 rounded-lg bg-white dark:bg-slate-800"
        />
      </div>

      <div>
        <label htmlFor="email" className="block text-sm font-medium mb-2">
          Email
        </label>
        <input
          type="email"
          id="email"
          required
          value={formData.email}
          onChange={(e) => setFormData({ ...formData, email: e.target.value })}
          className="w-full px-4 py-2 border border-slate-300 dark:border-slate-700 rounded-lg bg-white dark:bg-slate-800"
        />
      </div>

      <div>
        <label htmlFor="message" className="block text-sm font-medium mb-2">
          Message
        </label>
        <textarea
          id="message"
          required
          rows={4}
          value={formData.message}
          onChange={(e) => setFormData({ ...formData, message: e.target.value })}
          className="w-full px-4 py-2 border border-slate-300 dark:border-slate-700 rounded-lg bg-white dark:bg-slate-800"
        />
      </div>

      <button
        type="submit"
        disabled={status === 'loading'}
        className="btn-primary w-full disabled:opacity-50"
      >
        {status === 'loading' ? 'Sending...' : 'Send Message'}
      </button>

      {status === 'success' && (
        <p className="text-green-600">Message sent successfully!</p>
      )}
      {status === 'error' && (
        <p className="text-red-600">Failed to send message. Please try again.</p>
      )}
    </form>
  );
}
```

### 7. Blog Post Template

```astro
---
// src/pages/blog/[slug].astro
import { getCollection } from 'astro:content';
import BaseLayout from '@/layouts/BaseLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<BaseLayout title={post.data.title} description={post.data.description}>
  <article class="container mx-auto px-4 py-12 max-w-3xl">
    <header class="mb-8">
      <h1 class="text-4xl font-bold mb-4">{post.data.title}</h1>
      <div class="flex items-center gap-4 text-slate-600 dark:text-slate-400">
        <time>{post.data.publishedAt.toLocaleDateString()}</time>
        <span>·</span>
        <span>{post.data.author}</span>
      </div>
    </header>

    <div class="prose dark:prose-invert max-w-none">
      <Content />
    </div>
  </article>
</BaseLayout>
```

---

**All examples follow:**
- ✅ Responsive design (mobile-first)
- ✅ Dark mode support
- ✅ TypeScript types
- ✅ Accessibility standards
- ✅ Performance optimization
