# Keyword Strategy — Research & Implementation

## Keyword Research Process

### Step 1 — Seed Keywords
Start with what you know. For a Next.js agency:
- "Next.js development"
- "headless CMS"
- "Payload CMS"
- "web performance optimization"

### Step 2 — Expand with Free Tools (no API key needed)

```bash
# Google Suggest scraper (what people actually search):
curl -s "https://suggestqueries.google.com/complete/search?client=firefox&q=nextjs+seo" | python3 -c "
import sys, json
data = json.load(sys.stdin)
print('\n'.join(data[1]))
"

# Also People Also Ask / Related Searches:
# Manually check: https://www.google.com/search?q=nextjs+seo
# Use: https://alsoasked.com — free, shows PAA tree

# Check keyword volume via Ahrefs Free / Semrush Free / Ubersuggest
```

### Step 3 — Classify by Search Intent

| Intent | Signal Words | Content Type | Example |
|--------|-------------|-------------|---------|
| Informational | how, what, why, guide, tutorial | Blog post, guide | "how to add SEO to nextjs" |
| Commercial | best, vs, comparison, review, top | Comparison page | "nextjs vs remix for seo" |
| Transactional | buy, price, hire, agency, service | Service/product page | "hire nextjs developer" |
| Navigational | brand name, login, dashboard | Homepage | "payload cms login" |

**Match content type to intent — Google will not rank wrong content type for a query.**

### Step 4 — Group into Clusters

```
PILLAR: "Next.js SEO" (hub page targeting head term)
  ├── "nextjs metadata api tutorial" (blog post)
  ├── "nextjs sitemap generation" (blog post)
  ├── "nextjs schema markup" (blog post)
  ├── "nextjs lighthouse 100" (blog post)
  ├── "nextjs image optimization seo" (blog post)
  └── "nextjs core web vitals" (blog post)

PILLAR: "Payload CMS" (hub page)
  ├── "payload cms seo plugin" (blog post)
  ├── "payload cms vs contentful" (comparison)
  ├── "payload cms nextjs tutorial" (blog post)
  └── "payload cms image optimization" (blog post)
```

### Step 5 — Keyword Difficulty Tiers

**Start with Tier 3 (easy wins), then move up.**

| Tier | Difficulty | Volume | Timeline |
|------|-----------|--------|---------|
| Tier 1 — Head | 70–100 | 10K–100K/mo | 6–12 months |
| Tier 2 — Body | 30–70 | 1K–10K/mo | 3–6 months |
| Tier 3 — Long tail | 0–30 | 100–1K/mo | 1–3 months |

---

## Keyword Implementation in Content

```typescript
// For a target keyword "nextjs seo guide 2025":

// ✅ Title: keyword in first 3 words
// "Next.js SEO Guide: Rank #1 in 2025 | YourBrand"

// ✅ H1: natural inclusion
// <h1>Next.js SEO Guide: Everything You Need to Rank in 2025</h1>

// ✅ Meta: keyword + value
// "The complete Next.js SEO guide for 2025. Learn metadata, Core Web Vitals, 
//  schema markup, and sitemap generation. Includes code examples."

// ✅ First paragraph: keyword within first 100 words
// "If you're building with Next.js and want to rank on Google, 
//  this Next.js SEO guide covers everything from metadata setup to 
//  Core Web Vitals..."

// ✅ URL: /blog/nextjs-seo-guide (stop words removed)

// ✅ Image alt: "Next.js SEO configuration screenshot showing metadata API"

// ✅ At least one <h2> with keyword variation:
// <h2>Why Next.js SEO Is Different from Traditional SEO</h2>

// LSI keywords to include naturally:
// metadata API, App Router, Core Web Vitals, LCP, CLS, sitemap.xml,
// robots.txt, schema markup, Open Graph, structured data, canonical URL
```

---

## Keyword Mapping Document Template

Create one keyword map before building any site. Update as you add pages.

```markdown
| Page | Primary Keyword | Secondary Keywords | Intent | Monthly Volume | Difficulty |
|------|----------------|-------------------|--------|---------------|------------|
| / (Home) | nextjs development agency | hire nextjs developer, nextjs web dev | Navigational/Transactional | 1,200 | 45 |
| /services/nextjs | nextjs development services | custom nextjs website, nextjs developer | Transactional | 800 | 40 |
| /blog/nextjs-seo | nextjs seo guide | nextjs metadata, nextjs sitemap | Informational | 2,400 | 35 |
| /blog/payload-cms | payload cms tutorial | payload cms nextjs, headless cms | Informational | 900 | 28 |
```

---

## Competitor Keyword Gap Analysis (Free)

```bash
# 1. Find top competitors for your keyword
# Search your target keyword → identify who's ranking in top 5

# 2. Audit their content for keyword patterns
curl -s https://competitor.com/blog/their-top-post | \
  python3 -c "
import sys, re
from collections import Counter
text = sys.stdin.read().lower()
# Strip HTML
text = re.sub('<[^>]+>', ' ', text)
# Count words (crude keyword density)
words = re.findall(r'[a-z]{4,}', text)
counter = Counter(words)
# Remove stop words
stopwords = {'this','that','with','from','have','been','they','will','your','what','when','then','also','into','more','than','some'}
for word, count in counter.most_common(30):
    if word not in stopwords:
        print(f'{word}: {count}')
"

# 3. Check their title tags and meta
curl -s https://competitor.com | grep -iE '(<title>|name="description")'

# 4. Check their heading structure
curl -s https://competitor.com/their-top-post | grep -oE '<h[1-6][^>]*>[^<]+</h[1-6]>'
```

---

## Local SEO Keywords (if applicable)

```
Pattern: [service] + [location]
Examples:
- "next.js developer nairobi"
- "web design agency kenya"
- "nextjs development company nairobi"
- "hire web developer kenya"

Pattern: [service] + "near me"
- "web development agency near me" (lower value — use location instead)

Target: City-level first, then expand to country/region
```

---

## Keyword Tracking

```bash
# Check current ranking for a keyword (crude — use real tools for accurate data)
curl -s "https://www.google.com/search?q=nextjs+seo+guide" \
  -H "User-Agent: Mozilla/5.0" | \
  grep -oE 'https?://[^"&]+' | grep -v google | head -10

# Better: Use Google Search Console (free, accurate)
# → Search Results → Queries → filter by page or keyword
# Shows impressions, clicks, average position, CTR

# Paid tools (worth it at scale):
# Ahrefs: best backlink data
# Semrush: best keyword tracking
# Screaming Frog: best technical audit
```
