# Core Web Vitals — Deep Fix Reference

## Quick Diagnosis Commands

```bash
SITE="https://yoursite.com"

# Check TTFB (proxy for LCP server time)
curl -o /dev/null -s -w "TTFB: %{time_starttransfer}s\n" "$SITE"
# Target: < 0.8s. If > 1s, you have a server/caching problem.

# Check content size (affects FCP/LCP)
curl -s -o /dev/null -w "HTML size: %{size_download} bytes\n" "$SITE"
# Target: < 100KB HTML

# Check compression
curl -sI -H "Accept-Encoding: br,gzip" "$SITE" | grep -i content-encoding
# Should return: content-encoding: br (brotli) or gzip

# Check HTTP/2 (affects parallel loading)
curl -sI --http2 "$SITE" | head -2
# Should return: HTTP/2 200

# Check CDN cache hit
curl -sI "$SITE" | grep -i 'x-vercel-cache\|cf-cache-status\|x-cache'
# Should return HIT for cached pages
```

---

## LCP — Largest Contentful Paint (≤ 2.5s)

The LCP element is usually: hero image, hero heading, or large text block.

### Step 1: Identify your LCP element
Open Chrome DevTools → Performance tab → record page load → look for "LCP" marker. Or use:
```javascript
// In browser console:
new PerformanceObserver((list) => {
  const entries = list.getEntries()
  const lastEntry = entries[entries.length - 1]
  console.log('LCP element:', lastEntry.element)
  console.log('LCP time:', lastEntry.startTime)
}).observe({ type: 'largest-contentful-paint', buffered: true })
```

### Step 2: Fix LCP image
```tsx
// ❌ WRONG — no priority, no preload
<Image src="/hero.jpg" alt="Hero" width={1920} height={1080} />

// ✅ RIGHT — priority triggers <link rel="preload">
<Image
  src="/hero.jpg"
  alt="Hero — descriptive text"
  width={1920}
  height={1080}
  priority              // CRITICAL for LCP element
  quality={85}
  sizes="100vw"
  placeholder="blur"
  blurDataURL={blurDataURL}
/>
```

### Step 3: Fix LCP text (if heading is LCP)
```css
/* Ensure font loads fast — avoid FOIT (Flash of Invisible Text) */
@font-face {
  font-display: swap; /* Show fallback font immediately */
}

/* Size fallback font to match web font (reduces CLS on font swap) */
/* Use https://screenspan.net/fallback to generate these values */
@font-face {
  font-family: 'Inter-fallback';
  src: local('Arial');
  ascent-override: 90.20%;
  descent-override: 22.48%;
  line-gap-override: 0%;
  size-adjust: 107.40%;
}
```

### Step 4: Reduce server response time
```typescript
// Use ISR or SSG — avoid getServerSideProps for high-traffic pages
// app/page.tsx
export const revalidate = 3600 // Static with hourly revalidation

// For Payload CMS data — use fetch tags for precise revalidation
const data = await fetch(`${PAYLOAD_URL}/api/pages/home`, {
  next: { tags: ['home-page'], revalidate: 86400 }
})
```

### Step 5: Eliminate render-blocking resources
```bash
# Find render-blocking scripts:
curl -s https://yoursite.com | grep -oE '<script[^>]*src="[^"]*"[^>]*>' | grep -v 'async\|defer\|type="module"'
# Any result here is a render-blocking script — add async/defer
```

---

## CLS — Cumulative Layout Shift (≤ 0.1)

CLS happens when elements move unexpectedly during load.

### Fix 1: Always set image dimensions
```tsx
// ❌ WRONG — causes layout shift
<img src="/photo.jpg" />

// ✅ RIGHT — reserves space before image loads
<Image src="/photo.jpg" alt="Photo" width={800} height={600} />

// For unknown dimensions (e.g. from CMS), use aspect-ratio:
<div style={{ aspectRatio: '16/9', position: 'relative' }}>
  <Image src={imageUrl} alt={alt} fill style={{ objectFit: 'cover' }} />
</div>
```

### Fix 2: Reserve space for dynamic content
```tsx
// ❌ WRONG — button appears after JS loads, shifts layout
const [isLoggedIn, setIsLoggedIn] = useState(false)
useEffect(() => {
  setIsLoggedIn(checkAuth())
}, [])
return isLoggedIn ? <UserMenu /> : null // Shifts layout on load!

// ✅ RIGHT — reserve fixed dimensions
return (
  <div style={{ width: 120, height: 36 }}>
    {isLoggedIn ? <UserMenu /> : <LoginButton />}
  </div>
)
```

### Fix 3: Font display: swap + size-adjust
```typescript
// next/font handles this automatically:
import { Inter } from 'next/font/google'
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  adjustFontFallback: true, // ← This generates size-adjust for the fallback
})
// adjustFontFallback minimizes CLS when web font loads and swaps in
```

### Fix 4: Avoid inserting content above existing content
```tsx
// ❌ WRONG — cookie banner inserted above page pushes content down
<CookieBanner /> {/* Appears at top, shifts everything */}
<main>...</main>

// ✅ RIGHT — fixed position or appear from bottom
<CookieBanner style={{ position: 'fixed', bottom: 0, left: 0, right: 0 }} />
```

### Fix 5: Ads and embeds
```tsx
// Always reserve space for ads and embeds
<div style={{ minHeight: 250, width: '100%' }}>
  <AdComponent />
</div>
```

### Measure CLS in browser:
```javascript
// Browser console — measure CLS:
let clsValue = 0
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      clsValue += entry.value
      console.log('CLS so far:', clsValue, 'from:', entry.sources?.[0]?.node)
    }
  }
}).observe({ type: 'layout-shift', buffered: true })
```

---

## INP — Interaction to Next Paint (≤ 200ms)

INP replaced FID. It measures responsiveness to ALL interactions, not just the first.

### Find slow interactions:
```javascript
// Browser console:
new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.duration > 100) {
      console.log(`Slow ${entry.name}: ${entry.duration}ms`, entry)
    }
  }
}).observe({ type: 'event', durationThreshold: 100, buffered: true })
```

### Fix 1: Break up long tasks
```typescript
// ❌ WRONG — blocks main thread for > 50ms
function processLargeArray(items: any[]) {
  return items.map(item => expensiveOperation(item)) // 500ms+
}

// ✅ RIGHT — yield to browser between chunks
async function processLargeArray(items: any[]) {
  const results = []
  for (let i = 0; i < items.length; i += 50) {
    const chunk = items.slice(i, i + 50)
    results.push(...chunk.map(item => expensiveOperation(item)))
    // Yield control back to browser
    await new Promise(resolve => setTimeout(resolve, 0))
  }
  return results
}

// Or use scheduler.yield() in modern browsers:
async function processLargeArray(items: any[]) {
  const results = []
  for (const item of items) {
    results.push(expensiveOperation(item))
    if ('scheduler' in window && 'yield' in (window as any).scheduler) {
      await (window as any).scheduler.yield()
    }
  }
  return results
}
```

### Fix 2: Debounce expensive handlers
```typescript
import { useDeferredValue, useState, useTransition } from 'react'

// Search input — debounce filter
function SearchFilter({ items }: { items: any[] }) {
  const [query, setQuery] = useState('')
  const deferredQuery = useDeferredValue(query) // Low-priority update

  const filtered = items.filter(item =>
    item.title.toLowerCase().includes(deferredQuery.toLowerCase())
  )

  return (
    <>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)} // Immediate: responsive
        placeholder="Search..."
      />
      <ul>
        {filtered.map(item => <li key={item.id}>{item.title}</li>)}
      </ul>
    </>
  )
}
```

### Fix 3: Virtualize long lists
```tsx
// ❌ WRONG — renders 1000 DOM nodes
{posts.map(post => <PostCard key={post.id} post={post} />)}

// ✅ RIGHT — virtualizes, only renders visible items
npm install @tanstack/react-virtual

import { useVirtualizer } from '@tanstack/react-virtual'

function VirtualPostList({ posts }: { posts: any[] }) {
  const parentRef = useRef<HTMLDivElement>(null)
  const rowVirtualizer = useVirtualizer({
    count: posts.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 120, // estimated row height in px
    overscan: 5,
  })

  return (
    <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
      <div style={{ height: `${rowVirtualizer.getTotalSize()}px`, position: 'relative' }}>
        {rowVirtualizer.getVirtualItems().map(vItem => (
          <div
            key={vItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              transform: `translateY(${vItem.start}px)`,
            }}
          >
            <PostCard post={posts[vItem.index]} />
          </div>
        ))}
      </div>
    </div>
  )
}
```

### Fix 4: Move heavy work to Web Worker
```typescript
// workers/search.worker.ts
self.addEventListener('message', ({ data }) => {
  const { items, query } = data
  const results = items.filter((item: any) =>
    item.title.toLowerCase().includes(query.toLowerCase())
  )
  self.postMessage(results)
})

// Usage:
const worker = new Worker(new URL('./workers/search.worker.ts', import.meta.url))
worker.postMessage({ items, query })
worker.onmessage = ({ data }) => setResults(data)
```

---

## FCP — First Contentful Paint (≤ 1.8s)

```tsx
// 1. Inline critical CSS (avoid external stylesheet blocking)
// With Tailwind, critical CSS is usually small enough to inline via CSS-in-JS

// 2. Remove render-blocking scripts
// ❌ WRONG
<script src="/analytics.js"></script>

// ✅ RIGHT
<Script src="/analytics.js" strategy="afterInteractive" />

// 3. Preload critical fonts
// next/font handles this automatically — always use next/font

// 4. Use HTTP/2 Server Push or 103 Early Hints (Vercel supports this)
// next.config.ts
experimental: {
  earlyHints: true, // Send 103 Early Hints for critical resources
}
```

---

## TTFB — Time to First Byte (≤ 800ms)

```bash
# Measure TTFB breakdown:
curl -o /dev/null -s -w \
  "DNS: %{time_namelookup}s\nTCP: %{time_connect}s\nSSL: %{time_appconnect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" \
  https://yoursite.com

# If DNS > 0.1s → use faster DNS (Cloudflare, Google)
# If TCP > 0.1s → use CDN closer to users
# If SSL > 0.2s → enable OCSP stapling, TLS 1.3
# If TTFB > 0.8s → add caching, use SSG/ISR
```

### Fix high TTFB with ISR + caching:
```typescript
// pages that can be static → export const revalidate = N
// pages that must be dynamic → use edge runtime

// app/api/data/route.ts
export const runtime = 'edge' // Lowest possible TTFB

// Add aggressive caching headers to API routes:
export async function GET() {
  const data = await fetchData()
  return Response.json(data, {
    headers: {
      'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400',
    },
  })
}
```

---

## Web Vitals Monitoring

```tsx
// app/components/WebVitals.tsx — report to analytics
'use client'
import { useReportWebVitals } from 'next/web-vitals'

export function WebVitalsReporter() {
  useReportWebVitals((metric) => {
    // Send to Google Analytics 4
    if (typeof window !== 'undefined' && window.gtag) {
      window.gtag('event', metric.name, {
        event_category: 'Web Vitals',
        event_label: metric.id,
        value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
        non_interaction: true,
      })
    }

    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.log(`[Web Vitals] ${metric.name}: ${metric.value}`)
    }
  })
  return null
}

// Add to app/layout.tsx:
<WebVitalsReporter />
```

---

## Lighthouse CI (Automated Testing)

```bash
# Install
npm install --save-dev @lhci/cli

# Run against local build
npm run build && npm run start &
npx lhci collect --url=http://localhost:3000
npx lhci assert --preset=lighthouse:recommended
```

```json
// lighthouserc.json
{
  "ci": {
    "collect": {
      "url": ["http://localhost:3000", "http://localhost:3000/blog"],
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "categories:accessibility": ["error", {"minScore": 1.0}],
        "categories:best-practices": ["error", {"minScore": 1.0}],
        "categories:seo": ["error", {"minScore": 1.0}],
        "first-contentful-paint": ["warn", {"maxNumericValue": 1800}],
        "largest-contentful-paint": ["error", {"maxNumericValue": 2500}],
        "cumulative-layout-shift": ["error", {"maxNumericValue": 0.1}],
        "total-blocking-time": ["warn", {"maxNumericValue": 200}],
        "speed-index": ["warn", {"maxNumericValue": 3400}],
        "interactive": ["warn", {"maxNumericValue": 3800}]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI
on: [push, pull_request]
jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run build
      - run: npm run start &
      - run: npx lhci autorun
```
