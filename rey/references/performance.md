# REY — Performance & Deployment Reference

> Performance is a feature. Bukan afterthought.

---

## 🎯 TARGET METRICS

| Metric | Target | Tool |
|--------|--------|------|
| Lighthouse Performance | **95+** | Lighthouse / PageSpeed |
| Lighthouse Accessibility | **95+** | Lighthouse |
| Lighthouse Best Practices | **95+** | Lighthouse |
| Lighthouse SEO | **95+** | Lighthouse |
| LCP (Largest Contentful Paint) | **< 2.5s** | Core Web Vitals |
| FID / INP (Interaction to Next Paint) | **< 200ms** | Core Web Vitals |
| CLS (Cumulative Layout Shift) | **< 0.1** | Core Web Vitals |
| TTFB (Time to First Byte) | **< 200ms** | WebPageTest |
| Bundle size (initial JS, gzipped) | **< 150KB** | Bundle analyzer |
| Total page weight | **< 500KB** | DevTools |
| API response time (p95) | **< 200ms** | Sentry / Axiom |

---

## ⚡ FRONTEND PERFORMANCE CHECKLIST

### Images
- [ ] Gunakan `next/image` untuk automatic optimization, WebP, lazy loading
- [ ] Sertakan `width` dan `height` untuk menghindari CLS
- [ ] Gunakan `priority` prop untuk LCP image (hero, fold-above)
- [ ] Gunakan `sizes` prop yang tepat untuk responsive images
- [ ] Compress images sebelum upload (< 100KB untuk thumbnails)
- [ ] Gunakan CDN untuk static assets

### JavaScript
- [ ] Audit bundle dengan `@next/bundle-analyzer`
- [ ] Gunakan dynamic imports untuk heavy components
  ```tsx
  const HeavyChart = dynamic(() => import('./HeavyChart'), { 
    ssr: false,
    loading: () => <ChartSkeleton />
  })
  ```
- [ ] Tree-shake unused imports (hindari `import * as`)
- [ ] Gunakan `React.memo`, `useMemo`, `useCallback` dengan tepat (tidak berlebihan)
- [ ] Hindari re-renders tidak perlu dengan state management yang tepat
- [ ] Code-split berdasarkan routes (Next.js otomatis, tapi perhatikan shared chunks)

### CSS
- [ ] Tailwind CSS otomatis purge unused styles
- [ ] Hindari CSS-in-JS yang runtime-heavy
- [ ] Gunakan CSS variables untuk theming (bukan JS-based)
- [ ] Critical CSS di-inline untuk above-the-fold content
- [ ] Font loading dioptimasi dengan `next/font` (zero FOUT)

### Fonts
```tsx
// ✅ Optimal font loading
import { Geist } from 'next/font/google'
const font = Geist({ 
  subsets: ['latin'],
  display: 'swap',      // FOUT instead of FOIT
  preload: true,
  variable: '--font-geist'
})
```

### Caching Strategy
```typescript
// Next.js App Router fetch caching
// Static (cache forever until revalidate)
fetch(url, { next: { revalidate: 3600 } })

// No cache (always fresh)
fetch(url, { cache: 'no-store' })

// On-demand revalidation
import { revalidatePath, revalidateTag } from 'next/cache'
revalidatePath('/blog')
revalidateTag('posts')
```

### Route Strategy (Next.js)
```typescript
// Static pages = fastest possible
export const dynamic = 'force-static'

// Incremental Static Regeneration
export const revalidate = 60 // seconds

// Server-rendered (fresh data, good TTFB if API is fast)
export const dynamic = 'force-dynamic'
```

---

## 🗄️ DATABASE PERFORMANCE

### Indexing Rules
```sql
-- WAJIB: Index semua foreign keys
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Index untuk kolom yang sering di-filter
CREATE INDEX idx_posts_status ON posts(status);
CREATE INDEX idx_posts_published_at ON posts(published_at DESC);

-- Composite index untuk query umum
CREATE INDEX idx_posts_user_status ON posts(user_id, status);

-- Partial index untuk subset data
CREATE INDEX idx_active_users ON users(email) WHERE deleted_at IS NULL;

-- Full-text search
CREATE INDEX idx_posts_fts ON posts USING gin(
  to_tsvector('english', title || ' ' || content)
);
```

### N+1 Query Prevention
```typescript
// ❌ N+1 problem
const posts = await db.select().from(postsTable)
for (const post of posts) {
  const author = await db.select().from(usersTable).where(eq(usersTable.id, post.userId))
}

// ✅ Single query dengan join
const postsWithAuthors = await db
  .select({
    post: postsTable,
    author: usersTable
  })
  .from(postsTable)
  .leftJoin(usersTable, eq(postsTable.userId, usersTable.id))
```

### Connection Pooling
```typescript
// Drizzle + Supabase connection pooling
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'

// Use transaction pooler URL (port 6543) for serverless
const client = postgres(process.env.DATABASE_POOLER_URL!, { 
  prepare: false  // Required for PgBouncer
})
export const db = drizzle(client)
```

### Caching Layer (Redis/Upstash)
```typescript
import { Redis } from '@upstash/redis'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
})

// Cache pattern
async function getCachedData<T>(
  key: string, 
  fetchFn: () => Promise<T>,
  ttl = 60
): Promise<T> {
  const cached = await redis.get<T>(key)
  if (cached) return cached
  
  const data = await fetchFn()
  await redis.setex(key, ttl, data)
  return data
}
```

---

## 🔒 SECURITY CHECKLIST

### Application Security
- [ ] HTTPS everywhere (automatic dengan Vercel/Cloudflare)
- [ ] Security headers configured
  ```typescript
  // next.config.ts
  const securityHeaders = [
    { key: 'X-DNS-Prefetch-Control', value: 'on' },
    { key: 'X-Frame-Options', value: 'SAMEORIGIN' },
    { key: 'X-Content-Type-Options', value: 'nosniff' },
    { key: 'Referrer-Policy', value: 'origin-when-cross-origin' },
    { key: 'Permissions-Policy', value: 'camera=(), microphone=()' },
    {
      key: 'Content-Security-Policy',
      value: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline';"
    }
  ]
  ```
- [ ] Input validation dengan Zod di semua API endpoints
- [ ] Rate limiting pada API routes
  ```typescript
  import { Ratelimit } from '@upstash/ratelimit'
  const ratelimit = new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, '10 s'),
  })
  ```
- [ ] SQL injection: gunakan ORM (Drizzle/Prisma), jangan raw queries tanpa parameterization
- [ ] CSRF protection (Next.js handles this automatically untuk API routes)
- [ ] Sanitize user-generated content sebelum render (DOMPurify)
- [ ] Secrets hanya di environment variables, tidak di kode

### Auth Security
- [ ] JWT expiry pendek (15 min access token) + refresh token rotation
- [ ] Secure + HttpOnly cookies untuk tokens
- [ ] Logout invalidates server-side session
- [ ] Password policy: min 8 chars, complexity requirements
- [ ] Brute force protection (rate limiting pada auth endpoints)

---

## 🚀 DEPLOYMENT GUIDE

### Environment Structure
```
development  → local (.env.local)
staging      → preview deployment (Vercel preview / Railway staging)
production   → main deployment
```

### Environment Variables Template
```bash
# .env.example (COMMIT ini ke git)
# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME="My App"

# Database
DATABASE_URL=postgresql://...
DATABASE_POOLER_URL=postgresql://...  # For serverless

# Auth (Clerk)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...

# Storage
UPLOADTHING_SECRET=sk_...
UPLOADTHING_APP_ID=...

# Email (Resend)
RESEND_API_KEY=re_...

# Cache (Upstash)
UPSTASH_REDIS_REST_URL=https://...
UPSTASH_REDIS_REST_TOKEN=...

# Monitoring
NEXT_PUBLIC_SENTRY_DSN=https://...
SENTRY_AUTH_TOKEN=...

# Analytics
NEXT_PUBLIC_POSTHOG_KEY=phc_...
NEXT_PUBLIC_POSTHOG_HOST=https://app.posthog.com
```

### Vercel Deployment Config
```json
// vercel.json
{
  "framework": "nextjs",
  "regions": ["sin1"],
  "functions": {
    "src/app/api/**/*.ts": {
      "maxDuration": 30
    }
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" }
      ]
    }
  ]
}
```

### GitHub Actions CI/CD
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck
      - run: pnpm lint
      - run: pnpm test --run
      - run: pnpm build
```

---

## 📊 MONITORING SETUP

### Sentry (Error Tracking)
```typescript
// instrumentation.ts (Next.js)
export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./sentry.server.config')
  }
  if (process.env.NEXT_RUNTIME === 'edge') {
    await import('./sentry.edge.config')
  }
}
```

### PostHog (Analytics)
```tsx
// providers/posthog-provider.tsx
'use client'
import posthog from 'posthog-js'
import { PostHogProvider as PHProvider } from 'posthog-js/react'

export function PostHogProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
      capture_pageview: false, // Manual in Next.js
    })
  }, [])
  
  return <PHProvider client={posthog}>{children}</PHProvider>
}
```

---

## 🧪 TESTING STRATEGY

### Test Pyramid
```
E2E Tests (Playwright)          — Sedikit, critical paths only
   ↑
Integration Tests (Vitest)      — API routes, DB queries
   ↑
Unit Tests (Vitest)             — Utils, hooks, validators (banyak)
```

### Vitest Setup
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    environment: 'jsdom',
    setupFiles: ['./tests/setup.ts'],
    coverage: {
      provider: 'v8',
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 70,
      }
    }
  }
})
```

### Playwright E2E
```typescript
// tests/e2e/auth.spec.ts
import { test, expect } from '@playwright/test'

test('user can sign up and access dashboard', async ({ page }) => {
  await page.goto('/register')
  await page.fill('[name=email]', 'test@example.com')
  await page.fill('[name=password]', 'SecurePass123!')
  await page.click('[type=submit]')
  
  await expect(page).toHaveURL('/dashboard')
  await expect(page.getByRole('heading', { name: 'Dashboard' })).toBeVisible()
})
```

---

## 📋 PRE-LAUNCH CHECKLIST

### Performance
- [ ] Lighthouse score 90+ di semua kategori
- [ ] Core Web Vitals passing (Green)
- [ ] Images dioptimasi
- [ ] No unused JavaScript
- [ ] Fonts preloaded

### SEO
- [ ] Meta tags lengkap (title, description, OG tags)
- [ ] Sitemap.xml generated
- [ ] Robots.txt configured
- [ ] Structured data (JSON-LD) untuk relevant pages
- [ ] Canonical URLs set

### Security
- [ ] Security headers configured
- [ ] Rate limiting pada semua public APIs
- [ ] Environment variables verified di production
- [ ] No debug/console.log in production
- [ ] Error messages tidak expose sensitive info

### Monitoring
- [ ] Sentry configured + test error
- [ ] Uptime monitoring (Checkly)
- [ ] Database backups verified
- [ ] Alert thresholds set

### Accessibility
- [ ] Keyboard navigation works throughout
- [ ] Screen reader tested (VoiceOver/NVDA)
- [ ] Color contrast verified
- [ ] Focus states visible
- [ ] Alt text on all images
- [ ] Form labels properly associated

---

## 🌊 ADVANCED: STREAMING SSR & SUSPENSE

### Next.js Streaming Pattern
```tsx
// app/dashboard/page.tsx — stream components independently
import { Suspense } from 'react'

export default function DashboardPage() {
  return (
    <div className="grid grid-cols-2 gap-6">
      {/* Critical UI renders immediately */}
      <DashboardHeader />
      
      {/* Each section streams as data becomes available */}
      <Suspense fallback={<MetricsSkeleton />}>
        <MetricsSection />  {/* Fetches own data */}
      </Suspense>
      
      <Suspense fallback={<ChartSkeleton />}>
        <RevenueChart />    {/* Fetches own data */}
      </Suspense>
      
      <Suspense fallback={<TableSkeleton rows={10} />}>
        <RecentOrders />    {/* Fetches own data */}
      </Suspense>
    </div>
  )
}
// Result: Page shows skeleton immediately, each section pops in as ready
// No waterfall, no blocking, no unified loading state
```

---

## 🔧 WEB WORKERS (Offload Heavy Computation)

```typescript
// worker.ts (separate file, runs in background thread)
self.addEventListener('message', (event) => {
  const { data, type } = event.data
  
  switch (type) {
    case 'PROCESS_CSV': {
      const result = processLargeCSV(data) // Won't block UI
      self.postMessage({ type: 'CSV_RESULT', data: result })
      break
    }
    case 'CALCULATE_STATS': {
      const stats = calculateComplexStats(data)
      self.postMessage({ type: 'STATS_RESULT', data: stats })
      break
    }
  }
})

// React hook to use Web Worker
function useWorker<TInput, TOutput>(workerUrl: string) {
  const workerRef = useRef<Worker | null>(null)
  
  useEffect(() => {
    workerRef.current = new Worker(workerUrl, { type: 'module' })
    return () => workerRef.current?.terminate()
  }, [workerUrl])
  
  const process = useCallback((input: TInput): Promise<TOutput> => {
    return new Promise((resolve, reject) => {
      if (!workerRef.current) return reject(new Error('Worker not ready'))
      
      const handler = (e: MessageEvent) => {
        workerRef.current?.removeEventListener('message', handler)
        resolve(e.data)
      }
      
      workerRef.current.addEventListener('message', handler)
      workerRef.current.postMessage(input)
    })
  }, [])
  
  return { process }
}
```

---

## 📊 ADVANCED MONITORING

### Custom Performance Marks
```typescript
// Track user-perceived performance of key flows
function trackPerformance(name: string) {
  const start = performance.mark(`${name}-start`)
  
  return {
    end() {
      performance.mark(`${name}-end`)
      const measure = performance.measure(name, `${name}-start`, `${name}-end`)
      
      // Send to analytics
      if (window.posthog) {
        posthog.capture('performance_metric', {
          name,
          duration: measure.duration,
          page: window.location.pathname
        })
      }
    }
  }
}

// Usage
const perf = trackPerformance('checkout_flow')
await processCheckout()
perf.end() // Tracks exactly how long checkout takes for real users
```

### Real User Monitoring (RUM)
```typescript
// Capture Core Web Vitals automatically
import { onCLS, onINP, onLCP, onFCP, onTTFB } from 'web-vitals'

function sendToAnalytics({ name, value, rating }: Metric) {
  posthog.capture('web_vital', {
    metric: name,
    value: Math.round(name === 'CLS' ? value * 1000 : value),
    rating, // 'good' | 'needs-improvement' | 'poor'
    page: window.location.pathname
  })
}

onCLS(sendToAnalytics)
onINP(sendToAnalytics)
onLCP(sendToAnalytics)
onFCP(sendToAnalytics)
onTTFB(sendToAnalytics)
```
