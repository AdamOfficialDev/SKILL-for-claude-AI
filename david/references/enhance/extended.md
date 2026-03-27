# DAVID — UX Enhancement Reference: Extended (E7–E12)

Extended enhancement sub-systems and delivery format for Scanner EN.
Load for: feedback/communication, mobile experience, dark mode polish, performance perception, accessibility delight, data display, delivery format.

## Table of Contents
1. [E7 — Feedback & Communication Upgrader](#e7--feedback--communication-upgrader)
2. [E8 — Mobile Experience Enhancer](#e8--mobile-experience-enhancer)
3. [E9 — Dark Mode Polish](#e9--dark-mode-polish)
4. [E10 — Performance Perception Enhancer](#e10--performance-perception-enhancer)
5. [E11 — Accessibility Delight Layer](#e11--accessibility-delight-layer)
6. [E12 — Data Display Enhancer](#e12--data-display-enhancer)
7. [Enhancement Delivery Format](#enhancement-delivery-format)

---

## E7 — Feedback & Communication Upgrader

### Toast System Enhancement

```jsx
// 📊 Current — basic toast.success('Done')
// 🚀 DAVID ENHANCE — Rich contextual toasts

function RichToast({ type, title, description, action, icon: CustomIcon }) {
  const config = {
    success: { icon: CheckCircle, color: 'text-green-600', bg: 'bg-green-50 dark:bg-green-950 border-green-200 dark:border-green-800' },
    error:   { icon: XCircle,     color: 'text-red-600',   bg: 'bg-red-50 dark:bg-red-950 border-red-200 dark:border-red-800' },
    warning: { icon: AlertCircle, color: 'text-yellow-600', bg: 'bg-yellow-50 dark:bg-yellow-950 border-yellow-200 dark:border-yellow-800' },
    info:    { icon: Info,        color: 'text-blue-600',  bg: 'bg-blue-50 dark:bg-blue-950 border-blue-200 dark:border-blue-800' },
  }[type];

  const Icon = CustomIcon || config.icon;

  return (
    <div className={`flex items-start gap-3 p-4 rounded-xl border shadow-lg ${config.bg}`}>
      <Icon size={20} className={`mt-0.5 flex-shrink-0 ${config.color}`} />
      <div className="flex-1 min-w-0">
        <p className="text-sm font-medium text-gray-900 dark:text-gray-100">{title}</p>
        {description && (
          <p className="mt-0.5 text-sm text-gray-600 dark:text-gray-400">{description}</p>
        )}
        {action && (
          <button onClick={action.onClick} className="mt-2 text-sm font-medium text-blue-600 hover:text-blue-700">
            {action.label}
          </button>
        )}
      </div>
    </div>
  );
}

// Rich usage:
// toast.custom(<RichToast
//   type="success"
//   title="Project created"
//   description="'Alpha Dashboard' is ready to use"
//   action={{ label: 'Open project', onClick: () => router.push('/projects/alpha') }}
// />)
```

### Inline Feedback Enhancement

```jsx
// 🚀 DAVID ENHANCE — Contextual inline feedback
function SaveButton({ onSave, label = "Save changes" }) {
  const [status, setStatus] = useState('idle'); // idle | saving | saved | error

  async function handleSave() {
    setStatus('saving');
    try {
      await onSave();
      setStatus('saved');
      setTimeout(() => setStatus('idle'), 3000);
    } catch {
      setStatus('error');
      setTimeout(() => setStatus('idle'), 5000);
    }
  }

  const variants = {
    idle:   { label, icon: null, className: 'btn-primary' },
    saving: { label: 'Saving...', icon: <Spinner size={14} />, className: 'btn-primary opacity-70 cursor-wait' },
    saved:  { label: 'Saved',    icon: <Check size={14} />,   className: 'btn-success' },
    error:  { label: 'Failed — Try again', icon: <AlertCircle size={14} />, className: 'btn-danger' },
  }[status];

  return (
    <button
      onClick={handleSave}
      disabled={status === 'saving'}
      className={`flex items-center gap-2 ${variants.className}`}
    >
      {variants.icon}
      {variants.label}
    </button>
  );
}
```

---

## E8 — Mobile Experience Enhancer

### Bottom Sheet Pattern

```jsx
// 🚀 DAVID ENHANCE — Bottom sheet for mobile actions (vs dropdown)
function BottomSheet({ isOpen, onClose, title, children }) {
  return (
    <>
      {/* Backdrop */}
      {isOpen && (
        <div
          className="fixed inset-0 bg-black/40 z-40 md:hidden"
          onClick={onClose}
        />
      )}

      {/* Sheet */}
      <div
        role="dialog"
        aria-label={title}
        aria-modal="true"
        className={`fixed inset-x-0 bottom-0 z-50 bg-white dark:bg-gray-900
          rounded-t-2xl shadow-xl transition-transform duration-300 ease-out md:hidden
          ${isOpen ? 'translate-y-0' : 'translate-y-full'}`}
      >
        {/* Drag handle */}
        <div className="flex justify-center pt-3 pb-1">
          <div className="w-10 h-1 rounded-full bg-gray-300 dark:bg-gray-600" />
        </div>

        {/* Safe area bottom padding */}
        <div style={{ paddingBottom: 'env(safe-area-inset-bottom)' }}>
          <div className="px-4 pt-2 pb-6">
            <h2 className="text-base font-semibold mb-4">{title}</h2>
            {children}
          </div>
        </div>
      </div>
    </>
  );
}
```

### Swipe-to-Delete

```jsx
// 🚀 DAVID ENHANCE — Swipe actions for list items
function SwipeableListItem({ onDelete, onArchive, children }) {
  const [offset, setOffset] = useState(0);
  const [isDragging, setIsDragging] = useState(false);
  const startX = useRef(0);

  const THRESHOLD = 80; // px to trigger action

  function handleTouchStart(e) {
    startX.current = e.touches[0].clientX;
    setIsDragging(true);
  }

  function handleTouchMove(e) {
    if (!isDragging) return;
    const diff = e.touches[0].clientX - startX.current;
    setOffset(Math.max(-160, Math.min(0, diff))); // Left only, max -160px
  }

  function handleTouchEnd() {
    setIsDragging(false);
    if (offset < -THRESHOLD * 1.5) {
      onDelete();  // Full swipe = delete
    } else if (offset < -THRESHOLD) {
      setOffset(-80); // Partial = reveal actions
    } else {
      setOffset(0); // Snap back
    }
  }

  return (
    <div className="relative overflow-hidden">
      {/* Action buttons behind */}
      <div className="absolute right-0 inset-y-0 flex">
        <button onClick={onArchive} className="w-20 bg-blue-500 text-white text-xs font-medium flex items-center justify-center">
          Archive
        </button>
        <button onClick={onDelete} className="w-20 bg-red-500 text-white text-xs font-medium flex items-center justify-center">
          Delete
        </button>
      </div>

      {/* Content layer */}
      <div
        style={{ transform: `translateX(${offset}px)`, transition: isDragging ? 'none' : 'transform 200ms ease-out' }}
        onTouchStart={handleTouchStart}
        onTouchMove={handleTouchMove}
        onTouchEnd={handleTouchEnd}
        className="relative bg-white dark:bg-gray-900"
      >
        {children}
      </div>
    </div>
  );
}
```

---

## E9 — Dark Mode Polish

### Image Treatment in Dark Mode

```jsx
// 🚀 DAVID ENHANCE — Images in dark mode
function SmartImage({ src, alt, ...props }) {
  return (
    <img
      src={src}
      alt={alt}
      // Slightly dim images in dark mode — harsh bright images feel wrong in dark UI
      className="dark:brightness-90 dark:contrast-95 transition-[filter] duration-200"
      {...props}
    />
  );
}

// For logos and icons that are dark-only:
function AdaptiveLogo({ lightSrc, darkSrc, alt }) {
  return (
    <>
      <img src={lightSrc} alt={alt} className="dark:hidden" />
      <img src={darkSrc}  alt={alt} className="hidden dark:block" />
    </>
  );
}
```

### Shadow System for Dark Mode

```css
/* 🚀 DAVID ENHANCE — Proper shadow system for both modes */
:root {
  --shadow-sm: 0 1px 2px rgba(0,0,0,0.05), 0 1px 3px rgba(0,0,0,0.1);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.06);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.1), 0 4px 6px rgba(0,0,0,0.05);
  --shadow-xl: 0 20px 25px rgba(0,0,0,0.1), 0 10px 10px rgba(0,0,0,0.04);
}

.dark {
  /* Darker, more pronounced shadows for dark backgrounds */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.3);
  --shadow-md: 0 4px 6px rgba(0,0,0,0.4);
  --shadow-lg: 0 10px 15px rgba(0,0,0,0.5);
  --shadow-xl: 0 20px 25px rgba(0,0,0,0.6);
  /* Also add subtle highlight on top edge in dark mode */
  /* box-shadow: var(--shadow-md), inset 0 1px 0 rgba(255,255,255,0.05); */
}
```

---

## E10 — Performance Perception Enhancer

### Optimistic UI Patterns

```jsx
// 🚀 DAVID ENHANCE — Optimistic updates for instant-feel
function LikeButton({ postId, initialLiked, initialCount }) {
  const [liked, setLiked] = useState(initialLiked);
  const [count, setCount] = useState(initialCount);
  const [isAnimating, setIsAnimating] = useState(false);

  async function handleLike() {
    // 1. Immediate visual feedback (< 16ms)
    const newLiked = !liked;
    setLiked(newLiked);
    setCount(c => c + (newLiked ? 1 : -1));
    setIsAnimating(true);
    setTimeout(() => setIsAnimating(false), 300);

    // 2. API call in background
    try {
      await api.toggleLike(postId);
    } catch {
      // 3. Rollback only on failure
      setLiked(liked);
      setCount(initialCount);
      toast.error("Couldn't update — try again");
    }
  }

  return (
    <button
      onClick={handleLike}
      aria-label={liked ? 'Unlike' : 'Like'}
      aria-pressed={liked}
      className={`flex items-center gap-1.5 transition-colors
        ${liked ? 'text-red-500' : 'text-gray-400 hover:text-red-400'}`}
    >
      <Heart
        size={16}
        fill={liked ? 'currentColor' : 'none'}
        className={isAnimating ? 'animate-[heart-pop_300ms_ease-out]' : ''}
      />
      <span className="text-sm tabular-nums">{count.toLocaleString()}</span>
    </button>
  );
}
// @keyframes heart-pop { 0%{transform:scale(1)} 50%{transform:scale(1.4)} 100%{transform:scale(1)} }
```

### Instant Navigation Prefetch

```jsx
// 🚀 DAVID ENHANCE — Prefetch on hover for instant navigation feel
function PrefetchLink({ href, children, ...props }) {
  const router = useRouter();

  function handleMouseEnter() {
    router.prefetch(href); // Next.js: prefetch on hover, not just on mount
  }

  return (
    <Link href={href} onMouseEnter={handleMouseEnter} {...props}>
      {children}
    </Link>
  );
}
// Result: Page loads "instantly" because it was already prefetched when user hovered
```

---

## E11 — Accessibility Delight Layer

### Focus Management Enhancement

```jsx
// 🚀 DAVID ENHANCE — Smooth focus ring that follows keyboard
function useFocusVisible() {
  const [isKeyboard, setIsKeyboard] = useState(false);

  useEffect(() => {
    const handleMouseDown = () => setIsKeyboard(false);
    const handleKeyDown   = () => setIsKeyboard(true);
    document.addEventListener('mousedown', handleMouseDown);
    document.addEventListener('keydown',   handleKeyDown);
    return () => {
      document.removeEventListener('mousedown', handleMouseDown);
      document.removeEventListener('keydown',   handleKeyDown);
    };
  }, []);

  return isKeyboard;
}

// Global CSS for beautiful keyboard focus
// :focus-visible {
//   outline: 2px solid var(--color-primary);
//   outline-offset: 2px;
//   border-radius: 4px;
//   transition: outline-offset 100ms ease;
// }
// :focus:not(:focus-visible) { outline: none; }
```

### Skip Links Enhancement

```jsx
// 🚀 DAVID ENHANCE — Visible, well-designed skip links
function SkipLinks() {
  return (
    <div className="sr-only focus-within:not-sr-only">
      <a
        href="#main-content"
        className="fixed top-4 left-4 z-[9999] px-4 py-2 bg-blue-600 text-white text-sm font-medium rounded-lg shadow-lg
          focus:outline-none focus:ring-2 focus:ring-blue-300 focus:ring-offset-2
          animate-[fade-in_150ms_ease]"
      >
        Skip to main content
      </a>
      <a
        href="#main-nav"
        className="fixed top-4 left-48 z-[9999] px-4 py-2 bg-blue-600 text-white text-sm font-medium rounded-lg shadow-lg
          focus:outline-none focus:ring-2 focus:ring-blue-300 focus:ring-offset-2"
      >
        Skip to navigation
      </a>
    </div>
  );
}
```

---

## E12 — Data Display Enhancer

### Number Formatting System

```jsx
// 🚀 DAVID ENHANCE — Smart number display
function StatCard({ label, value, change, format = 'number' }) {
  function formatValue(v) {
    if (format === 'currency') return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', notation: 'compact' }).format(v);
    if (format === 'percent') return `${v.toFixed(1)}%`;
    if (v >= 1_000_000) return `${(v / 1_000_000).toFixed(1)}M`;
    if (v >= 1_000)     return `${(v / 1_000).toFixed(1)}K`;
    return v.toLocaleString();
  }

  const isPositive = change > 0;

  return (
    <div className="card-default p-5">
      <p className="text-sm text-gray-500">{label}</p>
      <p className="mt-1 text-3xl font-bold tracking-tight">
        <AnimatedNumber value={value} format={formatValue} />
      </p>
      {change !== undefined && (
        <div className={`mt-2 flex items-center gap-1 text-sm font-medium
          ${isPositive ? 'text-green-600' : 'text-red-600'}`}>
          {isPositive ? <TrendingUp size={14} /> : <TrendingDown size={14} />}
          {Math.abs(change).toFixed(1)}% vs last period
        </div>
      )}
    </div>
  );
}
```

### Progress Indicators

```jsx
// 🚀 DAVID ENHANCE — Rich progress visualization
function ProgressBar({ value, max, label, showLabel = true, variant = 'default' }) {
  const percent = Math.min(100, (value / max) * 100);
  const variants = {
    default: 'bg-blue-600',
    success: 'bg-green-500',
    warning: 'bg-yellow-500',
    danger:  'bg-red-500',
    // Auto-color based on value
    auto: percent >= 75 ? 'bg-red-500' : percent >= 50 ? 'bg-yellow-500' : 'bg-blue-600',
  };

  return (
    <div>
      {showLabel && (
        <div className="flex justify-between text-sm mb-1.5">
          <span className="font-medium text-gray-700 dark:text-gray-300">{label}</span>
          <span className="text-gray-500 tabular-nums">{value}/{max}</span>
        </div>
      )}
      <div
        role="progressbar"
        aria-valuenow={value}
        aria-valuemin={0}
        aria-valuemax={max}
        aria-label={label}
        className="h-2 bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden"
      >
        <div
          className={`h-full rounded-full transition-all duration-500 ease-out ${variants[variant]}`}
          style={{ width: `${percent}%` }}
        />
      </div>
    </div>
  );
}
```

---

## Enhancement Delivery Format

Setiap finding EN wajib menggunakan format ini. **Field `Evidence` adalah mandatory** — finding tanpa Evidence dianggap tidak valid dan tidak boleh dioutput.

```
🚀 ENHANCEMENT OPPORTUNITY [E1-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Scanner      : E1 — Loading State Upgrader
Component    : UserList.tsx (line 24-31)
Evidence     : Line 27: `if (loading) return <Spinner className="..." />`
               Search: skeleton, Skeleton, animate-pulse, shimmer, LoadingSkeleton → NOT FOUND (1–247 baris)
               → CONFIRMED: Basic spinner only, tidak ada skeleton implementation
Current Level: Basic (Level 1 — spinner only) — VERIFIED dari line 27
Target Level : Good (Level 3 — skeleton + shimmer)

Current UX:
  ❌ Generic spinner in center (line 27 — confirmed)
  ❌ Layout shifts when content loads (no skeleton = no layout reservation)
  ❌ No sense of content shape coming
  ❌ Feels slow even when fast

Enhanced UX:
  ✅ Skeleton matches exact content layout
  ✅ Shimmer animation signals "loading"
  ✅ Zero layout shift on content arrival
  ✅ Perceived 30-40% faster

Fix Confidence  : ✅ SAFE TO APPLY — visual only, no logic change
Effort          : Low (15 min)
UX Impact       : High — first impression of all list data

[DAVID provides complete implementation below]
```

**Evidence format untuk upgrade (fitur SUDAH ADA, bisa dinaikkan levelnya):**

```
Evidence     : Line 45-67: UserListSkeleton component FOUND — animate-pulse digunakan (Level 2).
               Search: shimmer, background-gradient, background-position → NOT FOUND.
               → Skeleton ada, shimmer belum: upgrade Level 2 → Level 3 tersedia.
Current Level: Functional (Level 2 — skeleton dengan pulse) — VERIFIED dari line 45-67
Target Level : Good (Level 3 — skeleton + shimmer animation)
```

**Enhancement impact rating:**

| Rating | Criteria |
|--------|---------|
| 🔥 High | First impression, primary user flows, >80% of sessions see this |
| ⚡ Medium | Secondary flows, >40% of sessions |
| 💡 Low | Edge cases, power users, <20% of sessions |

**DAVID always gives effort estimate:** Low (<30min) / Medium (30min-2h) / High (>2h)

