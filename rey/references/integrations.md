# REY — Integrations Guide

> Copy-paste ready integration setups. No more reading docs for hours.

---

## 💳 STRIPE — PAYMENTS

### Installation
```bash
pnpm add stripe @stripe/stripe-js @stripe/react-stripe-js
```

### Server Setup
```typescript
// lib/stripe.ts
import Stripe from "stripe"
import { env } from "@/env"

export const stripe = new Stripe(env.STRIPE_SECRET_KEY, {
  apiVersion: "2024-11-20.acacia",
  typescript: true,
})

// Price IDs from Stripe dashboard
export const PRICES = {
  pro_monthly: "price_xxx",
  pro_yearly: "price_yyy",
  enterprise_monthly: "price_zzz",
} as const

export type PriceId = (typeof PRICES)[keyof typeof PRICES]

// Create or retrieve customer
export async function getOrCreateStripeCustomer(userId: string, email: string) {
  // Check if user already has a Stripe customer ID in DB
  const user = await db.query.users.findFirst({ where: eq(users.id, userId) })

  if (user?.stripeCustomerId) {
    return stripe.customers.retrieve(user.stripeCustomerId)
  }

  const customer = await stripe.customers.create({
    email,
    metadata: { userId },
  })

  await db.update(users)
    .set({ stripeCustomerId: customer.id })
    .where(eq(users.id, userId))

  return customer
}
```

### Checkout Session
```typescript
// app/api/stripe/create-checkout/route.ts
import { NextResponse } from "next/server"
import { auth } from "@clerk/nextjs/server"
import { stripe, getOrCreateStripeCustomer, PRICES } from "@/lib/stripe"
import { env } from "@/env"

export async function POST(req: Request) {
  const { userId } = await auth()
  if (!userId) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const { priceId, successUrl, cancelUrl } = await req.json()

  // Get user from DB
  const user = await db.query.users.findFirst({ where: eq(users.clerkId, userId) })
  if (!user) return NextResponse.json({ error: "User not found" }, { status: 404 })

  const customer = await getOrCreateStripeCustomer(user.id, user.email)

  const session = await stripe.checkout.sessions.create({
    customer: customer.id,
    payment_method_types: ["card"],
    line_items: [{ price: priceId, quantity: 1 }],
    mode: "subscription",
    success_url: successUrl ?? `${env.NEXT_PUBLIC_APP_URL}/dashboard?checkout=success`,
    cancel_url: cancelUrl ?? `${env.NEXT_PUBLIC_APP_URL}/pricing`,
    metadata: { userId: user.id },
    subscription_data: {
      metadata: { userId: user.id },
      trial_period_days: 14, // Free trial
    },
    allow_promotion_codes: true,
    billing_address_collection: "auto",
  })

  return NextResponse.json({ url: session.url })
}
```

### Billing Portal (Manage Subscription)
```typescript
// app/api/stripe/portal/route.ts
export async function POST(req: Request) {
  const { userId } = await auth()
  if (!userId) return NextResponse.json({ error: "Unauthorized" }, { status: 401 })

  const user = await db.query.users.findFirst({ where: eq(users.clerkId, userId) })
  if (!user?.stripeCustomerId) {
    return NextResponse.json({ error: "No billing account found" }, { status: 404 })
  }

  const session = await stripe.billingPortal.sessions.create({
    customer: user.stripeCustomerId,
    return_url: `${env.NEXT_PUBLIC_APP_URL}/settings/billing`,
  })

  return NextResponse.json({ url: session.url })
}
```

### Webhook Handler (Critical — Keep Subscription in Sync)
```typescript
// app/api/webhooks/stripe/route.ts
import { stripe } from "@/lib/stripe"
import { db } from "@/lib/db"
import { users, subscriptions } from "@/lib/db/schema"
import { eq } from "drizzle-orm"

export async function POST(req: Request) {
  const body = await req.text()
  const signature = req.headers.get("stripe-signature")!

  let event: Stripe.Event
  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch {
    return new Response("Invalid signature", { status: 400 })
  }

  switch (event.type) {
    case "checkout.session.completed": {
      const session = event.data.object as Stripe.CheckoutSession
      const userId = session.metadata?.userId

      if (userId && session.subscription) {
        const subscription = await stripe.subscriptions.retrieve(
          session.subscription as string
        )
        await upsertSubscription(userId, subscription)
      }
      break
    }

    case "customer.subscription.updated":
    case "customer.subscription.deleted": {
      const subscription = event.data.object as Stripe.Subscription
      const userId = subscription.metadata?.userId

      if (userId) {
        await upsertSubscription(userId, subscription)

        if (event.type === "customer.subscription.deleted") {
          await db.update(users)
            .set({ plan: "free" })
            .where(eq(users.id, userId))
        }
      }
      break
    }

    case "invoice.payment_failed": {
      const invoice = event.data.object as Stripe.Invoice
      // Send payment failure email
      await sendPaymentFailureEmail(invoice)
      break
    }
  }

  return new Response("OK")
}

async function upsertSubscription(userId: string, subscription: Stripe.Subscription) {
  const priceId = subscription.items.data[0]?.price.id
  const plan = getPlanFromPriceId(priceId)

  await db.update(users)
    .set({ plan, updatedAt: new Date() })
    .where(eq(users.id, userId))

  await db.insert(subscriptions).values({
    userId,
    stripeSubscriptionId: subscription.id,
    stripeCustomerId: subscription.customer as string,
    stripePriceId: priceId ?? "",
    status: subscription.status,
    currentPeriodStart: new Date(subscription.current_period_start * 1000),
    currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    cancelAtPeriodEnd: subscription.cancel_at_period_end,
  }).onConflictDoUpdate({
    target: subscriptions.stripeSubscriptionId,
    set: {
      status: subscription.status,
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      cancelAtPeriodEnd: subscription.cancel_at_period_end,
    },
  })
}
```

### Pricing Component
```tsx
// components/features/billing/pricing-cards.tsx
"use client"

import { useState } from "react"
import { Button } from "@/components/ui/button"
import { PRICES } from "@/lib/stripe"
import { cn } from "@/lib/utils"

const plans = [
  {
    name: "Free",
    price: { monthly: 0, yearly: 0 },
    priceId: { monthly: null, yearly: null },
    features: ["5 projects", "Basic analytics", "Email support"],
    cta: "Get Started",
    popular: false,
  },
  {
    name: "Pro",
    price: { monthly: 29, yearly: 290 },
    priceId: { monthly: PRICES.pro_monthly, yearly: PRICES.pro_yearly },
    features: ["Unlimited projects", "Advanced analytics", "Priority support", "API access", "14-day free trial"],
    cta: "Start Free Trial",
    popular: true,
  },
]

export function PricingCards({ currentPlan }: { currentPlan?: string }) {
  const [billing, setBilling] = useState<"monthly" | "yearly">("monthly")
  const [loading, setLoading] = useState<string | null>(null)

  const handleCheckout = async (priceId: string) => {
    setLoading(priceId)
    try {
      const res = await fetch("/api/stripe/create-checkout", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ priceId }),
      })
      const { url } = await res.json()
      window.location.href = url
    } finally {
      setLoading(null)
    }
  }

  return (
    <div>
      {/* Billing toggle */}
      <div className="flex items-center justify-center gap-4 mb-8">
        <button
          className={cn("text-sm font-medium", billing === "monthly" ? "text-foreground" : "text-muted-foreground")}
          onClick={() => setBilling("monthly")}
        >
          Monthly
        </button>
        <div className="relative inline-flex h-6 w-11 cursor-pointer rounded-full bg-primary/20 transition-colors">
          <span className={cn("absolute top-1 h-4 w-4 rounded-full bg-primary transition-transform", billing === "yearly" ? "translate-x-6" : "translate-x-1")} />
        </div>
        <button
          className={cn("text-sm font-medium", billing === "yearly" ? "text-foreground" : "text-muted-foreground")}
          onClick={() => setBilling("yearly")}
        >
          Yearly <span className="text-green-500 font-semibold">-17%</span>
        </button>
      </div>

      <div className="grid md:grid-cols-2 gap-6 max-w-3xl mx-auto">
        {plans.map((plan) => (
          <div key={plan.name} className={cn(
            "relative rounded-2xl border p-6",
            plan.popular ? "border-primary shadow-lg shadow-primary/10" : "border-border"
          )}>
            {plan.popular && (
              <span className="absolute -top-3 left-1/2 -translate-x-1/2 bg-primary text-primary-foreground text-xs font-bold px-3 py-1 rounded-full">
                Most Popular
              </span>
            )}

            <h3 className="text-xl font-bold">{plan.name}</h3>
            <div className="mt-4 mb-6">
              <span className="text-4xl font-bold">${plan.price[billing]}</span>
              <span className="text-muted-foreground">/{billing === "monthly" ? "mo" : "yr"}</span>
            </div>

            <ul className="space-y-3 mb-6">
              {plan.features.map(feature => (
                <li key={feature} className="flex items-center gap-2 text-sm">
                  <span className="text-green-500">✓</span> {feature}
                </li>
              ))}
            </ul>

            <Button
              className="w-full"
              variant={plan.popular ? "default" : "outline"}
              disabled={currentPlan === plan.name.toLowerCase() || !plan.priceId[billing]}
              loading={loading === plan.priceId[billing]}
              onClick={() => plan.priceId[billing] && handleCheckout(plan.priceId[billing]!)}
            >
              {currentPlan === plan.name.toLowerCase() ? "Current Plan" : plan.cta}
            </Button>
          </div>
        ))}
      </div>
    </div>
  )
}
```

---

## 📧 EMAIL — RESEND + REACT EMAIL

### Setup
```bash
pnpm add resend react-email @react-email/components
```

### Email Templates
```tsx
// emails/welcome.tsx
import {
  Body, Button, Container, Head, Heading, Html,
  Img, Link, Preview, Section, Text, Tailwind
} from "@react-email/components"

interface WelcomeEmailProps {
  userName: string
  userEmail: string
  loginUrl: string
}

export default function WelcomeEmail({ userName, userEmail, loginUrl }: WelcomeEmailProps) {
  return (
    <Html>
      <Head />
      <Preview>Welcome to MyApp — you're all set!</Preview>
      <Tailwind>
        <Body className="bg-gray-50 font-sans">
          <Container className="mx-auto py-8 px-4 max-w-xl">
            {/* Logo */}
            <Section className="mb-8 text-center">
              <Img src="https://myapp.com/logo.png" width="120" height="40" alt="MyApp" />
            </Section>

            {/* Main content */}
            <Section className="bg-white rounded-2xl p-8 shadow-sm">
              <Heading className="text-2xl font-bold text-gray-900 mb-2">
                Welcome, {userName}! 👋
              </Heading>

              <Text className="text-gray-600 mb-6">
                Your account has been created successfully. Here's everything you need to get started.
              </Text>

              <Button
                href={loginUrl}
                className="bg-blue-600 text-white font-semibold px-6 py-3 rounded-xl w-full text-center"
              >
                Get Started →
              </Button>

              <Text className="text-sm text-gray-400 mt-6">
                If you didn't create this account, please ignore this email or{" "}
                <Link href="mailto:support@myapp.com" className="text-blue-600">
                  contact support
                </Link>.
              </Text>
            </Section>

            {/* Footer */}
            <Text className="text-center text-xs text-gray-400 mt-8">
              MyApp Inc. · 123 Street, City, Country<br />
              <Link href="{{unsubscribeUrl}}" className="text-gray-400">
                Unsubscribe
              </Link>
            </Text>
          </Container>
        </Body>
      </Tailwind>
    </Html>
  )
}
```

```tsx
// emails/reset-password.tsx
export default function ResetPasswordEmail({ resetUrl, userName }: { resetUrl: string; userName: string }) {
  return (
    <Html>
      <Head />
      <Preview>Reset your MyApp password</Preview>
      <Tailwind>
        <Body className="bg-gray-50 font-sans">
          <Container className="mx-auto py-8 px-4 max-w-xl">
            <Section className="bg-white rounded-2xl p-8">
              <Heading>Reset your password</Heading>
              <Text>Hi {userName}, we received a request to reset your password.</Text>
              <Button href={resetUrl} className="bg-blue-600 text-white px-6 py-3 rounded-xl">
                Reset Password
              </Button>
              <Text className="text-sm text-gray-400">
                This link expires in 1 hour. If you didn't request this, ignore this email.
              </Text>
            </Section>
          </Container>
        </Body>
      </Tailwind>
    </Html>
  )
}
```

### Email Service
```typescript
// lib/email.ts
import { Resend } from "resend"
import { render } from "@react-email/components"
import WelcomeEmail from "@/emails/welcome"
import ResetPasswordEmail from "@/emails/reset-password"
import { env } from "@/env"

const resend = new Resend(env.RESEND_API_KEY)

const FROM = "MyApp <noreply@myapp.com>"

export const email = {
  async sendWelcome(to: string, props: { userName: string; loginUrl: string }) {
    return resend.emails.send({
      from: FROM,
      to,
      subject: "Welcome to MyApp! 🎉",
      react: WelcomeEmail({ ...props, userEmail: to }),
    })
  },

  async sendResetPassword(to: string, props: { userName: string; resetUrl: string }) {
    return resend.emails.send({
      from: FROM,
      to,
      subject: "Reset your MyApp password",
      react: ResetPasswordEmail(props),
    })
  },

  async sendPaymentFailed(to: string, props: { userName: string; retryUrl: string }) {
    return resend.emails.send({
      from: FROM,
      to,
      subject: "Payment failed — action required",
      react: PaymentFailedEmail(props),
    })
  },

  async sendBatch(messages: Parameters<typeof resend.batch.send>[0]) {
    return resend.batch.send(messages)
  },
}
```

---

## 📤 FILE UPLOADS — UPLOADTHING

### Setup
```bash
pnpm add uploadthing @uploadthing/react
```

### File Router
```typescript
// lib/uploadthing.ts
import { createUploadthing, type FileRouter } from "uploadthing/next"
import { auth } from "@clerk/nextjs/server"

const f = createUploadthing()

export const uploadRouter = {
  // Profile image upload
  profileImage: f({ image: { maxFileSize: "4MB", maxFileCount: 1 } })
    .middleware(async () => {
      const { userId } = await auth()
      if (!userId) throw new Error("Unauthorized")
      return { userId }
    })
    .onUploadComplete(async ({ metadata, file }) => {
      // Update user avatar in DB
      await db.update(users)
        .set({ avatarUrl: file.url })
        .where(eq(users.clerkId, metadata.userId))

      return { url: file.url }
    }),

  // Document upload
  document: f({
    pdf: { maxFileSize: "16MB", maxFileCount: 5 },
    "application/msword": { maxFileSize: "16MB" },
    "application/vnd.openxmlformats-officedocument.wordprocessingml.document": { maxFileSize: "16MB" },
  })
    .middleware(async () => {
      const { userId } = await auth()
      if (!userId) throw new Error("Unauthorized")
      return { userId }
    })
    .onUploadComplete(async ({ metadata, file }) => {
      // Save to documents table
      const [doc] = await db.insert(documents).values({
        userId: metadata.userId,
        name: file.name,
        url: file.url,
        size: file.size,
        type: file.type,
      }).returning()

      return { documentId: doc.id, url: file.url }
    }),

  // Bulk images (e.g. product photos)
  productImages: f({ image: { maxFileSize: "8MB", maxFileCount: 10 } })
    .middleware(async () => {
      const { userId } = await auth()
      if (!userId) throw new Error("Unauthorized")
      return { userId }
    })
    .onUploadComplete(async ({ file }) => ({ url: file.url })),

} satisfies FileRouter

export type OurFileRouter = typeof uploadRouter
```

```typescript
// app/api/uploadthing/route.ts
import { createRouteHandler } from "uploadthing/next"
import { uploadRouter } from "@/lib/uploadthing"

export const { GET, POST } = createRouteHandler({ router: uploadRouter })
```

### Upload Components
```tsx
// components/ui/file-uploader.tsx
"use client"

import { UploadDropzone, UploadButton } from "@uploadthing/react"
import type { OurFileRouter } from "@/lib/uploadthing"
import { toast } from "sonner"

interface ProfileImageUploaderProps {
  onUpload: (url: string) => void
}

export function ProfileImageUploader({ onUpload }: ProfileImageUploaderProps) {
  return (
    <UploadButton<OurFileRouter, "profileImage">
      endpoint="profileImage"
      onClientUploadComplete={(res) => {
        const url = res[0]?.url
        if (url) {
          onUpload(url)
          toast.success("Profile image updated!")
        }
      }}
      onUploadError={(error) => {
        toast.error(`Upload failed: ${error.message}`)
      }}
      appearance={{
        button: "bg-primary text-primary-foreground rounded-xl px-4 py-2 text-sm font-medium",
        allowedContent: "text-xs text-muted-foreground mt-1",
      }}
    />
  )
}

export function DocumentDropzone({ onUpload }: { onUpload: (files: { url: string; name: string }[]) => void }) {
  return (
    <UploadDropzone<OurFileRouter, "document">
      endpoint="document"
      onClientUploadComplete={(res) => {
        const files = res.map(f => ({ url: f.url, name: f.name }))
        onUpload(files)
        toast.success(`${files.length} file(s) uploaded!`)
      }}
      onUploadError={(error) => toast.error(error.message)}
      appearance={{
        container: "border-2 border-dashed border-border rounded-xl p-8 cursor-pointer hover:border-primary/50 transition-colors",
        label: "text-sm font-medium text-foreground",
        allowedContent: "text-xs text-muted-foreground",
      }}
    />
  )
}
```

---

## 🔍 SEARCH — TYPESENSE

### Setup
```bash
pnpm add typesense
```

### Typesense Client
```typescript
// lib/search.ts
import Typesense from "typesense"
import { env } from "@/env"

export const searchClient = new Typesense.Client({
  nodes: [{
    host: env.TYPESENSE_HOST,
    port: 443,
    protocol: "https",
  }],
  apiKey: env.TYPESENSE_API_KEY,
  connectionTimeoutSeconds: 2,
})

// Collection schemas
export async function createProductsCollection() {
  await searchClient.collections().create({
    name: "products",
    fields: [
      { name: "id", type: "string" },
      { name: "name", type: "string" },
      { name: "description", type: "string" },
      { name: "price", type: "float" },
      { name: "category", type: "string", facet: true },
      { name: "tags", type: "string[]", facet: true },
      { name: "inStock", type: "bool", facet: true },
      { name: "createdAt", type: "int64" },
    ],
    default_sorting_field: "createdAt",
  })
}

// Index a document
export async function indexProduct(product: Product) {
  await searchClient.collections("products").documents().upsert({
    id: product.id,
    name: product.name,
    description: product.description,
    price: product.price,
    category: product.category,
    tags: product.tags,
    inStock: product.inStock,
    createdAt: Math.floor(product.createdAt.getTime() / 1000),
  })
}

// Search
export async function searchProducts(query: string, filters?: {
  category?: string
  inStock?: boolean
  priceMin?: number
  priceMax?: number
  page?: number
  perPage?: number
}) {
  const filterBy = [
    filters?.category && `category:${filters.category}`,
    filters?.inStock !== undefined && `inStock:${filters.inStock}`,
    filters?.priceMin && `price:>=${filters.priceMin}`,
    filters?.priceMax && `price:<=${filters.priceMax}`,
  ].filter(Boolean).join(" && ")

  const result = await searchClient.collections("products").documents().search({
    q: query,
    query_by: "name,description,tags",
    filter_by: filterBy || undefined,
    facet_by: "category,tags,inStock",
    page: filters?.page ?? 1,
    per_page: filters?.perPage ?? 20,
    sort_by: "_text_match:desc",
    highlight_affix_num_tokens: 3,
  })

  return {
    hits: result.hits ?? [],
    total: result.found,
    facets: result.facet_counts ?? [],
    page: result.page,
  }
}
```

---

## 📊 ANALYTICS — POSTHOG

### Setup
```bash
pnpm add posthog-js posthog-node
```

### Client Setup
```tsx
// components/providers/posthog-provider.tsx
"use client"

import posthog from "posthog-js"
import { PostHogProvider } from "posthog-js/react"
import { usePathname, useSearchParams } from "next/navigation"
import { useEffect, Suspense } from "react"
import { useUser } from "@clerk/nextjs"

function PostHogPageView() {
  const pathname = usePathname()
  const searchParams = useSearchParams()

  useEffect(() => {
    if (pathname) {
      let url = window.origin + pathname
      if (searchParams.toString()) url += `?${searchParams.toString()}`
      posthog.capture("$pageview", { "$current_url": url })
    }
  }, [pathname, searchParams])

  return null
}

function PostHogIdentify() {
  const { user } = useUser()

  useEffect(() => {
    if (user) {
      posthog.identify(user.id, {
        email: user.emailAddresses[0]?.emailAddress,
        name: user.fullName,
        created_at: user.createdAt,
      })
    } else {
      posthog.reset()
    }
  }, [user])

  return null
}

export function PostHogProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
      api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
      capture_pageview: false, // Manual in Next.js
      capture_pageleave: true,
      persistence: "localStorage",
    })
  }, [])

  return (
    <PostHogProvider client={posthog}>
      <Suspense>
        <PostHogPageView />
        <PostHogIdentify />
      </Suspense>
      {children}
    </PostHogProvider>
  )
}
```

### Event Tracking
```typescript
// lib/analytics.ts
import posthog from "posthog-js"

export const track = {
  // User events
  signUp: (method: string) => posthog.capture("user_signed_up", { method }),
  signIn: (method: string) => posthog.capture("user_signed_in", { method }),

  // Feature events
  projectCreated: (projectType: string) =>
    posthog.capture("project_created", { project_type: projectType }),

  featureUsed: (feature: string, properties?: Record<string, unknown>) =>
    posthog.capture("feature_used", { feature, ...properties }),

  // Business events
  checkoutStarted: (plan: string, billing: string) =>
    posthog.capture("checkout_started", { plan, billing }),

  subscriptionUpgraded: (from: string, to: string) =>
    posthog.capture("subscription_upgraded", { from_plan: from, to_plan: to }),

  // Error events
  errorEncountered: (errorType: string, context?: Record<string, unknown>) =>
    posthog.capture("error_encountered", { error_type: errorType, ...context }),
}

// Feature flags
export function useFeatureFlag(flag: string): boolean {
  return posthog.isFeatureEnabled(flag) ?? false
}
```

---

## 🔴 REAL-TIME — SUPABASE REALTIME

### Real-time Subscriptions
```typescript
// hooks/use-realtime.ts
"use client"

import { useEffect, useRef } from "react"
import { RealtimeChannel } from "@supabase/supabase-js"
import { supabase } from "@/lib/supabase"

// Subscribe to table changes
export function useRealtimeTable<T extends { id: string }>(
  table: string,
  options: {
    filter?: string
    onInsert?: (row: T) => void
    onUpdate?: (row: T) => void
    onDelete?: (row: T) => void
  }
) {
  const channelRef = useRef<RealtimeChannel | null>(null)

  useEffect(() => {
    const channel = supabase
      .channel(`realtime:${table}`)
      .on(
        "postgres_changes",
        {
          event: "*",
          schema: "public",
          table,
          filter: options.filter,
        },
        (payload) => {
          switch (payload.eventType) {
            case "INSERT":
              options.onInsert?.(payload.new as T)
              break
            case "UPDATE":
              options.onUpdate?.(payload.new as T)
              break
            case "DELETE":
              options.onDelete?.(payload.old as T)
              break
          }
        }
      )
      .subscribe()

    channelRef.current = channel

    return () => {
      channel.unsubscribe()
    }
  }, [table, options.filter])
}

// Presence (who's online)
export function usePresence(roomId: string, userInfo: { name: string; avatarUrl?: string }) {
  const [onlineUsers, setOnlineUsers] = useState<typeof userInfo[]>([])

  useEffect(() => {
    const channel = supabase.channel(`presence:${roomId}`, {
      config: { presence: { key: crypto.randomUUID() } },
    })

    channel
      .on("presence", { event: "sync" }, () => {
        const state = channel.presenceState<typeof userInfo>()
        const users = Object.values(state).flat()
        setOnlineUsers(users)
      })
      .subscribe(async (status) => {
        if (status === "SUBSCRIBED") {
          await channel.track(userInfo)
        }
      })

    return () => {
      channel.untrack()
      channel.unsubscribe()
    }
  }, [roomId])

  return onlineUsers
}
```

### Usage Examples
```tsx
// Real-time notifications
function NotificationsPanel({ userId }: { userId: string }) {
  const [notifications, setNotifications] = useState<Notification[]>([])

  useRealtimeTable<Notification>("notifications", {
    filter: `user_id=eq.${userId}`,
    onInsert: (notification) => {
      setNotifications(prev => [notification, ...prev])
      toast(notification.message) // Show toast for new notification
    },
  })

  return <NotificationList items={notifications} />
}

// Real-time collaboration indicator
function CollaboratorAvatars({ documentId }: { documentId: string }) {
  const { user } = useUser()
  const onlineUsers = usePresence(`doc:${documentId}`, {
    name: user?.fullName ?? "Anonymous",
    avatarUrl: user?.imageUrl,
  })

  return (
    <div className="flex -space-x-2">
      {onlineUsers.map((u, i) => (
        <Avatar key={i} src={u.avatarUrl} name={u.name} />
      ))}
    </div>
  )
}
```

---

## 🗺️ RATE LIMITING

```typescript
// lib/rate-limit.ts
import { Ratelimit } from "@upstash/ratelimit"
import { Redis } from "@upstash/redis"

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
})

// Different limiters for different operations
export const rateLimit = {
  // API calls: 100 per minute per user
  api: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(100, "1 m"),
    prefix: "ratelimit:api",
  }),

  // AI calls: 20 per hour per user (expensive!)
  ai: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(20, "1 h"),
    prefix: "ratelimit:ai",
  }),

  // Auth attempts: 5 per 15 minutes per IP
  auth: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5, "15 m"),
    prefix: "ratelimit:auth",
  }),

  // Email sends: 10 per hour
  email: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, "1 h"),
    prefix: "ratelimit:email",
  }),
}

// Middleware helper
export async function withRateLimit(
  limiter: typeof rateLimit.api,
  identifier: string
): Promise<{ success: boolean; remaining: number; resetAt: Date }> {
  const { success, remaining, reset } = await limiter.limit(identifier)
  return { success, remaining, resetAt: new Date(reset) }
}
```
