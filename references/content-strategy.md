# Content Strategy — Pillar Pages, Clusters & Calendar

## Topic Cluster Model

```
                    [PILLAR PAGE]
                   "Next.js SEO Hub"
                  (targets head term)
                        |
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
  [Cluster Post]  [Cluster Post]  [Cluster Post]
  "Metadata API"  "Sitemap Setup"  "Schema Markup"
     ↕ links          ↕ links         ↕ links
        └───────────────┴───────────────┘
              (all link back to pillar)
```

### Rules:
1. Pillar page = comprehensive, long-form, targets head keyword
2. Cluster pages = specific, in-depth, target long-tail keywords
3. Every cluster post links to the pillar page
4. Pillar page links to every cluster post
5. Cluster posts interlink with each other when relevant

---

## Pillar Page Template

```markdown
# [Primary Keyword]: The Complete Guide

## What Is [Topic]? (Definition + why it matters)
## [Aspect 1]: [Cluster page 1 topic] — [link to cluster]
## [Aspect 2]: [Cluster page 2 topic] — [link to cluster]
## [Aspect 3]: [Cluster page 3 topic] — [link to cluster]
## [Aspect 4]: [Cluster page 4 topic] — [link to cluster]
## Common Mistakes to Avoid
## Frequently Asked Questions
## Next Steps / CTA

Word count: 2,500–5,000+
Schema: Article + FAQPage + BreadcrumbList
Update frequency: Every 6 months (mark updatedAt)
```

---

## Content Calendar (for a dev/tech site)

| Week | Content Type | Target Keyword | Notes |
|------|-------------|---------------|-------|
| 1 | Pillar page | "nextjs seo" | Hub for all SEO content |
| 2 | Cluster post | "nextjs metadata api tutorial" | Links to pillar |
| 3 | Cluster post | "nextjs sitemap generation" | Links to pillar |
| 4 | Cluster post | "nextjs schema markup json-ld" | Links to pillar |
| 5 | Case study | "we increased traffic by 300%" | Links to service pages |
| 6 | Comparison | "nextjs vs gatsby seo" | Commercial intent |
| 7 | Cluster post | "nextjs core web vitals fix" | Links to pillar |
| 8 | FAQ page | "nextjs seo faq" | FAQPage schema |

**Frequency:** 1–2 high-quality posts per week beats 5 thin posts.

---

## Content Quality Checklist

Before publishing any piece of content:

```
[ ] Primary keyword in title (first 3 words preferred)
[ ] Primary keyword in H1
[ ] Primary keyword in first 100 words
[ ] Primary keyword in meta description
[ ] Primary keyword in at least one H2
[ ] Primary keyword in at least one image alt
[ ] URL contains primary keyword (stop words removed)
[ ] Word count appropriate for intent and competition
[ ] At least 3 internal links (to relevant pages)
[ ] At least 1 outbound link (to authoritative external source)
[ ] All images have descriptive alt text
[ ] Author bio with credentials shown
[ ] Published date + updated date visible
[ ] Schema markup (Article + Breadcrumb minimum)
[ ] Canonical URL set
[ ] OG image (1200×630) uploaded
[ ] Fact-checked and accurate
[ ] Readable at 8th-grade level (use Hemingway App)
[ ] Mobile readable (short paragraphs, headers, bullet points)
[ ] CTA at end (related content, newsletter, service/product)
```

---

## Content Refresh Strategy

Old content decays. Google favors fresh, updated content.

```typescript
// In Payload CMS — track when content was last updated for SEO
// Add to posts collection:
{
  name: 'lastUpdatedReason',
  type: 'textarea',
  label: 'Reason for Update (internal note)',
  admin: { description: 'What changed? e.g. "Updated stats for 2025, added new section on INP"' },
}

// Identify content to refresh:
// 1. Posts > 6 months old with declining traffic (Google Search Console)
// 2. Posts ranking #4–#15 (close — worth pushing over the edge)
// 3. Posts with outdated info (check for date references)
// 4. Posts with high impressions but low CTR (rewrite title/description)
```

### Refresh checklist:
```
[ ] Update publish date (set updatedAt to today)
[ ] Update any year references
[ ] Add new sections if topic has evolved
[ ] Update statistics and data points
[ ] Add new internal links to newer content
[ ] Improve title and meta description (check GSC for CTR)
[ ] Add new images / update screenshots
[ ] Add FAQ section (if missing)
[ ] Trigger revalidation in Next.js (webhook from Payload)
```

---

## Structured Content Types for SEO

```typescript
// High-value content types for search rankings:

// 1. How-To Guides — triggers rich result in Google
// Use HowTo schema + numbered steps

// 2. FAQ Pages — triggers FAQ rich result  
// Use FAQPage schema + Q&A format

// 3. List Posts — "10 Best X" — high CTR in search
// Include ListItem schema

// 4. Comparison Pages — "X vs Y" — high commercial intent
// Table format + clear winner section

// 5. Case Studies — builds authority
// Show before/after metrics, named client (if allowed)

// 6. Tutorials with Code — high engagement, backlink magnet
// Use SyntaxHighlighter, copyable code blocks

// 7. Tools/Calculators — infinite traffic, zero effort to maintain
// e.g. "Lighthouse Score Calculator", "CLS Calculator"
// Use SoftwareApplication schema
```

---

## Content Distribution (after publish)

```
Day 1: Publish + share on Twitter/X, LinkedIn
Day 1: Submit URL to Google Search Console → URL Inspection → Request indexing
Day 2: Share in relevant Slack communities / Discord servers
Day 3: Send to email newsletter (if you have one)
Week 1: Reach out to 3-5 sites linking to similar content — ask for link
Week 2: Repurpose into Twitter/X thread
Month 1: Check Google Search Console — is it indexed? Any impressions?
Month 3: Check rankings — optimize if #4–15
Month 6: Refresh if traffic declining
```

```bash
# Submit URL to Google for fast indexing:
# Google Search Console → URL Inspection → paste URL → Request indexing

# Verify indexing:
curl "https://www.google.com/search?q=site:yoursite.com/blog/your-post-slug"
# If result appears → indexed ✅
# If no result → not indexed yet (wait 1-7 days after requesting)
```
