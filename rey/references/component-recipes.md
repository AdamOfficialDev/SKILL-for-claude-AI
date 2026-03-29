# REY — Component Recipes

> Production-ready patterns. Copy. Customize. Ship.

---

## 🔄 ASYNC DATA PATTERNS

### Optimistic Updates
```tsx
// Pattern: update UI immediately, rollback on error
import { useMutation, useQueryClient } from "@tanstack/react-query"
import { toast } from "sonner"

function TodoItem({ todo }: { todo: Todo }) {
  const queryClient = useQueryClient()

  const toggleMutation = useMutation({
    mutationFn: async (id: string) => {
      const res = await fetch(`/api/todos/${id}/toggle`, { method: "PATCH" })
      if (!res.ok) throw new Error("Failed to toggle")
      return res.json()
    },
    // Called before mutation — optimistically update
    onMutate: async (id) => {
      await queryClient.cancelQueries({ queryKey: ["todos"] })
      const previous = queryClient.getQueryData(["todos"])

      queryClient.setQueryData(["todos"], (old: Todo[]) =>
        old.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
      )

      return { previous } // Context for rollback
    },
    onError: (err, id, context) => {
      // Rollback on error
      queryClient.setQueryData(["todos"], context?.previous)
      toast.error("Failed to update. Changes reverted.")
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ["todos"] })
    },
  })

  return (
    <div className="flex items-center gap-3">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleMutation.mutate(todo.id)}
        className="h-4 w-4"
      />
      <span className={todo.completed ? "line-through text-muted-foreground" : ""}>
        {todo.title}
      </span>
    </div>
  )
}
```

### Infinite Scroll with TanStack Query
```tsx
import { useInfiniteQuery } from "@tanstack/react-query"
import { useInView } from "react-intersection-observer"
import { useEffect } from "react"

interface FeedItem { id: string; content: string; createdAt: string }
interface PageData { items: FeedItem[]; nextCursor: string | null }

function InfiniteList() {
  const { ref, inView } = useInView({ threshold: 0.1 })

  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, status } =
    useInfiniteQuery({
      queryKey: ["feed"],
      queryFn: async ({ pageParam }) => {
        const params = new URLSearchParams({ limit: "20" })
        if (pageParam) params.set("cursor", pageParam)
        const res = await fetch(`/api/feed?${params}`)
        return res.json() as Promise<PageData>
      },
      initialPageParam: undefined as string | undefined,
      getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
    })

  useEffect(() => {
    if (inView && hasNextPage && !isFetchingNextPage) {
      fetchNextPage()
    }
  }, [inView, hasNextPage, isFetchingNextPage, fetchNextPage])

  if (status === "pending") return <FeedSkeleton count={5} />
  if (status === "error") return <ErrorState onRetry={() => window.location.reload()} />

  const items = data.pages.flatMap(page => page.items)

  return (
    <div className="space-y-4">
      {items.map(item => (
        <FeedCard key={item.id} item={item} />
      ))}

      {/* Trigger element */}
      <div ref={ref} className="py-4 flex justify-center">
        {isFetchingNextPage ? (
          <Spinner />
        ) : hasNextPage ? (
          <span className="text-muted-foreground text-sm">Load more...</span>
        ) : (
          <span className="text-muted-foreground text-sm">You've reached the end</span>
        )}
      </div>
    </div>
  )
}
```

### Suspense + Error Boundary Pattern
```tsx
import { Suspense } from "react"
import { ErrorBoundary } from "react-error-boundary"

function ErrorFallback({ error, resetErrorBoundary }: {
  error: Error
  resetErrorBoundary: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center py-12 px-4 text-center">
      <div className="text-4xl mb-4">⚠️</div>
      <h3 className="text-lg font-semibold mb-2">Something went wrong</h3>
      <p className="text-sm text-muted-foreground mb-4 max-w-sm">{error.message}</p>
      <Button variant="outline" onClick={resetErrorBoundary}>Try Again</Button>
    </div>
  )
}

// Usage pattern — REY always wraps async data components
function DataSection() {
  return (
    <ErrorBoundary FallbackComponent={ErrorFallback}>
      <Suspense fallback={<DataSkeleton />}>
        <AsyncDataComponent />
      </Suspense>
    </ErrorBoundary>
  )
}
```

---

## 📝 FORM PATTERNS

### Multi-Step Form (Wizard)
```tsx
"use client"

import { useState } from "react"
import { useForm, FormProvider } from "react-hook-form"
import { zodResolver } from "@hookform/resolvers/zod"
import { z } from "zod"
import { motion, AnimatePresence } from "motion/react"

// Step schemas
const step1Schema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
})

const step2Schema = z.object({
  company: z.string().min(1),
  role: z.string().min(1),
  teamSize: z.enum(["1", "2-10", "11-50", "50+"]),
})

const step3Schema = z.object({
  plan: z.enum(["starter", "pro", "enterprise"]),
  billing: z.enum(["monthly", "yearly"]),
})

const fullSchema = step1Schema.merge(step2Schema).merge(step3Schema)
type FormData = z.infer<typeof fullSchema>

const steps = [
  { id: 1, title: "Your Info", schema: step1Schema },
  { id: 2, title: "Company", schema: step2Schema },
  { id: 3, title: "Plan", schema: step3Schema },
]

export function MultiStepForm({ onComplete }: { onComplete: (data: FormData) => void }) {
  const [currentStep, setCurrentStep] = useState(0)
  const [direction, setDirection] = useState(1)

  const methods = useForm<FormData>({
    resolver: zodResolver(steps[currentStep].schema),
    mode: "onChange",
  })

  const isFirst = currentStep === 0
  const isLast = currentStep === steps.length - 1

  const goNext = methods.handleSubmit(() => {
    if (!isLast) {
      setDirection(1)
      setCurrentStep(s => s + 1)
    }
  })

  const goBack = () => {
    setDirection(-1)
    setCurrentStep(s => s - 1)
  }

  const handleFinalSubmit = methods.handleSubmit((data) => {
    onComplete(data)
  })

  return (
    <FormProvider {...methods}>
      <div className="max-w-lg mx-auto">
        {/* Progress */}
        <div className="flex gap-2 mb-8">
          {steps.map((step, i) => (
            <div key={step.id} className="flex-1">
              <div className={`h-2 rounded-full transition-colors ${
                i <= currentStep ? "bg-primary" : "bg-muted"
              }`} />
              <span className="text-xs text-muted-foreground mt-1">{step.title}</span>
            </div>
          ))}
        </div>

        {/* Step content */}
        <AnimatePresence mode="wait" custom={direction}>
          <motion.div
            key={currentStep}
            custom={direction}
            initial={{ opacity: 0, x: direction * 50 }}
            animate={{ opacity: 1, x: 0 }}
            exit={{ opacity: 0, x: direction * -50 }}
            transition={{ duration: 0.25 }}
          >
            {currentStep === 0 && <Step1 />}
            {currentStep === 1 && <Step2 />}
            {currentStep === 2 && <Step3 />}
          </motion.div>
        </AnimatePresence>

        {/* Navigation */}
        <div className="flex gap-3 mt-6">
          {!isFirst && (
            <Button variant="outline" onClick={goBack} className="flex-1">
              Back
            </Button>
          )}
          <Button
            onClick={isLast ? handleFinalSubmit : goNext}
            className="flex-1"
          >
            {isLast ? "Complete Setup" : "Continue"}
          </Button>
        </div>
      </div>
    </FormProvider>
  )
}
```

### Combobox (Searchable Select)
```tsx
"use client"

import { useState, useMemo } from "react"
import { Check, ChevronsUpDown, Search } from "lucide-react"
import { Popover, PopoverContent, PopoverTrigger } from "@/components/ui/popover"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { cn } from "@/lib/utils"

interface ComboboxOption { value: string; label: string; description?: string }

interface ComboboxProps {
  options: ComboboxOption[]
  value?: string
  onChange: (value: string) => void
  placeholder?: string
  searchPlaceholder?: string
  emptyText?: string
}

export function Combobox({
  options,
  value,
  onChange,
  placeholder = "Select...",
  searchPlaceholder = "Search...",
  emptyText = "No results found",
}: ComboboxProps) {
  const [open, setOpen] = useState(false)
  const [search, setSearch] = useState("")

  const filtered = useMemo(
    () => options.filter(o =>
      o.label.toLowerCase().includes(search.toLowerCase())
    ),
    [options, search]
  )

  const selected = options.find(o => o.value === value)

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button
          variant="outline"
          role="combobox"
          aria-expanded={open}
          className="w-full justify-between font-normal"
        >
          {selected ? selected.label : placeholder}
          <ChevronsUpDown className="ml-2 h-4 w-4 shrink-0 opacity-50" />
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-[--radix-popover-trigger-width] p-0" align="start">
        <div className="p-2 border-b">
          <div className="relative">
            <Search className="absolute left-2.5 top-2.5 h-4 w-4 text-muted-foreground" />
            <Input
              className="pl-8 border-0 focus-visible:ring-0"
              placeholder={searchPlaceholder}
              value={search}
              onChange={e => setSearch(e.target.value)}
            />
          </div>
        </div>

        <div className="max-h-48 overflow-y-auto p-1">
          {filtered.length === 0 ? (
            <div className="text-sm text-center text-muted-foreground py-6">{emptyText}</div>
          ) : (
            filtered.map(option => (
              <button
                key={option.value}
                className={cn(
                  "w-full flex items-center gap-2 rounded-md px-2 py-1.5 text-sm hover:bg-accent text-left",
                  value === option.value && "bg-accent"
                )}
                onClick={() => {
                  onChange(option.value)
                  setOpen(false)
                  setSearch("")
                }}
              >
                <Check className={cn(
                  "h-4 w-4 shrink-0",
                  value === option.value ? "opacity-100" : "opacity-0"
                )} />
                <div>
                  <div className="font-medium">{option.label}</div>
                  {option.description && (
                    <div className="text-xs text-muted-foreground">{option.description}</div>
                  )}
                </div>
              </button>
            ))
          )}
        </div>
      </PopoverContent>
    </Popover>
  )
}
```

---

## 🗂️ TABLE PATTERNS

### Full-Featured Data Table
```tsx
"use client"

import {
  useReactTable, getCoreRowModel, getSortedRowModel,
  getFilteredRowModel, getPaginationRowModel,
  flexRender, ColumnDef, SortingState, ColumnFiltersState
} from "@tanstack/react-table"
import { useState } from "react"
import { ArrowUpDown, ChevronLeft, ChevronRight } from "lucide-react"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[]
  data: TData[]
  searchColumn?: string
  searchPlaceholder?: string
}

export function DataTable<TData, TValue>({
  columns, data, searchColumn, searchPlaceholder = "Search..."
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = useState<SortingState>([])
  const [columnFilters, setColumnFilters] = useState<ColumnFiltersState>([])
  const [globalFilter, setGlobalFilter] = useState("")

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    onGlobalFilterChange: setGlobalFilter,
    state: { sorting, columnFilters, globalFilter },
    initialState: { pagination: { pageSize: 20 } },
  })

  return (
    <div>
      {/* Search */}
      <div className="flex items-center py-4">
        <Input
          placeholder={searchPlaceholder}
          value={globalFilter}
          onChange={e => setGlobalFilter(e.target.value)}
          className="max-w-sm"
        />
      </div>

      {/* Table */}
      <div className="rounded-xl border overflow-hidden">
        <table className="w-full text-sm">
          <thead className="bg-muted/50">
            {table.getHeaderGroups().map(headerGroup => (
              <tr key={headerGroup.id}>
                {headerGroup.headers.map(header => (
                  <th key={header.id} className="text-left px-4 py-3 font-medium text-muted-foreground">
                    {header.isPlaceholder ? null : (
                      <button
                        className={cn(
                          "flex items-center gap-1",
                          header.column.getCanSort() && "cursor-pointer hover:text-foreground"
                        )}
                        onClick={header.column.getToggleSortingHandler()}
                      >
                        {flexRender(header.column.columnDef.header, header.getContext())}
                        {header.column.getCanSort() && (
                          <ArrowUpDown className="h-4 w-4" />
                        )}
                      </button>
                    )}
                  </th>
                ))}
              </tr>
            ))}
          </thead>
          <tbody>
            {table.getRowModel().rows.length ? (
              table.getRowModel().rows.map(row => (
                <tr key={row.id} className="border-t hover:bg-muted/30 transition-colors">
                  {row.getVisibleCells().map(cell => (
                    <td key={cell.id} className="px-4 py-3">
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </td>
                  ))}
                </tr>
              ))
            ) : (
              <tr>
                <td colSpan={columns.length} className="h-24 text-center text-muted-foreground">
                  No results.
                </td>
              </tr>
            )}
          </tbody>
        </table>
      </div>

      {/* Pagination */}
      <div className="flex items-center justify-between py-4">
        <span className="text-sm text-muted-foreground">
          {table.getFilteredSelectedRowModel().rows.length > 0
            ? `${table.getFilteredSelectedRowModel().rows.length} of `
            : ""
          }
          {table.getFilteredRowModel().rows.length} row(s)
        </span>

        <div className="flex items-center gap-2">
          <Button
            variant="outline" size="icon"
            onClick={() => table.previousPage()}
            disabled={!table.getCanPreviousPage()}
          >
            <ChevronLeft className="h-4 w-4" />
          </Button>
          <span className="text-sm">
            Page {table.getState().pagination.pageIndex + 1} of {table.getPageCount()}
          </span>
          <Button
            variant="outline" size="icon"
            onClick={() => table.nextPage()}
            disabled={!table.getCanNextPage()}
          >
            <ChevronRight className="h-4 w-4" />
          </Button>
        </div>
      </div>
    </div>
  )
}

// Column definition helper
export function createSortableColumn<T>(key: keyof T, header: string): ColumnDef<T> {
  return {
    accessorKey: key as string,
    header: header,
    cell: ({ row }) => row.getValue(key as string),
  }
}
```

---

## 🎨 LAYOUT PATTERNS

### Command Menu (⌘K)
```tsx
"use client"

import { useEffect, useState } from "react"
import { useRouter } from "next/navigation"
import {
  CommandDialog, CommandEmpty, CommandGroup,
  CommandInput, CommandItem, CommandList, CommandSeparator
} from "@/components/ui/command"
import { Home, Settings, Users, CreditCard, LogOut } from "lucide-react"
import { useClerk } from "@clerk/nextjs"

export function CommandMenu() {
  const [open, setOpen] = useState(false)
  const router = useRouter()
  const { signOut } = useClerk()

  useEffect(() => {
    const down = (e: KeyboardEvent) => {
      if (e.key === "k" && (e.metaKey || e.ctrlKey)) {
        e.preventDefault()
        setOpen(prev => !prev)
      }
    }
    document.addEventListener("keydown", down)
    return () => document.removeEventListener("keydown", down)
  }, [])

  const navigate = (path: string) => {
    router.push(path)
    setOpen(false)
  }

  return (
    <CommandDialog open={open} onOpenChange={setOpen}>
      <CommandInput placeholder="Type a command or search..." />
      <CommandList>
        <CommandEmpty>No results found.</CommandEmpty>

        <CommandGroup heading="Navigation">
          {[
            { icon: Home, label: "Dashboard", path: "/dashboard" },
            { icon: Users, label: "Users", path: "/users" },
            { icon: Settings, label: "Settings", path: "/settings" },
            { icon: CreditCard, label: "Billing", path: "/settings/billing" },
          ].map(item => (
            <CommandItem key={item.path} onSelect={() => navigate(item.path)}>
              <item.icon className="mr-2 h-4 w-4" />
              {item.label}
            </CommandItem>
          ))}
        </CommandGroup>

        <CommandSeparator />

        <CommandGroup heading="Account">
          <CommandItem onSelect={() => signOut(() => router.push("/"))}>
            <LogOut className="mr-2 h-4 w-4" />
            Sign out
          </CommandItem>
        </CommandGroup>
      </CommandList>
    </CommandDialog>
  )
}
```

### Resizable Sidebar Layout
```tsx
"use client"

import { ResizableHandle, ResizablePanel, ResizablePanelGroup } from "@/components/ui/resizable"
import { cn } from "@/lib/utils"

interface ResizableLayoutProps {
  sidebar: React.ReactNode
  children: React.ReactNode
  defaultSidebarSize?: number
  minSidebarSize?: number
  maxSidebarSize?: number
}

export function ResizableLayout({
  sidebar,
  children,
  defaultSidebarSize = 20,
  minSidebarSize = 15,
  maxSidebarSize = 35,
}: ResizableLayoutProps) {
  return (
    <ResizablePanelGroup direction="horizontal" className="h-full">
      <ResizablePanel
        defaultSize={defaultSidebarSize}
        minSize={minSidebarSize}
        maxSize={maxSidebarSize}
        className="hidden md:flex"
      >
        <div className="h-full w-full overflow-y-auto border-r">
          {sidebar}
        </div>
      </ResizablePanel>

      <ResizableHandle withHandle className="hidden md:flex" />

      <ResizablePanel defaultSize={80}>
        <div className="h-full overflow-y-auto">
          {children}
        </div>
      </ResizablePanel>
    </ResizablePanelGroup>
  )
}
```

---

## 🔔 NOTIFICATION PATTERNS

### Toast Notifications (Sonner)
```typescript
// lib/toast.ts — Typed toast helpers
import { toast } from "sonner"

export const notify = {
  success: (message: string, description?: string) =>
    toast.success(message, { description }),

  error: (message: string, description?: string) =>
    toast.error(message, {
      description,
      duration: 6000, // Errors stay longer
    }),

  warning: (message: string, description?: string) =>
    toast.warning(message, { description }),

  info: (message: string, description?: string) =>
    toast.info(message, { description }),

  loading: (message: string) =>
    toast.loading(message),

  promise: <T>(
    promise: Promise<T>,
    messages: { loading: string; success: string; error: string }
  ) => toast.promise(promise, messages),

  action: (message: string, action: { label: string; onClick: () => void }) =>
    toast(message, {
      action: { label: action.label, onClick: action.onClick },
    }),
}

// Usage
await notify.promise(
  saveDocument(),
  {
    loading: "Saving document...",
    success: "Document saved!",
    error: "Failed to save document",
  }
)
```

---

## 🎛️ SETTINGS PATTERNS

### Settings Page with Sections
```tsx
// components/features/settings/settings-layout.tsx
"use client"

import { usePathname } from "next/navigation"
import Link from "next/link"
import { cn } from "@/lib/utils"
import { User, Lock, Bell, CreditCard, Palette, Users, Key } from "lucide-react"

const settingsNav = [
  { label: "Profile", href: "/settings", icon: User },
  { label: "Security", href: "/settings/security", icon: Lock },
  { label: "Notifications", href: "/settings/notifications", icon: Bell },
  { label: "Billing", href: "/settings/billing", icon: CreditCard },
  { label: "Appearance", href: "/settings/appearance", icon: Palette },
  { label: "Team", href: "/settings/team", icon: Users },
  { label: "API Keys", href: "/settings/api-keys", icon: Key },
]

export function SettingsLayout({ children }: { children: React.ReactNode }) {
  const pathname = usePathname()

  return (
    <div className="container max-w-5xl py-8">
      <div className="mb-6">
        <h1 className="text-2xl font-bold">Settings</h1>
        <p className="text-muted-foreground">Manage your account settings.</p>
      </div>

      <div className="flex gap-8">
        <nav className="w-48 shrink-0">
          <ul className="space-y-1">
            {settingsNav.map(item => (
              <li key={item.href}>
                <Link
                  href={item.href}
                  className={cn(
                    "flex items-center gap-2 rounded-lg px-3 py-2 text-sm transition-colors",
                    pathname === item.href
                      ? "bg-primary/10 text-primary font-medium"
                      : "text-muted-foreground hover:text-foreground hover:bg-muted"
                  )}
                >
                  <item.icon className="h-4 w-4" />
                  {item.label}
                </Link>
              </li>
            ))}
          </ul>
        </nav>

        <div className="flex-1 space-y-6">
          {children}
        </div>
      </div>
    </div>
  )
}

// Settings section card
export function SettingsSection({
  title,
  description,
  children,
}: {
  title: string
  description?: string
  children: React.ReactNode
}) {
  return (
    <div className="rounded-xl border p-6">
      <div className="mb-4">
        <h3 className="font-semibold">{title}</h3>
        {description && <p className="text-sm text-muted-foreground mt-1">{description}</p>}
      </div>
      {children}
    </div>
  )
}
```

---

## 📊 DASHBOARD PATTERNS

### Stats Cards
```tsx
// components/common/stats-card.tsx
import { LucideIcon, TrendingUp, TrendingDown, Minus } from "lucide-react"
import { Card } from "@/components/ui/card"
import { cn } from "@/lib/utils"

interface StatsCardProps {
  title: string
  value: string | number
  change?: number // percentage change
  icon: LucideIcon
  prefix?: string
  suffix?: string
  description?: string
}

export function StatsCard({
  title, value, change, icon: Icon, prefix = "", suffix = "", description
}: StatsCardProps) {
  const trend = change === undefined ? null : change > 0 ? "up" : change < 0 ? "down" : "flat"

  return (
    <Card className="p-5">
      <div className="flex items-center justify-between mb-3">
        <span className="text-sm font-medium text-muted-foreground">{title}</span>
        <div className="h-9 w-9 rounded-lg bg-primary/10 flex items-center justify-center">
          <Icon className="h-5 w-5 text-primary" />
        </div>
      </div>

      <div className="flex items-end justify-between">
        <div>
          <span className="text-2xl font-bold">
            {prefix}{typeof value === "number" ? value.toLocaleString() : value}{suffix}
          </span>
          {description && (
            <p className="text-xs text-muted-foreground mt-0.5">{description}</p>
          )}
        </div>

        {trend && (
          <div className={cn(
            "flex items-center gap-1 text-xs font-medium rounded-md px-2 py-1",
            trend === "up" && "text-green-600 bg-green-50 dark:bg-green-950",
            trend === "down" && "text-red-600 bg-red-50 dark:bg-red-950",
            trend === "flat" && "text-gray-500 bg-gray-100 dark:bg-gray-800",
          )}>
            {trend === "up" && <TrendingUp className="h-3 w-3" />}
            {trend === "down" && <TrendingDown className="h-3 w-3" />}
            {trend === "flat" && <Minus className="h-3 w-3" />}
            {Math.abs(change!)}%
          </div>
        )}
      </div>
    </Card>
  )
}

// Usage
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
  <StatsCard title="Total Revenue" value={48320} prefix="$" change={12.5} icon={DollarSign} />
  <StatsCard title="Active Users" value={2847} change={-3.1} icon={Users} />
  <StatsCard title="Conversions" value={18.2} suffix="%" change={0} icon={BarChart2} />
  <StatsCard title="Avg. Session" value="4m 32s" change={8.4} icon={Clock} />
</div>
```

---

## 🎭 EMPTY STATE PATTERNS

```tsx
// components/common/empty-state.tsx
import { LucideIcon } from "lucide-react"
import { Button } from "@/components/ui/button"
import { motion } from "motion/react"

interface EmptyStateProps {
  icon?: LucideIcon
  illustration?: React.ReactNode
  title: string
  description?: string
  action?: { label: string; onClick: () => void; icon?: LucideIcon }
  secondaryAction?: { label: string; onClick: () => void }
  compact?: boolean
}

export function EmptyState({
  icon: Icon,
  illustration,
  title,
  description,
  action,
  secondaryAction,
  compact = false,
}: EmptyStateProps) {
  return (
    <motion.div
      initial={{ opacity: 0, scale: 0.95 }}
      animate={{ opacity: 1, scale: 1 }}
      className={cn(
        "flex flex-col items-center justify-center text-center",
        compact ? "py-8 px-4" : "py-16 px-6"
      )}
    >
      {illustration ? (
        <div className="mb-4">{illustration}</div>
      ) : Icon ? (
        <div className="mb-4 rounded-full bg-muted p-4">
          <Icon className={cn("text-muted-foreground", compact ? "h-6 w-6" : "h-10 w-10")} />
        </div>
      ) : null}

      <h3 className={cn("font-semibold text-foreground", compact ? "text-base" : "text-lg")}>
        {title}
      </h3>

      {description && (
        <p className="mt-2 text-sm text-muted-foreground max-w-sm">
          {description}
        </p>
      )}

      {(action || secondaryAction) && (
        <div className="flex gap-3 mt-6">
          {secondaryAction && (
            <Button variant="outline" onClick={secondaryAction.onClick}>
              {secondaryAction.label}
            </Button>
          )}
          {action && (
            <Button onClick={action.onClick}>
              {action.icon && <action.icon className="mr-2 h-4 w-4" />}
              {action.label}
            </Button>
          )}
        </div>
      )}
    </motion.div>
  )
}
```

---

## ⌨️ KEYBOARD SHORTCUTS

```typescript
// hooks/use-keyboard-shortcut.ts
import { useEffect } from "react"

type Modifier = "ctrl" | "meta" | "shift" | "alt"
type KeyCombo = {
  key: string
  modifiers?: Modifier[]
}

export function useKeyboardShortcut(
  combo: KeyCombo | KeyCombo[],
  callback: (e: KeyboardEvent) => void,
  options: { enabled?: boolean; preventDefault?: boolean } = {}
) {
  const { enabled = true, preventDefault = true } = options

  useEffect(() => {
    if (!enabled) return

    const combos = Array.isArray(combo) ? combo : [combo]

    const handler = (e: KeyboardEvent) => {
      const matched = combos.some(c => {
        const keyMatch = e.key.toLowerCase() === c.key.toLowerCase()
        const modMatch = (c.modifiers ?? []).every(mod => {
          if (mod === "ctrl") return e.ctrlKey
          if (mod === "meta") return e.metaKey
          if (mod === "shift") return e.shiftKey
          if (mod === "alt") return e.altKey
          return false
        })
        const noExtraModifiers = !c.modifiers?.length || (
          e.ctrlKey === (c.modifiers.includes("ctrl")) &&
          e.metaKey === (c.modifiers.includes("meta")) &&
          e.shiftKey === (c.modifiers.includes("shift")) &&
          e.altKey === (c.modifiers.includes("alt"))
        )
        return keyMatch && modMatch
      })

      if (matched) {
        if (preventDefault) e.preventDefault()
        callback(e)
      }
    }

    window.addEventListener("keydown", handler)
    return () => window.removeEventListener("keydown", handler)
  }, [enabled, callback])
}

// Usage
useKeyboardShortcut({ key: "k", modifiers: ["meta"] }, () => setCommandOpen(true))
useKeyboardShortcut({ key: "s", modifiers: ["meta"] }, () => handleSave())
useKeyboardShortcut({ key: "Escape" }, () => handleClose(), { enabled: isOpen })
```
