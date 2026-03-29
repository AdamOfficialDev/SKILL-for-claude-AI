# DAVID — Accessibility Reference (WCAG 2.2)

Full checklist for Scanner F. Load when auditing HTML, JSX, Vue, Svelte, or Angular templates.

---

## Table of Contents
1. [Quick Severity Guide](#quick-severity-guide)
2. [Level A — Critical](#level-a--critical-always-fix)
3. [Level AA — Important](#level-aa--important)
4. [React / JSX Specific](#react--jsx-specific)
5. [WCAG 2.2 New Criteria (2023)](#wcag-22-new-criteria-2023)

---


## Quick Severity Guide

| WCAG Level | DAVID Severity | Action |
|------------|---------------|--------|
| A (must) | 🔴 P0 / 🟠 P1 | Always fix |
| AA (should) | 🟡 P2 | Fix in accessibility-sensitive context |
| AAA (may) | 🟢 P3 | Note only |

---

## Level A — Critical (Always Fix)

### 1.1.1 Non-text Content
Every image, icon, chart, and non-text element needs a text alternative.

```html
<!-- ❌ Missing alt -->
<img src="logo.png">
<img src="chart.png">

<!-- ✅ Descriptive alt -->
<img src="logo.png" alt="Acme Corp logo">
<img src="chart.png" alt="Bar chart showing Q3 revenue of $2.4M, up 18% from Q2">

<!-- ✅ Decorative images get empty alt (still required!) -->
<img src="divider.png" alt="">

<!-- ❌ Icon button with no label -->
<button><svg>...</svg></button>

<!-- ✅ Icon button with aria-label -->
<button aria-label="Close dialog"><svg aria-hidden="true">...</svg></button>
```

### 1.3.1 Info and Relationships
Structure and relationships conveyed visually must be programmatically determinable.

```html
<!-- ❌ Visual table not marked up as table -->
<div class="table-row">
  <div class="cell">Name</div>
  <div class="cell">Email</div>
</div>

<!-- ✅ Proper table markup -->
<table>
  <thead><tr><th>Name</th><th>Email</th></tr></thead>
  <tbody><tr><td>Alice</td><td>alice@example.com</td></tr></tbody>
</table>

<!-- ❌ Required field only indicated by color -->
<label>Email <span style="color:red">*</span></label>
<input type="email">

<!-- ✅ Required field with aria-required -->
<label>Email <span aria-hidden="true" style="color:red">*</span></label>
<input type="email" aria-required="true">
```

### 1.3.3 Sensory Characteristics
Instructions don't rely solely on shape, color, size, or spatial position.

```html
<!-- ❌ Color-only instruction -->
<p>Required fields are shown in red.</p>

<!-- ✅ Multiple cues -->
<p>Required fields are marked with an asterisk (*) and highlighted.</p>
```

### 2.1.1 Keyboard
All functionality available via keyboard, no keyboard traps.

```javascript
// ❌ Click-only handler — keyboard users can't activate
element.addEventListener('click', handler);

// ✅ Also handle keyboard
element.addEventListener('click', handler);
element.addEventListener('keydown', (e) => {
    if (e.key === 'Enter' || e.key === ' ') handler(e);
});
element.setAttribute('role', 'button');
element.setAttribute('tabindex', '0');
```

```javascript
// ❌ Focus trap — user can tab in but not out
function openModal() {
    modal.style.display = 'block';
    // No focus management — focus stays on background
}

// ✅ Managed focus + trap within modal + restore on close
function openModal() {
    modal.style.display = 'block';
    const firstFocusable = modal.querySelector('button, [href], input');
    firstFocusable?.focus();
    trapFocusInModal(modal);
}
function closeModal() {
    modal.style.display = 'none';
    triggerButton.focus();  // Restore focus to trigger element
}
```

### 2.4.3 Focus Order
Focus order preserves meaning and operability.

```html
<!-- ❌ tabindex values create confusing tab order -->
<input tabindex="3">
<input tabindex="1">
<input tabindex="2">

<!-- ✅ Natural DOM order, no positive tabindex -->
<input>
<input>
<input>
<!-- Use tabindex="0" to ADD to natural order, tabindex="-1" to remove -->
```

### 4.1.2 Name, Role, Value
All UI components have accessible name, role, and state.

```html
<!-- ❌ Custom toggle with no semantics -->
<div class="toggle" onclick="toggle()"></div>

<!-- ✅ Proper ARIA -->
<div role="switch" aria-checked="false" tabindex="0"
     onclick="toggle()" onkeydown="handleKey(event)">
  Dark mode
</div>

<!-- ❌ Loading state conveyed only visually -->
<button class="loading">Submit</button>

<!-- ✅ State communicated to AT -->
<button aria-busy="true" aria-label="Submitting...">
  <span aria-hidden="true">⟳</span> Submit
</button>
```

---

## Level AA — Important

### 1.4.3 Contrast (Minimum)
- Normal text: contrast ratio ≥ 4.5:1
- Large text (18pt / 14pt bold): contrast ratio ≥ 3:1
- UI components and graphical objects: ≥ 3:1

**Common contrast failures:**
```css
/* ❌ Gray text on white — fails 4.5:1 */
color: #999999; background: #ffffff;  /* Ratio: 2.85:1 */
color: #767676; background: #ffffff;  /* Ratio: 4.48:1 — just barely fails */

/* ✅ Sufficient contrast */
color: #6b6b6b; background: #ffffff;  /* Ratio: 5.74:1 — passes */
color: #595959; background: #ffffff;  /* Ratio: 7.0:1 — AAA level */
```

**Check placeholders too:**
```css
/* ❌ Placeholder often fails contrast */
::placeholder { color: #aaaaaa; }  /* 2.32:1 — fails */

/* ✅ */
::placeholder { color: #767676; }  /* 4.54:1 — passes */
```

### 1.4.4 Resize Text
Text must be resizable up to 200% without loss of content or functionality. Avoid fixed px heights on text containers; use `min-height` or `em` units.

### 2.4.1 Bypass Blocks
Skip navigation link required if there's repeated navigation before main content.

```html
<!-- ✅ Skip link (can be visually hidden until focused) -->
<a href="#main-content" class="skip-link">Skip to main content</a>
<nav>...</nav>
<main id="main-content">...</main>
```

```css
.skip-link {
    position: absolute;
    top: -40px;
    left: 0;
}
.skip-link:focus {
    top: 0;  /* Visible on focus */
}
```

### 2.4.6 Headings and Labels
Headings and labels are descriptive.

```html
<!-- ❌ Generic headings -->
<h2>Details</h2>
<h2>More information</h2>

<!-- ✅ Descriptive headings -->
<h2>Shipping details</h2>
<h2>Payment information</h2>
```

### 3.1.1 Language of Page
`<html>` must have `lang` attribute.

```html
<!-- ❌ Missing lang -->
<html>

<!-- ✅ -->
<html lang="en">
<html lang="id">  <!-- Indonesian -->
<html lang="ar" dir="rtl">  <!-- Arabic, right-to-left -->
```

### 3.3.1 Error Identification
Form errors identified and described in text.

```html
<!-- ❌ Error only shown visually (red border) -->
<input type="email" class="error">

<!-- ✅ Error associated with input via aria-describedby -->
<input type="email" aria-invalid="true" aria-describedby="email-error">
<p id="email-error" role="alert">
  Please enter a valid email address (example: user@domain.com)
</p>
```

### 3.3.2 Labels or Instructions
Instructions provided before form input.

```html
<!-- ❌ Instructions only after the input -->
<input type="password">
<p>Password must be 8+ characters with one number.</p>

<!-- ✅ Instructions before or associated with input -->
<label for="pwd">
  Password
  <span class="hint">8+ characters, at least one number</span>
</label>
<input type="password" id="pwd" aria-describedby="pwd-hint">
<p id="pwd-hint">Must contain at least one number and one uppercase letter.</p>
```

---

## React / JSX Specific

```jsx
// ❌ onClick on non-interactive element
<div onClick={handleClick}>Click me</div>

// ✅ Use interactive element or add ARIA
<button onClick={handleClick}>Click me</button>
// OR if div is truly needed:
<div role="button" tabIndex={0} onClick={handleClick}
     onKeyDown={(e) => e.key === 'Enter' && handleClick()}>
  Click me
</div>

// ❌ Icon with no label
<button><CloseIcon /></button>

// ✅
<button aria-label="Close dialog">
  <CloseIcon aria-hidden="true" />
</button>

// ❌ Modal without focus management
function Modal({ isOpen }) {
    return isOpen ? <div>{children}</div> : null;
}

// ✅ Modal with proper ARIA and focus trap
function Modal({ isOpen, onClose, title }) {
    const ref = useRef();
    useEffect(() => {
        if (isOpen) ref.current?.focus();
    }, [isOpen]);
    return isOpen ? (
        <div role="dialog" aria-modal="true" aria-labelledby="modal-title">
            <h2 id="modal-title">{title}</h2>
            <button ref={ref} onClick={onClose} aria-label="Close dialog">×</button>
            {children}
        </div>
    ) : null;
}
```

---

## WCAG 2.2 New Criteria (2023)

### 2.4.11 Focus Not Obscured (AA)
Keyboard focus indicator must not be entirely hidden by sticky headers or other content.

### 2.4.12 Focus Not Obscured — Enhanced (AAA)
Focus indicator must be fully visible.

### 2.5.3 Label in Name
For UI components with text labels, the accessible name must contain the visible text label.

```html
<!-- ❌ Accessible name doesn't contain visible text -->
<button aria-label="Navigate to next page">Next →</button>
<!-- Screen reader says "Navigate to next page", visual user sees "Next →" — mismatch -->

<!-- ✅ Accessible name includes visible text -->
<button aria-label="Next page">Next →</button>
```

### 2.5.8 Target Size (AA)
Interactive targets must be at least 24×24 CSS pixels (2.2 added this).

```css
/* ❌ Tiny click target */
button { width: 16px; height: 16px; }

/* ✅ Minimum 24×24 */
button { min-width: 24px; min-height: 24px; }
/* Recommended: 44×44 for comfortable touch targets */
```
