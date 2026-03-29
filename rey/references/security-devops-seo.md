# REY — Security, DevOps & SEO Guide

---

## 🔒 PART 1: SECURITY HARDENING

### Security Headers (Next.js)
```typescript
// next.config.ts
const securityHeaders = [
  { key: "X-DNS-Prefetch-Control", value: "on" },
  { key: "X-Frame-Options", value: "SAMEORIGIN" },
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  { key: "Permissions-Policy", value: "camera=(), microphone=(), geolocation=(), browsing-topics=()" },
  {
    key: "Strict-Transport-Security",
    value: "max-age=63072000; includeSubDomains; preload",
  },
  {
    key: "Content-Security-Policy",
    value: [
      "default-src 'self'",
      "script-src 'self' 'unsafe-eval' 'unsafe-inline' https://clerk.myapp.com",
      "style-src 'self' 'unsafe-inline' https://fonts.googleapis.com",
      "font-src 'self' https://fonts.gstatic.com",
      "img-src 'self' data: blob: https://img.clerk.com https://uploadthing.com",
      "connect-src 'self' https://*.supabase.co https://clerk.myapp.com",
      "frame-src 'self' https://js.stripe.com",
      "object-src 'none'",
      "base-uri 'self'",
      "form-action 'self'",
    ].join("; "),
  },
]

const nextConfig: NextConfig = {
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: securityHeaders,
      },
    ]
  },
}
```

### Input Validation (API Routes)
```typescript
// lib/api-helpers.ts
import { z } from "zod"
import { NextResponse } from "next/server"

export function validateRequest<T>(schema: z.ZodSchema<T>, data: unknown):
  | { success: true; data: T }
  | { success: false; response: NextResponse } {
  const result = schema.safeParse(data)

  if (!result.success) {
    return {
      success: false,
      response: NextResponse.json(
        {
          success: false,
          error: {
            code: "VALIDATION_ERROR",
            message: "Invalid input",
            details: result.error.errors,
          },
        },
        { status: 422 }
      ),
    }
  }

  return { success: true, data: result.data }
}

// Usage in route handler
export async function POST(req: Request) {
  const body = await req.json()
  const validation = validateRequest(createPostSchema, body)
  if (!validation.success) return validation.response

  const { title, content } = validation.data
  // ... proceed with validated data
}
```

### SQL Injection Prevention
```typescript
// Always use ORM or parameterized queries
// ✅ Safe: Drizzle ORM
const user = await db.select().from(users).where(eq(users.email, userInput))

// ✅ Safe: Parameterized raw SQL
const result = await db.execute(
  sql`SELECT * FROM users WHERE email = ${userInput}`
)

// ❌ NEVER do this:
const result = await db.execute(`SELECT * FROM users WHERE email = '${userInput}'`)
```

### XSS Prevention
```typescript
// Server-side: Sanitize HTML content before storing/rendering
import DOMPurify from "isomorphic-dompurify"

export function sanitizeHTML(html: string): string {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ["b", "i", "em", "strong", "a", "p", "br", "ul", "ol", "li"],
    ALLOWED_ATTR: ["href", "target", "rel"],
    FORCE_BODY: true,
  })
}

// Client-side: Never use dangerouslySetInnerHTML with user content
// ✅ Safe:
<p>{userContent}</p>

// ✅ Safe (if HTML is needed and sanitized):
<div dangerouslySetInnerHTML={{ __html: sanitizeHTML(userContent) }} />

// ❌ Never:
<div dangerouslySetInnerHTML={{ __html: userContent }} />
```

### Auth Guards
```typescript
// lib/guards.ts — Composable auth guards for API routes
import { auth } from "@clerk/nextjs/server"
import { NextResponse } from "next/server"
import { db } from "@/lib/db"
import { users } from "@/lib/db/schema"
import { eq } from "drizzle-orm"

type Handler = (req: Request, context: { user: User; userId: string }) => Promise<Response>

export function withAuth(handler: Handler) {
  return async (req: Request) => {
    const { userId } = await auth()
    if (!userId) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 })
    }

    const user = await db.query.users.findFirst({
      where: eq(users.clerkId, userId),
    })

    if (!user) {
      return NextResponse.json({ error: "User not found" }, { status: 404 })
    }

    return handler(req, { user, userId })
  }
}

export function withAdminAuth(handler: Handler) {
  return withAuth(async (req, context) => {
    if (context.user.role !== "admin") {
      return NextResponse.json({ error: "Forbidden" }, { status: 403 })
    }
    return handler(req, context)
  })
}

// Usage
export const GET = withAuth(async (req, { user }) => {
  return NextResponse.json({ data: user })
})

export const DELETE = withAdminAuth(async (req, { user }) => {
  // Only admins can reach here
})
```

### OWASP Top 10 Checklist
```
A01 — Broken Access Control
  ✅ Row-Level Security in Supabase
  ✅ Auth guards on all protected routes
  ✅ Check resource ownership before CRUD

A02 — Cryptographic Failures
  ✅ HTTPS everywhere (Vercel/Cloudflare auto)
  ✅ Secrets in env vars, not code
  ✅ HSTS header configured

A03 — Injection
  ✅ Drizzle ORM (parameterized queries)
  ✅ Zod validation on all inputs
  ✅ DOMPurify for HTML content

A04 — Insecure Design
  ✅ Rate limiting on auth endpoints
  ✅ Account lockout after failed attempts
  ✅ Separate tokens per use case

A05 — Security Misconfiguration
  ✅ Security headers configured
  ✅ Error messages don't leak internals
  ✅ Debug mode off in production

A06 — Vulnerable Components
  ✅ pnpm audit in CI/CD
  ✅ Dependabot / Renovate enabled
  ✅ Snyk or similar tool

A07 — Auth Failures
  ✅ Clerk handles auth (battle-tested)
  ✅ Short-lived JWT tokens
  ✅ Secure session management

A08 — Software/Data Integrity
  ✅ Webhook signatures verified (Stripe, Clerk)
  ✅ Subresource Integrity for CDN scripts

A09 — Logging/Monitoring Failures
  ✅ Sentry for errors
  ✅ Auth events logged
  ✅ Suspicious activity alerting

A10 — SSRF
  ✅ Validate/allowlist URLs before fetching
  ✅ Don't fetch user-provided URLs server-side
```

---

## 🐋 PART 2: DEVOPS & INFRASTRUCTURE

### Docker Setup
```dockerfile
# Dockerfile (multi-stage build for Next.js)
FROM node:20-alpine AS base

# Dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json pnpm-lock.yaml* ./
RUN corepack enable pnpm && pnpm install --frozen-lockfile

# Build
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED=1
ENV NODE_ENV=production
ARG NEXT_PUBLIC_APP_URL
ENV NEXT_PUBLIC_APP_URL=$NEXT_PUBLIC_APP_URL

RUN corepack enable pnpm && pnpm build

# Production
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml (for local development)
services:
  app:
    build: .
    ports:
      - "3000:3000"
    env_file: .env.local
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### GitHub Actions — Full CI/CD
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

concurrency:
  group: deploy-production
  cancel-in-progress: false # Never cancel deploys mid-flight

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: pnpm

      - run: pnpm install --frozen-lockfile
      - run: pnpm typecheck
      - run: pnpm lint
      - run: pnpm test:run
        env:
          DATABASE_URL: postgresql://postgres:test@localhost:5432/testdb

  deploy:
    name: Deploy to Vercel
    needs: test
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: pnpm

      - run: pnpm install --frozen-lockfile

      - name: Deploy
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

  notify:
    name: Notify
    needs: deploy
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Slack notification
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "${{ needs.deploy.result == 'success' && '✅' || '❌' }} Deploy ${{ needs.deploy.result }} — ${{ github.event.head_commit.message }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Dependabot Config
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: "/"
    schedule:
      interval: weekly
      day: monday
    groups:
      patch-updates:
        applies-to: version-updates
        update-types: ["minor", "patch"]
    ignore:
      - dependency-name: "*"
        update-types: ["version-update:semver-major"]

  - package-ecosystem: github-actions
    directory: "/"
    schedule:
      interval: monthly
```

### Database Migrations in CI
```bash
# Run migrations before deployment
# Add to deploy workflow before "Deploy" step:

- name: Run DB Migrations
  run: pnpm db:migrate
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

---

## 🌐 PART 3: SEO & METADATA

### Next.js Metadata API (App Router)
```typescript
// app/layout.tsx — Root metadata
import type { Metadata, Viewport } from "next"

export const metadata: Metadata = {
  metadataBase: new URL(process.env.NEXT_PUBLIC_APP_URL!),

  title: {
    default: "MyApp — Your App Tagline",
    template: "%s | MyApp",
  },

  description: "Clear, compelling description. Under 160 chars. Include main keyword.",

  keywords: ["keyword1", "keyword2", "keyword3"],

  authors: [{ name: "MyApp Inc.", url: "https://myapp.com" }],

  creator: "MyApp Inc.",

  openGraph: {
    type: "website",
    locale: "en_US",
    url: process.env.NEXT_PUBLIC_APP_URL,
    siteName: "MyApp",
    title: "MyApp — Your App Tagline",
    description: "OG description — can be slightly longer than meta description",
    images: [
      {
        url: "/og-image.png",
        width: 1200,
        height: 630,
        alt: "MyApp Preview",
      },
    ],
  },

  twitter: {
    card: "summary_large_image",
    title: "MyApp — Your App Tagline",
    description: "Twitter card description",
    images: ["/og-image.png"],
    creator: "@yourtwitterhandle",
  },

  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      "max-image-preview": "large",
      "max-snippet": -1,
    },
  },

  verification: {
    google: "your-google-verification-code",
  },
}

export const viewport: Viewport = {
  themeColor: [
    { media: "(prefers-color-scheme: light)", color: "#ffffff" },
    { media: "(prefers-color-scheme: dark)", color: "#0a0a0a" },
  ],
  width: "device-width",
  initialScale: 1,
}
```

### Dynamic OG Images
```typescript
// app/api/og/route.tsx
import { ImageResponse } from "@vercel/og"
import { NextRequest } from "next/server"

export const runtime = "edge"

export async function GET(req: NextRequest) {
  const { searchParams } = new URL(req.url)
  const title = searchParams.get("title") ?? "MyApp"
  const description = searchParams.get("description") ?? ""

  return new ImageResponse(
    (
      <div
        style={{
          height: "100%",
          width: "100%",
          display: "flex",
          flexDirection: "column",
          alignItems: "flex-start",
          justifyContent: "flex-end",
          backgroundImage: "linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%)",
          padding: 60,
        }}
      >
        <div style={{ display: "flex", alignItems: "center", marginBottom: 24 }}>
          <div style={{
            width: 48, height: 48, borderRadius: 12,
            background: "#3b82f6", display: "flex",
            alignItems: "center", justifyContent: "center",
            marginRight: 12,
          }}>
            <span style={{ color: "white", fontSize: 24, fontWeight: 700 }}>M</span>
          </div>
          <span style={{ color: "#94a3b8", fontSize: 20 }}>MyApp</span>
        </div>

        <div style={{ color: "white", fontSize: 52, fontWeight: 700, lineHeight: 1.1, marginBottom: 16 }}>
          {title}
        </div>

        {description && (
          <div style={{ color: "#94a3b8", fontSize: 24, lineHeight: 1.4 }}>
            {description}
          </div>
        )}
      </div>
    ),
    {
      width: 1200,
      height: 630,
    }
  )
}

// Usage in metadata:
// openGraph: { images: [`/api/og?title=${encodeURIComponent(post.title)}`] }
```

### Sitemap
```typescript
// app/sitemap.ts
import { MetadataRoute } from "next"
import { db } from "@/lib/db"
import { posts } from "@/lib/db/schema"
import { eq } from "drizzle-orm"

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = process.env.NEXT_PUBLIC_APP_URL!

  // Static routes
  const staticRoutes: MetadataRoute.Sitemap = [
    { url: baseUrl, lastModified: new Date(), changeFrequency: "daily", priority: 1 },
    { url: `${baseUrl}/features`, lastModified: new Date(), changeFrequency: "monthly", priority: 0.8 },
    { url: `${baseUrl}/pricing`, lastModified: new Date(), changeFrequency: "weekly", priority: 0.9 },
    { url: `${baseUrl}/blog`, lastModified: new Date(), changeFrequency: "daily", priority: 0.7 },
  ]

  // Dynamic routes (blog posts)
  const publishedPosts = await db.select({
    slug: posts.slug,
    updatedAt: posts.updatedAt,
  }).from(posts).where(eq(posts.status, "published"))

  const dynamicRoutes: MetadataRoute.Sitemap = publishedPosts.map(post => ({
    url: `${baseUrl}/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: "monthly",
    priority: 0.6,
  }))

  return [...staticRoutes, ...dynamicRoutes]
}
```

### Structured Data (JSON-LD)
```typescript
// components/common/structured-data.tsx
export function OrganizationSchema() {
  const schema = {
    "@context": "https://schema.org",
    "@type": "Organization",
    name: "MyApp Inc.",
    url: process.env.NEXT_PUBLIC_APP_URL,
    logo: `${process.env.NEXT_PUBLIC_APP_URL}/logo.png`,
    contactPoint: {
      "@type": "ContactPoint",
      email: "support@myapp.com",
      contactType: "customer support",
    },
    sameAs: [
      "https://twitter.com/myapp",
      "https://github.com/myapp",
    ],
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  )
}

export function ArticleSchema({ post }: { post: Post }) {
  const schema = {
    "@context": "https://schema.org",
    "@type": "Article",
    headline: post.title,
    description: post.excerpt,
    image: post.coverImage,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      "@type": "Person",
      name: post.author.name,
      url: `${process.env.NEXT_PUBLIC_APP_URL}/authors/${post.author.slug}`,
    },
    publisher: {
      "@type": "Organization",
      name: "MyApp",
      logo: { "@type": "ImageObject", url: `${process.env.NEXT_PUBLIC_APP_URL}/logo.png` },
    },
  }

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  )
}
```

---

## 🌍 PART 4: INTERNATIONALIZATION (i18n)

### next-intl Setup
```bash
pnpm add next-intl
```

```
messages/
├── en.json
├── id.json
└── es.json
```

```json
// messages/en.json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "loading": "Loading...",
    "error": "Something went wrong",
    "success": "Success!"
  },
  "auth": {
    "signIn": "Sign In",
    "signUp": "Sign Up",
    "signOut": "Sign Out",
    "email": "Email address",
    "password": "Password"
  },
  "dashboard": {
    "title": "Dashboard",
    "welcome": "Welcome back, {name}!",
    "stats": {
      "revenue": "Total Revenue",
      "users": "Active Users"
    }
  }
}
```

```typescript
// i18n/routing.ts
import { defineRouting } from "next-intl/routing"

export const routing = defineRouting({
  locales: ["en", "id", "es"],
  defaultLocale: "en",
  localePrefix: "as-needed", // English = /, Indonesian = /id/
})
```

```typescript
// middleware.ts — update for i18n
import createMiddleware from "next-intl/middleware"
import { clerkMiddleware, createRouteMatcher } from "@clerk/nextjs/server"
import { routing } from "@/i18n/routing"

const handleI18nRouting = createMiddleware(routing)
const isPublicRoute = createRouteMatcher(["/", "/sign-in(.*)", "/sign-up(.*)", "/api/(.*)"])

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) await auth.protect()
  return handleI18nRouting(req)
})
```

```typescript
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from "next-intl"
import { getMessages } from "next-intl/server"
import { notFound } from "next/navigation"
import { routing } from "@/i18n/routing"

export default async function LocaleLayout({
  children,
  params: { locale },
}: {
  children: React.ReactNode
  params: { locale: string }
}) {
  if (!routing.locales.includes(locale as any)) notFound()

  const messages = await getMessages()

  return (
    <NextIntlClientProvider messages={messages}>
      {children}
    </NextIntlClientProvider>
  )
}
```

```tsx
// Usage in components
import { useTranslations } from "next-intl"

function Dashboard() {
  const t = useTranslations("dashboard")
  const { user } = useUser()

  return (
    <div>
      <h1>{t("title")}</h1>
      <p>{t("welcome", { name: user?.firstName })}</p>
    </div>
  )
}

// Server component
import { getTranslations } from "next-intl/server"

async function ServerPage() {
  const t = await getTranslations("common")
  return <button>{t("save")}</button>
}
```
