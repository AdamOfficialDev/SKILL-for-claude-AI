# DAVID — UI/UX Audit: Foundation Patterns

Core visual and component foundation for Scanner O.
Load for: typography, spacing, color tokens, component state matrix, loading states.

## Table of Contents
1. [Visual Hierarchy & Typography](#visual-hierarchy--typography)
2. [Spacing & Layout Grid](#spacing--layout-grid)
3. [Color System & Tokens](#color-system--tokens)
4. [Component State Inventory](#component-state-inventory)

---

## Visual Hierarchy & Typography

### Type Scale Systems

A consistent type scale is the foundation of visual hierarchy. Flag any codebase that doesn't use one.

**Recommended scales:**

Major Third (1.25×):
```
12px → 15px → 19px → 24px → 30px → 37px → 46px
```

Perfect Fourth (1.333×):
```
12px → 16px → 21px → 28px → 37px → 50px
```

Tailwind's default (close to Major Third):
```
text-xs   = 12px
text-sm   = 14px
text-base = 16px  ← Body default
text-lg   = 18px
text-xl   = 20px
text-2xl  = 24px
text-3xl  = 30px
text-4xl  = 36px
text-5xl  = 48px
```

**DAVID checks:** Are all font-size values from ONE of these scales? Any rogue sizes (13px, 15px, 17px, 22px) = violation.

### Line Height Guide

```css
/* Optimal line-height by use case */
.display-text   { line-height: 1.1; }   /* Large headings — tight */
.heading        { line-height: 1.25; }  /* h1-h3 */
.subheading     { line-height: 1.4; }   /* h4-h6 */
.body-text      { line-height: 1.6; }   /* Paragraphs — most readable */
.small-text     { line-height: 1.5; }   /* Captions, labels */
.code           { line-height: 1.7; }   /* Code blocks — needs space */

/* ❌ Violations DAVID flags */
line-height: 1;      /* No breathing room — text touches */
line-height: 2;      /* Too loose — disconnected lines */
line-height: 20px;   /* Pixel value — breaks when font size changes */
```

### Font Weight Discipline

Most UIs only need 2 weights. Using more creates inconsistency.

```css
/* ✅ Two-weight system */
--font-regular: 400;   /* Body text, labels, secondary info */
--font-medium:  500;   /* Headings, emphasized labels, buttons */

/* ❌ Too many weights — visual noise */
font-weight: 300;  /* Thin — hard to read on screens */
font-weight: 600;  /* Semi-bold — often unnecessary */
font-weight: 700;  /* Bold — reserve for exceptional emphasis only */
font-weight: 800;  /* Extra-bold — almost never needed in UI */
```

---

## Spacing & Layout Grid

### 4px / 8px Grid System

All spacing should be multiples of 4px (or 8px for larger gaps).

```
4px  = 1 unit  → tight internal padding (icon to label)
8px  = 2 units → small gaps (between form fields in same group)
12px = 3 units → medium padding (button padding-inline)
16px = 4 units → standard gap (between list items, section padding)
24px = 6 units → comfortable padding (card padding)
32px = 8 units → section separation
48px = 12 units → major section breaks
64px = 16 units → page-level spacing
```

**Tailwind equivalent:**
```
p-1=4px  p-2=8px  p-3=12px  p-4=16px  p-6=24px  p-8=32px  p-12=48px  p-16=64px
```

**DAVID flags:**
- `padding: 13px` — not on grid → violation
- `gap: 7px` — not on grid → violation  
- `margin: 5px` — not on grid → violation
- `padding: 10px` — not on 4px grid (but ok on 2px grid if team chose it — check consistency)

### Consistent Component Spacing

Components of the same type must have identical spacing between them.

```jsx
// ❌ Inconsistent spacing — no rhythm
<div className="mb-4">Card 1</div>
<div className="mb-2">Card 2</div>
<div className="mb-6">Card 3</div>

// ✅ Consistent spacing — use gap on parent
<div className="flex flex-col gap-4">
  <div>Card 1</div>
  <div>Card 2</div>
  <div>Card 3</div>
</div>
```

---

## Color System & Tokens

### Semantic vs Palette Colors

```css
/* ❌ Using palette colors directly — brittle in dark mode */
.button-primary { background: #3b82f6; color: #ffffff; }
.alert-error    { background: #fee2e2; color: #991b1b; }

/* ✅ Semantic tokens — swap the token value for dark mode */
.button-primary { background: var(--color-action-primary); color: var(--color-on-action); }
.alert-error    { background: var(--color-danger-surface); color: var(--color-danger-text); }
```

### Minimum Color Contrast Ratios (WCAG AA)

| Text Type | Background | Min Ratio |
|-----------|-----------|-----------|
| Normal text (<18pt) | Any | 4.5:1 |
| Large text (≥18pt bold, ≥24pt) | Any | 3:1 |
| UI components (borders, icons) | Adjacent | 3:1 |
| Decorative elements | — | No requirement |
| Disabled controls | — | No requirement |

**Common failures DAVID looks for:**

| Foreground | Background | Actual Ratio | Verdict |
|-----------|-----------|-------------|---------|
| #6b7280 (gray-500) | #ffffff | 4.63:1 | ✅ Pass |
| #9ca3af (gray-400) | #ffffff | 2.85:1 | ❌ Fail |
| #d1d5db (gray-300) | #ffffff | 1.62:1 | ❌ Fail |
| #6b7280 (gray-500) | #f9fafb | 4.22:1 | ❌ Fail (barely) |
| #ffffff | #3b82f6 (blue-500) | 3.00:1 | ❌ Fail (normal text) |
| #ffffff | #2563eb (blue-600) | 4.54:1 | ✅ Pass |

### Color as Sole Indicator

```jsx
// ❌ Color is the ONLY way to tell required from optional
<label style={{ color: required ? 'red' : 'gray' }}>Email</label>

// ✅ Color + shape/text indicator
<label>
  Email 
  {required && <span aria-hidden="true" style={{color:'red'}}> *</span>}
  {required && <span className="sr-only">(required)</span>}
</label>
```

---

## Component State Inventory

### Complete State Matrix

Use this matrix to audit every interactive component. Flag any missing state as P1 (HIGH).

| Component | Default | Hover | Focus | Active | Disabled | Loading | Empty | Error | Success |
|-----------|---------|-------|-------|--------|----------|---------|-------|-------|---------|
| Button | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | — | — | — |
| Input | ✅ | — | ✅ | — | ✅ | — | — | ✅ | ✅ |
| Select | ✅ | — | ✅ | — | ✅ | — | ✅ | ✅ | — |
| Checkbox | ✅ | ✅ | ✅ | — | ✅ | — | — | ✅ | — |
| Link | ✅ | ✅ | ✅ | ✅ | ✅ | — | — | — | — |
| Card | ✅ | ✅* | — | — | — | ✅ | ✅ | ✅ | — |
| List | ✅ | — | — | — | — | ✅ | ✅ | ✅ | — |
| Modal | ✅ | — | — | — | — | ✅ | — | ✅ | — |
| Table | ✅ | — | — | — | — | ✅ | ✅ | ✅ | — |
| Toast | — | — | — | — | — | — | — | ✅ | ✅ |

*Only if card is interactive (clickable)

### Loading State Hierarchy

Not all loading states are equal — use the right one:

```
FULL PAGE LOAD
└── Use: Skeleton screen (not spinner)
    Why: Preserves layout, reduces perceived wait time by 40%

SECTION/WIDGET LOAD  
└── Use: Skeleton within container bounds
    Why: Prevents layout shift

BUTTON ACTION (form submit, save)
└── Use: Spinner INSIDE button + disable button
    Why: Shows action is processing, prevents double-submit

INLINE DATA (single field update)
└── Use: Small spinner next to the field
    Why: Minimal disruption

BACKGROUND SYNC
└── Use: Subtle top progress bar (like GitHub/YouTube)
    Why: Non-intrusive

NEVER USE: Full-page spinner for everything
```

---

