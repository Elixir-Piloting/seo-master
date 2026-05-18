# Next.js 16 SEO — Complete Reference

## App Router Architecture for SEO

### Metadata Inheritance Chain
```
app/layout.tsx          → site-wide defaults
app/(marketing)/layout.tsx → section overrides  
app/blog/layout.tsx     → blog section overrides
app/blog/[slug]/page.tsx → page-specific (highest priority)
```

Each level MERGES with parent (not replaces). Only override what changes.

---

## Full Metadata Config (layout.tsx)

```typescript
import type { Metadata, Viewport } from 'next'

// Separate viewport export — required in Next.js 14+
export const viewport: Viewport = {
  width: 'device-width',
  initialScale: 1,
  maximumScale: 5,
  themeColor: [
    { media: '(prefers-color-scheme: light)', color: '#ffffff' },
    { media: '(prefers-color-scheme: dark)', color: '#0a0a0a' },
  ],
}

export const metadata: Metadata = {
  metadataBase: new URL(process.env.NEXT_PUBLIC_SITE_URL || 'https://yoursite.com'),

  title: {
    default: 'Your Brand — Primary Keyword',
    template: '%s | Your Brand',
    absolute: undefined, // Use this on pages where you DON'T want the template
  },

  description: 'Compelling 150-160 char description with your primary keyword and value proposition. Make users want to click.',

  applicationName: 'Your App Name',
  referrer: 'origin-when-cross-origin',
  keywords: ['keyword1', 'keyword2', 'keyword3'],
  authors: [{ name: 'Author', url: 'https://yoursite.com/about' }],
  creator: 'Your Company',
  publisher: 'Your Company',
  formatDetection: {
    email: false,
    address: false,
    telephone: false,
  },

  // Canonical URL
  alternates: {
    canonical: '/',  // Relative — metadataBase resolves it
    languages: {
      'en-US': '/en-us',
      'fr-FR': '/fr-fr',
    },
  },

  // Open Graph
  openGraph: {
    title: 'Your Brand — Primary Keyword',
    description: 'Compelling OG description up to 200 chars.',
    url: '/',
    siteName: 'Your Brand',
    locale: 'en_US',
    type: 'website',
    images: [
      {
        url: '/og-image.jpg',    // 1200×630px, < 8MB
        width: 1200,
        height: 630,
        alt: 'Descriptive alt text for OG image',
        type: 'image/jpeg',
      },
    ],
  },

  // Twitter / X
  twitter: {
    card: 'summary_large_image',
    title: 'Your Brand — Primary Keyword',
    description: 'Twitter description, up to 200 chars.',
    site: '@yourtwitterhandle',
    creator: '@yourtwitterhandle',
    images: {
      url: '/og-image.jpg',
      alt: 'Descriptive alt text',
    },
  },

  // Robots
  robots: {
    index: true,
    follow: true,
    nocache: false,
    googleBot: {
      index: true,
      follow: true,
      noimageindex: false,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },

  // Icons
  icons: {
    icon: [
      { url: '/favicon.ico', sizes: 'any' },
      { url: '/icon.svg', type: 'image/svg+xml' },
      { url: '/favicon-16x16.png', sizes: '16x16', type: 'image/png' },
      { url: '/favicon-32x32.png', sizes: '32x32', type: 'image/png' },
    ],
    apple: [
      { url: '/apple-touch-icon.png', sizes: '180x180', type: 'image/png' },
    ],
    other: [
      { rel: 'mask-icon', url: '/safari-pinned-tab.svg', color: '#000000' },
    ],
  },

  // PWA manifest
  manifest: '/site.webmanifest',

  // Verification tokens
  verification: {
    google: process.env.GOOGLE_VERIFICATION_TOKEN,
    yandex: process.env.YANDEX_VERIFICATION_TOKEN,
    bing: process.env.BING_VERIFICATION_TOKEN,
  },

  // Category
  category: 'technology',
}
```

---

## generateMetadata — Dynamic Pages

```typescript
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next'
import { notFound } from 'next/navigation'

interface Props {
  params: { slug: string }
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug)
  if (!post) return { title: 'Not Found' }

  const canonicalUrl = `${process.env.NEXT_PUBLIC_SITE_URL}/blog/${params.slug}`
  const ogImageUrl = post.seo?.ogImage?.url || post.featuredImage?.url || '/og-default.jpg'

  return {
    title: post.seo?.title || post.title,
    description: post.seo?.description || post.excerpt,
    
    alternates: {
      canonical: canonicalUrl,
    },

    openGraph: {
      title: post.seo?.ogTitle || post.title,
      description: post.seo?.ogDescription || post.excerpt,
      url: canonicalUrl,
      type: 'article',
      publishedTime: post.publishedAt,
      modifiedTime: post.updatedAt,
      authors: post.author ? [post.author.name] : [],
      tags: post.tags?.map((t: any) => t.label) || [],
      images: [
        {
          url: ogImageUrl,
          width: 1200,
          height: 630,
          alt: post.seo?.ogImageAlt || post.title,
        },
      ],
    },

    twitter: {
      card: 'summary_large_image',
      title: post.seo?.title || post.title,
      description: post.seo?.description || post.excerpt,
      images: [ogImageUrl],
    },

    // No-index support
    robots: post.seo?.noIndex
      ? { index: false, follow: false }
      : { index: true, follow: true },
  }
}

// Static paths for SSG
export async function generateStaticParams() {
  const posts = await getAllPostSlugs() // fetch from Payload
  return posts.map((post) => ({ slug: post.slug }))
}
```

---

## Breadcrumbs Component (SEO + Accessibility)

```tsx
// components/Breadcrumbs.tsx
interface Crumb {
  label: string
  href?: string
}

export function Breadcrumbs({ crumbs }: { crumbs: Crumb[] }) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: crumbs.map((crumb, i) => ({
      '@type': 'ListItem',
      position: i + 1,
      name: crumb.label,
      item: crumb.href ? `${process.env.NEXT_PUBLIC_SITE_URL}${crumb.href}` : undefined,
    })),
  }

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
      />
      <nav aria-label="Breadcrumb">
        <ol className="flex items-center gap-2 text-sm text-muted-foreground">
          {crumbs.map((crumb, i) => (
            <li key={i} className="flex items-center gap-2">
              {crumb.href ? (
                <a href={crumb.href} className="hover:text-foreground transition-colors">
                  {crumb.label}
                </a>
              ) : (
                <span aria-current="page">{crumb.label}</span>
              )}
              {i < crumbs.length - 1 && <span aria-hidden="true">/</span>}
            </li>
          ))}
        </ol>
      </nav>
    </>
  )
}
```

---

## next/image — SEO Best Practices

```tsx
// Always include:
// 1. Descriptive alt text (keyword-natural, not stuffed)
// 2. Correct sizes prop for responsive images
// 3. priority on above-the-fold / LCP images
// 4. blurDataURL for perceived performance

// Generate blur placeholder for Payload media:
async function getBlurDataUrl(imageUrl: string): Promise<string> {
  const response = await fetch(imageUrl)
  const buffer = await response.arrayBuffer()
  const base64 = Buffer.from(buffer).toString('base64')
  const mimeType = response.headers.get('content-type') || 'image/jpeg'
  return `data:${mimeType};base64,${base64}`
}

// Component:
<Image
  src={imageUrl}
  alt={imageAlt}
  width={imageWidth}
  height={imageHeight}
  priority={isAboveFold}
  quality={isHero ? 90 : 80}
  placeholder="blur"
  blurDataURL={blurDataURL}
  sizes={isFullWidth
    ? '100vw'
    : '(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw'
  }
/>
```

---

## next/font — Zero CLS from Fonts

```typescript
// lib/fonts.ts — centralize fonts
import { Inter, Merriweather, JetBrains_Mono } from 'next/font/google'
import localFont from 'next/font/local'

export const sansFont = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-sans',
  preload: true,
  fallback: ['system-ui', '-apple-system', 'sans-serif'],
  adjustFontFallback: true, // Reduces CLS from font swap
})

export const serifFont = Merriweather({
  subsets: ['latin'],
  weight: ['300', '400', '700', '900'],
  display: 'swap',
  variable: '--font-serif',
  fallback: ['Georgia', 'serif'],
  adjustFontFallback: true,
})

export const monoFont = JetBrains_Mono({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-mono',
  fallback: ['Consolas', 'monospace'],
})

// Local font (fastest — no external request):
export const brandFont = localFont({
  src: [
    { path: '../public/fonts/brand-regular.woff2', weight: '400', style: 'normal' },
    { path: '../public/fonts/brand-bold.woff2', weight: '700', style: 'normal' },
  ],
  display: 'swap',
  variable: '--font-brand',
  preload: true,
  fallback: ['system-ui'],
})
```

---

## Structured Page Template

```tsx
// app/blog/[slug]/page.tsx — complete SEO-optimized page
import { Breadcrumbs } from '@/components/Breadcrumbs'
import { SchemaOrg } from '@/components/SchemaOrg'
import Image from 'next/image'
import { notFound } from 'next/navigation'

export default async function BlogPostPage({ params }: { params: { slug: string } }) {
  const post = await getPost(params.slug)
  if (!post) notFound()

  const articleSchema = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    description: post.excerpt,
    image: post.featuredImage?.url,
    author: {
      '@type': 'Person',
      name: post.author?.name,
      url: `${process.env.NEXT_PUBLIC_SITE_URL}/authors/${post.author?.slug}`,
    },
    publisher: {
      '@type': 'Organization',
      name: 'Your Brand',
      logo: { '@type': 'ImageObject', url: `${process.env.NEXT_PUBLIC_SITE_URL}/logo.png` },
    },
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': `${process.env.NEXT_PUBLIC_SITE_URL}/blog/${post.slug}`,
    },
  }

  return (
    <article itemScope itemType="https://schema.org/Article">
      <SchemaOrg schema={articleSchema} />
      
      <Breadcrumbs crumbs={[
        { label: 'Home', href: '/' },
        { label: 'Blog', href: '/blog' },
        { label: post.title },
      ]} />

      <h1 itemProp="headline">{post.title}</h1>

      {post.featuredImage && (
        <Image
          src={post.featuredImage.url}
          alt={post.featuredImage.alt || post.title}
          width={1200}
          height={630}
          priority    // This is the LCP element
          sizes="(max-width: 1200px) 100vw, 1200px"
          itemProp="image"
        />
      )}

      <div itemProp="articleBody">
        {/* Render Payload rich text or blocks */}
      </div>

      {/* Article metadata for Google */}
      <meta itemProp="datePublished" content={post.publishedAt} />
      <meta itemProp="dateModified" content={post.updatedAt} />
    </article>
  )
}
```

---

## Pagination SEO

```typescript
// app/blog/page/[page]/page.tsx
export async function generateMetadata({ params }: { params: { page: string } }): Promise<Metadata> {
  const page = parseInt(params.page)
  return {
    title: page === 1 ? 'Blog' : `Blog — Page ${page}`,
    // Noindex on pages 2+ (optional strategy — some SEOs prefer indexing all)
    robots: page > 1 ? { index: false, follow: true } : undefined,
    alternates: {
      canonical: page === 1 ? '/blog' : `/blog/page/${page}`,
    },
  }
}
```

---

## 404 & Error Pages

```tsx
// app/not-found.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Page Not Found',
  robots: { index: false, follow: true },
}

export default function NotFound() {
  return (
    <main>
      <h1>404 — Page Not Found</h1>
      <p>The page you're looking for doesn't exist.</p>
      <a href="/">Return Home</a>
    </main>
  )
}
```
