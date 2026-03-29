# REY — Premium Design System Reference

> Design premium bukan soal biaya. Ini soal keputusan yang disengaja di setiap level.

---

## 🎨 REY DESIGN PHILOSOPHY

### The Premium Triad
```
INTENTIONAL × POLISHED × PERFORMANT = PREMIUM
```

1. **Intentional** — Setiap keputusan visual punya alasan. Warna ini kenapa? Spacing ini kenapa?
2. **Polished** — Detail yang biasanya diabaikan: hover states, loading skeletons, empty states, error states
3. **Performant** — UI yang indah tapi lambat bukan premium. Animations 60fps, no jank.

### Anti-Patterns yang WAJIB Dihindari
- ❌ Generic purple-to-blue gradient hero section
- ❌ Overused card grid layout tanpa visual hierarchy
- ❌ "Coming soon" atau placeholder content di delivery
- ❌ Animations yang ada hanya karena bisa, bukan karena useful
- ❌ Modal untuk setiap interaction — prefer inline, sheet, atau toast
- ❌ Dark mode yang hanya invert warna (bukan truly designed)
- ❌ Mobile view yang hanya "responsive" tapi tidak mobile-first

---

## 🎭 AESTHETIC DIRECTIONS

REY memilih aesthetic berdasarkan konteks project. Setiap project HARUS punya identitas yang jelas:

### 1. LUXURY MINIMAL
*Untuk: SaaS premium, portfolio profesional, fintech*
- Palette: Off-white + deep navy/charcoal + gold accent
- Typography: Serif display (Playfair, Cormorant) + Sans body (Geist, DM Sans)
- Spacing: Very generous — let content breathe
- Motion: Subtle, slow, purposeful (ease-in-out 600ms+)
- Signature: Thin borders, lots of whitespace, precise alignment

### 2. DARK TECH / CYBERPUNK
*Untuk: Dev tools, gaming, AI products, crypto*
- Palette: Near-black (#0a0a0f) + electric accent (cyan, green, purple neon)
- Typography: Mono/tech fonts (Geist Mono, JetBrains Mono, Space Mono) + Grotesk
- Pattern: Subtle grid/dot backgrounds, glow effects, glass morphism
- Motion: Snappy, fast, typewriter effects, glitch animations
- Signature: Glow shadows, code-like elements, scan lines

### 3. ORGANIC / MODERN
*Untuk: Health, wellness, food, lifestyle brands*
- Palette: Warm neutrals (cream, sand) + earth tones + sage/terracotta
- Typography: Humanist sans (Nunito, Outfit) + Optional serif accent
- Texture: Noise texture, paper grain, organic shapes (blob, waves)
- Motion: Natural, springy (Framer spring physics), scroll parallax
- Signature: Rounded corners (radius 20px+), soft shadows, organic SVGs

### 4. BOLD EDITORIAL
*Untuk: Media, news, creative agencies, fashion*
- Palette: Black + white dominant + ONE bold accent color
- Typography: Strong contrast — massive display + small body
- Layout: Asymmetric grid, overlapping elements, full-bleed images
- Motion: Dramatic reveal, scroll-triggered animations
- Signature: High contrast, unconventional grid, expressive type

### 5. CLEAN PRODUCTIVITY
*Untuk: Dashboards, admin tools, B2B SaaS, internal tools*
- Palette: White/light gray base + muted accent + semantic colors
- Typography: System-like (Geist, Inter) — clarity over character
- Density: Information-dense but organized
- Motion: Minimal — only feedback animations (loading, success, error)
- Signature: Consistent spacing system, clear hierarchy, data tables

---

## 🎨 COLOR SYSTEM

### REY's Color Architecture
```css
:root {
  /* Brand */
  --brand-50: #f0f9ff;
  --brand-100: #e0f2fe;
  --brand-500: #0ea5e9;  /* Primary */
  --brand-600: #0284c7;  /* Primary hover */
  --brand-900: #0c4a6e;
  
  /* Neutrals (NOT pure black/white) */
  --neutral-0: #fafafa;     /* Background */
  --neutral-50: #f5f5f5;
  --neutral-100: #e5e5e5;
  --neutral-200: #d4d4d4;
  --neutral-400: #a3a3a3;   /* Muted text */
  --neutral-600: #525252;   /* Secondary text */
  --neutral-800: #262626;   /* Primary text */
  --neutral-950: #0a0a0a;   /* Near black */
  
  /* Semantic */
  --success: #22c55e;
  --warning: #f59e0b;
  --error: #ef4444;
  --info: #3b82f6;
  
  /* Surface */
  --bg-primary: var(--neutral-0);
  --bg-secondary: var(--neutral-50);
  --bg-elevated: #ffffff;
  --border: var(--neutral-200);
  --border-subtle: var(--neutral-100);
}

/* Dark mode — truly designed, not inverted */
.dark {
  --bg-primary: #0a0a0f;
  --bg-secondary: #111118;
  --bg-elevated: #1a1a24;
  --border: #2a2a3a;
  --border-subtle: #1f1f2e;
  --neutral-800: #e4e4e7;  /* Text becomes light */
  --neutral-600: #a1a1aa;
  --neutral-400: #71717a;
}
```

### Color Usage Rules
- **60-30-10 Rule**: 60% neutral, 30% secondary, 10% accent
- **Contrast**: Minimum 4.5:1 for body text, 3:1 for large text (WCAG AA)
- **State colors**: Always pair color with another indicator (icon, text, pattern) — never rely on color alone

---

## 📐 SPACING & LAYOUT SYSTEM

### Spacing Scale (Tailwind compatible)
```
4px   → space-1  (micro gaps, icon padding)
8px   → space-2  (tight inline spacing)
12px  → space-3  (compact elements)
16px  → space-4  (default element spacing)
24px  → space-6  (section internal spacing)
32px  → space-8  (component spacing)
48px  → space-12 (section gaps)
64px  → space-16 (page section padding)
96px  → space-24 (hero/landing sections)
128px → space-32 (maximum section spacing)
```

### Container Widths
```
sm:  640px  (mobile breakpoint)
md:  768px  (tablet)
lg:  1024px (laptop)
xl:  1280px (desktop — DEFAULT max-width)
2xl: 1536px (wide screen — use sparingly)
```

### Typography Scale
```
xs:   12px / 1.5  (captions, labels)
sm:   14px / 1.5  (secondary text, metadata)
base: 16px / 1.6  (body text)
lg:   18px / 1.5  (lead text)
xl:   20px / 1.4  (card titles, subheadings)
2xl:  24px / 1.3  (section subheadings)
3xl:  30px / 1.2  (h3)
4xl:  36px / 1.15 (h2)
5xl:  48px / 1.1  (h1)
6xl:  60px / 1.05 (hero headings)
7xl:  72px / 1.0  (display / hero XL)
8xl:  96px / 0.95 (ultra display)
```

---

## ✍️ TYPOGRAPHY SYSTEM

### Pairing Rules
REY selalu pair dua font: **Display** (headings) + **Body** (text):

| Aesthetic | Display Font | Body Font | Fallback Stack |
|-----------|-------------|-----------|----------------|
| Luxury | Cormorant Garamond | DM Sans | serif / sans-serif |
| Tech | Space Grotesk | Geist Mono | monospace |
| Modern | Bricolage Grotesque | Inter | sans-serif |
| Editorial | Fraunces | IBM Plex Sans | serif / sans-serif |
| Organic | Lora | Nunito | serif / sans-serif |
| Productivity | Geist | Geist | sans-serif |

### Implementation (Next.js)
```typescript
// lib/fonts.ts
import { Cormorant_Garamond, DM_Sans } from 'next/font/google'

export const displayFont = Cormorant_Garamond({
  subsets: ['latin'],
  weight: ['400', '500', '600', '700'],
  variable: '--font-display',
  display: 'swap',
})

export const bodyFont = DM_Sans({
  subsets: ['latin'],
  variable: '--font-body',
  display: 'swap',
})
```

---

## ✨ MOTION & ANIMATION

### Motion Principles
1. **Purposeful**: Animasi ada untuk komunikasi, bukan dekorasi
2. **Responsive**: User yang prefer-reduced-motion harus dihormati
3. **Performant**: CSS transitions > JS animations untuk simple cases
4. **Consistent**: Gunakan easing dan duration yang konsisten

### Standard Durations
```
instant:  0ms    (no transition — immediate feedback)
micro:    100ms  (hover states, button press)
fast:     200ms  (dropdown open, tooltip)
normal:   300ms  (modal open, slide)
slow:     500ms  (page transitions, reveals)
dramatic: 800ms+ (hero animations, storytelling)
```

### Easing Functions
```css
:root {
  --ease-out: cubic-bezier(0.0, 0, 0.2, 1);      /* Elements entering screen */
  --ease-in: cubic-bezier(0.4, 0, 1, 1);          /* Elements leaving screen */
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);    /* Elements moving on screen */
  --ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1); /* Bouncy, playful */
  --ease-sharp: cubic-bezier(0.4, 0, 0.6, 1);     /* Quick, snappy */
}
```

### Framer Motion Patterns
```tsx
// Page transition
const pageVariants = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
  exit: { opacity: 0, y: -20 }
}

// Stagger children (for lists, cards)
const containerVariants = {
  animate: { transition: { staggerChildren: 0.1 } }
}

const itemVariants = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0, transition: { duration: 0.4 } }
}

// Always respect reduced motion
const { prefersReducedMotion } = useReducedMotion()
```

---

## 🧩 COMPONENT STANDARDS

### Every Component MUST Have:
- ✅ **Default state** — looks right with any content length
- ✅ **Loading state** — skeleton, spinner, atau placeholder
- ✅ **Empty state** — illustration + CTA, bukan blank space
- ✅ **Error state** — clear message + recovery action
- ✅ **Disabled state** — visually distinct, cursor not-allowed
- ✅ **Hover + Focus states** — untuk accessibility
- ✅ **Dark mode variant** — properly designed, not inverted
- ✅ **Mobile responsive** — tidak hanya mengecil, tapi re-laid-out

### Semantic HTML Rules
```tsx
// ✅ Correct
<nav aria-label="Main navigation">
<main id="main-content">
<article>
<aside aria-label="Sidebar">
<button type="button" onClick={handleClick}>

// ❌ Wrong
<div onClick={handleClick}> // should be button
<div className="nav">      // should be nav
<span className="title">  // should be h2/h3/etc
```

### shadcn/ui Customization Pattern
```tsx
// components/ui/button.tsx — extended from shadcn
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-lg font-medium transition-all focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default: "bg-brand-600 text-white hover:bg-brand-700 shadow-sm",
        outline: "border border-border bg-transparent hover:bg-neutral-50",
        ghost: "hover:bg-neutral-100 hover:text-neutral-900",
        premium: "bg-gradient-to-r from-brand-600 to-purple-600 text-white shadow-lg hover:shadow-brand-500/25 hover:scale-[1.02] active:scale-[0.98]",
      },
      size: {
        sm: "h-8 px-3 text-xs",
        default: "h-10 px-4 py-2 text-sm",
        lg: "h-12 px-6 text-base",
        xl: "h-14 px-8 text-lg",
      },
    },
  }
)
```

---

## 📱 RESPONSIVE DESIGN RULES

### Mobile-First Breakpoints
```
Default (mobile): 0px+
sm: 640px+   (large mobile / small tablet)
md: 768px+   (tablet)
lg: 1024px+  (laptop)
xl: 1280px+  (desktop)
2xl: 1536px+ (wide)
```

### Touch Targets
- Minimum tap target: **44×44px** (Apple HIG) / **48×48px** (Google Material)
- Spacing between tappable elements: minimum **8px**
- Never rely on hover for critical functionality on mobile

### Navigation Patterns
- Mobile: Bottom nav bar ATAU hamburger → full-screen menu
- Tablet: Collapsed sidebar atau top nav
- Desktop: Full sidebar ATAU top nav dengan dropdown

---

## 🖼️ IMAGE & MEDIA GUIDELINES

```tsx
// Always use next/image
import Image from 'next/image'

<Image
  src={src}
  alt={descriptiveAlt}  // Never empty for content images
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  className="object-cover"
  placeholder="blur"
  blurDataURL={blurDataURL}
/>
```

**Aspect ratios yang umum digunakan:**
- Hero images: 16:9 atau 21:9 (cinematic)
- Card thumbnails: 16:9 atau 3:2
- Avatar: 1:1
- Product images: 1:1 atau 4:3
- Blog cover: 16:9 atau 2:1

---

## 🎬 ADVANCED ANIMATION PATTERNS

### Scroll-Triggered Animations
```tsx
// useInView hook pattern
function useInView(options?: IntersectionObserverInit) {
  const ref = useRef<HTMLElement>(null)
  const [isInView, setIsInView] = useState(false)
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        setIsInView(true)
        observer.disconnect() // Animate only once
      }
    }, { threshold: 0.1, ...options })
    
    if (ref.current) observer.observe(ref.current)
    return () => observer.disconnect()
  }, [])
  
  return { ref, isInView }
}

// Usage with Framer Motion
function FeatureCard({ feature }: { feature: Feature }) {
  const { ref, isInView } = useInView()
  
  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 40 }}
      animate={isInView ? { opacity: 1, y: 0 } : {}}
      transition={{ duration: 0.6, ease: [0.25, 0.46, 0.45, 0.94] }}
    >
      {/* Content */}
    </motion.div>
  )
}
```

### Micro-Interactions Library
```tsx
// Animated counter
function AnimatedCounter({ value, duration = 1000 }: { value: number; duration?: number }) {
  const [displayed, setDisplayed] = useState(0)
  
  useEffect(() => {
    const start = performance.now()
    const startValue = displayed
    
    const update = (time: number) => {
      const elapsed = time - start
      const progress = Math.min(elapsed / duration, 1)
      // Ease out cubic
      const eased = 1 - Math.pow(1 - progress, 3)
      setDisplayed(Math.round(startValue + (value - startValue) * eased))
      
      if (progress < 1) requestAnimationFrame(update)
    }
    
    requestAnimationFrame(update)
  }, [value, duration])
  
  return <span>{displayed.toLocaleString()}</span>
}

// Button with haptic-like press effect
const PremiumButton = ({ children, onClick }: ButtonProps) => (
  <motion.button
    whileHover={{ scale: 1.02 }}
    whileTap={{ scale: 0.97 }}
    transition={{ type: 'spring', stiffness: 400, damping: 17 }}
    onClick={onClick}
    className="..."
  >
    {children}
  </motion.button>
)

// Magnetic hover effect
function MagneticButton({ children }: { children: React.ReactNode }) {
  const ref = useRef<HTMLButtonElement>(null)
  const x = useMotionValue(0)
  const y = useMotionValue(0)
  
  const handleMouseMove = (e: React.MouseEvent) => {
    const rect = ref.current?.getBoundingClientRect()
    if (!rect) return
    const centerX = rect.left + rect.width / 2
    const centerY = rect.top + rect.height / 2
    x.set((e.clientX - centerX) * 0.3)
    y.set((e.clientY - centerY) * 0.3)
  }
  
  const handleMouseLeave = () => {
    x.set(0)
    y.set(0)
  }
  
  return (
    <motion.button
      ref={ref}
      style={{ x, y }}
      transition={{ type: 'spring', stiffness: 150, damping: 15 }}
      onMouseMove={handleMouseMove}
      onMouseLeave={handleMouseLeave}
    >
      {children}
    </motion.button>
  )
}
```

---

## ♿ ACCESSIBILITY DEEP-DIVE

### Focus Management
```tsx
// Dialog focus trap
function Modal({ isOpen, onClose, children }: ModalProps) {
  const firstFocusRef = useRef<HTMLButtonElement>(null)
  
  useEffect(() => {
    if (isOpen) {
      // Store last focused element
      const lastFocused = document.activeElement as HTMLElement
      firstFocusRef.current?.focus()
      
      return () => {
        lastFocused?.focus() // Restore focus on close
      }
    }
  }, [isOpen])
  
  // Trap focus inside modal
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Escape') onClose()
    
    if (e.key === 'Tab') {
      const focusables = getFocusableElements(e.currentTarget as HTMLElement)
      const first = focusables[0]
      const last = focusables[focusables.length - 1]
      
      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault()
        last.focus()
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault()
        first.focus()
      }
    }
  }
  
  return isOpen ? (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      onKeyDown={handleKeyDown}
    >
      <button ref={firstFocusRef} onClick={onClose} aria-label="Close modal">×</button>
      <h2 id="modal-title">...</h2>
      {children}
    </div>
  ) : null
}
```

### ARIA Live Regions (Announcements)
```tsx
// Announce dynamic content to screen readers
function useAnnouncer() {
  const announcerRef = useRef<HTMLDivElement>(null)
  
  const announce = useCallback((message: string, priority: 'polite' | 'assertive' = 'polite') => {
    if (!announcerRef.current) return
    announcerRef.current.setAttribute('aria-live', priority)
    announcerRef.current.textContent = ''
    setTimeout(() => {
      if (announcerRef.current) announcerRef.current.textContent = message
    }, 50)
  }, [])
  
  const Announcer = () => (
    <div
      ref={announcerRef}
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    />
  )
  
  return { announce, Announcer }
}

// Usage
const { announce, Announcer } = useAnnouncer()
const handleSave = async () => {
  await saveData()
  announce('Changes saved successfully') // Screen reader will read this
}
```

### Color Contrast Utilities
```typescript
// Check contrast ratio (WCAG 2.1)
function getRelativeLuminance(r: number, g: number, b: number): number {
  const [rs, gs, bs] = [r, g, b].map(c => {
    const sRGB = c / 255
    return sRGB <= 0.04045 ? sRGB / 12.92 : Math.pow((sRGB + 0.055) / 1.055, 2.4)
  })
  return 0.2126 * rs + 0.7152 * gs + 0.0722 * bs
}

function getContrastRatio(color1: string, color2: string): number {
  // Parse hex to RGB and calculate
  const l1 = getRelativeLuminance(...hexToRgb(color1))
  const l2 = getRelativeLuminance(...hexToRgb(color2))
  const lighter = Math.max(l1, l2)
  const darker = Math.min(l1, l2)
  return (lighter + 0.05) / (darker + 0.05)
}

// WCAG targets:
// AA: 4.5:1 for normal text, 3:1 for large text (18px+ or 14px bold)
// AAA: 7:1 for normal text, 4.5:1 for large text
```

---

## 📊 DATA VISUALIZATION PRINCIPLES

### When to use which chart
```
Comparison over time    → Line chart
Part of whole          → Donut/Pie (max 5 segments)
Distribution           → Histogram, Box plot
Correlation            → Scatter plot
Ranking                → Horizontal bar chart
Progress to goal       → Progress bar, Gauge
Single KPI             → Number + trend indicator
Multi-dimensional      → Heatmap, Radar chart
```

### Chart.js Premium Defaults
```typescript
Chart.defaults.font.family = 'Geist, sans-serif'
Chart.defaults.color = 'hsl(0, 0%, 40%)'

const premiumLineConfig: ChartConfiguration = {
  type: 'line',
  options: {
    responsive: true,
    maintainAspectRatio: false,
    interaction: { intersect: false, mode: 'index' },
    plugins: {
      legend: { display: false },
      tooltip: {
        backgroundColor: 'hsl(0, 0%, 10%)',
        titleColor: 'hsl(0, 0%, 95%)',
        bodyColor: 'hsl(0, 0%, 70%)',
        padding: 12,
        borderRadius: 8,
      }
    },
    scales: {
      x: {
        grid: { display: false },
        border: { display: false },
      },
      y: {
        grid: { color: 'hsl(0, 0%, 90%)' },
        border: { display: false, dash: [4, 4] },
      }
    },
    elements: {
      line: { tension: 0.4, borderWidth: 2 },
      point: { radius: 0, hoverRadius: 6 }
    }
  }
}
```
