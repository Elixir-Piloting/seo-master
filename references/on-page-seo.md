# On-Page SEO — Rules & Patterns

## Title Tag Formula

**Format:** `Primary Keyword — Secondary Keyword | Brand Name`
- 50–60 characters (Google truncates at ~580px width ≈ 60 chars)
- Front-load the primary keyword (first 3 words matter most)
- Use `—` (em dash) or `|` (pipe) as separators — NOT hyphens
- Never duplicate titles across pages
- Include numbers/years for freshness: "10 Best…" or "Guide (2025)"

### Examples by page type:
```
Homepage:    "Next.js Development Agency | YourBrand"
Blog post:   "Next.js SEO: Complete 2025 Guide | YourBrand"
Product:     "Enterprise CMS — Headless & Fast | YourBrand"
Service:     "Custom Web Development Services | YourBrand"
Location:    "Web Design in Nairobi, Kenya | YourBrand"
FAQ:         "Frequently Asked Questions | YourBrand"
About:       "About Us — Our Story & Mission | YourBrand"
```

---

## Meta Description Formula

**Format:** Action verb + primary keyword + value prop + CTA
- 150–160 characters
- Include primary keyword naturally (not stuffed)
- End with a soft CTA: "Learn more", "Get started", "See pricing"
- No duplicate descriptions — every page unique
- Active voice, present tense

### Examples:
```
Blog post:
"Master Next.js SEO with our complete 2025 guide. Learn metadata, 
sitemaps, schema, and Core Web Vitals fixes. Rank #1 on Google today."

Service page:
"Custom Next.js websites built for speed and rankings. We deliver 
Lighthouse 100 scores and top-5 Google results. Get a free audit."

Product:
"The headless CMS that ranks. Payload CMS with built-in SEO fields,
image optimization, and one-click sitemaps. Try free for 14 days."
```

---

## Heading Structure

### Rules:
- Exactly **one `<h1>`** per page — contains primary keyword
- `<h2>` for major sections — include secondary keywords
- `<h3>` for subsections — include LSI (related) keywords
- `<h4>`–`<h6>` sparingly
- Never skip levels: don't go h1 → h3
- Headings should work as a standalone outline

### Template (blog post):
```markdown
<h1>Next.js SEO: The Complete 2025 Guide</h1>
  <h2>Why SEO Matters for Next.js Apps</h2>
  <h2>Setting Up the Metadata API</h2>
    <h3>Root Layout Metadata</h3>
    <h3>Dynamic Page Metadata</h3>
    <h3>Open Graph Tags</h3>
  <h2>Generating Your Sitemap</h2>
  <h2>Core Web Vitals Optimization</h2>
    <h3>Fixing LCP</h3>
    <h3>Eliminating CLS</h3>
    <h3>Improving INP</h3>
  <h2>Schema Markup for Rich Results</h2>
  <h2>Frequently Asked Questions</h2>
```

---

## URL Structure

### Rules:
- Lowercase only: `/blog/nextjs-seo-guide`
- Hyphens only (not underscores, not spaces): `seo-guide` not `seo_guide`
- Descriptive: `/products/headless-cms` not `/products/item-42`
- Short: ≤ 5 meaningful words (exclude stop words)
- No trailing slash inconsistency — pick one, redirect the other
- No date in URL (ages poorly): `/blog/seo` not `/blog/2025/01/seo`
- No query parameters for public content: use path instead

### URL hierarchy:
```
yoursite.com/                         → Homepage
yoursite.com/blog/                    → Blog index
yoursite.com/blog/nextjs-seo-guide    → Blog post
yoursite.com/products/                → Products index
yoursite.com/products/payload-cms     → Product page
yoursite.com/services/web-development → Service page
yoursite.com/about/                   → About
yoursite.com/contact/                 → Contact
```

### Redirect common variants:
```typescript
// next.config.ts — 301 redirects for URL hygiene
async redirects() {
  return [
    // Remove trailing slash (or add — pick one)
    { source: '/about/', destination: '/about', permanent: true },
    // Old URL patterns
    { source: '/blog/:slug/', destination: '/blog/:slug', permanent: true },
    // Category removal
    { source: '/category/:cat/:slug', destination: '/blog/:slug', permanent: true },
  ]
},
```

---

## Content Quality Rules

### Word count targets (not a hard rule — relevance > length):
- Blog posts: 1,200–2,500 words
- Pillar/guide pages: 2,500–5,000+ words
- Product pages: 400–800 words
- Service pages: 600–1,200 words
- Homepage: 300–600 words

### E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness):
```tsx
// Show author expertise on every post
<div className="author-bio" itemScope itemType="https://schema.org/Person">
  <Image src={author.avatar} alt={author.name} width={64} height={64} />
  <div>
    <span itemProp="name">{author.name}</span>
    <span itemProp="jobTitle">{author.jobTitle}</span>
    <p itemProp="description">{author.bio}</p>
    <a href={author.linkedIn} itemProp="sameAs">LinkedIn</a>
  </div>
</div>

// Show published + updated dates (signals freshness to Google)
<time dateTime={post.publishedAt}>Published: {formatDate(post.publishedAt)}</time>
<time dateTime={post.updatedAt}>Updated: {formatDate(post.updatedAt)}</time>
```

### Keyword density:
- Primary keyword: 1–2% density (not stuffed)
- Use keyword in: title, h1, first 100 words, meta description, at least one h2, image alt
- LSI keywords throughout: related terms Google expects to see
- Never repeat keyword more than once per heading

---

## Internal Linking Strategy

```typescript
// Systematic internal linking — do this for every piece of content

// 1. Homepage → all main sections (hub page)
// 2. Blog index → latest + featured posts
// 3. Blog posts → related posts (same category/tag)
// 4. Blog posts → relevant product/service pages
// 5. Product pages → case studies / blog posts about the product
// 6. Service pages → relevant blog content, contact page

// Auto-related posts component:
async function RelatedPosts({ currentSlug, tags }: { currentSlug: string, tags: string[] }) {
  const related = await fetch(
    `${PAYLOAD_URL}/api/posts?where[tags.label][in]=${tags.join(',')}&where[slug][not_equals]=${currentSlug}&limit=3&depth=1`
  ).then(r => r.json())

  return (
    <section aria-label="Related articles">
      <h2>Related Articles</h2>
      {related.docs.map((post: any) => (
        <a key={post.id} href={`/blog/${post.slug}`}>
          {post.title}
        </a>
      ))}
    </section>
  )
}
```

### Anchor text rules:
```tsx
// ❌ WRONG — non-descriptive
<a href="/blog/seo-guide">click here</a>
<a href="/blog/seo-guide">read more</a>
<a href="/blog/seo-guide">this article</a>

// ✅ RIGHT — keyword-rich, descriptive
<a href="/blog/seo-guide">our complete Next.js SEO guide</a>
<a href="/services/web-development">Next.js web development services</a>
<a href="/products/payload-cms">Payload CMS features and pricing</a>
```

---

## Image SEO

### Alt text rules:
```tsx
// Informative images — describe what's in the image, naturally include keyword
<Image alt="Next.js sitemap configuration in VS Code" ... />
<Image alt="Lighthouse 100 score for performance, accessibility, best practices, SEO" ... />
<Image alt="Payload CMS SEO plugin settings showing meta title and description fields" ... />

// Decorative images — empty alt (not null, not missing — empty string)
<Image alt="" role="presentation" ... />

// Don't:
// ❌ alt="image123.jpg"
// ❌ alt="photo"
// ❌ alt="SEO SEO SEO keyword keyword"
// ❌ Missing alt attribute entirely

// File names also matter:
// ❌ IMG_4892.jpg
// ✅ nextjs-seo-guide-screenshot.jpg
// ✅ payload-cms-admin-panel.webp
```

### Image dimensions in Payload CMS:
```typescript
// Always store and expose image dimensions from Payload
// So Next.js <Image> can render without layout shift

// In your Payload Media collection, dimensions are stored automatically.
// Fetch and use them:
const image = post.featuredImage
return (
  <Image
    src={image.url}
    alt={image.alt}
    width={image.width}    // From Payload
    height={image.height}  // From Payload
    sizes="(max-width: 768px) 100vw, 800px"
  />
)
```

---

## Canonical URLs — Full Pattern

```typescript
// Rule: Every public page must have a canonical URL
// Rule: Paginated pages canonicalize to themselves (not page 1)
// Rule: Sort/filter variants canonicalize to base URL

// Correct canonical patterns:
// Homepage:          https://yoursite.com
// Blog post:         https://yoursite.com/blog/post-slug
// Blog page 2:       https://yoursite.com/blog/page/2  (NOT /blog/)
// Product filtered:  https://yoursite.com/products/  (NOT /products/?color=red)
// www variant:       Redirect www → non-www (301), set canonical to non-www

// In Next.js metadata:
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const canonicalUrl = `${process.env.NEXT_PUBLIC_SITE_URL}/blog/${params.slug}`
  return {
    alternates: {
      canonical: canonicalUrl,
    },
  }
}

// Verify canonical is rendered:
// curl -s https://yoursite.com/blog/my-post | grep canonical
// Expected: <link rel="canonical" href="https://yoursite.com/blog/my-post"/>
```

---

## Hreflang for Multi-language Sites

```typescript
// For international/multilingual sites, add hreflang to every page

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  return {
    alternates: {
      canonical: `https://yoursite.com/en/${params.slug}`,
      languages: {
        'en': `https://yoursite.com/en/${params.slug}`,
        'fr': `https://yoursite.com/fr/${params.slug}`,
        'es': `https://yoursite.com/es/${params.slug}`,
        'x-default': `https://yoursite.com/en/${params.slug}`,
      },
    },
  }
}

// Verify:
// curl -s https://yoursite.com/en/my-page | grep hreflang
// Expected: <link rel="alternate" hrefLang="fr" href="https://yoursite.com/fr/my-page"/>
```

---

## Pre-deploy On-Page Checklist

```bash
SITE="https://yoursite.com"
PAGE="$SITE/blog/example-post"

# 1. Title tag exists and < 60 chars
TITLE=$(curl -s "$PAGE" | grep -oP '(?<=<title>)[^<]+')
echo "Title ($((${#TITLE})) chars): $TITLE"

# 2. Meta description exists and < 160 chars
DESC=$(curl -s "$PAGE" | grep -oP '(?<=name="description" content=")[^"]+')
echo "Description ($((${#DESC})) chars): $DESC"

# 3. H1 exists (exactly one)
H1_COUNT=$(curl -s "$PAGE" | grep -cE '<h1[^>]*>')
echo "H1 count (should be 1): $H1_COUNT"

# 4. Canonical is set
curl -s "$PAGE" | grep 'canonical'

# 5. OG tags complete
curl -s "$PAGE" | grep -E 'og:(title|description|image|url)'

# 6. No noindex on public pages
NOINDEX=$(curl -s "$PAGE" | grep -i noindex)
[ -z "$NOINDEX" ] && echo "✅ No noindex" || echo "❌ NOINDEX FOUND: $NOINDEX"

# 7. JSON-LD present
curl -s "$PAGE" | grep 'application/ld+json' | wc -l

# 8. Images have alt text
MISSING_ALT=$(curl -s "$PAGE" | grep -oE '<img[^>]+>' | grep -v 'alt=')
[ -z "$MISSING_ALT" ] && echo "✅ All images have alt" || echo "❌ Missing alt: $MISSING_ALT"
```
