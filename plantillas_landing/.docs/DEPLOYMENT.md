# Deployment Guide - bMOI Landing Page

Complete guide for deploying the bMOI Landing Page to production.

## Deployment Options

| Platform | Cost | Difficulty | Speed | Best For |
|----------|------|------------|-------|----------|
| **AWS S3 + CloudFront** | $ | Medium | Fast | Production |
| **Netlify** | Free/$$ | Easy | Fastest | Quick deploy |
| **Vercel** | Free/$$ | Easy | Fastest | Quick deploy |
| **Cloudflare Pages** | Free | Easy | Fastest | Global CDN |

---

## Option 1: AWS S3 + CloudFront (Recommended)

### Prerequisites

```bash
# Install AWS CLI
brew install awscli  # macOS
# or download from https://aws.amazon.com/cli/

# Configure AWS credentials
aws configure
# AWS Access Key ID: YOUR_KEY
# AWS Secret Access Key: YOUR_SECRET
# Default region: us-east-1
```

### Step 1: Build Project

```bash
npm run build
# Output: dist/ folder
```

### Step 2: Create S3 Bucket

```bash
# Create bucket
aws s3 mb s3://bmoi-landing-prod

# Enable static website hosting
aws s3 website s3://bmoi-landing-prod \
  --index-document index.html \
  --error-document 404.html
```

### Step 3: Configure Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::bmoi-landing-prod/*"
    }
  ]
}
```

Apply policy:
```bash
aws s3api put-bucket-policy \
  --bucket bmoi-landing-prod \
  --policy file://bucket-policy.json
```

### Step 4: Create CloudFront Distribution

```bash
# Create distribution
aws cloudfront create-distribution \
  --origin-domain-name bmoi-landing-prod.s3.amazonaws.com \
  --default-root-object index.html

# Note the Distribution ID (E1234567890ABC)
```

### Step 5: Deploy

```bash
# Sync files to S3
aws s3 sync dist/ s3://bmoi-landing-prod --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

### Step 6: Custom Domain (Optional)

1. **Request SSL Certificate (ACM)**
   ```bash
   aws acm request-certificate \
     --domain-name bmoi.com \
     --subject-alternative-names www.bmoi.com \
     --validation-method DNS
   ```

2. **Validate Certificate** (add DNS records)

3. **Update CloudFront Distribution**
   - Add custom domain (Alternate Domain Names)
   - Add SSL certificate

4. **Update DNS (Route 53 or your DNS provider)**
   ```
   bmoi.com → CloudFront Distribution (ALIAS)
   www.bmoi.com → CloudFront Distribution (ALIAS)
   ```

### Automation Script

Create `deploy.sh`:
```bash
#!/bin/bash
set -e

echo "Building project..."
npm run build

echo "Deploying to S3..."
aws s3 sync dist/ s3://bmoi-landing-prod --delete

echo "Invalidating CloudFront..."
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"

echo "✅ Deployment complete!"
```

Make executable:
```bash
chmod +x deploy.sh
./deploy.sh
```

---

## Option 2: Netlify

### Step 1: Install Netlify CLI

```bash
npm install -g netlify-cli

# Login
netlify login
```

### Step 2: Deploy

```bash
# Build
npm run build

# Deploy
netlify deploy --prod --dir=dist
```

### Step 3: Configure `netlify.toml`

```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    X-XSS-Protection = "1; mode=block"
```

### Step 4: Connect Git Repository

1. Push to GitHub
2. Go to Netlify Dashboard
3. "New site from Git"
4. Select repository
5. Auto-deploy on push!

---

## Option 3: Vercel

### Step 1: Install Vercel CLI

```bash
npm install -g vercel

# Login
vercel login
```

### Step 2: Deploy

```bash
# Deploy
vercel --prod
```

### Step 3: Configure `vercel.json`

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        }
      ]
    }
  ]
}
```

---

## Option 4: Cloudflare Pages

### Via Git Integration

1. Push to GitHub
2. Go to Cloudflare Dashboard
3. "Create a project"
4. Select repository
5. Build settings:
   - Build command: `npm run build`
   - Build output: `dist`
6. Deploy!

### Via CLI

```bash
# Install Wrangler
npm install -g wrangler

# Login
wrangler login

# Deploy
wrangler pages publish dist --project-name bmoi-landing
```

---

## CI/CD with GitHub Actions

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to S3

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Deploy to S3
        uses: jakejarvis/s3-sync-action@master
        with:
          args: --delete
        env:
          AWS_S3_BUCKET: bmoi-landing-prod
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SOURCE_DIR: 'dist'

      - name: Invalidate CloudFront
        uses: chetan/invalidate-cloudfront-action@v2
        env:
          DISTRIBUTION: ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }}
          PATHS: '/*'
          AWS_REGION: 'us-east-1'
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

---

## Environment Variables

### Production `.env`

```bash
PUBLIC_API_URL=https://api.bmoi.com
PUBLIC_APP_URL=https://app.bmoi.com
PUBLIC_SITE_URL=https://bmoi.com
PUBLIC_GA_ID=G-XXXXXXXXXX
PUBLIC_GTM_ID=GTM-XXXXXXX
```

### Security Notes

- ⚠️ Never commit `.env` to Git
- ✅ Use secrets in GitHub Actions
- ✅ Use environment variables in Netlify/Vercel

---

## Post-Deployment Checklist

- [ ] SSL certificate active (HTTPS)
- [ ] Custom domain configured
- [ ] DNS records updated
- [ ] Lighthouse score 95+
- [ ] Google Analytics working
- [ ] Contact form working
- [ ] All links functional
- [ ] Mobile responsive
- [ ] Dark mode working
- [ ] sitemap.xml accessible
- [ ] robots.txt accessible

---

## Monitoring

### Uptime Monitoring

**Free tools:**
- UptimeRobot (https://uptimerobot.com)
- Pingdom (https://pingdom.com)

### Performance Monitoring

**Tools:**
- Google PageSpeed Insights
- Lighthouse CI
- WebPageTest

### Error Monitoring

**Tools:**
- Sentry (optional for static sites)
- Google Analytics (track errors as events)

---

## Rollback Strategy

### S3 + CloudFront

```bash
# List recent deployments (S3 versioning)
aws s3api list-object-versions \
  --bucket bmoi-landing-prod

# Restore previous version
aws s3 cp s3://bmoi-landing-prod/index.html \
  s3://bmoi-landing-prod/index.html \
  --version-id VERSION_ID
```

### Netlify/Vercel

- Use dashboard to rollback to previous deployment
- One-click rollback available

---

## Cost Estimation

### AWS S3 + CloudFront

**Low traffic (< 10K visits/month):**
- S3 storage: $0.10
- S3 requests: $0.05
- CloudFront: $0.50
- **Total: ~$0.65/month**

**Medium traffic (100K visits/month):**
- S3 storage: $0.10
- S3 requests: $0.50
- CloudFront: $5.00
- **Total: ~$5.60/month**

### Netlify/Vercel (Free Tier)

- ✅ 100GB bandwidth
- ✅ Unlimited sites
- ✅ HTTPS included
- ✅ Auto-deploy from Git

**Perfect for small/medium projects!**

---

## Troubleshooting

### Issue: 403 Forbidden on S3

**Solution:** Check bucket policy allows public read

### Issue: CloudFront not updating

**Solution:** Create cache invalidation

```bash
aws cloudfront create-invalidation \
  --distribution-id E1234567890ABC \
  --paths "/*"
```

### Issue: SSL certificate not working

**Solution:** Certificate must be in `us-east-1` for CloudFront

---

**Deployment complete! 🚀**

Your landing page is now live and optimized for global performance!
