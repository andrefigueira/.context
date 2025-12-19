# SEO: Search Engine Optimization Overview

This document defines SEO implementation patterns, structured data schemas, and performance optimization strategies for [Your Project Name].

## Technical SEO Requirements

### Meta Tag Specification

```yaml
# meta-requirements.yaml - Deterministic meta tag rules
required_meta_tags:
  title:
    max_length: 60
    format: "{Page Title} | {Brand Name}"
    unique: true
    example: "Pricing Plans | Acme Software"

  description:
    max_length: 160
    min_length: 120
    unique: true
    include_cta: recommended
    example: "Compare our pricing plans. Start free, scale as you grow. No credit card required."

  canonical:
    required: true
    format: absolute_url
    trailing_slash: false
    example: "https://example.com/pricing"

  robots:
    default: "index, follow"
    noindex_pages:
      - "/admin/*"
      - "/internal/*"
      - "/preview/*"
      - "/*?preview=*"

  viewport:
    required: true
    value: "width=device-width, initial-scale=1"

open_graph:
  og:title:
    max_length: 60
    fallback: title tag
  og:description:
    max_length: 200
    fallback: meta description
  og:image:
    dimensions: "1200x630"
    format: jpg | png | webp
    max_size: "8MB"
    required: true
  og:type:
    values: [website, article, product]
    default: website
  og:url:
    required: true
    matches: canonical

twitter:
  twitter:card:
    values: [summary, summary_large_image]
    default: summary_large_image
  twitter:site:
    format: "@username"
  twitter:image:
    dimensions: "1200x600"
    fallback: og:image
```

### Implementation

```tsx
// components/SEO/SEO.tsx
interface SEOProps {
  title: string;
  description: string;
  canonical?: string;
  image?: {
    url: string;
    width?: number;
    height?: number;
    alt?: string;
  };
  type?: 'website' | 'article' | 'product';
  noindex?: boolean;
  article?: {
    publishedTime: string;
    modifiedTime?: string;
    author: string;
    section?: string;
    tags?: string[];
  };
}

const BRAND_NAME = 'Acme Software';
const SITE_URL = 'https://example.com';
const DEFAULT_IMAGE = '/images/og-default.jpg';

export function SEO({
  title,
  description,
  canonical,
  image,
  type = 'website',
  noindex = false,
  article,
}: SEOProps) {
  const fullTitle = `${title} | ${BRAND_NAME}`;
  const canonicalUrl = canonical ?? (typeof window !== 'undefined' ? window.location.href : '');
  const imageUrl = image?.url ?? DEFAULT_IMAGE;

  return (
    <Head>
      {/* Primary Meta Tags */}
      <title>{fullTitle}</title>
      <meta name="description" content={description} />
      <link rel="canonical" href={canonicalUrl} />
      {noindex && <meta name="robots" content="noindex, nofollow" />}

      {/* Open Graph */}
      <meta property="og:type" content={type} />
      <meta property="og:title" content={fullTitle} />
      <meta property="og:description" content={description} />
      <meta property="og:url" content={canonicalUrl} />
      <meta property="og:image" content={imageUrl} />
      {image?.width && <meta property="og:image:width" content={String(image.width)} />}
      {image?.height && <meta property="og:image:height" content={String(image.height)} />}
      {image?.alt && <meta property="og:image:alt" content={image.alt} />}

      {/* Twitter */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={fullTitle} />
      <meta name="twitter:description" content={description} />
      <meta name="twitter:image" content={imageUrl} />

      {/* Article specific */}
      {article && (
        <>
          <meta property="article:published_time" content={article.publishedTime} />
          {article.modifiedTime && (
            <meta property="article:modified_time" content={article.modifiedTime} />
          )}
          <meta property="article:author" content={article.author} />
          {article.section && (
            <meta property="article:section" content={article.section} />
          )}
          {article.tags?.map((tag) => (
            <meta key={tag} property="article:tag" content={tag} />
          ))}
        </>
      )}
    </Head>
  );
}
```

## Structured Data Schemas

### JSON-LD Implementation

```yaml
# structured-data-schemas.yaml
schemas:
  Organization:
    required_fields:
      - "@type": "Organization"
      - name: string
      - url: url
      - logo: url
    optional_fields:
      - sameAs: url[] (social profiles)
      - contactPoint: ContactPoint

  WebSite:
    required_fields:
      - "@type": "WebSite"
      - name: string
      - url: url
    optional_fields:
      - potentialAction: SearchAction

  Article:
    required_fields:
      - "@type": "Article"
      - headline: string (max 110 chars)
      - image: url[]
      - datePublished: ISO8601
      - author: Person | Organization
    optional_fields:
      - dateModified: ISO8601
      - publisher: Organization

  Product:
    required_fields:
      - "@type": "Product"
      - name: string
      - image: url[]
      - description: string
    optional_fields:
      - sku: string
      - brand: Brand
      - offers: Offer
      - aggregateRating: AggregateRating
      - review: Review[]

  BreadcrumbList:
    required_fields:
      - "@type": "BreadcrumbList"
      - itemListElement: ListItem[]
    list_item_fields:
      - "@type": "ListItem"
      - position: integer
      - name: string
      - item: url

  FAQPage:
    required_fields:
      - "@type": "FAQPage"
      - mainEntity: Question[]
    question_fields:
      - "@type": "Question"
      - name: string (the question)
      - acceptedAnswer:
          "@type": "Answer"
          text: string
```

### Implementation Examples

```tsx
// components/StructuredData/OrganizationSchema.tsx
export function OrganizationSchema() {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Acme Software',
    url: 'https://example.com',
    logo: 'https://example.com/logo.png',
    sameAs: [
      'https://twitter.com/acme',
      'https://linkedin.com/company/acme',
      'https://github.com/acme',
    ],
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+1-800-555-0123',
      contactType: 'customer service',
      availableLanguage: ['English'],
    },
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}

// components/StructuredData/ArticleSchema.tsx
interface ArticleSchemaProps {
  title: string;
  description: string;
  image: string;
  datePublished: string;
  dateModified?: string;
  author: {
    name: string;
    url?: string;
  };
}

export function ArticleSchema({
  title,
  description,
  image,
  datePublished,
  dateModified,
  author,
}: ArticleSchemaProps) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: title.slice(0, 110),
    description,
    image: [image],
    datePublished,
    dateModified: dateModified ?? datePublished,
    author: {
      '@type': 'Person',
      name: author.name,
      url: author.url,
    },
    publisher: {
      '@type': 'Organization',
      name: 'Acme Software',
      logo: {
        '@type': 'ImageObject',
        url: 'https://example.com/logo.png',
      },
    },
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}

// components/StructuredData/BreadcrumbSchema.tsx
interface BreadcrumbItem {
  name: string;
  url: string;
}

export function BreadcrumbSchema({ items }: { items: BreadcrumbItem[] }) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: items.map((item, index) => ({
      '@type': 'ListItem',
      position: index + 1,
      name: item.name,
      item: item.url,
    })),
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

## URL Structure

```yaml
url_patterns:
  format:
    lowercase: true
    separator: "-"
    trailing_slash: false
    max_length: 75

  hierarchy:
    blog:
      list: "/blog"
      category: "/blog/category/{slug}"
      post: "/blog/{slug}"
    products:
      list: "/products"
      category: "/products/{category}"
      detail: "/products/{category}/{slug}"
    pages:
      static: "/{slug}"
      nested: "/{parent}/{slug}"

  redirects:
    301_required:
      - http -> https
      - www -> non-www (or vice versa)
      - trailing slash normalization
      - old urls -> new urls
    preserve: query parameters where applicable

  localization:
    pattern: "/{locale}/{path}"
    default_locale_prefix: false
    hreflang_required: true
    example:
      - "https://example.com/pricing" (English, default)
      - "https://example.com/es/precios" (Spanish)
      - "https://example.com/de/preise" (German)
```

## Performance Requirements

```yaml
core_web_vitals:
  LCP:
    target: "< 2.5s"
    good: "< 2.5s"
    needs_improvement: "2.5s - 4.0s"
    poor: "> 4.0s"
    optimization:
      - preload hero images
      - optimize server response time
      - use CDN for assets

  FID:
    target: "< 100ms"
    good: "< 100ms"
    needs_improvement: "100ms - 300ms"
    poor: "> 300ms"
    optimization:
      - minimize main thread work
      - break up long tasks
      - use web workers for heavy computation

  CLS:
    target: "< 0.1"
    good: "< 0.1"
    needs_improvement: "0.1 - 0.25"
    poor: "> 0.25"
    optimization:
      - set explicit dimensions on images/embeds
      - reserve space for dynamic content
      - avoid inserting content above existing content

  INP:
    target: "< 200ms"
    good: "< 200ms"
    needs_improvement: "200ms - 500ms"
    poor: "> 500ms"
```

### Image Optimization

```tsx
// components/OptimizedImage.tsx
interface OptimizedImageProps {
  src: string;
  alt: string;
  width: number;
  height: number;
  priority?: boolean;
  sizes?: string;
}

export function OptimizedImage({
  src,
  alt,
  width,
  height,
  priority = false,
  sizes = '100vw',
}: OptimizedImageProps) {
  return (
    <picture>
      {/* WebP for modern browsers */}
      <source
        type="image/webp"
        srcSet={generateSrcSet(src, 'webp')}
        sizes={sizes}
      />
      {/* AVIF for cutting edge browsers */}
      <source
        type="image/avif"
        srcSet={generateSrcSet(src, 'avif')}
        sizes={sizes}
      />
      {/* Fallback */}
      <img
        src={src}
        alt={alt}
        width={width}
        height={height}
        loading={priority ? 'eager' : 'lazy'}
        decoding="async"
        fetchPriority={priority ? 'high' : 'auto'}
      />
    </picture>
  );
}

function generateSrcSet(src: string, format: string): string {
  const widths = [640, 750, 828, 1080, 1200, 1920];
  return widths
    .map((w) => `${transformUrl(src, w, format)} ${w}w`)
    .join(', ');
}
```

## Sitemap & Robots

### Sitemap Generation

```typescript
// scripts/generate-sitemap.ts
interface SitemapEntry {
  url: string;
  lastmod?: string;
  changefreq?: 'always' | 'hourly' | 'daily' | 'weekly' | 'monthly' | 'yearly' | 'never';
  priority?: number;
  images?: Array<{ url: string; title?: string; caption?: string }>;
}

const sitemapConfig = {
  baseUrl: 'https://example.com',
  pages: [
    { path: '/', changefreq: 'weekly', priority: 1.0 },
    { path: '/pricing', changefreq: 'weekly', priority: 0.9 },
    { path: '/features', changefreq: 'monthly', priority: 0.8 },
    { path: '/blog', changefreq: 'daily', priority: 0.8 },
  ],
  dynamicRoutes: [
    { pattern: '/blog/:slug', source: 'posts' },
    { pattern: '/products/:slug', source: 'products' },
  ],
};

async function generateSitemap(): Promise<string> {
  const entries: SitemapEntry[] = [];

  // Static pages
  for (const page of sitemapConfig.pages) {
    entries.push({
      url: `${sitemapConfig.baseUrl}${page.path}`,
      changefreq: page.changefreq,
      priority: page.priority,
    });
  }

  // Dynamic routes
  for (const route of sitemapConfig.dynamicRoutes) {
    const items = await fetchItems(route.source);
    for (const item of items) {
      entries.push({
        url: `${sitemapConfig.baseUrl}${route.pattern.replace(':slug', item.slug)}`,
        lastmod: item.updatedAt,
        priority: 0.7,
      });
    }
  }

  return buildXml(entries);
}
```

### Robots.txt

```txt
# robots.txt
User-agent: *
Allow: /

# Block admin and internal pages
Disallow: /admin/
Disallow: /internal/
Disallow: /api/
Disallow: /_next/
Disallow: /preview/

# Block query parameter variations
Disallow: /*?preview=
Disallow: /*?ref=
Disallow: /*?utm_

# Crawl delay (optional, not all bots respect this)
Crawl-delay: 1

# Sitemap location
Sitemap: https://example.com/sitemap.xml
Sitemap: https://example.com/sitemap-blog.xml
Sitemap: https://example.com/sitemap-products.xml
```

## Monitoring & Validation

```yaml
seo_monitoring:
  tools:
    - Google Search Console
    - Bing Webmaster Tools
    - Ahrefs / SEMrush (optional)

  alerts:
    critical:
      - indexing errors > 0
      - core web vitals fail
      - 5xx errors on key pages
      - canonical mismatches
    warning:
      - 404 errors increase
      - coverage drops
      - mobile usability issues

  audits:
    frequency: weekly
    checks:
      - all pages have unique titles
      - all pages have meta descriptions
      - no broken internal links
      - structured data validates
      - images have alt text
      - canonical tags present
```

## Decision History & Trade-offs

### SSR vs SSG for SEO Pages
**Decision**: Use SSG for marketing pages, SSR for dynamic content
**Rationale**:
- SSG provides fastest possible TTFB
- Marketing pages change infrequently
- SSR for personalized or frequently updated content

**Trade-offs**:
- Need to trigger rebuilds for SSG content updates
- More complex deployment pipeline

### Image Format Strategy
**Decision**: Serve AVIF > WebP > JPEG with picture element
**Rationale**:
- AVIF provides 50% smaller files than WebP
- WebP has 95%+ browser support
- Fallback ensures universal compatibility

**Trade-offs**:
- Requires generating multiple formats
- Increased storage requirements
- Build time increases

---

**Next**: Review [../guidelines.md](../guidelines.md) for development workflow and deployment procedures.
