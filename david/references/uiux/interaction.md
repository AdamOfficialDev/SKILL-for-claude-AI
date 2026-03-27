# DAVID — UI/UX Audit: Interaction Patterns

Interaction and feedback patterns for Scanner O.
Load for: forms, mobile, animation, dark mode, skeleton screens, empty states, microcopy, performance UX, responsive design.

## Table of Contents
1. [Form UX Deep Patterns](#form-ux-deep-patterns)
2. [Mobile UX Patterns](#mobile-ux-patterns)
3. [Animation Easing Reference](#animation-easing-reference)
4. [Dark Mode Implementation](#dark-mode-implementation)
5. [Skeleton Screen Patterns](#skeleton-screen-patterns)
6. [Empty State Templates](#empty-state-templates)
7. [Microcopy Patterns](#microcopy-patterns)
8. [Performance UX Metrics](#performance-ux-metrics)
9. [Responsive Design Patterns](#responsive-design-patterns)

---

## Form UX Deep Patterns

### Input Field Anatomy

```
┌────────────────────────────────────┐
│ Label text               (optional) │  ← Always ABOVE input
│ Hint text (additional context)      │  ← Optional, below label
├────────────────────────────────────┤
│ Input field                         │  ← The actual control
│ Placeholder (example value only)    │
└────────────────────────────────────┘
  Error message (appears on blur)      ← Below input, in error state
  Character count (if maxlength)       ← Below input, right-aligned
```

### Validation Timing Matrix

| Trigger | Use Case | Never Use For |
|---------|---------|---------------|
| `onBlur` | All standard validation | — |
| `onChange` after first error | Re-validation after user corrects | Initial validation |
| `onSubmit` | Cross-field validation | Single field validation |
| `onMount` | Pre-filled data that may be invalid | Fresh empty forms |

### Password Field Requirements

```jsx
// ✅ Complete password field with all UX requirements
function PasswordField({ label, value, onChange, showStrength }) {
  const [visible, setVisible] = useState(false);
  
  return (
    <div>
      <label htmlFor="password">{label}</label>
      <div style={{ position: 'relative' }}>
        <input
          id="password"
          type={visible ? 'text' : 'password'}
          value={value}
          onChange={onChange}
          autoComplete="current-password"  // ← Allows password managers
          aria-describedby={showStrength ? 'pw-strength' : undefined}
        />
        <button
          type="button"           // ← Must be type="button" or it submits form
          onClick={() => setVisible(v => !v)}
          aria-label={visible ? 'Hide password' : 'Show password'}
          style={{ position: 'absolute', right: 12, top: '50%', transform: 'translateY(-50%)' }}
        >
          {visible ? <EyeOff size={16} /> : <Eye size={16} />}
        </button>
      </div>
      {showStrength && (
        <div id="pw-strength" role="status" aria-live="polite">
          <PasswordStrengthBar value={value} />
        </div>
      )}
    </div>
  );
}
```

### Multi-Step Form Requirements

```jsx
// ✅ Multi-step form with all required UX elements
function MultiStepForm() {
  return (
    <>
      {/* Progress indicator — required */}
      <nav aria-label="Form progress">
        <ol>
          {steps.map((step, i) => (
            <li key={i} aria-current={i === currentStep ? 'step' : undefined}>
              {step.label}
            </li>
          ))}
        </ol>
      </nav>
      
      {/* Step content */}
      <form onSubmit={handleSubmit}>
        {/* Step [currentStep] content */}
        
        {/* Navigation — BOTH back and forward always visible */}
        <div>
          {currentStep > 0 && (
            <button type="button" onClick={goBack}>
              Back  {/* Never hide the back button */}
            </button>
          )}
          <button type={isLastStep ? 'submit' : 'button'} onClick={goForward}>
            {isLastStep ? 'Submit' : 'Continue'}
          </button>
        </div>
      </form>
      
      {/* Auto-save indicator if applicable */}
      {lastSaved && <p aria-live="polite">Draft saved {lastSaved}</p>}
    </>
  );
}
```

---

## Mobile UX Patterns

### Touch Target Reference

| Element | Minimum | Recommended | Implementation |
|---------|---------|-------------|----------------|
| Icon button | 44×44px | 48×48px | `min-w-[44px] min-h-[44px]` |
| Text link | 44px tall | — | `py-2 inline-block` |
| List item tap area | Full row height | — | `py-3` minimum |
| Floating action button | 56×56px | 56×56px | Standard FAB spec |
| Bottom tab bar item | Full bar height × 1/N | — | Use `flex-1` |
| Checkbox/radio | 44×44px touch area | — | Use `appearance-none` + pseudo-element |

### Safe Area Insets (Notch / Home Indicator)

```css
/* ✅ Respect device safe areas */
.bottom-nav {
  padding-bottom: env(safe-area-inset-bottom);
  /* Ensures content doesn't hide behind home indicator on iPhone */
}

.header {
  padding-top: env(safe-area-inset-top);
  /* Respects notch/dynamic island */
}

/* Full safe area padding utility */
.safe-padding {
  padding-top: env(safe-area-inset-top);
  padding-right: env(safe-area-inset-right);
  padding-bottom: env(safe-area-inset-bottom);
  padding-left: env(safe-area-inset-left);
}
```

### Mobile Input Keyboard Types

```html
<!-- DAVID checks these are set correctly on mobile forms -->
<input type="email"   inputmode="email">     <!-- Email keyboard -->
<input type="tel"     inputmode="tel">       <!-- Phone keyboard -->
<input type="number"  inputmode="numeric">   <!-- Number pad -->
<input type="text"    inputmode="decimal">   <!-- Decimal number (prices) -->
<input type="text"    inputmode="url">       <!-- URL keyboard -->
<input type="search"  inputmode="search">    <!-- Search keyboard with "Go" -->
```

### 100vh Problem

```css
/* ❌ Classic 100vh bug — viewport changes when mobile browser chrome shows/hides */
.full-screen { height: 100vh; }  /* Jumps when address bar appears/disappears */

/* ✅ Solution hierarchy */
.full-screen { 
  height: 100vh;    /* Fallback for old browsers */
  height: 100dvh;   /* Dynamic viewport height — best solution */
}

/* Alternative: JS approach for older browser support */
/* document.documentElement.style.setProperty('--vh', `${window.innerHeight * 0.01}px`); */
/* .full-screen { height: calc(var(--vh, 1vh) * 100); } */
```

---

## Animation Easing Reference

### Easing Curve Selection Guide

```css
/* ENTERING elements (appear, expand, slide in) */
/* → ease-out: starts fast, decelerates — feels like arriving */
transition-timing-function: cubic-bezier(0.0, 0.0, 0.2, 1);  /* Material ease-out */
transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1); /* Spring — slight overshoot */

/* EXITING elements (disappear, collapse, slide out) */
/* → ease-in: starts slow, accelerates — feels like leaving */
transition-timing-function: cubic-bezier(0.4, 0.0, 1, 1);  /* Material ease-in */

/* MOVING within screen (reorder, shift position) */
/* → ease-in-out: smooth acceleration and deceleration */
transition-timing-function: cubic-bezier(0.4, 0.0, 0.2, 1);  /* Material standard */

/* MICRO-INTERACTIONS (button click, toggle, hover) */
/* → ease-out, very short */
transition-timing-function: ease-out;
```

### Duration Reference

```css
/* Micro-interactions (hover, toggle, click feedback) */
--duration-micro: 100ms;

/* Small components (tooltip show/hide, dropdown open) */
--duration-fast: 150ms;

/* Medium components (modal, drawer, sheet) */
--duration-moderate: 250ms;

/* Page transitions */
--duration-slow: 350ms;

/* NEVER: > 500ms for UI interactions (feels laggy) */
/* NEVER: < 80ms for visual feedback (imperceptible) */
```

### Properties to Animate (GPU-Friendly)

```css
/* ✅ Compositor-layer properties — no layout recalc, smooth 60fps */
transform: translateX() translateY() scale() rotate();
opacity: 0 to 1;
filter: blur() brightness();  /* Use sparingly */

/* ❌ Layout-triggering properties — avoid animating these */
width, height          /* Triggers layout */
top, left, right, bottom  /* Use transform instead */
margin, padding         /* Triggers layout */
border-width            /* Triggers layout */
font-size               /* Triggers layout */
background-color        /* Use opacity over a colored overlay instead */
```

### `prefers-reduced-motion` Implementation

```css
/* Pattern 1: Disable all animation */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* Pattern 2: Provide alternative (better UX) */
.notification {
  animation: slide-in 300ms ease-out;
}
@media (prefers-reduced-motion: reduce) {
  .notification {
    animation: fade-in 150ms ease-out;  /* Still animated, just no movement */
  }
}

/* Pattern 3: Opt-in pattern (safest) */
@media (prefers-reduced-motion: no-preference) {
  .hero { animation: float 3s ease-in-out infinite; }
}
```

---

## Dark Mode Implementation

### Color Token Strategy

```css
/* Layer 1: Raw palette (never use in components) */
--blue-500: #3b82f6;
--blue-600: #2563eb;
--gray-900: #111827;

/* Layer 2: Semantic tokens (use in components) */
:root {
  --color-bg-primary:    #ffffff;
  --color-bg-secondary:  #f9fafb;
  --color-bg-elevated:   #ffffff;
  --color-text-primary:  #111827;
  --color-text-secondary:#6b7280;
  --color-border:        #e5e7eb;
  --color-action:        #2563eb;
  --color-action-text:   #ffffff;
  --color-danger:        #dc2626;
  --color-success:       #16a34a;
}

@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-primary:    #111827;
    --color-bg-secondary:  #1f2937;
    --color-bg-elevated:   #1f2937;  /* NOT #ffffff */
    --color-text-primary:  #f9fafb;
    --color-text-secondary:#9ca3af;
    --color-border:        #374151;
    --color-action:        #60a5fa;  /* Lighter blue for dark bg */
    --color-action-text:   #111827;
    --color-danger:        #f87171;  /* Lighter red for dark bg */
    --color-success:       #4ade80;  /* Lighter green for dark bg */
  }
}
```

### Elevation in Dark Mode

In light mode, elevation = shadow. In dark mode, elevation = lighter background.

```css
/* Light mode: shadow creates depth */
.card-base     { background: #ffffff; box-shadow: 0 1px 3px rgba(0,0,0,0.1); }
.card-elevated { background: #ffffff; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.card-floating { background: #ffffff; box-shadow: 0 10px 15px rgba(0,0,0,0.1); }

/* Dark mode: lighter bg creates depth, shadow becomes subtle */
@media (prefers-color-scheme: dark) {
  .card-base     { background: #1e2433; box-shadow: 0 1px 3px rgba(0,0,0,0.4); }
  .card-elevated { background: #242b3d; box-shadow: 0 4px 6px rgba(0,0,0,0.4); }
  .card-floating { background: #2a3347; box-shadow: 0 10px 15px rgba(0,0,0,0.4); }
}
```

### Tailwind Dark Mode Checklist

For every color class, DAVID verifies a `dark:` counterpart exists:

```html
<!-- ❌ Incomplete dark mode -->
<div class="bg-white text-gray-900 border border-gray-200">
  <h2 class="text-gray-800">Title</h2>
  <p class="text-gray-600">Body</p>
</div>

<!-- ✅ Complete dark mode -->
<div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100 border border-gray-200 dark:border-gray-700">
  <h2 class="text-gray-800 dark:text-gray-200">Title</h2>
  <p class="text-gray-600 dark:text-gray-400">Body</p>
</div>
```

---

## Skeleton Screen Patterns

### Skeleton vs Spinner Decision Tree

```
Is this a FULL PAGE / LARGE SECTION loading? 
  YES → Skeleton screen (mirrors final layout)

Is this a SMALL INLINE element (single value, 1 line)?
  YES → Inline spinner or "..." placeholder

Is this a BUTTON ACTION (submit, save, delete)?
  YES → Spinner inside button, keep button disabled

Is this BACKGROUND SYNC (auto-save, sync)?
  YES → Subtle top progress bar or status dot
  
Is this a LIST / TABLE loading?
  YES → Skeleton rows (5-7 rows, mirroring real content)
```

### React Skeleton Patterns

```jsx
// ✅ Content-shaped skeleton
function CardSkeleton() {
  return (
    <div className="animate-pulse">
      {/* Avatar */}
      <div className="rounded-full bg-gray-200 dark:bg-gray-700 h-12 w-12" />
      
      {/* Title line */}
      <div className="mt-4 h-4 bg-gray-200 dark:bg-gray-700 rounded w-3/4" />
      
      {/* Body lines */}
      <div className="mt-2 h-3 bg-gray-200 dark:bg-gray-700 rounded w-full" />
      <div className="mt-2 h-3 bg-gray-200 dark:bg-gray-700 rounded w-5/6" />
      
      {/* CTA button shape */}
      <div className="mt-4 h-9 bg-gray-200 dark:bg-gray-700 rounded w-24" />
    </div>
  );
}

// ✅ Table row skeleton
function TableRowSkeleton({ rows = 5 }) {
  return Array.from({ length: rows }).map((_, i) => (
    <tr key={i} className="animate-pulse">
      <td className="px-4 py-3">
        <div className="h-4 bg-gray-200 dark:bg-gray-700 rounded w-24" />
      </td>
      <td className="px-4 py-3">
        <div className="h-4 bg-gray-200 dark:bg-gray-700 rounded w-32" />
      </td>
      <td className="px-4 py-3">
        <div className="h-4 bg-gray-200 dark:bg-gray-700 rounded w-16" />
      </td>
    </tr>
  ));
}
```

---

## Empty State Templates

### Required Elements

```
Every empty state MUST have:
1. Visual element (illustration, icon, or emoji)
2. Headline — what is empty + why
3. Body text — what the user can do
4. CTA — at least one action button

Optional but recommended:
- Search suggestion if empty is due to filter/search
- "Clear filters" button if user applied filters
```

### Empty State by Context

```jsx
// ✅ Empty list (first time, no data yet)
function EmptyProjectList({ onCreateProject }) {
  return (
    <div className="text-center py-16">
      <FolderOpen className="mx-auto h-16 w-16 text-gray-300" aria-hidden="true" />
      <h3 className="mt-4 text-lg font-medium text-gray-900 dark:text-gray-100">
        No projects yet
      </h3>
      <p className="mt-2 text-sm text-gray-500">
        Get started by creating your first project.
      </p>
      <button onClick={onCreateProject} className="mt-6 btn-primary">
        Create project
      </button>
    </div>
  );
}

// ✅ Empty search results
function EmptySearchResults({ query, onClear }) {
  return (
    <div className="text-center py-16">
      <Search className="mx-auto h-12 w-12 text-gray-300" aria-hidden="true" />
      <h3 className="mt-4 text-lg font-medium">
        No results for "{query}"
      </h3>
      <p className="mt-2 text-sm text-gray-500">
        Try a different search term or check your spelling.
      </p>
      <button onClick={onClear} className="mt-4 btn-secondary">
        Clear search
      </button>
    </div>
  );
}

// ✅ Empty filtered results
function EmptyFilteredResults({ onClearFilters }) {
  return (
    <div className="text-center py-12">
      <FilterX className="mx-auto h-12 w-12 text-gray-300" aria-hidden="true" />
      <h3 className="mt-4 text-base font-medium">No matches found</h3>
      <p className="mt-1 text-sm text-gray-500">
        No items match your current filters.
      </p>
      <button onClick={onClearFilters} className="mt-4 btn-secondary">
        Clear all filters
      </button>
    </div>
  );
}
```

---

## Microcopy Patterns

### Button Label Formula

```
[Verb] + [Object] = Clear button label

❌ Vague    → ✅ Specific
Submit      → Create account / Send message / Book appointment
Save        → Save changes / Save draft / Save to library
Yes         → Yes, delete / Yes, cancel subscription / Yes, archive
No          → Keep it / Keep subscription / Undo
Cancel      → Don't delete / Go back / Discard changes
Continue    → Continue to payment / Next step / Set up profile
OK          → Got it / I understand / Sounds good
```

### Error Message Formula

```
[What happened] + [Why] + [How to fix]

❌ "Error"
✅ "Couldn't save — check your internet connection and try again"

❌ "Invalid"
✅ "Phone number must be 10 digits (e.g. 0812-3456-7890)"

❌ "Not found"
✅ "We couldn't find an account with that email. Try signing up instead?"
```

### Toast Notification Guidelines

```javascript
// Duration by type
const TOAST_DURATION = {
  success: 3000,    // 3s — user just needs to know it worked
  info:    4000,    // 4s — slightly more to read
  warning: 5000,    // 5s — needs attention
  error:   0,       // Persistent — requires user action to dismiss
};

// Content guidelines:
// ✅ "Message sent"  (success — past tense, confirms action)
// ✅ "Saving..."     (info — present tense, shows progress)
// ✅ "Low storage"   (warning — noun phrase, state)
// ✅ "Couldn't send. Check your connection." (error — what + why + implicit action)

// ❌ "Success!"      (what succeeded?)
// ❌ "Error"         (what error?)
// ❌ "Done."         (what's done?)
```

### Confirmation Dialog Formula

```
Title:   [Action verb] + [Object]?
Body:    Clear consequence in one sentence. No "Are you sure?" ever.
Buttons: [Confirm action verb] + [Cancel/Keep verb]

❌ Bad:
Title: "Are you sure?"
Body:  "This cannot be undone."
Buttons: [Yes] [No]

✅ Good:
Title: "Delete 'Project Alpha'?"  
Body:  "This will permanently remove all 47 files and team access. No undo."
Buttons: [Delete project] [Keep project]

✅ Good (lighter action):
Title: "Log out of all devices?"
Body:  "You'll need to sign in again on all your other devices."
Buttons: [Log out everywhere] [Cancel]
```

---

## Performance UX Metrics

### Core Web Vitals Thresholds

| Metric | Good | Needs Improvement | Poor |
|--------|------|------------------|------|
| LCP (Largest Contentful Paint) | ≤2.5s | 2.5–4.0s | >4.0s |
| FID (First Input Delay) | ≤100ms | 100–300ms | >300ms |
| CLS (Cumulative Layout Shift) | ≤0.1 | 0.1–0.25 | >0.25 |
| INP (Interaction to Next Paint) | ≤200ms | 200–500ms | >500ms |

### CLS Prevention Checklist

```html
<!-- ✅ Reserve space for images -->
<img src="hero.jpg" width="1200" height="630" alt="Hero image"
     style="max-width:100%; height:auto;">

<!-- ✅ Reserve space for video -->
<div style="aspect-ratio: 16/9;">
  <video src="intro.mp4"></video>
</div>

<!-- ✅ Reserve space for ads/embeds -->
<div style="min-height: 250px;">
  <!-- Ad content loads here -->
</div>

<!-- ✅ Avoid injecting content above existing content -->
<!-- Never: pushing existing content down when banner appears -->
<!-- Use: overlay/fixed position for notifications, cookies banners -->
```

### Optimistic UI Patterns

```javascript
// ✅ Optimistic update — immediate feedback, rollback on error
async function handleLike(postId) {
  // 1. Immediately update UI
  setLiked(true);
  setLikeCount(c => c + 1);
  
  try {
    // 2. Send to server
    await api.likePost(postId);
  } catch (error) {
    // 3. Rollback on failure
    setLiked(false);
    setLikeCount(c => c - 1);
    toast.error("Couldn't like post. Try again.");
  }
}

// ✅ Optimistic delete
async function handleDelete(itemId) {
  // Store original for rollback
  const originalItems = [...items];
  
  // Immediately remove from list
  setItems(items.filter(i => i.id !== itemId));
  
  try {
    await api.deleteItem(itemId);
  } catch (error) {
    // Restore on failure
    setItems(originalItems);
    toast.error("Couldn't delete item. Please try again.");
  }
}
```

---

## Responsive Design Patterns

### Breakpoint System Reference

```css
/* Tailwind default breakpoints — most common system */
/* sm:  640px  — landscape phone */
/* md:  768px  — tablet portrait */
/* lg:  1024px — tablet landscape / small laptop */
/* xl:  1280px — desktop */
/* 2xl: 1536px — large desktop */

/* Mobile-first (always prefer this) */
.grid {
  grid-template-columns: 1fr;             /* Mobile default */
}
@media (min-width: 640px) {
  .grid { grid-template-columns: 1fr 1fr; }  /* sm: 2 cols */
}
@media (min-width: 1024px) {
  .grid { grid-template-columns: repeat(3, 1fr); }  /* lg: 3 cols */
}
```

### Container Width Patterns

```css
/* ✅ Consistent max-width system */
.container-narrow  { max-width: 640px;  margin: 0 auto; padding: 0 16px; }
.container-default { max-width: 1024px; margin: 0 auto; padding: 0 24px; }
.container-wide    { max-width: 1280px; margin: 0 auto; padding: 0 32px; }
.container-full    { max-width: 100%;   padding: 0 40px; }

/* ❌ DAVID flags: no consistent container system */
/* Different pages using different max-widths without semantic reason */
```

### Text Overflow Safety

```css
/* ✅ Always add these to dynamic text content */
.user-content {
  overflow-wrap: break-word;  /* Break long URLs/strings */
  word-break: break-word;     /* Fallback */
  hyphens: auto;              /* Optional: hyphenate long words */
}

/* ✅ Single-line truncation */
.truncate-single {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;  /* Must have a max-width for ellipsis to work */
}

/* ✅ Multi-line clamp */
.truncate-multi-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

---

