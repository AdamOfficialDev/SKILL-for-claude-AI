# DAVID — SEO Audit Reference (Scanner T)

Full SEO pattern library. Load for Next.js, Nuxt, Remix, SvelteKit, or any SSR app.

---

## Critical SEO Checklist (P1 — Fix Immediately)

### Title Tag
```html
<!-- ❌ Missing entirely -->
<head><meta charset="utf-8"></head>

<!-- ❌ Generic/same for all pages -->
<title>My App</title>

<!-- ❌ Too long (>60 chars gets truncated in SERPs) -->
<title>Buy Premium Organic Coffee Beans Online | Best Quality Single Origin Coffee | Free Shipping</title>

<!-- ✅ Unique, descriptive, 50-60 chars, keyword-first -->
<title>Organic Coffee Beans | Single Origin, Free Shipping</title>
```

### Meta Description
```html
<!-- ❌ Missing — Google generates one, often badly -->
<!-- no meta description -->

<!-- ❌ Too short or generic -->
<meta name="description" content="Welcome to our website.">

<!-- ❌ Too long (>160 chars gets truncated) -->

<!-- ✅ Unique per page, 150-160 chars, includes CTA -->
<meta name="description" content="Shop 20+ single-origin organic coffee beans. Roasted fresh, shipped within 24h. Free shipping on orders over $40. Try our starter pack today.">
```

### Next.js Metadata (App Router)
```typescript
// ✅ Per-page metadata
export const metadata: Metadata = {
  title: 'Product Name | Brand',
  description: 'Unique description for this page, 150-160 chars',
  openGraph: {
    title: 'Product Name',
    description: 'OG description (can differ from meta description)',
    images: [{ url: 'https://example.com/og-image.jpg', width: 1200, height: 630 }],
    url: 'https://example.com/products/slug',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Product Name',
    description: 'Twitter card description',
    images: ['https://example.com/twitter-image.jpg'],
  },
  alternates: {
    canonical: 'https://example.com/products/slug',
  },
  robots: {
    index: true,  // Explicit — don't leave to default
    follow: true,
  },
};

// ✅ Dynamic metadata for dynamic routes
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const product = await getProduct(params.slug);
  return {
    title: `${product.name} | Brand`,
    description: product.description.slice(0, 155),
    openGraph: {
      images: [product.imageUrl],
    },
  };
}
```

---

## Open Graph Tags

```html
<!-- ✅ Full OG set — check all are present and non-empty -->
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Page description for social sharing">
<meta property="og:image" content="https://example.com/og-image.jpg">  <!-- ABSOLUTE URL! -->
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">
<meta property="og:url" content="https://example.com/current-page">   <!-- ABSOLUTE URL! -->
<meta property="og:type" content="website">  <!-- or article, product -->
<meta property="og:site_name" content="Brand Name">

<!-- Twitter Card -->
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@brandhandle">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Description">
<meta name="twitter:image" content="https://example.com/twitter-image.jpg">
```

**DAVID checks:**
- OG image is absolute URL (relative paths don't work in social previews)
- OG image is 1200×630px (recommended) or at least 600×315px (minimum)
- og:url matches the canonical URL
- Each page has unique og:title and og:description

---

## Canonical URL

```html
<!-- ✅ Every page needs a canonical — prevents duplicate content penalty -->
<link rel="canonical" href="https://example.com/products/coffee-beans">

<!-- ❌ Common mistakes DAVID detects: -->

<!-- 1. Canonical pointing to wrong URL -->
<link rel="canonical" href="https://example.com/products/coffee-beans?ref=email">
<!-- Should strip query params in canonical -->

<!-- 2. Self-referential canonical on paginated pages -->
<!-- Page 2 canonical should NOT point to page 1 -->

<!-- 3. Canonical using http when site is https -->
<link rel="canonical" href="http://example.com/products/coffee-beans">

<!-- 4. Missing canonical on duplicate content (www vs non-www) -->
```

---

## Heading Hierarchy

```html
<!-- ❌ Multiple H1 tags -->
<h1>Page Title</h1>
<h1>Section Title</h1>  <!-- Should be h2 -->

<!-- ❌ Skipping heading levels -->
<h1>Page Title</h1>
<h3>Subsection</h3>  <!-- Missing h2 -->

<!-- ❌ Using headings for styling instead of hierarchy -->
<h4 class="big-text">Not actually a heading</h4>  <!-- Should be <p> or <span> -->

<!-- ✅ One H1, logical hierarchy -->
<h1>Main Page Topic</h1>
  <h2>Section 1</h2>
    <h3>Subsection 1.1</h3>
  <h2>Section 2</h2>
    <h3>Subsection 2.1</h3>
```

---

## The noindex Trap

```typescript
// ❌ CRITICAL — accidentally blocking production indexing

// Next.js app/layout.tsx
export const metadata = {
  robots: 'noindex, nofollow',  // BLOCKS ENTIRE SITE
};

// Even worse — in environment-conditional that's broken:
export const metadata = {
  robots: process.env.NODE_ENV === 'production' ? 'index' : 'noindex',
  // If NODE_ENV not set in production → defaults to 'noindex' → invisible to Google
};

// ✅ Safe pattern
export const metadata = {
  robots: {
    index: process.env.NEXT_PUBLIC_ALLOW_INDEXING === 'true',
    follow: process.env.NEXT_PUBLIC_ALLOW_INDEXING === 'true',
  },
};
```

**DAVID flags ANY `noindex` in production code with 🔴 CRITICAL — requires manual verification.**

---

## Structured Data (Schema.org)

```html
<!-- ✅ Article schema -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Article Title",
  "author": { "@type": "Person", "name": "Author Name" },
  "datePublished": "2024-01-15",
  "dateModified": "2024-01-20",
  "image": "https://example.com/article-image.jpg",
  "publisher": {
    "@type": "Organization",
    "name": "Brand Name",
    "logo": { "@type": "ImageObject", "url": "https://example.com/logo.png" }
  }
}
</script>

<!-- ✅ Product schema -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "description": "Product description",
  "image": "https://example.com/product.jpg",
  "offers": {
    "@type": "Offer",
    "price": "29.99",
    "priceCurrency": "USD",
    "availability": "https://schema.org/InStock"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.8",
    "reviewCount": "127"
  }
}
</script>

<!-- ✅ Breadcrumb schema -->
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com" },
    { "@type": "ListItem", "position": 2, "name": "Products", "item": "https://example.com/products" },
    { "@type": "ListItem", "position": 3, "name": "Coffee Beans" }
  ]
}
</script>
```

---

## Image SEO

```html
<!-- ❌ Missing alt (both a11y AND SEO failure) -->
<img src="/products/coffee.jpg">

<!-- ❌ Generic alt -->
<img src="/products/coffee.jpg" alt="image">
<img src="/products/coffee.jpg" alt="product photo">

<!-- ✅ Descriptive alt with keyword -->
<img src="/products/coffee.jpg" alt="Ethiopian Yirgacheffe single-origin coffee beans 250g">

<!-- ✅ Next.js Image component (required for Next.js SEO) -->
<Image
  src="/products/coffee.jpg"
  alt="Ethiopian Yirgacheffe single-origin coffee beans 250g"
  width={800}
  height={600}
  priority={true}  // For LCP images — loads eagerly
/>
```

---

## Technical SEO Checklist

| Check | What DAVID Looks For |
|-------|---------------------|
| `robots.txt` | Exists and doesn't block important paths |
| `sitemap.xml` | Exists, accessible, referenced in robots.txt |
| HTTPS | All internal links use https, no mixed content |
| Mobile viewport | `<meta name="viewport" content="width=device-width, initial-scale=1">` |
| Page speed | No render-blocking resources (check script placement) |
| Core Web Vitals | No `width`/`height` missing on images (CLS), no large unoptimized images (LCP) |
| Pagination | `rel="next"` and `rel="prev"` on paginated pages |
| Hreflang | Multi-language sites need `<link rel="alternate" hreflang="...">` |
| 404 handling | Custom 404 page that returns actual 404 status code (not 200 with "not found" text) |
