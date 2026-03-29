# REY — Library Arsenal Reference

> Library yang tepat bisa hemat minggu development. Library yang salah bisa jadi technical debt bertahun-tahun.
> REY hanya recommend library yang: battle-tested, actively maintained, TypeScript-first, dan production-proven.

---

## ⚡ TIER SYSTEM

```
TIER S — REY install ini di hampir setiap project
TIER A — Highly recommended untuk use case spesifik
TIER B — Good option, tapi ada alternatif
TIER C — Niche / situational — hanya jika benar-benar butuh
```

---

## 🎨 UI COMPONENTS & PRIMITIVES

### Headless Component Libraries (Unstyled — Lu yang kontrol style)

| Library | Tier | Install | Why |
|---------|------|---------|-----|
| **Radix UI** | S | `@radix-ui/react-*` | Best accessibility, composable, battle-tested oleh Linear, Vercel, dll |
| **shadcn/ui** | S | `npx shadcn@latest add` | Copy-paste ownership, built on Radix, zero vendor lock-in |
| **Headless UI** | A | `@headlessui/react` | Tailwind team, simpler API tapi less comprehensive dari Radix |
| **React Aria** | A | `react-aria-components` | Adobe, extreme a11y focus, lebih verbose tapi most complete |
| **Ark UI** | B | `@ark-ui/react` | Newer, Chakra team, growing fast |

**REY Default:** shadcn/ui (karena copy-paste = owned code, no upgrade headaches)

### Specific UI Components (Best-in-class per category)

| Category | Library | Tier | Notes |
|----------|---------|------|-------|
| **Command Menu** | **cmdk** | S | `cmdk` — ⌘K palette, used by Linear, Vercel. Pair dengan shadcn |
| **Toast/Notifications** | **Sonner** | S | `sonner` — by Emile (shadcn author), beautiful defaults, stacking |
| **Drawer/Sheet** | **Vaul** | S | `vaul` — mobile-first drawer, snap points, used by shadcn sheet |
| **Dialog/Modal** | Radix Dialog | S | Built into shadcn, best a11y |
| **Tooltip** | **Floating UI** | S | `@floating-ui/react` — powers most tooltip libs, most accurate positioning |
| **Date Picker** | **React DayPicker** | A | `react-day-picker` — accessible, composable, shadcn calendar uses this |
| **Color Picker** | `react-colorful` | A | Tiny (2.8KB), no deps, tree-shakeable |
| **Combobox/Select** | Radix Select + cmdk | S | Combine both for searchable select |
| **Multi-select** | `react-select` | B | Heavy but feature-rich, atau build dengan cmdk |
| **Phone Input** | `react-phone-number-input` | A | International format, flag icons |
| **OTP Input** | `input-otp` | A | `input-otp` — by shadcn, accessible, copy-paste ready |
| **Stepper** | Custom atau `stepperize` | B | Build sendiri dengan shadcn lebih flexible |
| **Resizable Panels** | `react-resizable-panels` | A | `react-resizable-panels` — used by shadcn, smooth DX |
| **Carousel** | `embla-carousel-react` | A | Best performance, touch-friendly, shadcn carousel uses this |
| **Virtualized List** | **TanStack Virtual** | S | `@tanstack/react-virtual` — render 1M+ rows performantly |

---

## 📝 FORMS

| Library | Tier | Why REY Recommends |
|---------|------|-------------------|
| **React Hook Form** | S | `react-hook-form` — Best DX, uncontrolled, minimal re-renders. Pair dengan Zod. |
| **Zod** | S | `zod` — Schema validation, type inference, REY default validator |
| **@hookform/resolvers** | S | `@hookform/resolvers` — Connect RHF dengan Zod/Yup/etc |
| **Conform** | A | `conform-to` — Server Action-native forms untuk Next.js App Router. Lebih seamless dengan Server Actions |
| **Valibot** | B | Alternative Zod, smaller bundle tapi ecosystem lebih kecil |

**REY Decision:**
```
Client-side heavy form → React Hook Form + Zod + @hookform/resolvers
Server Actions form    → Conform + Zod (native progressive enhancement)
Simple form            → React Hook Form saja, tanpa library tambahan
```

### Form UX Libraries
| Library | Use Case |
|---------|---------|
| `@tanstack/react-form` | Alternative RHF, lebih type-safe tapi newer |
| `nuqs` | Sync form/filter state ke URL params (lihat State section) |

---

## 📊 DATA TABLES

| Library | Tier | Notes |
|---------|------|-------|
| **TanStack Table v8** | S | `@tanstack/react-table` — Headless, 100% customizable, sorting/filtering/pagination built-in. REY default. |
| **AG Grid Community** | A | Feature-rich, Excel-like, good for enterprise dashboards. Free tier ada. |
| **React Data Grid** | B | Adazzle, good performance |
| **shadcn Data Table** | S | Build on top TanStack Table, shadcn styling — start here |

**TanStack Table minimal setup:**
```tsx
import {
  flexRender,
  getCoreRowModel,
  getSortedRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  useReactTable,
} from '@tanstack/react-table'
```

---

## 🔄 DATA FETCHING & STATE

### Server State (API data)
| Library | Tier | Why |
|---------|------|-----|
| **TanStack Query v5** | S | `@tanstack/react-query` — caching, background sync, optimistic updates, devtools. REY #1 choice. |
| **SWR** | A | `swr` — Vercel's library, simpler API, good for Next.js. Lighter than TQ. |
| **tRPC** | S | `trpc` — End-to-end type-safe APIs. Pair dengan Next.js + Drizzle untuk full-stack TS. |

**REY Decision:**
```
Next.js full-stack, same repo   → tRPC + TanStack Query
Separate frontend/backend        → TanStack Query + REST/GraphQL
Simple project                  → SWR (less boilerplate)
RSC-heavy (Next.js App Router)  → Server Components + minimal client fetching
```

### Client State (UI State)
| Library | Tier | Best For |
|---------|------|---------|
| **Zustand** | S | `zustand` — Simple global state. 1KB, no boilerplate. REY default for global state. |
| **Jotai** | S | `jotai` — Atomic state model. Better for: many small pieces of state that can compose. |
| **Valtio** | B | `valtio` — Proxy-based, mutable style. Good for complex nested state. |
| **React Context** | A | Built-in. Good for: theme, auth user, locale. NOT for frequent updates. |
| **XState** | B | `xstate` — State machines. Overkill untuk kebanyakan kasus, tapi perfect untuk complex flows (wizard, multi-step) |

**REY Decision:**
```
Global UI state (sidebar open, theme, user prefs)  → Zustand
Many independent atoms (filters, selections)         → Jotai  
Complex wizard/multi-step flow                       → XState atau Zustand dengan explicit states
Per-component state                                  → useState / useReducer
```

### URL State
| Library | Tier | Why |
|---------|------|-----|
| **nuqs** | S | `nuqs` — Type-safe URL search params. Filter, pagination, tabs, sort dalam URL. REY wajib install. |

```tsx
// nuqs usage — filter state in URL
import { useQueryState, parseAsInteger, parseAsString } from 'nuqs'

function ProductsPage() {
  const [page, setPage] = useQueryState('page', parseAsInteger.withDefault(1))
  const [category, setCategory] = useQueryState('category', parseAsString)
  const [sort, setSort] = useQueryState('sort', parseAsString.withDefault('newest'))
  
  // URL: /products?page=2&category=shoes&sort=price-asc
  // Shareable, bookmarkable, back-button works!
}
```

---

## 🎬 ANIMATION & MOTION

| Library | Tier | Best For |
|---------|------|---------|
| **Motion (Framer Motion v11)** | S | `motion` — React animations, gestures, layout animations, exit animations. REY #1. |
| **GSAP** | A | `gsap` — Complex timeline animations, ScrollTrigger, morphing. When Motion isn't enough. |
| **Auto Animate** | A | `@formkit/auto-animate` — Add one line, get automatic list/show/hide animations. Magic. |
| **React Spring** | B | `react-spring` — Physics-based, more flexible than Motion but worse DX |
| **Lottie** | A | `lottie-react` — Play After Effects animations. Good for illustrations/empty states |
| **Rive** | A | `@rive-app/react-canvas` — Interactive animations, better than Lottie for interactive |
| **Three.js** | B | `three` + `@react-three/fiber` + `@react-three/drei` — 3D, WebGL, immersive |
| **Spline** | B | `@splinetool/react-spline` — No-code 3D scenes, embed Spline designs |

**Auto Animate — one of REY's favorite hidden gems:**
```tsx
import { useAutoAnimate } from '@formkit/auto-animate/react'

function List({ items }: { items: string[] }) {
  const [parent] = useAutoAnimate()
  return (
    <ul ref={parent}>
      {items.map(item => <li key={item}>{item}</li>)}
    </ul>
  )
  // Adding/removing/reordering items = automatic smooth animation. Zero config.
}
```

**View Transitions API (Native browser, no library needed):**
```tsx
// next.config.ts — enable experimental
experimental: { viewTransition: true }

// Usage in components
import { unstable_ViewTransition as ViewTransition } from 'react'

<ViewTransition name="hero-image">
  <img src={product.image} />
</ViewTransition>
// Navigating to detail page = smooth image morphing animation
```

---

## 📐 LAYOUT & DRAG-AND-DROP

| Library | Tier | Notes |
|---------|------|-------|
| **dnd-kit** | S | `@dnd-kit/core` + `@dnd-kit/sortable` — Best DnD for React. Accessible, performant. Kanban, sortable lists. |
| **react-beautiful-dnd** | C | Deprecated by Atlassian. DON'T use. |
| **react-dropzone** | A | `react-dropzone` — File drag-and-drop upload zone. |
| **react-grid-layout** | B | Resizable + draggable grid (dashboard widgets) |

**dnd-kit minimal kanban:**
```tsx
import { DndContext, closestCenter } from '@dnd-kit/core'
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable'
```

---

## 📝 RICH TEXT & CONTENT

| Library | Tier | Use Case |
|---------|------|---------|
| **Tiptap** | S | `@tiptap/react` — Extensible, ProseMirror-based. Best overall rich text editor. Free + Pro. |
| **Plate** | A | `@udecode/plate` — Plugin-based, shadcn-compatible, Notion-like. |
| **Lexical** | A | `lexical` — Facebook/Meta, performant, but more setup required |
| **Milkdown** | B | Plugin-based, WYSIWYG markdown |
| **MDX** | S | For content sites, documentation — not an editor, a format |
| **react-markdown** | S | `react-markdown` — Render markdown safely. Essential for AI chat, docs. |

---

## 📈 CHARTS & DATA VISUALIZATION

| Library | Tier | Best For |
|---------|------|---------|
| **Recharts** | S | `recharts` — React-native, SVG-based, composable. REY default for most charts. |
| **Tremor** | A | `@tremor/react` — Pre-built dashboard components, shadcn-compatible. Fast to build dashboards. |
| **Chart.js + react-chartjs-2** | A | Canvas-based, more chart types, better performance for large datasets |
| **Nivo** | A | `@nivo/core` — Beautiful defaults, great for complex visualizations |
| **Visx** | B | Airbnb, low-level D3 + React, steep learning curve |
| **D3.js** | B | The foundation. Use only if custom viz that no library supports |
| **Observable Plot** | B | Great for exploratory data analysis |

**REY Chart Decision:**
```
Standard dashboard charts (bar, line, area, pie)  → Recharts
Pre-built dashboard components fast               → Tremor
Custom/complex visualization                      → Nivo atau D3
Large dataset (10k+ points)                       → Chart.js (canvas > SVG at scale)
```

---

## 🗓️ DATE & TIME

| Library | Tier | Notes |
|---------|------|-------|
| **date-fns** | S | `date-fns` — Tree-shakeable, immutable, pure functions. REY default. |
| **Temporal API** | A | Native browser API (Polyfill: `@js-temporal/polyfill`). The future. |
| **Day.js** | A | `dayjs` — Moment.js replacement, 2KB, plugin-based |
| **Luxon** | B | `luxon` — Moment.js successor from same author, more complete |

**REY: Avoid Moment.js** — 67KB, mutable, deprecated-ish.

---

## 🔠 UTILITY LIBRARIES

| Library | Tier | Notes |
|---------|------|-------|
| **clsx** | S | `clsx` — Merge classNames conditionally. Always use. |
| **tailwind-merge** | S | `tailwind-merge` — Merge Tailwind classes without conflicts. Always use. |
| **class-variance-authority** | S | `class-variance-authority` — Build variant-based components (like shadcn does). |
| **cn() helper** | S | `import { clsx } from 'clsx'; import { twMerge } from 'tailwind-merge'` — combine both |
| **lodash-es** | A | `lodash-es` — Utility functions, tree-shakeable ES module version |
| **remeda** | A | `remeda` — TypeScript-first lodash alternative, better types |
| **nanoid** | S | `nanoid` — Tiny unique ID generator. Use instead of uuid for client-side |
| **uuid** | S | `uuid` — Standard UUID generation for server-side |
| **ms** | A | `ms` — Convert time strings: `ms('2 days')` → `172800000` |
| **pretty-bytes** | A | Human-readable file sizes: `prettyBytes(1337)` → `'1.34 kB'` |
| **numeral** | B | Number formatting: currency, percentages, large numbers |

**The cn() helper — install in every project:**
```typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// Usage:
<div className={cn(
  "base-class",
  isActive && "active-class",
  variant === 'danger' && "text-red-500",
  className  // Allow parent override without conflicts
)} />
```

---

## 🛡️ VALIDATION & ERROR HANDLING

| Library | Tier | Notes |
|---------|------|-------|
| **Zod** | S | Schema validation everywhere — forms, API, env vars |
| **@t3-oss/env-nextjs** | S | Type-safe environment variables with Zod. Fail at build if missing. |
| **Sentry** | S | `@sentry/nextjs` — Error tracking, session replay, performance. |
| **neverthrow** | A | `neverthrow` — Result/Either type for functional error handling |

**T3 Env — type-safe env vars:**
```typescript
// env.ts
import { createEnv } from "@t3-oss/env-nextjs"
import { z } from "zod"

export const env = createEnv({
  server: {
    DATABASE_URL: z.string().url(),
    CLERK_SECRET_KEY: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_APP_URL: z.string().url(),
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().min(1),
  },
  runtimeEnv: {
    DATABASE_URL: process.env.DATABASE_URL,
    // ...
  },
})
// Accessing env.DATABASE_URL is type-safe AND validated at startup
```

---

## 🧪 TESTING LIBRARIES

| Library | Tier | Use |
|---------|------|-----|
| **Vitest** | S | `vitest` — Fast unit/integration tests. Jest-compatible API. REY default. |
| **Testing Library** | S | `@testing-library/react` — Test components as users would use them |
| **Playwright** | S | `@playwright/test` — E2E browser tests. REY default for e2e. |
| **MSW** | S | `msw` — Mock Service Worker. Mock APIs in tests AND browser. |
| **Faker.js** | A | `@faker-js/faker` — Generate test data |

---

## 🌐 NETWORKING & API

| Library | Tier | Notes |
|---------|------|-------|
| **Ky** | A | `ky` — Modern fetch wrapper, retry logic, timeout, hooks. Better than axios for modern projects. |
| **Axios** | B | `axios` — Still valid, but heavier than ky. Use if team already familiar. |
| **ofetch** | A | `ofetch` — Nitro/Nuxt team, isomorphic, great DX |
| **tRPC** | S | Full-stack type-safe RPC — mentioned in Data Fetching |

---

## 📸 MEDIA & FILES

| Library | Tier | Use Case |
|---------|------|---------|
| **UploadThing** | S | `uploadthing` — File uploads for Next.js. Handles everything. |
| **react-dropzone** | A | `react-dropzone` — Drag-and-drop upload UX |
| **@vercel/og** | S | `@vercel/og` — Dynamic OG images. Edge-compatible. |
| **sharp** | A | `sharp` — Server-side image processing (Node.js only) |
| **react-zoom-pan-pinch** | B | Image zoom/pan for product images, maps |
| **react-image-crop** | B | `react-image-crop` — Avatar/image cropper |
| **react-pdf** | B | `@react-pdf/renderer` — Generate PDFs in React |

---

## 🎯 USER EXPERIENCE ENHANCEMENTS

| Library | Tier | Notes |
|---------|------|-------|
| **react-hot-toast** | B | Simpler toasts — tapi REY prefer **Sonner** |
| **driver.js** | A | `driver.js` — Product tours, onboarding, feature spotlight. Framework agnostic. |
| **Shepherd.js** | B | Tour library, React wrapper available |
| **react-confetti** | B | `react-confetti` — Celebration effect for key moments |
| **use-sound** | B | `use-sound` — Sound effects for micro-interactions |
| **hotkeys-js** | A | `hotkeys-js` atau `react-hotkeys-hook` — Keyboard shortcut management |
| **react-joyride** | B | User onboarding tours |

---

## 🗺️ MAPS & GEOLOCATION

| Library | Tier | Notes |
|---------|------|-------|
| **react-map-gl** | A | `react-map-gl` — Mapbox GL wrapper, best DX |
| **Leaflet + react-leaflet** | A | Open source, free tiles, lighter |
| **Google Maps React** | B | `@vis.gl/react-google-maps` — If Google ecosystem required |

---

## 🔔 REAL-TIME & WEBSOCKETS

| Library | Tier | Notes |
|---------|------|-------|
| **Supabase Realtime** | S | Built-in if using Supabase. Postgres changes → UI update. |
| **Liveblocks** | A | `@liveblocks/react` — Multiplayer/collaborative features (cursors, presence) |
| **Ably** | A | Managed WebSockets, generous free tier |
| **Socket.io** | A | `socket.io-client` — Self-hosted, bidirectional, rooms |
| **PartyKit** | B | Durable Objects-based, Cloudflare ecosystem |

---

## 📱 MOBILE (React Native / Expo)

| Library | Tier | Notes |
|---------|------|-------|
| **NativeWind** | S | `nativewind` — Tailwind CSS for React Native |
| **React Native Paper** | A | Material Design components for RN |
| **Tamagui** | A | `tamagui` — Unified web + native components |
| **Expo Router** | S | `expo-router` — File-based routing for Expo (like Next.js) |
| **React Native Reanimated** | S | `react-native-reanimated` — Smooth 60fps animations |
| **React Native Gesture Handler** | S | `react-native-gesture-handler` — Native gestures |
| **MMKV** | S | `react-native-mmkv` — Fastest key-value storage for RN |
| **WatermelonDB** | A | Local-first database for RN, good for offline |
| **Zustand** | S | Works identically in RN as web |

---

## 🧩 REY LIBRARY BUNDLES (Pre-configured per project type)

### Bundle: SAAS WEBAPP (Complete)
```bash
# Core
pnpm add next react react-dom typescript

# Styling
pnpm add tailwindcss @tailwindcss/forms @tailwindcss/typography
pnpm add class-variance-authority clsx tailwind-merge

# shadcn/ui (run after above)
pnpm dlx shadcn@latest init

# Components
pnpm add cmdk sonner vaul @floating-ui/react
pnpm add embla-carousel-react react-resizable-panels
pnpm add input-otp react-day-picker

# Forms
pnpm add react-hook-form @hookform/resolvers zod

# State & Data
pnpm add zustand @tanstack/react-query nuqs
pnpm add @trpc/server @trpc/client @trpc/react-query @trpc/next

# Animation
pnpm add motion @formkit/auto-animate

# Tables
pnpm add @tanstack/react-table

# Utils
pnpm add nanoid date-fns
pnpm add @t3-oss/env-nextjs

# Auth
pnpm add @clerk/nextjs

# DB
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit

# Monitoring
pnpm add @sentry/nextjs

# Email
pnpm add resend react-email

# Dev
pnpm add -D vitest @testing-library/react @playwright/test msw
pnpm add -D prettier prettier-plugin-tailwindcss eslint
pnpm add -D husky lint-staged @commitlint/cli @commitlint/config-conventional
```

### Bundle: DASHBOARD / ADMIN
```bash
# Above SaaS bundle PLUS:
pnpm add @tanstack/react-virtual  # Virtual scrolling
pnpm add recharts                 # Charts
pnpm add @dnd-kit/core @dnd-kit/sortable @dnd-kit/utilities  # Drag-and-drop
pnpm add react-dropzone           # File uploads
```

### Bundle: CONTENT / BLOG
```bash
# Minimal Next.js PLUS:
pnpm add @tiptap/react @tiptap/pm @tiptap/starter-kit  # Rich text
pnpm add react-markdown remark-gfm rehype-highlight    # Render markdown
pnpm add @vercel/og               # OG images
pnpm add fuse.js                  # Client-side search
```

### Bundle: E-COMMERCE
```bash
# SaaS bundle PLUS:
pnpm add react-zoom-pan-pinch     # Product image zoom
pnpm add react-image-crop         # Image management
pnpm add stripe @stripe/stripe-js @stripe/react-stripe-js  # Payments
```

### Bundle: LANDING PAGE (Astro)
```bash
pnpm create astro@latest -- --template minimal
pnpm add @astrojs/tailwind @astrojs/react
pnpm add motion                   # Animations
pnpm add @formkit/auto-animate    # Auto animations
```
