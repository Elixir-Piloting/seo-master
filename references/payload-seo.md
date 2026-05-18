# Payload CMS SEO — Complete Reference

## Installation

```bash
# Official SEO plugin
npm install @payloadcms/plugin-seo

# Rich text to HTML (for meta description extraction)
npm install @payloadcms/richtext-lexical
```

---

## Payload Config with Full SEO Setup

```typescript
// payload.config.ts
import { buildConfig } from 'payload'
import { seoPlugin } from '@payloadcms/plugin-seo'
import type { GenerateTitle, GenerateURL, GenerateDescription, GenerateImage } from '@payloadcms/plugin-seo/types'
import { Pages } from './collections/Pages'
import { Posts } from './collections/Posts'
import { Media } from './collections/Media'

const generateTitle: GenerateTitle<any> = ({ doc }) => {
  return doc?.title ? `${doc.title} — My Brand` : 'My Brand'
}

const generateURL: GenerateURL<any> = ({ doc, collectionConfig }) => {
  const baseUrl = process.env.NEXT_PUBLIC_SITE_URL || 'https://yoursite.com'
  
  switch (collectionConfig?.slug) {
    case 'posts':
      return `${baseUrl}/blog/${doc?.slug}`
    case 'pages':
      return `${baseUrl}/${doc?.slug === 'home' ? '' : doc?.slug}`
    case 'products':
      return `${baseUrl}/products/${doc?.slug}`
    default:
      return baseUrl
  }
}

const generateDescription: GenerateDescription<any> = ({ doc }) => {
  if (doc?.excerpt) return doc.excerpt
  // Extract first 160 chars from rich text if excerpt not set
  if (doc?.content?.root?.children?.[0]?.children?.[0]?.text) {
    return doc.content.root.children[0].children[0].text.substring(0, 160)
  }
  return ''
}

const generateImage: GenerateImage<any> = ({ doc }) => {
  return doc?.featuredImage || null
}

export default buildConfig({
  collections: [Pages, Posts, Media],
  
  plugins: [
    seoPlugin({
      collections: ['pages', 'posts', 'products'],
      globals: ['settings'],
      uploadsCollection: 'media',
      generateTitle,
      generateURL,
      generateDescription,
      generateImage,
      tabbedUI: true,   // Shows SEO in a separate tab — cleaner UX
      fieldOverrides: {
        title: { label: 'SEO Title (50-60 chars)', admin: { description: 'Appears in Google search results' } },
        description: { label: 'Meta Description (150-160 chars)', admin: { description: 'Short summary for search results' } },
      },
    }),
  ],
})
```

---

## Posts Collection (Full SEO-Ready)

```typescript
// collections/Posts.ts
import { CollectionConfig } from 'payload'
import { lexicalEditor } from '@payloadcms/richtext-lexical'
import { slugField } from '../fields/slug'
import { populatePublishedAt } from '../hooks/populatePublishedAt'

export const Posts: CollectionConfig = {
  slug: 'posts',
  admin: {
    useAsTitle: 'title',
    defaultColumns: ['title', 'status', 'publishedAt', 'author'],
    listSearchableFields: ['title', 'excerpt', 'slug'],
  },
  access: {
    read: ({ req: { user } }) => {
      // Published posts are public; drafts require auth
      if (user) return true
      return { _status: { equals: 'published' } }
    },
  },
  versions: {
    drafts: {
      autosave: { interval: 5000 },
    },
  },
  hooks: {
    beforeChange: [populatePublishedAt],
    afterChange: [
      // Trigger Next.js on-demand revalidation
      async ({ doc, operation }) => {
        if (operation === 'update' || operation === 'create') {
          try {
            await fetch(
              `${process.env.NEXT_PUBLIC_SITE_URL}/api/revalidate?secret=${process.env.REVALIDATION_SECRET}`,
              {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                  path: `/blog/${doc.slug}`,
                  tag: 'posts',
                }),
              }
            )
          } catch (error) {
            console.error('Revalidation failed:', error)
          }
        }
      },
    ],
  },
  fields: [
    {
      name: 'title',
      type: 'text',
      required: true,
      maxLength: 120,
    },
    slugField(), // Reusable slug field — see below
    {
      name: 'status',
      type: 'select',
      options: ['draft', 'published', 'archived'],
      defaultValue: 'draft',
      required: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'publishedAt',
      type: 'date',
      admin: {
        position: 'sidebar',
        date: { pickerAppearance: 'dayAndTime' },
      },
    },
    {
      name: 'author',
      type: 'relationship',
      relationTo: 'users',
      admin: { position: 'sidebar' },
    },
    {
      name: 'categories',
      type: 'relationship',
      relationTo: 'categories',
      hasMany: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'tags',
      type: 'relationship',
      relationTo: 'tags',
      hasMany: true,
      admin: { position: 'sidebar' },
    },
    {
      name: 'excerpt',
      type: 'textarea',
      maxLength: 300,
      admin: { description: 'Short summary (150-300 chars). Used in meta description if SEO description is empty.' },
    },
    {
      name: 'featuredImage',
      type: 'upload',
      relationTo: 'media',
      admin: { description: 'Main image — also used as OG image if no SEO image is set. Min 1200×630.' },
    },
    {
      name: 'content',
      type: 'richText',
      editor: lexicalEditor({}),
      required: true,
    },
    // SEO fields are ADDED BY THE PLUGIN automatically.
    // No need to manually add seo group when using @payloadcms/plugin-seo.
    
    // Manually add these extra SEO fields the plugin doesn't cover:
    {
      name: 'canonicalUrl',
      type: 'text',
      label: 'Canonical URL Override',
      admin: {
        description: 'Leave blank to use auto-generated canonical. Only set if this content lives at another URL.',
      },
    },
    {
      name: 'noIndex',
      type: 'checkbox',
      label: 'Exclude from Search Engines (noindex)',
      defaultValue: false,
      admin: { description: 'Check ONLY for private, thin, or duplicate content pages.' },
    },
    {
      name: 'schemaOverride',
      type: 'json',
      label: 'Custom JSON-LD Schema',
      admin: { description: 'Advanced: Override auto-generated schema. Must be valid JSON-LD.' },
    },
    {
      name: 'priority',
      type: 'number',
      label: 'Sitemap Priority (0.0 – 1.0)',
      defaultValue: 0.7,
      min: 0,
      max: 1,
      admin: { position: 'sidebar', description: 'Controls <priority> in sitemap.xml' },
    },
  ],
}
```

---

## Reusable Slug Field

```typescript
// fields/slug.ts
import { Field } from 'payload'

export const slugField = (sourceField = 'title'): Field => ({
  name: 'slug',
  type: 'text',
  unique: true,
  required: true,
  index: true,
  admin: {
    position: 'sidebar',
    description: 'URL-safe identifier. Auto-generated from title. Lowercase, hyphens only.',
    components: {
      Field: '@/components/SlugField', // Custom component with "auto" button
    },
  },
  hooks: {
    beforeValidate: [
      ({ value, data }) => {
        // Auto-generate slug from title if not set
        if (!value && data?.[sourceField]) {
          return data[sourceField]
            .toLowerCase()
            .replace(/[^a-z0-9\s-]/g, '')    // Remove special chars
            .replace(/\s+/g, '-')             // Spaces to hyphens
            .replace(/-+/g, '-')              // Deduplicate hyphens
            .replace(/^-|-$/g, '')            // Trim leading/trailing hyphens
        }
        // Sanitize existing slug
        if (value) {
          return value
            .toLowerCase()
            .replace(/[^a-z0-9\s-]/g, '')
            .replace(/\s+/g, '-')
            .replace(/-+/g, '-')
            .replace(/^-|-$/g, '')
        }
        return value
      },
    ],
  },
  validate: (value) => {
    if (!value) return 'Slug is required'
    if (!/^[a-z0-9-]+$/.test(value)) return 'Slug must be lowercase letters, numbers, and hyphens only'
    if (value.length > 100) return 'Slug must be under 100 characters'
    return true
  },
})
```

---

## Media Collection (Image SEO)

```typescript
// collections/Media.ts
import { CollectionConfig } from 'payload'

export const Media: CollectionConfig = {
  slug: 'media',
  upload: {
    staticDir: 'public/media',
    mimeTypes: ['image/jpeg', 'image/png', 'image/webp', 'image/avif', 'image/svg+xml'],
    imageSizes: [
      // Thumbnail
      { name: 'thumbnail', width: 400, height: 300, position: 'centre', formatOptions: { format: 'webp', options: { quality: 80 } } },
      // Card
      { name: 'card', width: 768, height: 512, position: 'centre', formatOptions: { format: 'webp', options: { quality: 80 } } },
      // Feature
      { name: 'feature', width: 1024, height: 576, position: 'centre', formatOptions: { format: 'webp', options: { quality: 85 } } },
      // Hero
      { name: 'hero', width: 1920, height: 1080, position: 'centre', formatOptions: { format: 'webp', options: { quality: 85 } } },
      // OG image
      { name: 'og', width: 1200, height: 630, position: 'centre', formatOptions: { format: 'jpeg', options: { quality: 90 } } },
    ],
    adminThumbnail: 'thumbnail',
  },
  fields: [
    {
      name: 'alt',
      type: 'text',
      required: true,
      label: 'Alt Text',
      admin: { description: 'Describe the image for screen readers and search engines. Be specific and descriptive.' },
    },
    {
      name: 'caption',
      type: 'text',
      label: 'Caption (shown below image)',
    },
    {
      name: 'credit',
      type: 'text',
      label: 'Image Credit / Attribution',
    },
  ],
}
```

---

## Payload Hooks for SEO

### populatePublishedAt hook
```typescript
// hooks/populatePublishedAt.ts
import { FieldHook } from 'payload'

export const populatePublishedAt: FieldHook = ({ data, operation, value }) => {
  if (operation === 'create' || operation === 'update') {
    if (data?._status === 'published' && !value) {
      return new Date().toISOString()
    }
  }
  return value
}
```

### Auto-revalidate on publish
```typescript
// hooks/revalidatePage.ts
export const revalidatePage = async ({ doc, collectionConfig }: any) => {
  const baseUrl = process.env.NEXT_PUBLIC_SITE_URL
  const secret = process.env.REVALIDATION_SECRET

  const pathMap: Record<string, (doc: any) => string> = {
    posts: (d) => `/blog/${d.slug}`,
    pages: (d) => d.slug === 'home' ? '/' : `/${d.slug}`,
    products: (d) => `/products/${d.slug}`,
  }

  const getPath = pathMap[collectionConfig.slug]
  if (!getPath) return

  const path = getPath(doc)

  try {
    const res = await fetch(`${baseUrl}/api/revalidate?secret=${secret}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ path, tag: collectionConfig.slug }),
    })
    if (!res.ok) console.error('Revalidation failed:', await res.text())
    else console.log(`Revalidated: ${path}`)
  } catch (err) {
    console.error('Revalidation error:', err)
  }
}
```

---

## Fetching SEO Data from Payload (Next.js side)

```typescript
// lib/payload.ts
const PAYLOAD_URL = process.env.PAYLOAD_URL || 'http://localhost:3001'

// Fetch a post with all SEO fields
export async function getPost(slug: string) {
  const res = await fetch(
    `${PAYLOAD_URL}/api/posts?where[slug][equals]=${slug}&depth=2&limit=1`,
    {
      next: { tags: ['posts', `post-${slug}`] },
    }
  )
  if (!res.ok) return null
  const data = await res.json()
  return data.docs?.[0] || null
}

// Fetch all slugs for generateStaticParams
export async function getAllPostSlugs() {
  const res = await fetch(`${PAYLOAD_URL}/api/posts?limit=1000&depth=0&select=slug`, {
    next: { revalidate: 3600 },
  })
  const data = await res.json()
  return data.docs || []
}

// Fetch global SEO settings
export async function getGlobalSEO() {
  const res = await fetch(`${PAYLOAD_URL}/api/globals/settings?depth=1`, {
    next: { revalidate: 86400 },
  })
  return res.json()
}
```

---

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_SITE_URL=https://yoursite.com
PAYLOAD_URL=https://cms.yoursite.com          # or http://localhost:3001 in dev
PAYLOAD_SECRET=your-very-long-random-secret
DATABASE_URI=mongodb://localhost/yourdb       # or postgres://...
REVALIDATION_SECRET=your-revalidation-secret
GOOGLE_VERIFICATION_TOKEN=your-token
NEXT_PUBLIC_GA_ID=G-XXXXXXXXXX
```
