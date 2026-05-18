---
name: seo-master
description: >
  A production-grade SEO AI skill that audits, fixes, and optimizes websites to achieve
  Lighthouse 100 scores, 85+ SEO tool ratings, zero critical warnings, and top-5 Google
  rankings. Trigger this skill for ANY of: building a new Next.js site, adding SEO to
  Payload CMS, fixing Lighthouse performance/SEO/accessibility scores, optimizing Core Web
  Vitals (LCP, CLS, FID/INP), generating sitemap.xml, robots.txt, schema markup, Open Graph
  tags, JSON-LD, meta tags, image optimization, font optimization, page speed issues, running
  SEO audits via curl, checking headers, fixing 404s, canonical URLs, hreflang, structured data
  errors, Google Search Console warnings, Ahrefs/Semrush/Screaming Frog errors, missing alt
  text, slow TTFB, render-blocking resources, unused JavaScript, layout shift, cumulative layout
  shift, largest contentful paint, interaction to next paint, or anything related to making a
  site rank #1–5 on Google. Always use this skill when the user mentions Next.js SEO, Payload
  CMS SEO, Lighthouse scores, or wants their site to rank higher. Never skip this skill for
  performance or ranking questions — it contains critical implementation patterns and bash
  audit commands the model cannot replicate from memory alone.
---

# SEO Master — Production SEO for Next.js + Payload CMS

A comprehensive, code-first SEO skill targeting:
- ✅ Lighthouse 100/100/100/100 (Performance / Accessibility / Best Practices / SEO)
- ✅ 85+ on Ahrefs Site Audit, Semrush, Screaming Frog, Moz
- ✅ Zero critical SEO warnings or errors
- ✅ Top-5 Google rankings for target keywords
- ✅ Perfect Core Web Vitals (LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms)

---

## PHASE 0 — Audit First, Fix Second

Before writing any code, run a live audit. Always do this when given a URL.

```bash
# === 1. HTTP Headers & Redirects ===
curl -sI https://DOMAIN.com | head -30
curl -sIL https://DOMAIN.com  # Follow all redirects, show full chain

# === 2. Check canonical, robots, sitemap in <head> ===
curl -s https://DOMAIN.com | grep -iE '(canonical|robots|sitemap|og:|twitter:|viewport|charset|description)' | head -40

# === 3. Verify sitemap exists and is valid XML ===
curl -s https://DOMAIN.com/sitemap.xml | head -60
curl -s https://DOMAIN.com/sitemap_index.xml | head -30

# === 4. Check robots.txt ===
curl -s https://DOMAIN.com/robots.txt

# === 5. Check page title, h1, meta description ===
curl -s https://DOMAIN.com | grep -iE '(<title>|<h1|<meta name="description")' | head -10

# === 6. Check for noindex tags (should NOT be on public pages) ===
curl -s https://DOMAIN.com | grep -i 'noindex'

# === 7. Check response time (TTFB proxy) ===
curl -o /dev/null -s -w "DNS: %{time_namelookup}s | Connect: %{time_connect}s | TTFB: %{time_starttransfer}s | Total: %{time_total}s\n" https://DOMAIN.com

# === 8. Check compression (gzip/brotli) ===
curl -sI -H "Accept-Encoding: br, gzip" https://DOMAIN.com | grep -i 'content-encoding'

# === 9. Check cache-control headers ===
curl -sI https://DOMAIN.com/favicon.ico | grep -i 'cache'
curl -sI https://DOMAIN.com/_next/static/chunks/main.js 2>/dev/null | grep -i 'cache'

# === 10. Validate structured data (JSON-LD) ===
curl -s https://DOMAIN.com | grep -A 50 'application/ld+json'

# === 11. Check image alt attributes ===
curl -s https://DOMAIN.com | grep -oE '<img[^>]*>' | grep -v 'alt="[^"]' | head -20

# === 12. Check Open Graph completeness ===
curl -s https://DOMAIN.com | grep -iE 'og:(title|description|image|url|type)'

# === 13. Check Twitter Card ===
curl -s https://DOMAIN.com | grep -i 'twitter:'

# === 14. Check HTTP/2 or HTTP/3 support ===
curl -sI --http2 https://DOMAIN.com | head -5

# === 15. Check security headers (affects Best Practices score) ===
curl -sI https://DOMAIN.com | grep -iE '(strict-transport|x-content-type|x-frame|content-security|permissions-policy|referrer-policy)'
```

**Interpret results and create a prioritized fix list before writing any code.**

---

## PHASE 1 — Next.js 16 SEO Foundation

Read `references/nextjs-seo.md` for full implementation.  
**Quick reference:**

### 1.1 Metadata API (App Router — required)

Every page/layout MUST export metadata. Never use bare `<head>` tags.

```typescript
// app/layout.tsx — Site-wide defaults
import type { Metadata } from 'next'

export const metadata: Metadata = {
  metadataBase: new URL('https://yoursite.com'),
  title: {
    default: 'Your Site Name',
    template: '%s | Your Site Name',   // Page title | Site name
  },
  description: 'Your default meta description — 150-160 chars, keyword-rich.',
  keywords: ['keyword1', 'keyword2'],
  authors: [{ name: 'Author Name' }],
  creator: 'Your Company',
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
  openGraph: {
    type: 'website',
    locale: 'en_US',
    url: 'https://yoursite.com',
    siteName: 'Your Site Name',
    title: 'Your Site Name',
    description: 'Your OG description.',
    images: [{ url: '/og-image.jpg', width: 1200, height: 630, alt: 'Site preview' }],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Your Site Name',
    description: 'Your Twitter description.',
    images: ['/og-image.jpg'],
    creator: '@yourhandle',
  },
  alternates: {
    canonical: 'https://yoursite.com',
  },
  verification: {
    google: 'your-google-verification-token',
  },
}
```

### 1.2 Dynamic Metadata per Page

```typescript
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next'

export async function generateMetadata({ params }: { params: { slug: string } }): Promise<Metadata> {
  const post = await getPost(params.slug) // fetch from Payload CMS

  return {
    title: post.seo?.title || post.title,
    description: post.seo?.description || post.excerpt,
    alternates: { canonical: `https://yoursite.com/blog/${params.slug}` },
    openGraph: {
      title: post.seo?.ogTitle || post.title,
      description: post.seo?.ogDescription || post.excerpt,
      images: post.seo?.ogImage ? [{ url: post.seo.ogImage.url, width: 1200, height: 630 }] : [],
      type: 'article',
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: [post.author?.name],
    },
  }
}
```

### 1.3 Sitemap (auto-generated)

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next'

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://yoursite.com'

  // Fetch dynamic pages from Payload CMS
  const posts = await fetch(`${process.env.PAYLOAD_URL}/api/posts?limit=1000&depth=0`)
    .then(r => r.json()).then(d => d.docs)
  const pages = await fetch(`${process.env.PAYLOAD_URL}/api/pages?limit=1000&depth=0`)
    .then(r => r.json()).then(d => d.docs)

  const staticRoutes: MetadataRoute.Sitemap = [
    { url: baseUrl, lastModified: new Date(), changeFrequency: 'daily', priority: 1 },
    { url: `${baseUrl}/about`, lastModified: new Date(), changeFrequency: 'monthly', priority: 0.8 },
    { url: `${baseUrl}/blog`, lastModified: new Date(), changeFrequency: 'daily', priority: 0.9 },
    { url: `${baseUrl}/contact`, lastModified: new Date(), changeFrequency: 'yearly', priority: 0.5 },
  ]

  const dynamicPostRoutes: MetadataRoute.Sitemap = posts.map((post: any) => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: new Date(post.updatedAt),
    changeFrequency: 'weekly',
    priority: 0.7,
  }))

  const dynamicPageRoutes: MetadataRoute.Sitemap = pages.map((page: any) => ({
    url: `${baseUrl}/${page.slug}`,
    lastModified: new Date(page.updatedAt),
    changeFrequency: 'monthly',
    priority: 0.8,
  }))

  return [...staticRoutes, ...dynamicPostRoutes, ...dynamicPageRoutes]
}
```

### 1.4 Robots.txt

```typescript
// app/robots.ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: '*',
        allow: '/',
        disallow: ['/api/', '/admin/', '/_next/', '/private/'],
      },
      {
        userAgent: 'Googlebot',
        allow: '/',
        disallow: ['/api/', '/admin/'],
      },
    ],
    sitemap: 'https://yoursite.com/sitemap.xml',
    host: 'https://yoursite.com',
  }
}
```

---

## PHASE 2 — Payload CMS SEO Integration

Read `references/payload-seo.md` for full implementation.

### 2.1 SEO Plugin (official)

```bash
npm install @payloadcms/plugin-seo
```

```typescript
// payload.config.ts
import { seoPlugin } from '@payloadcms/plugin-seo'
import { GenerateTitle, GenerateURL, GenerateDescription } from '@payloadcms/plugin-seo/types'

const generateTitle: GenerateTitle = ({ doc }) => `${doc.title} | My Site`
const generateURL: GenerateURL = ({ doc }) => `https://yoursite.com/${doc?.slug}`
const generateDescription: GenerateDescription = ({ doc }) => doc?.excerpt

export default buildConfig({
  plugins: [
    seoPlugin({
      collections: ['pages', 'posts', 'products'],
      globals: ['settings'],
      uploadsCollection: 'media',
      generateTitle,
      generateURL,
      generateDescription,
      tabbedUI: true,
    }),
  ],
})
```

### 2.2 Manual SEO Field Group (if not using plugin)

```typescript
// collections/Posts.ts
import { CollectionConfig } from 'payload/types'

const Posts: CollectionConfig = {
  slug: 'posts',
  fields: [
    // ... your content fields
    {
      name: 'seo',
      label: 'SEO',
      type: 'group',
      fields: [
        { name: 'title', type: 'text', maxLength: 60, label: 'SEO Title (max 60 chars)' },
        { name: 'description', type: 'textarea', maxLength: 160, label: 'Meta Description (max 160 chars)' },
        { name: 'keywords', type: 'text', label: 'Keywords (comma-separated)' },
        { name: 'ogTitle', type: 'text', maxLength: 70, label: 'OG Title' },
        { name: 'ogDescription', type: 'textarea', maxLength: 200, label: 'OG Description' },
        { name: 'ogImage', type: 'upload', relationTo: 'media', label: 'OG Image (1200×630)' },
        { name: 'noIndex', type: 'checkbox', label: 'No Index (exclude from Google)' },
        { name: 'canonicalUrl', type: 'text', label: 'Canonical URL (leave blank for auto)' },
        { name: 'schema', type: 'json', label: 'Custom JSON-LD Schema' },
      ],
    },
  ],
}
```

---

## PHASE 3 — Performance (Lighthouse 100)

Read `references/performance.md` for complete configs.

### Critical next.config.ts settings

```typescript
// next.config.ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  // Image optimization
  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 31536000, // 1 year
    remotePatterns: [
      { protocol: 'https', hostname: 'your-payload-domain.com' },
    ],
  },
  // Compression
  compress: true,
  // Strict mode for better React perf
  reactStrictMode: true,
  // Standalone for optimal Docker builds
  output: 'standalone',
  // Experimental optimizations
  experimental: {
    optimizePackageImports: ['lucide-react', '@radix-ui/react-icons', 'lodash'],
    turbo: {},
  },
  // Security headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-XSS-Protection', value: '1; mode=block' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=(), geolocation=()' },
          { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains; preload' },
        ],
      },
      // Immutable cache for Next.js static assets
      {
        source: '/_next/static/(.*)',
        headers: [{ key: 'Cache-Control', value: 'public, max-age=31536000, immutable' }],
      },
      // Long cache for media
      {
        source: '/images/(.*)',
        headers: [{ key: 'Cache-Control', value: 'public, max-age=86400, stale-while-revalidate=604800' }],
      },
    ]
  },
  // Redirects for SEO (www → non-www, etc.)
  async redirects() {
    return [
      {
        source: '/home',
        destination: '/',
        permanent: true, // 301
      },
    ]
  },
}

export default nextConfig
```

---

## PHASE 4 — Schema Markup (JSON-LD)

Read `references/schema-markup.md` for all schema types.

### Reusable SchemaOrg component

```tsx
// components/SchemaOrg.tsx
interface SchemaOrgProps {
  schema: Record<string, unknown> | Record<string, unknown>[]
}

export function SchemaOrg({ schema }: SchemaOrgProps) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  )
}

// Usage in layout.tsx:
// <SchemaOrg schema={organizationSchema} />
```

---

## PHASE 5 — Fonts & Assets

```tsx
// app/layout.tsx — Optimal font loading
import { Inter, Playfair_Display } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',           // Never block rendering
  preload: true,
  variable: '--font-inter',
  fallback: ['system-ui', 'arial'],
})

const playfair = Playfair_Display({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-playfair',
  weight: ['400', '700'],
})
```

---

## PHASE 6 — Image Optimization Rules

Always use `next/image`. Never use bare `<img>` tags on content pages.

```tsx
import Image from 'next/image'

// Hero image (above fold) — always eager + high priority
<Image
  src="/hero.jpg"
  alt="Descriptive alt text with keyword naturally included"
  width={1200}
  height={630}
  priority           // Preloads — use for LCP element
  quality={85}
  sizes="100vw"
/>

// Below-fold images — lazy by default
<Image
  src="/feature.jpg"
  alt="Specific description of what's shown"
  width={800}
  height={450}
  quality={80}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

---

## PHASE 7 — On-Page SEO Rules

**Title tags:**
- 50–60 characters
- Include primary keyword near the front
- Format: `Primary Keyword — Secondary Keyword | Brand`

**Meta descriptions:**
- 150–160 characters
- Include keyword naturally, include a CTA
- Never duplicate across pages

**Headings:**
- One `<h1>` per page, includes primary keyword
- `<h2>` for major sections
- `<h3>` for subsections — logical hierarchy only
- Never skip levels (h1 → h3)

**URL structure:**
- Lowercase, hyphens only, no underscores
- Descriptive: `/blog/nextjs-seo-guide` not `/blog/post-123`
- Short: ≤ 5 words preferred
- No stop words: not `/blog/how-to-do-seo-in-nextjs`

**Internal linking:**
- Every page should link to 3–7 relevant internal pages
- Use keyword-rich anchor text (not "click here")
- Link from high-authority pages to important new pages

---

## PHASE 8 — Core Web Vitals Fixes

Read `references/core-web-vitals.md` for deep-dive solutions.

| Metric | Target | Common Causes | Fix |
|--------|--------|--------------|-----|
| LCP | ≤ 2.5s | Slow server, large hero image, render-blocking CSS | Use `priority` on hero image, CDN, ISR |
| CLS | ≤ 0.1 | Images without dimensions, web fonts, ads | Always set width/height, `font-display: swap` |
| INP | ≤ 200ms | Heavy JS on interaction, long tasks | Code split, `useTransition`, defer non-critical JS |
| FCP | ≤ 1.8s | Render-blocking resources | Inline critical CSS, defer scripts |
| TTFB | ≤ 800ms | Slow API, no caching | ISR/SSG, edge functions, Payload caching |

---

## PHASE 9 — ISR & Caching Strategy

```typescript
// Static pages with ISR (best for SEO + performance)
export const revalidate = 3600 // Revalidate every hour

// Or per-fetch
const data = await fetch('https://...', {
  next: { revalidate: 3600 }
})

// On-demand revalidation (call from Payload webhook)
// app/api/revalidate/route.ts
import { revalidatePath, revalidateTag } from 'next/cache'
import { NextRequest } from 'next/server'

export async function POST(req: NextRequest) {
  const secret = req.nextUrl.searchParams.get('secret')
  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Invalid secret' }, { status: 401 })
  }
  const { path, tag } = await req.json()
  if (tag) revalidateTag(tag)
  if (path) revalidatePath(path)
  return Response.json({ revalidated: true })
}
```

---

## PHASE 10 — Audit Checklist (run before every deploy)

```bash
# Run all checks against your staging URL
SITE="https://staging.yoursite.com"

echo "=== SEO AUDIT ==="
echo "--- Robots.txt ---"
curl -s "$SITE/robots.txt"

echo "--- Sitemap ---"
curl -s "$SITE/sitemap.xml" | grep '<loc>' | head -20

echo "--- Title & Meta ---"
curl -s "$SITE" | grep -iE '(<title>|name="description"|canonical)'

echo "--- OG Tags ---"
curl -s "$SITE" | grep 'og:'

echo "--- JSON-LD ---"
curl -s "$SITE" | python3 -c "
import sys, re, json
html = sys.stdin.read()
schemas = re.findall(r'<script type=\"application/ld\+json\">(.*?)</script>', html, re.DOTALL)
for s in schemas:
    try: print(json.dumps(json.loads(s), indent=2))
    except: print('Invalid JSON-LD found!')
"

echo "--- Response Time ---"
curl -o /dev/null -s -w "TTFB: %{time_starttransfer}s | Total: %{time_total}s\n" "$SITE"

echo "--- Compression ---"
curl -sI -H "Accept-Encoding: br,gzip" "$SITE" | grep -i 'content-encoding'

echo "--- Security Headers ---"
curl -sI "$SITE" | grep -iE '(strict-transport|x-content|x-frame|content-security|permissions)'

echo "--- HTTP/2 ---"
curl -sI --http2 "$SITE" | head -2

echo "--- Noindex Check (should be empty on public pages) ---"
curl -s "$SITE" | grep -i 'noindex'

echo "--- Canonical ---"
curl -s "$SITE" | grep -i 'canonical'

echo "--- Image Alt Attributes (missing alt = problem) ---"
curl -s "$SITE" | grep -oE '<img[^>]+>' | grep -v 'alt=' | head -10

echo "=== DONE ==="
```

---

## Reference Files

| File | When to Read |
|------|-------------|
| `references/nextjs-seo.md` | Full Next.js 16 App Router SEO patterns |
| `references/payload-seo.md` | Payload CMS collections, hooks, webhooks for SEO |
| `references/performance.md` | Lighthouse 100 configs, bundle analysis, code splitting |
| `references/schema-markup.md` | All JSON-LD schema types with full examples |
| `references/core-web-vitals.md` | LCP, CLS, INP deep fixes with code |
| `references/on-page-seo.md` | Title/meta rules, heading structure, internal linking |
| `references/keyword-strategy.md` | Keyword research process, intent mapping, clustering |
| `references/content-strategy.md` | Pillar pages, topic clusters, content calendar |
| `references/monitoring.md` | Search Console, Ahrefs, Vercel Analytics setup |

---

## Priority Order for New Projects

1. **Domain & HTTPS** — SSL, www redirect, custom domain
2. **next.config.ts** — Headers, compression, images
3. **Root metadata** — metadataBase, default title template
4. **Robots.txt + Sitemap** — Index all public pages
5. **Per-page metadata** — Title, description, canonical, OG
6. **JSON-LD schemas** — Organization, WebSite, BreadcrumbList minimum
7. **Image optimization** — `next/image` everywhere, `priority` on LCP
8. **Font optimization** — `next/font`, `display: swap`
9. **Payload SEO plugin** — SEO fields on all collections
10. **ISR/caching** — Revalidation strategy
11. **Core Web Vitals** — After build, audit and fix
12. **Content & keywords** — On-page content optimization
13. **Internal linking** — Systematic internal link audit
14. **Monitoring** — GSC, Analytics, uptime
