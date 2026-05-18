# 🚀 SEO Master — Claude Skill

A production-grade SEO skill for Claude that audits, fixes, and optimizes websites to achieve:

- ✅ **Lighthouse 100/100/100/100** — Performance, Accessibility, Best Practices, SEO
- ✅ **85+ SEO tool scores** — Ahrefs, Semrush, Screaming Frog, Moz
- ✅ **Zero critical warnings or errors**
- ✅ **Top-5 Google rankings** for target keywords
- ✅ **Perfect Core Web Vitals** — LCP ≤ 2.5s, CLS ≤ 0.1, INP ≤ 200ms

Built specifically for **Next.js 16** + **Payload CMS** stacks, with curl-based audit commands, automated ISR revalidation, JSON-LD schema generation, and Lighthouse CI integration.

---

## Install

```bash
npx skill install YOUR_GITHUB_USERNAME/seo-master
```

Or in Claude → Settings → Skills → Install from GitHub → paste the repo URL.

---

## What's Inside

```
seo-master/
├── SKILL.md                        # Main skill — triggers on any SEO request
└── references/
    ├── nextjs-seo.md               # Full Next.js 16 App Router SEO patterns
    ├── payload-seo.md              # Payload CMS collections, hooks, SEO plugin
    ├── performance.md              # Lighthouse 100 configs, bundle analysis
    ├── schema-markup.md            # All JSON-LD schema types with full examples
    ├── core-web-vitals.md          # LCP, CLS, INP, FCP deep fixes with code
    ├── on-page-seo.md              # Title/meta rules, headings, internal linking
    ├── keyword-strategy.md         # Research process, intent mapping, clusters
    ├── content-strategy.md         # Pillar pages, topic clusters, content calendar
    └── monitoring.md               # GSC, GA4, Vercel Analytics, audit scripts
```

---

## Features

### 🔍 Live Site Auditing
Run 15+ `curl` checks against any URL — TTFB, headers, canonical, schema, compression, security headers, noindex detection, image alt text, OG tags, and more.

### ⚡ Next.js 16 SEO Foundation
- Full Metadata API setup (root layout + dynamic per-page)
- Auto-generated `sitemap.ts` pulling from Payload CMS
- `robots.ts` with fine-grained bot rules
- `next/font` with zero CLS font loading
- `next/image` patterns for LCP optimization

### 🧩 Payload CMS Integration
- Official `@payloadcms/plugin-seo` config
- SEO-ready Media collection with auto image resizing
- Reusable slug field with auto-sanitization
- On-publish revalidation webhook to Next.js

### 📊 Schema Markup (JSON-LD)
Organization, WebSite, Article, FAQ, Product, LocalBusiness, HowTo, BreadcrumbList — with validation commands.

### 🏎️ Performance & Core Web Vitals
- Bundle splitting, dynamic imports, code splitting strategy
- LCP, CLS, INP, FCP, TTFB root-cause diagnosis + fixes
- Lighthouse CI config for GitHub Actions
- Web Worker patterns for heavy computation

### 📈 Keyword & Content Strategy
- Search intent classification
- Topic cluster / pillar page model
- Content calendar templates
- Competitor gap analysis with `curl`

### 📡 Monitoring
- Google Search Console setup + weekly checks
- GA4 + Vercel Analytics integration
- Automated monthly audit bash script
- KPI dashboard with targets

---

## Usage

Once installed, just talk to Claude naturally:

> *"Audit my site at https://mysite.com"*  
> *"Add SEO to my Next.js app"*  
> *"My Lighthouse score is 67 — fix it"*  
> *"Set up the Payload CMS SEO plugin"*  
> *"Generate schema markup for my blog posts"*  
> *"My LCP is 4.2 seconds, what's wrong?"*  
> *"Create a sitemap.ts for my Next.js 16 app"*

The skill auto-triggers on any SEO, performance, Lighthouse, or ranking question.

---

## Stack Compatibility

| Technology | Support |
|-----------|---------|
| Next.js 14+ (App Router) | ✅ Full |
| Next.js 16 | ✅ Full |
| Payload CMS 2.x / 3.x | ✅ Full |
| Vercel | ✅ Full |
| Any Node.js host | ✅ Partial |
| WordPress / other CMS | ⚠️ Concepts apply, not code |

---

## License

MIT — use freely, contributions welcome.
