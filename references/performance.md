# Performance — Lighthouse 100 Reference

## Lighthouse Score Breakdown

| Category | Weight | What Fails It |
|----------|--------|--------------|
| Performance | LCP, CLS, INP, FCP, Speed Index, TBT | Slow server, large images, render-blocking JS/CSS, long tasks |
| Accessibility | 100% | Missing alt, no ARIA labels, poor contrast, non-semantic HTML |
| Best Practices | 100% | HTTP (not HTTPS), security headers, no deprecated APIs |
| SEO | 100% | Missing meta, noindex on public pages, broken links, non-crawlable content |

---

## next.config.ts — Maximum Performance

```typescript
// next.config.ts
import type { NextConfig } from 'next'
import bundleAnalyzer from '@next/bundle-analyzer'

const withBundleAnalyzer = bundleAnalyzer({
  enabled: process.env.ANALYZE === 'true',
})

const nextConfig: NextConfig = {
  reactStrictMode: true,
  compress: true,
  poweredByHeader: false,        // Remove X-Powered-By header

  images: {
    formats: ['image/avif', 'image/webp'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    minimumCacheTTL: 60 * 60 * 24 * 365, // 1 year
    dangerouslyAllowSVG: false,
    contentSecurityPolicy: "default-src 'self'; script-src 'none'; sandbox;",
  },

  experimental: {
    optimizePackageImports: [
      'lucide-react',
      '@radix-ui/react-icons',
      '@heroicons/react',
      'lodash',
      'date-fns',
      'framer-motion',
    ],
    // Partial prerendering (PPR) — Next.js 15+
    // ppr: true,
  },

  // Bundle splitting
  webpack: (config, { dev, isServer }) => {
    if (!dev && !isServer) {
      config.optimization.splitChunks = {
        chunks: 'all',
        cacheGroups: {
          vendor: {
            test: /[\\/]node_modules[\\/]/,
            name(module: any) {
              const packageName = module.context.match(/[\\/]node_modules[\\/](.*?)([\\/]|$)/)[1]
              return `npm.${packageName.replace('@', '')}`
            },
          },
        },
      }
    }
    return config
  },

  async headers() {
    return [
      // Security headers — required for Lighthouse Best Practices 100
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Strict-Transport-Security',
            value: 'max-age=31536000; includeSubDomains; preload',
          },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
          { key: 'X-XSS-Protection', value: '1; mode=block' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          {
            key: 'Permissions-Policy',
            value: 'camera=(), microphone=(), geolocation=(), interest-cohort=()',
          },
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-eval' 'unsafe-inline' https://www.googletagmanager.com https://www.google-analytics.com",
              "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
              "font-src 'self' https://fonts.gstatic.com",
              "img-src 'self' data: blob: https:",
              "connect-src 'self' https://www.google-analytics.com",
              "frame-ancestors 'none'",
            ].join('; '),
          },
        ],
      },
      // Immutable cache for Next.js static chunks
      {
        source: '/_next/static/(.*)',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=31536000, immutable' },
        ],
      },
      // Images cache
      {
        source: '/images/(.*)',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=86400, stale-while-revalidate=604800' },
        ],
      },
      // Media from Payload
      {
        source: '/media/(.*)',
        headers: [
          { key: 'Cache-Control', value: 'public, max-age=86400, stale-while-revalidate=2592000' },
        ],
      },
    ]
  },
}

export default withBundleAnalyzer(nextConfig)
```

---

## Analyze Your Bundle

```bash
# Install analyzer
npm install --save-dev @next/bundle-analyzer

# Run analysis
ANALYZE=true npm run build

# This opens a visual treemap of your JS bundle.
# Target: no single chunk > 200KB (gzipped)
# Move large dependencies to dynamic imports
```

---

## Code Splitting — Dynamic Imports

```tsx
// BEFORE — bad: entire library shipped to client
import { HeavyComponent } from 'heavy-library'

// AFTER — good: lazy loaded only when needed
import dynamic from 'next/dynamic'

const HeavyComponent = dynamic(
  () => import('heavy-library').then(mod => mod.HeavyComponent),
  {
    loading: () => <div className="animate-pulse h-64 bg-muted rounded" />,
    ssr: false, // set true if component needs SSR
  }
)

// Conditionally load (e.g., only on user interaction)
const VideoPlayer = dynamic(() => import('@/components/VideoPlayer'), {
  ssr: false,
})

// In component:
const [showVideo, setShowVideo] = useState(false)
return showVideo ? <VideoPlayer /> : <button onClick={() => setShowVideo(true)}>Play</button>
```

---

## Critical CSS Inlining (avoid render-blocking)

```tsx
// app/layout.tsx — inline critical CSS only
// Tailwind handles this well — BUT for custom critical styles:

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        {/* Inline only critical above-fold CSS */}
        <style dangerouslySetInnerHTML={{ __html: `
          /* Critical: layout, fonts, hero */
          *,*::before,*::after{box-sizing:border-box}
          body{margin:0;font-family:var(--font-sans,system-ui);line-height:1.5}
          .hero{min-height:100svh;display:flex;align-items:center}
        `}} />
      </head>
      <body>{children}</body>
    </html>
  )
}
```

---

## Reduce Total Blocking Time (TBT)

TBT = time the main thread is blocked > 50ms. Target: < 200ms.

```tsx
// Use useTransition for non-urgent state updates
import { useTransition, useState } from 'react'

function SearchComponent() {
  const [query, setQuery] = useState('')
  const [results, setResults] = useState([])
  const [isPending, startTransition] = useTransition()

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(e.target.value) // Urgent: update input immediately
    startTransition(() => {
      // Non-urgent: can be interrupted
      setResults(search(e.target.value))
    })
  }

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending ? <Spinner /> : <ResultsList results={results} />}
    </div>
  )
}

// Defer non-critical scripts
// In layout.tsx:
<Script
  src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXX"
  strategy="afterInteractive"  // loads after page is interactive
/>

// For non-essential third-party scripts:
<Script src="https://..." strategy="lazyOnload" />
// strategy options:
// "beforeInteractive" — blocks page (use for polyfills only)
// "afterInteractive"  — after hydration (analytics, chat widgets)
// "lazyOnload"        — during idle time (surveys, social embeds)
// "worker"            — in web worker (experimental, fastest)
```

---

## Preloading Critical Resources

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <head>
        {/* Preconnect to external domains */}
        <link rel="preconnect" href="https://fonts.googleapis.com" />
        <link rel="preconnect" href="https://fonts.gstatic.com" crossOrigin="anonymous" />
        <link rel="preconnect" href="https://www.googletagmanager.com" />
        
        {/* DNS prefetch for domains we don't connect to immediately */}
        <link rel="dns-prefetch" href="https://cms.yoursite.com" />
        
        {/* Preload hero image (helps LCP) */}
        <link
          rel="preload"
          as="image"
          href="/hero.jpg"
          imageSrcSet="/hero-640.jpg 640w, /hero-1200.jpg 1200w"
          imageSizes="100vw"
        />
      </head>
      <body>{children}</body>
    </html>
  )
}
```

---

## Server Components Strategy (zero JS shipped)

```tsx
// Server component (default in App Router) — NO JS sent to client
// Use for: data fetching, static content, non-interactive UI
async function BlogList() {
  const posts = await getPosts() // Direct DB/API call, no useEffect
  return (
    <ul>
      {posts.map(post => (
        <li key={post.id}>
          <a href={`/blog/${post.slug}`}>{post.title}</a>
        </li>
      ))}
    </ul>
  )
}

// Client component — only when you need interactivity
'use client'
import { useState } from 'react'

function LikeButton({ postId }: { postId: string }) {
  const [liked, setLiked] = useState(false)
  return <button onClick={() => setLiked(!liked)}>{liked ? '❤️' : '🤍'}</button>
}

// PATTERN: Keep client components small and at the leaf
// Server: Layout, page, data fetching, static content
// Client: Forms, buttons, modals, animations, browser APIs
```

---

## Edge Functions (Lowest TTFB)

```typescript
// For pages that need dynamic data but max performance
// middleware.ts — runs at the edge (fastest possible)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // Geolocation, A/B testing, auth redirects — all at the edge
  const country = request.geo?.country || 'US'
  const response = NextResponse.next()
  response.headers.set('x-user-country', country)
  return response
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}

// Route handler at edge:
// app/api/hello/route.ts
export const runtime = 'edge' // Runs at CDN edge, 0ms cold start

export async function GET() {
  return Response.json({ hello: 'world' })
}
```

---

## ISR — Incremental Static Regeneration

```typescript
// Best of SSG + SSR for SEO

// Option 1: Time-based (simple)
export const revalidate = 3600 // Revalidate page every hour

// Option 2: Tag-based (precise) — preferred
export default async function Page() {
  const data = await fetch('https://...', {
    next: {
      tags: ['posts'],          // Tag for targeted revalidation
      revalidate: 86400,        // Max age 24h as fallback
    }
  })
  // ...
}

// Trigger revalidation from Payload webhook:
import { revalidateTag, revalidatePath } from 'next/cache'

export async function POST(req: Request) {
  const { secret, tag, path } = await req.json()
  if (secret !== process.env.REVALIDATION_SECRET) return new Response('Unauthorized', { status: 401 })

  if (tag) revalidateTag(tag)
  if (path) revalidatePath(path)

  return Response.json({ revalidated: true, timestamp: Date.now() })
}
```

---

## Vercel / Deployment Optimizations

```json
// vercel.json
{
  "functions": {
    "app/api/**": { "maxDuration": 30 }
  },
  "regions": ["iad1"],           // Pick region closest to your users
  "framework": "nextjs"
}
```

```bash
# Check Vercel Edge Network caching:
curl -sI https://yoursite.com | grep -i 'x-vercel-cache'
# HIT = served from edge cache (fastest)
# MISS = fetched from origin (first request or revalidated)
# STALE = served stale while revalidating
```

---

## Accessibility — Lighthouse 100

```tsx
// Required for Lighthouse Accessibility 100:

// 1. lang attribute on <html>
<html lang="en">

// 2. Every image has descriptive alt
<Image alt="Specific description" ... />
// Decorative images:
<Image alt="" role="presentation" ... />

// 3. Buttons have accessible text
<button aria-label="Close dialog">✕</button>

// 4. Links are descriptive (not "click here")
<a href="/blog/seo-guide">Read our Next.js SEO guide</a>

// 5. Form inputs have labels
<label htmlFor="email">Email address</label>
<input id="email" type="email" />

// 6. Color contrast ≥ 4.5:1 (normal text), 3:1 (large text)
// Test: https://webaim.org/resources/contrastchecker/

// 7. Focus visible styles
:focus-visible { outline: 2px solid #005fcc; outline-offset: 2px; }

// 8. Skip to main content link
<a href="#main" className="sr-only focus:not-sr-only">Skip to content</a>

// 9. ARIA landmarks
<header role="banner">
<nav aria-label="Main navigation">
<main id="main">
<footer role="contentinfo">

// 10. Heading hierarchy (never skip)
<h1>Page title</h1>
  <h2>Section</h2>
    <h3>Subsection</h3>
```

---

## Bundle Analysis Commands

```bash
# Analyze bundle size
ANALYZE=true npm run build

# Check what's being loaded
npx source-map-explorer .next/static/chunks/*.js

# Find large dependencies
npx cost-of-modules

# Check page weight
curl -so /dev/null -w '%{size_download}' https://yoursite.com
# Target: < 100KB HTML, < 50KB critical CSS, < 200KB critical JS

# Check gzip ratio
curl -s -H "Accept-Encoding: gzip" -o /dev/null -w '%{size_download}' https://yoursite.com
```
