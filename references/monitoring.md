# Monitoring & Analytics — Full Setup Reference

## Google Search Console (Free — Most Important)

### Setup:
1. Go to https://search.google.com/search-console
2. Add property → URL prefix → enter your domain
3. Verify via:
   - HTML tag (paste into `<head>`) — easiest with Next.js metadata
   - DNS record — most reliable

```typescript
// app/layout.tsx — GSC verification
export const metadata: Metadata = {
  verification: {
    google: process.env.GOOGLE_VERIFICATION_TOKEN,
    // Get token from GSC → Settings → Ownership verification → HTML tag
    // Token is the content="..." value (without "google-site-verification=")
  },
}
```

### Submit sitemap:
1. GSC → Sitemaps → add `/sitemap.xml` → Submit
2. Check status — should say "Success" within 24–48 hours
3. Check "Discovered URLs" count

### Weekly GSC checks:
```
Performance → Search Results:
  - Impressions: going up? ✅
  - Clicks: going up? ✅
  - Average position: going down (closer to 1)? ✅
  - CTR: > 3%? ✅ (rewrite titles/descriptions if lower)

Coverage → Errors:
  - 404 errors → fix with redirects
  - Redirect errors → fix redirect chains
  - Submitted and blocked by robots.txt → fix robots.txt
  - Duplicate without user-selected canonical → fix canonicals

Enhancements:
  - Core Web Vitals → fix any "Poor" URLs
  - Rich results → check schema errors
```

---

## Google Analytics 4 (GA4)

```bash
# Install
npm install @next/third-parties
```

```typescript
// app/layout.tsx
import { GoogleAnalytics } from '@next/third-parties/google'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
      {process.env.NEXT_PUBLIC_GA_ID && (
        <GoogleAnalytics gaId={process.env.NEXT_PUBLIC_GA_ID} />
      )}
    </html>
  )
}

// .env.local
// NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
```

### Key GA4 events to track:
```typescript
// lib/analytics.ts
export function trackEvent(eventName: string, params?: Record<string, any>) {
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('event', eventName, params)
  }
}

// Usage:
// CTA clicks:
trackEvent('cta_click', { cta_location: 'hero', cta_text: 'Get Started' })

// Contact form submission:
trackEvent('generate_lead', { method: 'contact_form' })

// Blog scroll depth:
// (GA4 tracks scroll automatically — enable in Events settings)

// Outbound link clicks:
// (GA4 tracks these automatically)
```

### GA4 Conversions to set up:
- form_submit
- contact_click (click on phone/email)
- generate_lead
- purchase (if e-commerce)

---

## Vercel Analytics + Speed Insights

```bash
npm install @vercel/analytics @vercel/speed-insights
```

```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react'
import { SpeedInsights } from '@vercel/speed-insights/next'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        {children}
        <Analytics />       {/* Page view tracking */}
        <SpeedInsights />   {/* Real-user Core Web Vitals */}
      </body>
    </html>
  )
}
```

SpeedInsights shows **real user** LCP, CLS, INP — more accurate than Lighthouse (lab data).

---

## Automated SEO Health Monitoring

### Uptime + performance monitoring:
```bash
# Simple uptime check (add to cron):
#!/bin/bash
SITE="https://yoursite.com"
STATUS=$(curl -o /dev/null -s -w "%{http_code}" "$SITE")
TTFB=$(curl -o /dev/null -s -w "%{time_starttransfer}" "$SITE")

if [ "$STATUS" != "200" ]; then
  echo "🚨 SITE DOWN: $SITE returned HTTP $STATUS"
  # Send alert (email, Slack webhook, etc.)
fi

if (( $(echo "$TTFB > 2.0" | bc -l) )); then
  echo "⚠️ SLOW TTFB: ${TTFB}s on $SITE"
fi
```

### Slack alerting for SEO issues:
```bash
# Send to Slack webhook:
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"🚨 SEO Alert: Sitemap returned 404 on yoursite.com"}' \
  $SLACK_WEBHOOK_URL
```

---

## Periodic Audit Commands

### Monthly full site audit:
```bash
#!/bin/bash
SITE="https://yoursite.com"
echo "=========================================="
echo "Monthly SEO Audit: $SITE"
echo "Date: $(date)"
echo "=========================================="

echo ""
echo "=== 1. RESPONSE CODES ==="
for path in / /about /blog /contact /sitemap.xml /robots.txt; do
  STATUS=$(curl -o /dev/null -s -w "%{http_code}" "$SITE$path")
  REDIRECT=$(curl -o /dev/null -s -w "%{redirect_url}" "$SITE$path")
  echo "  $path → HTTP $STATUS ${REDIRECT:+→ $REDIRECT}"
done

echo ""
echo "=== 2. PERFORMANCE (TTFB) ==="
curl -o /dev/null -s -w "  Homepage TTFB: %{time_starttransfer}s (target: <0.8s)\n" "$SITE"

echo ""
echo "=== 3. COMPRESSION ==="
ENCODING=$(curl -sI -H "Accept-Encoding: br,gzip" "$SITE" | grep -i content-encoding)
echo "  $ENCODING"

echo ""
echo "=== 4. SECURITY HEADERS ==="
curl -sI "$SITE" | grep -iE "(strict-transport|x-content-type|x-frame|referrer-policy|permissions-policy)" | sed 's/^/  /'

echo ""
echo "=== 5. META TAGS (Homepage) ==="
curl -s "$SITE" | grep -iE '(<title>|name="description"|canonical|og:title|og:image)' | head -10 | sed 's/^/  /'

echo ""
echo "=== 6. SITEMAP ==="
SITEMAP_STATUS=$(curl -o /dev/null -s -w "%{http_code}" "$SITE/sitemap.xml")
SITEMAP_URLS=$(curl -s "$SITE/sitemap.xml" | grep -c '<loc>')
echo "  Status: HTTP $SITEMAP_STATUS"
echo "  URLs indexed: $SITEMAP_URLS"

echo ""
echo "=== 7. ROBOTS.TXT ==="
curl -s "$SITE/robots.txt"

echo ""
echo "=== 8. JSON-LD SCHEMAS ==="
curl -s "$SITE" | grep -c 'application/ld+json'
echo " schema blocks found on homepage"

echo ""
echo "=== 9. IMAGE ALT AUDIT (Homepage) ==="
MISSING=$(curl -s "$SITE" | grep -oE '<img[^>]+>' | grep -v 'alt=' | wc -l)
echo "  Images missing alt text: $MISSING"

echo ""
echo "=========================================="
echo "Audit complete. Check GSC for deeper data."
echo "=========================================="
```

---

## Search Console API (Advanced)

```typescript
// lib/gsc.ts — fetch ranking data programmatically
// Requires Google API credentials

async function getSearchConsoleData(siteUrl: string, startDate: string, endDate: string) {
  const auth = new google.auth.GoogleAuth({
    keyFile: 'credentials.json',
    scopes: ['https://www.googleapis.com/auth/webmasters.readonly'],
  })

  const searchConsole = google.searchconsole({ version: 'v1', auth })

  const response = await searchConsole.searchanalytics.query({
    siteUrl,
    requestBody: {
      startDate,
      endDate,
      dimensions: ['query', 'page'],
      rowLimit: 100,
      dimensionFilterGroups: [
        {
          filters: [
            {
              dimension: 'page',
              expression: siteUrl,
              operator: 'contains',
            },
          ],
        },
      ],
    },
  })

  return response.data.rows || []
}
```

---

## KPI Dashboard — What to Track

| Metric | Tool | Target | Frequency |
|--------|------|--------|-----------|
| Organic clicks | GSC | +10% MoM | Weekly |
| Average position | GSC | Moving toward #1–5 | Weekly |
| Core Web Vitals (real) | Vercel Speed Insights | LCP<2.5s, CLS<0.1 | Weekly |
| Lighthouse scores | Lighthouse CI / PageSpeed | All 90+ | Each deploy |
| Indexed pages | GSC → Coverage | All public pages | Monthly |
| Crawl errors | GSC → Coverage | 0 critical errors | Monthly |
| Backlinks | Ahrefs / GSC Links | Growing | Monthly |
| Keyword rankings | GSC / Ahrefs | Top 5 for targets | Monthly |
| TTFB | curl / Vercel | < 0.8s | Monthly |
| Sitemap URLs | curl /sitemap.xml | Matches total pages | Monthly |
