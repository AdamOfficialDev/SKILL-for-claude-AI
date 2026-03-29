# REY — Stack Selection Reference

> REY adalah opinionated. Bukan "semua bagus tergantung kebutuhan" — REY punya stance yang jelas, dengan reasoning yang kuat.

---

## 🏆 REY'S OPINIONATED STACK MATRIX

### Frontend

| Use Case | REY Recommends | Why |
|----------|---------------|-----|
| Web App / SaaS | **Next.js 14+ (App Router)** | SSR/SSG/ISR built-in, RSC, file-based routing, Vercel ecosystem |
| Mobile App | **React Native + Expo** | Largest ecosystem, OTA updates, web reuse |
| Dashboard / Admin | **Next.js + shadcn/ui** | Rapid development, accessible, customizable |
| Static Marketing | **Astro** | Zero JS by default, island architecture, blazing fast |
| Interactive Docs | **Nextra / Docusaurus** | MDX support, built-in search |
| E-commerce | **Next.js + Shopify Storefront API** atau **Medusa.js** | Production-tested, flexible |
| Real-time App | **Next.js + Socket.io** atau **Liveblocks** | Simplicity vs scale |
| 3D / WebGL | **Three.js + React Three Fiber** | Best React integration |

### Styling

| Scenario | REY Recommends | Why |
|----------|---------------|-----|
| Default choice | **Tailwind CSS** | Utility-first, purged CSS, design system friendly |
| Component lib | **shadcn/ui** (Radix UI based) | Copy-paste, fully owned, accessible |
| Animation | **Framer Motion** (React) / **GSAP** (complex) | Best DX, production-grade |
| Icons | **Lucide React** | Consistent, tree-shakeable |
| Fonts | **next/font** + Google Fonts atau variable fonts | Zero FOUT, optimized loading |

### Backend

| Use Case | REY Recommends | Why |
|----------|---------------|-----|
| REST API | **Node.js + Hono** atau **Fastify** | Edge-compatible, fast, TypeScript-first |
| Full-stack (simple) | **Next.js API Routes / Route Handlers** | Co-location, less infra |
| Full-stack (complex) | **Node.js + Hono** (separate service) | Decoupled, scalable |
| Python needed | **FastAPI** | Async, auto-docs, Pydantic validation |
| Serverless | **Hono on Cloudflare Workers** | Edge performance, global |
| GraphQL | **Pothos + GraphQL Yoga** | Code-first, type-safe |
| Real-time | **Socket.io** atau **Liveblocks** atau **Ably** | Depends on scale |

### Database

| Use Case | REY Recommends | Why |
|----------|---------------|-----|
| Relational (default) | **PostgreSQL** via **Supabase** | Managed, realtime, auth built-in, free tier |
| Relational (self-host) | **PostgreSQL + Drizzle ORM** | Type-safe, lightweight, SQL-like syntax |
| Alternative ORM | **Prisma** | Better DX for rapid development, auto-migrations |
| NoSQL | **MongoDB Atlas** | Flexible schema, great free tier |
| Key-Value / Cache | **Redis** via **Upstash** | Serverless-compatible, REST API |
| Full-text Search | **Typesense** (self-host) atau **Algolia** (managed) | Better than elasticsearch for most |
| Vector DB | **Pinecone** atau **pgvector** (Supabase) | Depends on existing stack |
| Analytics | **ClickHouse** atau **Tinybird** | OLAP optimized |

### Authentication

| Scenario | REY Recommends | Why |
|----------|---------------|-----|
| Default | **Clerk** | Best DX, UI components included, social + email |
| Open source | **Auth.js (NextAuth v5)** | Framework agnostic, self-hosted |
| Enterprise | **WorkOS** | SSO, SCIM, audit logs |
| If using Supabase | **Supabase Auth** | Native integration |

### File Storage

| REY Recommends | Why |
|---------------|-----|
| **Cloudflare R2** | S3-compatible, no egress fees |
| **Supabase Storage** | If already using Supabase |
| **UploadThing** | Best DX for Next.js, handles everything |

### Email

| REY Recommends | Why |
|---------------|-----|
| **Resend** | Best developer experience, React Email |
| **React Email** | Template builder, beautiful defaults |

### Payments

| REY Recommends | Why |
|---------------|-----|
| **Stripe** | Industry standard, best docs |
| **Lemon Squeezy** | Simpler for digital products, handles VAT |

### Deployment & Infra

| Use Case | REY Recommends | Why |
|----------|---------------|-----|
| Frontend | **Vercel** | Native Next.js, edge network, preview deploys |
| Frontend alt | **Cloudflare Pages** | Free, global edge, R2 integration |
| Backend API | **Railway** | Simple, PostgreSQL included, no cold starts |
| Backend alt | **Render** | Good free tier, easy scaling |
| Containers | **Fly.io** | Docker, persistent volumes, global |
| Serverless Workers | **Cloudflare Workers** | Edge, cheap, fast |
| VPS (full control) | **Hetzner Cloud** + **Coolify** (self-managed PaaS) | Cheap, powerful |

### Monitoring & Observability

| REY Recommends | Why |
|---------------|-----|
| **Sentry** | Error tracking, session replay, performance |
| **PostHog** | Product analytics + feature flags (open source option) |
| **Axiom** | Log management, cheap |
| **Checkly** | Synthetic monitoring |

---

## 🚦 DECISION FLOWCHART

```
START: Jenis project?
│
├─ WEB APP / SAAS
│   └─ Stack: Next.js + Tailwind + shadcn + Supabase + Clerk + Vercel
│      └─ Heavy backend? → Add Railway + Hono
│
├─ MOBILE APP  
│   └─ Stack: React Native + Expo + Supabase + Clerk + EAS Build
│
├─ LANDING PAGE / MARKETING
│   └─ Stack: Astro + Tailwind + Resend → Deploy ke Cloudflare Pages
│
├─ INTERNAL TOOL / ADMIN
│   └─ Stack: Next.js + shadcn + Supabase → Deploy ke Railway/Vercel
│
├─ API-ONLY SERVICE
│   └─ Stack: Hono + Drizzle + PostgreSQL (Railway) → Docker
│
└─ E-COMMERCE
    └─ Custom: Next.js + Medusa.js + Stripe
    └─ Shopify-based: Next.js + Shopify Storefront API
```

---

## ⚖️ TRADE-OFF CHEATSHEET

### Drizzle vs Prisma
- **Drizzle**: Lebih lightweight, SQL-like syntax, better for edge, type inference excellent. REY default untuk production.
- **Prisma**: Better DX untuk rapid dev, richer ecosystem, migrations lebih mature. Pilih ini kalau timnya besar.

### Next.js App Router vs Pages Router
- **App Router**: Default sekarang, RSC, Streaming, better caching. REY selalu pilih ini.
- **Pages Router**: Hanya kalau ada legacy codebase atau library incompatibility.

### Supabase vs Firebase
- **Supabase**: PostgreSQL (relational!), open source, self-hostable, row-level security. REY lebih suka ini.
- **Firebase**: Google ecosystem, Firestore NoSQL, easier auth flow. Pilih kalau sudah existing.

### Vercel vs Cloudflare Pages
- **Vercel**: Terbaik untuk Next.js, preview deploys, analytics. Tapi lebih mahal di scale.
- **Cloudflare Pages**: Gratis bandwidth, global edge, tapi butuh lebih banyak config.

---

## 🛠️ STANDARD TOOLING (Always Recommend)

```json
{
  "language": "TypeScript (strict mode)",
  "linting": "ESLint + @typescript-eslint",
  "formatting": "Prettier",
  "git-hooks": "Husky + lint-staged",
  "commit-convention": "Conventional Commits",
  "testing": "Vitest (unit) + Playwright (e2e)",
  "ci-cd": "GitHub Actions",
  "env-management": "T3 Env (type-safe env vars)",
  "validation": "Zod",
  "state-management": "Zustand (client) / TanStack Query (server)",
  "api-client": "tRPC (full-stack TS) atau Hono RPC",
  "documentation": "Typedoc + Storybook (components)"
}
```

---

## 📋 REY'S OPINIONATED DEFAULTS

Ketika user tidak specify, REY pakai ini:
- **Language**: TypeScript, strict
- **Package Manager**: pnpm
- **Node version**: LTS terbaru (via .nvmrc)
- **CSS**: Tailwind CSS
- **Component**: shadcn/ui
- **Auth**: Clerk (atau Supabase Auth jika Supabase dipakai)
- **DB**: PostgreSQL via Supabase
- **ORM**: Drizzle
- **Deploy**: Vercel (frontend) + Railway (backend)
- **Error tracking**: Sentry
- **Analytics**: PostHog

---

## 🤖 AI/ML INTEGRATION STACK

### LLM Integration
| Use Case | REY Recommends | Why |
|----------|---------------|-----|
| LLM API calls | **Vercel AI SDK** | Streaming, React hooks, multi-provider |
| Provider: Default | **Anthropic Claude** | Best for coding, reasoning, structured output |
| Provider: Fast/Cheap | **Groq (Llama)** | Ultra-fast inference, cheap |
| Provider: Multimodal | **GPT-4o** | Best vision + audio |
| Local LLM | **Ollama** | Privacy, no API cost, dev environment |
| Embeddings | **text-embedding-3-small** (OpenAI) | Best price/perf |
| Vector Search | **pgvector** (Supabase) atau **Pinecone** | SQL-integrated vs managed |

### AI SDK Pattern (Next.js)
```typescript
// app/api/chat/route.ts
import { streamText } from 'ai'
import { anthropic } from '@ai-sdk/anthropic'

export async function POST(req: Request) {
  const { messages } = await req.json()
  
  const result = streamText({
    model: anthropic('claude-opus-4-5'),
    system: 'You are a helpful assistant.',
    messages,
    maxTokens: 1024,
  })
  
  return result.toDataStreamResponse()
}
```

### RAG (Retrieval Augmented Generation) Stack
```
Document Ingestion: LangChain / LlamaIndex / custom
Chunking: tiktoken (token-aware splitting)
Embeddings: text-embedding-3-small (OpenAI)
Vector Store: pgvector (Supabase) / Pinecone
Retrieval: Semantic search + optional keyword (hybrid)
Generation: Claude / GPT-4o with retrieved context
```

---

## 📦 MONOREPO SETUP

### When to use Monorepo
- Multiple apps sharing code (web + mobile + API)
- Shared component library across projects
- Team > 5 engineers with multiple deployables

### REY Monorepo Stack
```bash
# Turborepo (REY default — simpler, faster)
npx create-turbo@latest

# Nx (more features, steeper learning curve)
npx create-nx-workspace
```

### Turborepo Structure
```
monorepo/
├── apps/
│   ├── web/          # Next.js main app
│   ├── docs/         # Astro/Nextra docs
│   ├── admin/        # Next.js admin panel
│   └── api/          # Hono API (if separate service)
├── packages/
│   ├── ui/           # Shared component library
│   ├── db/           # Drizzle schema + queries (shared)
│   ├── auth/         # Auth utilities (shared)
│   ├── config/       # Shared configs (eslint, ts, tailwind)
│   └── types/        # Shared TypeScript types
├── turbo.json
└── package.json
```

---

## 🌐 EDGE COMPUTING PATTERNS

### Cloudflare Workers Stack
```typescript
// worker.ts — runs at edge globally
import { Hono } from 'hono'
import { cors } from 'hono/cors'

const app = new Hono<{ Bindings: CloudflareBindings }>()

app.use('*', cors())

// KV Storage (key-value, globally replicated)
app.get('/config/:key', async (c) => {
  const value = await c.env.KV.get(c.req.param('key'))
  return c.json({ value })
})

// D1 Database (SQLite at edge)
app.get('/users/:id', async (c) => {
  const { results } = await c.env.DB.prepare(
    'SELECT * FROM users WHERE id = ?'
  ).bind(c.req.param('id')).all()
  return c.json(results[0])
})

// R2 Storage (S3-compatible)
app.get('/files/:key', async (c) => {
  const object = await c.env.R2.get(c.req.param('key'))
  if (!object) return c.notFound()
  return new Response(object.body)
})

export default app
```

### Edge + Origin Pattern
```
[Request]
    ↓
[Cloudflare Edge]
    → Cache hit? → Return immediately (< 50ms globally)
    → Auth token? → Verify at edge (no origin roundtrip)
    → Geolocation routing? → Route to nearest origin
    → Bot detection? → Block at edge
    ↓
[Origin Server] (only for cache misses / authenticated requests)
```
