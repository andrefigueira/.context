# Rule: SEO

Rules for pages, metadata, and search-engine-facing output.

## Applies to
`src/app/**/page.*`, `src/pages/**`, `public/sitemap*`, `public/robots*`, `src/head/**`

Page files also match `ui.md` by the `*.tsx`/`*.jsx` suffix. Load both for page-level work.

## Hard rules
1. Every public page sets a unique `<title>` and meta description
2. Every page has canonical URL configured. No duplicate-content ambiguity
3. Open Graph and Twitter card metadata on every page that can be shared
4. Structured data (JSON-LD) for pages that match a schema.org type (Article, Product, FAQ, etc.)
5. No client-side-only rendering for primary page content. Search engines need SSR or static
6. Images have width and height attributes to prevent layout shift
7. Internal links use descriptive anchor text, not "click here"
8. 404 and 500 pages return the correct HTTP status code

## Patterns

Metadata block (adapt to your framework):

```ts
export const metadata = {
  title: "Page title | Site",
  description: "Under 160 characters, specific, answers a query.",
  alternates: { canonical: "/path" },
  openGraph: { title, description, type: "article" },
};
```

## Load on demand
- Meta tags, structured data, Core Web Vitals → `.context/seo/overview.md`
- Performance budgets → `.context/performance.md`
