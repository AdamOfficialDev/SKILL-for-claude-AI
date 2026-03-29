# REY — Project Scaffolding Templates

> Copy. Customize. Ship. Jangan mulai dari nol.

---

## 🚀 TEMPLATE 1: NEXT.JS SAAS STARTER

### Init Command
```bash
# Buat project baru
pnpm create next-app@latest my-app --typescript --tailwind --app --src-dir --import-alias "@/*"
cd my-app

# Core dependencies
pnpm add drizzle-orm postgres drizzle-kit
pnpm add @clerk/nextjs
pnpm add uploadthing @uploadthing/react
pnpm add resend react-email
pnpm add @upstash/redis @upstash/ratelimit
pnpm add zod
pnpm add zustand
pnpm add @tanstack/react-query @tanstack/react-query-devtools
pnpm add framer-motion
pnpm add lucide-react

# shadcn/ui setup
pnpm dlx shadcn@latest init
pnpm dlx shadcn@latest add button input label card dialog sheet toast sonner

# Dev dependencies
pnpm add -D @types/node vitest @vitejs/plugin-react jsdom
pnpm add -D prettier prettier-plugin-tailwindcss
pnpm add -D @playwright/test
pnpm add -D husky lint-staged @commitlint/cli @commitlint/config-conventional
pnpm add -D @next/bundle-analyzer
```

### package.json scripts
```json
{
  "scripts": {
    "dev": "next dev --turbo",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit",
    "format": "prettier --write .",
    "test": "vitest",
    "test:e2e": "playwright test",
    "test:coverage": "vitest run --coverage",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate",
    "db:studio": "drizzle-kit studio",
    "analyze": "ANALYZE=true next build",
    "prepare": "husky"
  }
}
```

### tsconfig.json (strict)
```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": { "@/*": ["./src/*"] }
  }
}
```

### next.config.ts
```typescript
import type { NextConfig } from 'next'

const config: NextConfig = {
  experimental: {
    typedRoutes: true,
  },
  images: {
    remotePatterns: [
      { hostname: 'utfs.io' },       // UploadThing
      { hostname: '*.supabase.co' }, // Supabase Storage
    ],
  },
}

export default config
```

### src/lib/db/schema.ts (Drizzle baseline)
```typescript
import { pgTable, uuid, text, timestamp, pgEnum, boolean, jsonb } from 'drizzle-orm/pg-core'

export const roleEnum = pgEnum('role', ['user', 'admin', 'superadmin'])
export const planEnum = pgEnum('plan', ['free', 'pro', 'enterprise'])

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  clerkId: text('clerk_id').unique().notNull(),
  email: text('email').unique().notNull(),
  name: text('name').notNull(),
  avatarUrl: text('avatar_url'),
  role: roleEnum('role').default('user').notNull(),
  plan: planEnum('plan').default('free').notNull(),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
  deletedAt: timestamp('deleted_at'),
})

export type User = typeof users.$inferSelect
export type NewUser = typeof users.$inferInsert
```

### src/middleware.ts (Clerk auth)
```typescript
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server'

const isPublicRoute = createRouteMatcher([
  '/',
  '/sign-in(.*)',
  '/sign-up(.*)',
  '/api/webhooks(.*)',
])

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) {
    await auth.protect()
  }
})

export const config = {
  matcher: ['/((?!.+\\.[\\w]+$|_next).*)', '/', '/(api|trpc)(.*)'],
}
```

---

## 🏗️ TEMPLATE 2: LANDING PAGE (ASTRO)

```bash
pnpm create astro@latest my-landing -- --template minimal --typescript strict
cd my-landing

pnpm add @astrojs/tailwind tailwindcss
pnpm add @astrojs/react react react-dom @types/react @types/react-dom
pnpm add astro-icon
```

### Key Astro pages structure
```
src/
├── pages/
│   ├── index.astro      # Landing page
│   └── api/
│       └── contact.ts   # Contact form endpoint
├── components/
│   ├── Hero.astro
│   ├── Features.astro
│   ├── Pricing.astro
│   ├── Testimonials.astro
│   └── CTA.astro
├── layouts/
│   └── Layout.astro     # Base layout with SEO
└── styles/
    └── global.css
```

---

## 📱 TEMPLATE 3: EXPO MOBILE APP

```bash
npx create-expo-app@latest my-app --template
cd my-app

npx expo install expo-router expo-constants expo-status-bar
npx expo install @clerk/clerk-expo
npx expo install @tanstack/react-query
npx expo install zustand
npx expo install react-native-reanimated react-native-gesture-handler
```

---

## 🔧 TEMPLATE 4: HONO API SERVICE

```bash
mkdir my-api && cd my-api
pnpm init
pnpm add hono @hono/node-server
pnpm add drizzle-orm postgres
pnpm add zod @hono/zod-validator
pnpm add @upstash/redis @upstash/ratelimit
pnpm add -D typescript tsx @types/node drizzle-kit
```

```typescript
// src/index.ts
import { Hono } from 'hono'
import { cors } from 'hono/cors'
import { logger } from 'hono/logger'
import { prettyJSON } from 'hono/pretty-json'
import { serve } from '@hono/node-server'

const app = new Hono()

app.use('*', cors({ origin: process.env.ALLOWED_ORIGINS?.split(',') || '*' }))
app.use('*', logger())
app.use('*', prettyJSON())

// Health check
app.get('/health', (c) => c.json({ status: 'ok', timestamp: new Date().toISOString() }))

// Routes
app.route('/api/v1/users', usersRouter)
app.route('/api/v1/posts', postsRouter)

// 404 handler
app.notFound((c) => c.json({ error: 'Not found' }, 404))

// Error handler
app.onError((err, c) => {
  console.error(err)
  return c.json({ error: 'Internal server error' }, 500)
})

serve({ fetch: app.fetch, port: Number(process.env.PORT) || 3001 }, (info) => {
  console.log(`🚀 Server running on http://localhost:${info.port}`)
})
```

---

## 🎨 TEMPLATE 5: DESIGN SYSTEM COMPONENT

### Premium Card Component
```tsx
// components/ui/premium-card.tsx
import { cn } from '@/lib/utils'

interface PremiumCardProps extends React.HTMLAttributes<HTMLDivElement> {
  variant?: 'default' | 'elevated' | 'glass' | 'bordered'
  padding?: 'sm' | 'md' | 'lg'
}

const paddingMap = {
  sm: 'p-4',
  md: 'p-6',
  lg: 'p-8',
}

const variantMap = {
  default: 'bg-card text-card-foreground shadow-sm',
  elevated: 'bg-card text-card-foreground shadow-lg hover:shadow-xl transition-shadow duration-300',
  glass: 'bg-white/10 backdrop-blur-md border border-white/20 text-white',
  bordered: 'bg-transparent border-2 border-border hover:border-primary/50 transition-colors',
}

export function PremiumCard({ 
  className, 
  variant = 'default',
  padding = 'md',
  children,
  ...props 
}: PremiumCardProps) {
  return (
    <div
      className={cn(
        'rounded-xl',
        variantMap[variant],
        paddingMap[padding],
        className
      )}
      {...props}
    >
      {children}
    </div>
  )
}
```

### Skeleton Loading Component
```tsx
// components/ui/skeleton.tsx
import { cn } from '@/lib/utils'

function Skeleton({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div
      className={cn(
        'animate-pulse rounded-md bg-muted',
        className
      )}
      {...props}
    />
  )
}

// Usage examples
function CardSkeleton() {
  return (
    <div className="p-6 space-y-4">
      <Skeleton className="h-4 w-3/4" />
      <Skeleton className="h-4 w-1/2" />
      <div className="space-y-2">
        <Skeleton className="h-3 w-full" />
        <Skeleton className="h-3 w-full" />
        <Skeleton className="h-3 w-4/5" />
      </div>
    </div>
  )
}
```

### Empty State Component
```tsx
// components/ui/empty-state.tsx
import { cn } from '@/lib/utils'
import { LucideIcon } from 'lucide-react'
import { Button } from './button'

interface EmptyStateProps {
  icon?: LucideIcon
  title: string
  description?: string
  action?: {
    label: string
    onClick: () => void
  }
  className?: string
}

export function EmptyState({ 
  icon: Icon, 
  title, 
  description, 
  action, 
  className 
}: EmptyStateProps) {
  return (
    <div className={cn(
      'flex flex-col items-center justify-center py-16 px-4 text-center',
      className
    )}>
      {Icon && (
        <div className="mb-4 rounded-full bg-muted p-4">
          <Icon className="h-8 w-8 text-muted-foreground" />
        </div>
      )}
      <h3 className="text-lg font-semibold text-foreground">{title}</h3>
      {description && (
        <p className="mt-2 text-sm text-muted-foreground max-w-sm">{description}</p>
      )}
      {action && (
        <Button className="mt-6" onClick={action.onClick}>
          {action.label}
        </Button>
      )}
    </div>
  )
}
```

---

## 📊 TEMPLATE 6: API RESPONSE STANDARDS

```typescript
// lib/api-response.ts
type ApiSuccess<T> = {
  success: true
  data: T
  meta?: {
    total?: number
    page?: number
    perPage?: number
    totalPages?: number
  }
}

type ApiError = {
  success: false
  error: {
    code: string
    message: string
    details?: unknown
  }
}

export type ApiResponse<T> = ApiSuccess<T> | ApiError

// Helper functions
export const success = <T>(data: T, meta?: ApiSuccess<T>['meta']): ApiSuccess<T> => ({
  success: true,
  data,
  ...(meta && { meta }),
})

export const error = (code: string, message: string, details?: unknown): ApiError => ({
  success: false,
  error: { code, message, ...(details && { details }) },
})

// Standard error codes
export const ErrorCodes = {
  UNAUTHORIZED: 'UNAUTHORIZED',
  FORBIDDEN: 'FORBIDDEN',
  NOT_FOUND: 'NOT_FOUND',
  VALIDATION_ERROR: 'VALIDATION_ERROR',
  CONFLICT: 'CONFLICT',
  RATE_LIMITED: 'RATE_LIMITED',
  INTERNAL_ERROR: 'INTERNAL_ERROR',
} as const
```
