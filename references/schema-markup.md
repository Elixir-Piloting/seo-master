# Schema Markup (JSON-LD) — Complete Reference

## Always Include on Every Site

### 1. Organization Schema (site-wide, in layout.tsx)

```typescript
// lib/schema.ts
export function getOrganizationSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Your Company Name',
    url: 'https://yoursite.com',
    logo: {
      '@type': 'ImageObject',
      url: 'https://yoursite.com/logo.png',
      width: 512,
      height: 512,
    },
    description: 'Your company description.',
    foundingDate: '2020',
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+1-555-000-0000',
      contactType: 'customer service',
      availableLanguage: ['English'],
    },
    address: {
      '@type': 'PostalAddress',
      streetAddress: '123 Main St',
      addressLocality: 'City',
      addressRegion: 'State',
      postalCode: '00000',
      addressCountry: 'US',
    },
    sameAs: [
      'https://twitter.com/yourhandle',
      'https://linkedin.com/company/yourcompany',
      'https://github.com/yourcompany',
      'https://www.facebook.com/yourcompany',
    ],
  }
}

### 2. WebSite Schema with SiteLinks Searchbox
export function getWebsiteSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'WebSite',
    name: 'Your Brand',
    url: 'https://yoursite.com',
    description: 'Site description',
    potentialAction: {
      '@type': 'SearchAction',
      target: {
        '@type': 'EntryPoint',
        urlTemplate: 'https://yoursite.com/search?q={search_term_string}',
      },
      'query-input': 'required name=search_term_string',
    },
  }
}
```

---

## Page-Specific Schemas

### Article / Blog Post

```typescript
export function getArticleSchema(post: any) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Article',
    '@id': `https://yoursite.com/blog/${post.slug}#article`,
    headline: post.title,
    description: post.excerpt,
    image: {
      '@type': 'ImageObject',
      url: post.featuredImage?.url,
      width: 1200,
      height: 630,
    },
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      '@type': 'Person',
      name: post.author?.name,
      url: `https://yoursite.com/authors/${post.author?.slug}`,
      image: post.author?.avatar?.url,
      jobTitle: post.author?.jobTitle,
      sameAs: post.author?.socialLinks || [],
    },
    publisher: getOrganizationSchema(),
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': `https://yoursite.com/blog/${post.slug}`,
    },
    wordCount: post.wordCount,
    keywords: post.tags?.map((t: any) => t.label).join(', '),
    articleSection: post.categories?.map((c: any) => c.name).join(', '),
    inLanguage: 'en-US',
    isAccessibleForFree: true,
  }
}
```

### FAQ Page

```typescript
export function getFAQSchema(faqs: Array<{ question: string; answer: string }>) {
  return {
    '@context': 'https://schema.org',
    '@type': 'FAQPage',
    mainEntity: faqs.map(faq => ({
      '@type': 'Question',
      name: faq.question,
      acceptedAnswer: {
        '@type': 'Answer',
        text: faq.answer,
      },
    })),
  }
}
```

### Product

```typescript
export function getProductSchema(product: any) {
  return {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.images?.map((img: any) => img.url),
    sku: product.sku,
    brand: {
      '@type': 'Brand',
      name: 'Your Brand',
    },
    offers: {
      '@type': 'Offer',
      url: `https://yoursite.com/products/${product.slug}`,
      priceCurrency: 'USD',
      price: product.price,
      priceValidUntil: new Date(Date.now() + 86400000 * 30).toISOString().split('T')[0],
      availability: product.inStock
        ? 'https://schema.org/InStock'
        : 'https://schema.org/OutOfStock',
      seller: {
        '@type': 'Organization',
        name: 'Your Company',
      },
      shippingDetails: {
        '@type': 'OfferShippingDetails',
        shippingRate: {
          '@type': 'MonetaryAmount',
          value: '0',
          currency: 'USD',
        },
        deliveryTime: {
          '@type': 'ShippingDeliveryTime',
          businessDays: { '@type': 'QuantitativeValue', minValue: 3, maxValue: 7 },
        },
      },
    },
    aggregateRating: product.reviews?.count > 0 ? {
      '@type': 'AggregateRating',
      ratingValue: product.reviews.average,
      reviewCount: product.reviews.count,
      bestRating: 5,
      worstRating: 1,
    } : undefined,
  }
}
```

### Local Business

```typescript
export function getLocalBusinessSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'LocalBusiness', // Or: Restaurant, MedicalClinic, LawFirm, etc.
    name: 'Business Name',
    image: 'https://yoursite.com/storefront.jpg',
    '@id': 'https://yoursite.com/#business',
    url: 'https://yoursite.com',
    telephone: '+1-555-000-0000',
    priceRange: '$$',
    address: {
      '@type': 'PostalAddress',
      streetAddress: '123 Main St',
      addressLocality: 'City',
      addressRegion: 'CA',
      postalCode: '94102',
      addressCountry: 'US',
    },
    geo: {
      '@type': 'GeoCoordinates',
      latitude: 37.7749,
      longitude: -122.4194,
    },
    openingHoursSpecification: [
      {
        '@type': 'OpeningHoursSpecification',
        dayOfWeek: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday'],
        opens: '09:00',
        closes: '17:00',
      },
      {
        '@type': 'OpeningHoursSpecification',
        dayOfWeek: ['Saturday'],
        opens: '10:00',
        closes: '14:00',
      },
    ],
    sameAs: ['https://www.google.com/maps/place/...', 'https://yelp.com/biz/...'],
    aggregateRating: {
      '@type': 'AggregateRating',
      ratingValue: '4.8',
      reviewCount: '124',
    },
  }
}
```

### SaaS / Software

```typescript
export function getSoftwareSchema() {
  return {
    '@context': 'https://schema.org',
    '@type': 'SoftwareApplication',
    name: 'Your App',
    applicationCategory: 'BusinessApplication',
    operatingSystem: 'Web, iOS, Android',
    url: 'https://yoursite.com',
    description: 'What your software does.',
    featureList: 'Feature 1, Feature 2, Feature 3',
    screenshot: 'https://yoursite.com/screenshot.png',
    offers: {
      '@type': 'Offer',
      price: '29',
      priceCurrency: 'USD',
      priceSpecification: {
        '@type': 'UnitPriceSpecification',
        price: '29',
        priceCurrency: 'USD',
        billingDuration: 'P1M',
      },
    },
    aggregateRating: {
      '@type': 'AggregateRating',
      ratingValue: '4.9',
      reviewCount: '450',
    },
  }
}
```

### How-To

```typescript
export function getHowToSchema(steps: Array<{ name: string; text: string; image?: string }>) {
  return {
    '@context': 'https://schema.org',
    '@type': 'HowTo',
    name: 'How to Do X',
    description: 'Step-by-step guide for doing X',
    totalTime: 'PT30M',
    estimatedCost: { '@type': 'MonetaryAmount', currency: 'USD', value: '0' },
    step: steps.map((step, i) => ({
      '@type': 'HowToStep',
      position: i + 1,
      name: step.name,
      text: step.text,
      image: step.image,
    })),
  }
}
```

### BreadcrumbList

```typescript
export function getBreadcrumbSchema(crumbs: Array<{ name: string; url: string }>) {
  return {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: crumbs.map((crumb, i) => ({
      '@type': 'ListItem',
      position: i + 1,
      name: crumb.name,
      item: crumb.url,
    })),
  }
}
```

---

## SchemaOrg Component + Multiple Schemas

```tsx
// components/SchemaOrg.tsx
export function SchemaOrg({ schema }: { schema: object | object[] }) {
  const schemaArray = Array.isArray(schema) ? schema : [schema]
  return (
    <>
      {schemaArray.map((s, i) => (
        <script
          key={i}
          type="application/ld+json"
          dangerouslySetInnerHTML={{ __html: JSON.stringify(s) }}
        />
      ))}
    </>
  )
}

// Usage in app/layout.tsx:
<SchemaOrg schema={[getOrganizationSchema(), getWebsiteSchema()]} />

// Usage in blog post page:
<SchemaOrg schema={[
  getArticleSchema(post),
  getBreadcrumbSchema([
    { name: 'Home', url: 'https://yoursite.com' },
    { name: 'Blog', url: 'https://yoursite.com/blog' },
    { name: post.title, url: `https://yoursite.com/blog/${post.slug}` },
  ]),
]} />
```

---

## Schema Validation

```bash
# Validate via Google's Rich Results Test API:
curl "https://searchconsole.googleapis.com/v1/urlTestingTools/richResults:run?url=https://yoursite.com/blog/my-post" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Or extract and validate locally:
curl -s https://yoursite.com | python3 -c "
import sys, re, json
html = sys.stdin.read()
schemas = re.findall(r'<script type=\"application/ld\+json\">(.*?)</script>', html, re.DOTALL)
print(f'Found {len(schemas)} JSON-LD blocks:')
for i, s in enumerate(schemas, 1):
    try:
        parsed = json.loads(s)
        print(f'  [{i}] ✅ {parsed.get(\"@type\", \"Unknown\")}')
    except json.JSONDecodeError as e:
        print(f'  [{i}] ❌ Invalid JSON: {e}')
"
```

---

## Schema Quick-Check by Page Type

| Page | Required Schemas |
|------|-----------------|
| Homepage | Organization, WebSite, BreadcrumbList |
| Blog listing | WebPage, BreadcrumbList |
| Blog post | Article (or BlogPosting), BreadcrumbList |
| Product page | Product (with Offer, AggregateRating), BreadcrumbList |
| FAQ page | FAQPage, BreadcrumbList |
| About page | Organization, Person (team), BreadcrumbList |
| Contact page | LocalBusiness (if physical), BreadcrumbList |
| How-to guide | HowTo, Article, BreadcrumbList |
| Event page | Event, BreadcrumbList |
| Recipe page | Recipe, BreadcrumbList |
