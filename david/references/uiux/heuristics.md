# DAVID — UI/UX Audit: Heuristics & System Design

Heuristics, design systems, and information architecture for Scanner O.
Load for: framework-specific UX, Nielsen's 10 heuristics, cognitive load, design system consistency, navigation IA, emotional design.

## Table of Contents
1. [Framework-Specific UX Patterns](#framework-specific-ux-patterns)
2. [Nielsen's 10 Heuristics — Code Pattern Library (O2)](#nielsens-10-heuristics--code-pattern-library-o2)
3. [Cognitive Load Laws — Implementation Patterns (O5)](#cognitive-load-laws--implementation-patterns-o5)
4. [Design System Consistency Audit (O10)](#design-system-consistency-audit-o10)
5. [Navigation & Information Architecture (O12)](#navigation--information-architecture-o12)
6. [Emotional Design & Delight Patterns (O14)](#emotional-design--delight-patterns-o14)

---

## Framework-Specific UX Patterns

### React / Next.js

```jsx
// ✅ Suspense + ErrorBoundary for all async content
function ProductPage({ id }) {
  return (
    <ErrorBoundary fallback={<ErrorState onRetry={retry} />}>
      <Suspense fallback={<ProductSkeleton />}>
        <ProductContent id={id} />
      </Suspense>
    </ErrorBoundary>
  );
}

// ✅ useTransition for non-urgent updates (prevents UI blocking)
function SearchPage() {
  const [isPending, startTransition] = useTransition();
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  function handleSearch(e) {
    setQuery(e.target.value);  // Input updates immediately
    startTransition(() => {
      setResults(search(e.target.value));  // Heavy operation deferred
    });
  }
  
  return (
    <>
      <input value={query} onChange={handleSearch} />
      {isPending && <SearchingSkeleton />}
      {!isPending && <SearchResults results={results} />}
    </>
  );
}
```

### Tailwind UI/UX Anti-Patterns

```html
<!-- ❌ Interactive div — not keyboard accessible -->
<div class="cursor-pointer" onClick={handleClick}>Click me</div>

<!-- ❌ overflow-hidden on parent breaks sticky children -->
<div class="overflow-hidden">
  <nav class="sticky top-0"><!-- Won't stick! --></nav>
</div>

<!-- ❌ h-screen on mobile — uses wrong viewport -->
<div class="h-screen"><!-- Use h-dvh instead --></div>

<!-- ❌ Fixed widths without responsive override -->
<div class="w-96"><!-- Overflows on mobile --></div>

<!-- ✅ Responsive width -->
<div class="w-full md:w-96"><!-- Full width on mobile, fixed on md+ --></div>
```

### Vue / Nuxt

```vue
<!-- ✅ Always provide loading + error + empty states in templates -->
<template>
  <div>
    <!-- Loading state -->
    <div v-if="loading" aria-live="polite" aria-busy="true">
      <SkeletonCard v-for="i in 3" :key="i" />
    </div>
    
    <!-- Error state -->
    <div v-else-if="error" role="alert">
      <ErrorState :message="error.message" @retry="fetchData" />
    </div>
    
    <!-- Empty state -->
    <div v-else-if="items.length === 0">
      <EmptyState title="No items" description="Create one to get started">
        <template #action>
          <button @click="createItem">Create item</button>
        </template>
      </EmptyState>
    </div>
    
    <!-- Data state -->
    <ItemList v-else :items="items" />
  </div>
</template>
```

---

## Nielsen's 10 Heuristics — Code Pattern Library (O2)

Every heuristic has concrete, detectable code patterns. DAVID evaluates all 10 from the code itself.

### H1 — Visibility of System Status

User harus selalu tahu apa yang sedang terjadi.

```jsx
// ❌ No feedback — user doesn't know if action worked
async function handleSubmit() {
  await api.saveData(form);
  // Nothing changes — user clicks again, double submit
}

// ✅ Status visible at every stage
async function handleSubmit() {
  setStatus('saving');          // "Saving..." indicator shows
  try {
    await api.saveData(form);
    setStatus('saved');         // "Saved ✓" shows for 2s
    setTimeout(() => setStatus('idle'), 2000);
  } catch {
    setStatus('error');         // "Failed — try again" shows
  }
}
```

**DAVID checks:**
- Every async action has 3 states: loading, success, error
- Progress bars on operations > 2s
- Background sync shows subtle status dot
- File uploads show percentage progress, not just spinner
- Step wizards show current position

---

### H2 — Match Between System and Real World

Language and metaphors match user's mental model, not developer's.

```jsx
// ❌ Developer language
<button>Execute query</button>
<p>Error code: 403</p>
<label>User entity ID</label>
<button>Deprecate record</button>

// ✅ User language
<button>Search</button>
<p>You don't have permission to view this page</p>
<label>Your name</label>
<button>Archive</button>
```

**DAVID checks:**
- No error codes exposed to user without human explanation
- No technical jargon in UI copy (entity, query, execute, instantiate, deprecate)
- Icons match universal conventions (floppy disk = save, house = home, magnifying glass = search)
- Date formats match user's locale, not ISO 8601 default
- Currency and number formatting follows locale conventions

---

### H3 — User Control and Freedom

User can always undo, cancel, or escape.

```jsx
// ❌ Destructive with no escape
async function handleDelete(id) {
  await api.delete(id);
  // No confirmation, no undo — gone forever
}

// ✅ Confirm + undo window
async function handleDelete(id) {
  // Option A: Confirmation dialog
  if (!confirm('Delete this item? This cannot be undone.')) return;
  
  // Option B: Soft delete with undo toast (better UX)
  const item = items.find(i => i.id === id);
  setItems(items.filter(i => i.id !== id)); // Optimistic remove
  
  toast.custom(({ visible }) => (
    <div>
      <span>"{item.name}" deleted</span>
      <button onClick={() => {
        setItems(prev => [...prev, item]); // Restore
        toast.dismiss();
      }}>Undo</button>
    </div>
  ), { duration: 5000 }); // 5s undo window
  
  await api.delete(id); // Actually delete after toast expires
}
```

**DAVID checks:**
- Modals have X button AND backdrop click to close AND Escape key handler
- Forms have Cancel button that discards changes
- Multi-step flows have Back button at every step
- Destructive actions have undo option OR confirmation dialog
- Navigation never traps user (no dead ends)
- `<a>` links that open new tab have visual indicator

---

### H4 — Consistency and Standards

Same action = same UI pattern everywhere.

```jsx
// ❌ Inconsistent patterns across files
// file: UserCard.tsx
<button onClick={deleteUser}>Delete</button>  // No icon, red text

// file: ProjectCard.tsx
<IconButton icon={Trash} onClick={deleteProject} />  // Icon only

// file: TeamSettings.tsx
<MenuItem label="Remove" onClick={deleteTeam} destructive />  // Menu item

// ✅ Consistent delete pattern — one component, used everywhere
<DestructiveAction
  label="Delete"
  onConfirm={handleDelete}
  confirmMessage="This cannot be undone"
/>
```

**DAVID checks (via Cross-file Scanner):**
- Same action has same visual treatment across all files
- Primary buttons look identical in all locations
- Form validation error messages follow same format everywhere
- Loading states use same component (not different spinners in different places)
- Modal/dialog pattern is identical — not 3 different implementations
- Date formatting is consistent across all displays
- Currency formatting is consistent

---

### H5 — Error Prevention

Prevent errors before they happen — better than great error messages.

```jsx
// ❌ Error possible — no prevention
<input type="text" placeholder="Enter date" onChange={handleDate} />
// User types: "next monday" → not a date → error

// ✅ Prevent error with right input type
<input type="date" onChange={handleDate} min={today} />
// Browser enforces date format, min prevents past dates

// ❌ Submit enabled even when form invalid
<button onClick={submit}>Create account</button>

// ✅ Progressive enable — submit only when ready
<button
  onClick={submit}
  disabled={!email || !password || password.length < 8 || !agreedToTerms}
  aria-disabled={!isValid}
>
  Create account
</button>

// ❌ No confirmation on mass action
<button onClick={() => deleteAll(selectedIds)}>
  Delete {selectedIds.length} items
</button>

// ✅ Confirmation with specific consequence
<button onClick={() => setShowConfirm(true)}>
  Delete {selectedIds.length} items
</button>
// Dialog: "Delete 47 items? This will permanently remove them from all projects."
```

**DAVID checks:**
- Destructive buttons disabled until explicit confirmation checkbox ticked
- Forms disable submit until required fields filled
- File upload validates type/size BEFORE upload starts
- Date pickers prevent invalid ranges (end before start)
- Phone/email inputs use correct type + pattern validation
- Password fields show strength meter before submission

---

### H6 — Recognition Over Recall

Make information visible — don't make user remember.

```jsx
// ❌ User must remember — context lost
<input placeholder="Search" />
// After searching: results show but query cleared from input

// ✅ Query persists — user sees what they searched for
<input value={searchQuery} onChange={e => setSearchQuery(e.target.value)} />

// ❌ Form loses data on navigation
function Step2({ onNext }) {
  // No saved state — back button = lose everything
}

// ✅ Form remembers
function Step2({ onNext, initialValues }) {
  const [values, setValues] = useState(initialValues || {}); // Restore on back
}

// ❌ User must remember keyboard shortcut
// Nothing in UI shows Ctrl+K opens search

// ✅ Shortcut hinted in UI
<button>
  Search <kbd className="ml-2 opacity-50">⌘K</kbd>
</button>
```

**DAVID checks:**
- Search query remains visible in input after searching
- Breadcrumbs show where user is in navigation
- Multi-step forms restore previous answers on Back
- Recently used items surfaced in relevant contexts
- Keyboard shortcuts shown as hints near triggering elements
- Dropdown/select shows selected value clearly

---

### H7 — Flexibility and Efficiency

Support both novice and expert users.

```jsx
// ❌ One-size-fits-all — slow for power users
// Every action requires 4 clicks through menus

// ✅ Keyboard shortcuts for frequent actions
useEffect(() => {
  function handleKeyDown(e) {
    if ((e.metaKey || e.ctrlKey) && e.key === 'k') openCommandPalette();
    if ((e.metaKey || e.ctrlKey) && e.key === 'n') createNew();
    if ((e.metaKey || e.ctrlKey) && e.key === 's') save();
  }
  document.addEventListener('keydown', handleKeyDown);
  return () => document.removeEventListener('keydown', handleKeyDown);
}, []);

// ✅ Bulk actions for power users
{selectedIds.length > 0 && (
  <BulkActionBar
    count={selectedIds.length}
    actions={[
      { label: 'Move', onClick: handleBulkMove },
      { label: 'Tag', onClick: handleBulkTag },
      { label: 'Delete', onClick: handleBulkDelete, destructive: true },
    ]}
  />
)}
```

**DAVID checks:**
- Common actions have keyboard shortcuts documented
- Bulk selection available for list items
- Power users can skip intro/tutorial steps
- Frequently used settings accessible without deep navigation
- Command palette or search-as-navigation available for complex apps

---

### H8 — Aesthetic and Minimalist Design

Every element earns its place. Extra elements compete with essential ones.

```jsx
// ❌ Information overload — too much competing
<Card>
  <Badge>NEW</Badge>
  <Badge>SALE</Badge>
  <Badge>LIMITED</Badge>
  <h3>Product name</h3>
  <Rating stars={4.5} count={1283} />
  <Price original={99} discounted={49} />
  <p>Full description of the product spanning three lines...</p>
  <TagList tags={['electronics', 'wireless', 'bluetooth', 'audio', 'premium']} />
  <Button>Add to cart</Button>
  <Button variant="secondary">Save for later</Button>
  <Button variant="ghost">Compare</Button>
  <Button variant="ghost">Share</Button>
</Card>

// ✅ Hierarchy — one primary action, secondary on hover/expansion
<Card>
  <h3>Product name</h3>
  <div className="flex items-center gap-2">
    <Rating stars={4.5} />
    <Price discounted={49} original={99} />
  </div>
  <Button>Add to cart</Button>
  {/* Other actions appear on hover or in expanded view */}
</Card>
```

**DAVID checks:**
- Maximum 1 primary CTA per view section
- Badge count per card ≤ 2 (more = noise)
- Navigation items ≤ 7 (Hick's Law)
- Modal content fits without scroll for 95% of content
- No redundant labels (icon with tooltip = enough, don't add text label too if space is tight)

---

### H9 — Help Users Recognize, Diagnose, and Recover from Errors

Error messages: human language + specific cause + actionable fix.

```jsx
// ❌ All three violations
<p className="text-red-500">Error 422</p>
<p className="text-red-500">Validation failed</p>
<p className="text-red-500">Invalid input</p>

// ✅ Human language + specific + actionable
<p role="alert" className="text-red-600">
  Email address is already in use.{' '}
  <a href="/login">Sign in instead</a> or{' '}
  <button onClick={triggerPasswordReset}>reset your password</button>.
</p>

// ✅ Field-level error with exact fix
<div>
  <input
    aria-invalid={!!errors.phone}
    aria-describedby="phone-error"
  />
  {errors.phone && (
    <p id="phone-error" role="alert" className="text-red-600 text-sm mt-1">
      Phone number must be 10–13 digits. Example: 0812-3456-7890
    </p>
  )}
</div>
```

**DAVID checks:**
- No raw error codes exposed (422, 500, ECONNREFUSED)
- Error messages explain what went wrong AND how to fix
- Field errors appear next to the specific field (not generic "form has errors")
- Network errors suggest checking connection / retrying
- 404 page offers navigation options, not just "page not found"

---

### H10 — Help and Documentation

Context-sensitive help at the right moment.

```jsx
// ❌ No help at all — user is on their own
<input placeholder="API key" />

// ✅ Contextual help exactly where needed
<div>
  <label className="flex items-center gap-1">
    API Key
    <Tooltip content="Find your API key in Settings → Developer → API Keys">
      <HelpCircle size={14} className="text-gray-400" />
    </Tooltip>
  </label>
  <input placeholder="sk-..." />
  <a href="/docs/api-keys" className="text-xs text-blue-600 mt-1 block">
    How to get your API key →
  </a>
</div>

// ✅ Inline help for complex fields
<div>
  <label>Webhook URL</label>
  <input type="url" placeholder="https://your-server.com/webhook" />
  <p className="text-xs text-gray-500 mt-1">
    We'll send POST requests here when events occur.{' '}
    <a href="/docs/webhooks">View payload format →</a>
  </p>
</div>
```

**DAVID checks:**
- Complex inputs have hint text or tooltip explaining expected format
- First-run experience has onboarding hints
- Error states link to relevant docs when fix isn't obvious
- Technical features (webhooks, API keys, regex) have inline examples

---

## Cognitive Load Laws — Implementation Patterns (O5)

### Hick's Law — Reduce Number of Choices

**Law:** Decision time increases logarithmically with number of options.
**Rule:** > 7 choices at once = cognitive overload. Group or progressively disclose.

```jsx
// ❌ 12 navigation items — violates Hick's Law
<nav>
  <a>Home</a><a>Products</a><a>Pricing</a><a>Blog</a>
  <a>Docs</a><a>API</a><a>Community</a><a>Support</a>
  <a>About</a><a>Careers</a><a>Press</a><a>Legal</a>
</nav>

// ✅ Grouped with progressive disclosure
<nav>
  <a>Home</a>
  <a>Products</a>       {/* ≤ 7 top-level items */}
  <a>Pricing</a>
  <DropdownMenu label="Resources">  {/* Group sub-items */}
    <a>Blog</a><a>Docs</a><a>API</a><a>Community</a>
  </DropdownMenu>
  <DropdownMenu label="Company">
    <a>About</a><a>Careers</a><a>Press</a>
  </DropdownMenu>
  <a>Support</a>
</nav>

// ❌ All settings on one page — overwhelming
<SettingsPage>
  {/* 47 different settings in one flat list */}
</SettingsPage>

// ✅ Settings grouped by category with tabs/sections
<SettingsPage>
  <SettingsSection title="Account">...</SettingsSection>
  <SettingsSection title="Notifications">...</SettingsSection>
  <SettingsSection title="Privacy">...</SettingsSection>
  <SettingsSection title="Billing">...</SettingsSection>
  <SettingsSection title="Advanced">...</SettingsSection>
</SettingsPage>
```

**DAVID flags:**
- Navigation > 7 top-level items → suggest grouping
- Settings page > 15 items flat → suggest tabbed sections
- Dropdown with > 10 options → suggest search-in-dropdown or autocomplete
- Radio group with > 5 options → suggest select dropdown
- Form with > 8 fields on one step → suggest splitting into steps

---

### Miller's Law — Chunk Information

**Law:** Working memory holds ~7 (±2) items at once.
**Rule:** Group related information into chunks of 4–7 items.

```jsx
// ❌ 16 digits in one block — impossible to verify
<p>Card: 4532015112830366</p>

// ✅ Chunked — matches mental model of physical card
<p>Card: 4532 0151 1283 0366</p>

// ❌ Long form with no visual grouping
<form>
  <input label="First name" />
  <input label="Last name" />
  <input label="Email" />
  <input label="Phone" />
  <input label="Address line 1" />
  <input label="Address line 2" />
  <input label="City" />
  <input label="State" />
  <input label="Zip" />
  <input label="Country" />
  <input label="Company" />
</form>

// ✅ Chunked into logical groups
<form>
  <FieldGroup label="Personal info">
    <input label="First name" /><input label="Last name" />
    <input label="Email" /><input label="Phone" />
  </FieldGroup>
  <FieldGroup label="Address">
    <input label="Address line 1" /><input label="Address line 2" />
    <input label="City" /><input label="State" /><input label="Zip" />
    <input label="Country" />
  </FieldGroup>
  <FieldGroup label="Company (optional)">
    <input label="Company name" />
  </FieldGroup>
</form>
```

**DAVID flags:**
- List items > 10 with no grouping → suggest headers or sections
- Phone/card/ID numbers displayed without formatting separators
- Form fields > 8 on one step without visual grouping
- Table columns > 8 without column grouping or hiding

---

### Fitts's Law — Proximity and Size of Targets

**Law:** Time to reach a target = f(distance, size). Closer + bigger = faster.
**Rule:** Primary actions must be large and close to where user's attention is.

```jsx
// ❌ Primary CTA far from content, small
<div>
  <article>Very long content...</article>
  <div className="flex justify-start gap-2 mt-2">
    <button className="px-2 py-1 text-sm">Save</button>     {/* Small, left-aligned */}
    <button className="px-2 py-1 text-sm">Cancel</button>
  </div>
</div>

// ✅ Primary CTA large and in thumb zone
<div>
  <article>Very long content...</article>
  <div className="flex justify-end gap-3 mt-6">
    <button className="btn-ghost">Cancel</button>
    <button className="btn-primary px-6 py-3 text-base font-medium">
      Save changes
    </button>
  </div>
</div>

// ❌ Mobile: primary action far from thumb
<div className="flex flex-col min-h-screen">
  <header>...</header>
  <main>Content</main>
  <button className="fixed top-4 right-4">Add item</button>  {/* Top right = hard to reach */}
</div>

// ✅ Mobile: primary action in thumb zone
<button className="fixed bottom-6 right-6 rounded-full w-14 h-14 shadow-lg">
  <Plus />  {/* Bottom right = natural thumb position */}
</button>
```

**DAVID flags:**
- Primary CTA smaller than secondary CTA
- Mobile primary action above the fold (top of screen) — should be bottom
- Clickable area < 44px × 44px on mobile
- Delete button same size as primary action (should be smaller/secondary)
- Primary action not visually dominant (color, size, weight)

---

## Design System Consistency Audit (O10)

### Token Drift Detection

```jsx
// ❌ Same color, different sources — token drift
// file: Button.tsx
<button style={{ background: '#3b82f6' }}>     {/* Hardcoded hex */}
// file: Badge.tsx
<span className="bg-blue-500">                 {/* Tailwind class */}
// file: Alert.tsx
<div style={{ background: 'var(--primary)' }}> {/* CSS var — different name! */}
// file: Card.tsx
<div className="bg-[#3b82f6]">               {/* Arbitrary Tailwind value */}

// ✅ Single source of truth
// All use: className="bg-primary" or var(--color-primary)
```

**DAVID scans for:**
- Same hex color appearing as hardcoded value AND as design token in different files
- Multiple names for same semantic concept (`--primary`, `--color-primary`, `--brand`, `primary-500`)
- Tailwind arbitrary values `bg-[#xxx]` when a token exists for that color
- Spacing values not from 4px/8px grid (`mt-[13px]`, `p-[7px]`)

---

### Component Duplication Detection

```jsx
// ❌ Same component built twice with slight variations
// file: UserCard.tsx
function UserCard({ user }) {
  return <div className="rounded-lg border p-4 flex gap-3">
    <Avatar src={user.avatar} size={40} />
    <div>
      <p className="font-medium">{user.name}</p>
      <p className="text-sm text-gray-500">{user.email}</p>
    </div>
  </div>;
}

// file: TeamMemberCard.tsx (found in another folder)
function TeamMemberCard({ member }) {
  return <div className="rounded-lg border p-4 flex gap-3">
    <img src={member.avatarUrl} className="w-10 h-10 rounded-full" />
    <div>
      <p className="font-semibold">{member.fullName}</p>
      <p className="text-xs text-gray-400">{member.emailAddress}</p>
    </div>
  </div>;
}
// Both render identical UI with slightly different prop names — consolidate!
```

**DAVID's duplication report:**
```
🔗 COMPONENT DUPLICATION [O10-DUP-001]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Components : UserCard.tsx + TeamMemberCard.tsx
Similarity : ~85% structural match
Diff       : Different prop names, slight typography variation
Action     : Consolidate into PersonCard({ name, email, avatar, size? })
Savings    : Remove 1 duplicate, single point of maintenance
```

---

### Icon Library Consistency

```jsx
// ❌ Mixed icon libraries — visual inconsistency
import { Search } from 'lucide-react';           // Lucide style
import { FiUser } from 'react-icons/fi';          // Feather style
import SearchIcon from '@heroicons/react/solid';  // Heroicons style
import { BiHome } from 'react-icons/bi';          // Boxicons style

// ✅ One library, one style throughout
import { Search, User, Home, Settings } from 'lucide-react'; // Pick one, stay with it
```

---

### Button Variant Audit

```jsx
// DAVID checks that button variants are consistent across the codebase:
// Scan for all button-like elements and verify:
// 1. Primary button: one style only
// 2. Secondary button: one style only
// 3. Destructive button: one style only — never labeled same as primary

// ❌ 4 different "primary" button implementations found:
<button className="bg-blue-600 text-white px-4 py-2 rounded">     {/* Button.tsx */}
<button className="bg-blue-500 text-white px-3 py-1.5 rounded-md"> {/* Form.tsx */}
<button className="btn btn-primary">                                 {/* Settings.tsx */}
<button style={{background:'#2563eb',color:'white',padding:'8px 16px'}}> {/* Modal.tsx */}
```

---

## Navigation & Information Architecture (O12)

### Breadcrumb Requirements

```jsx
// ✅ Complete breadcrumb implementation
function Breadcrumb({ items }) {
  return (
    <nav aria-label="Breadcrumb">
      <ol className="flex items-center gap-1 text-sm">
        {items.map((item, i) => (
          <li key={i} className="flex items-center gap-1">
            {i > 0 && <ChevronRight size={14} className="text-gray-400" aria-hidden />}
            {i === items.length - 1 ? (
              <span aria-current="page" className="text-gray-900 font-medium">
                {item.label}
              </span>
            ) : (
              <a href={item.href} className="text-gray-500 hover:text-gray-700">
                {item.label}
              </a>
            )}
          </li>
        ))}
      </ol>
    </nav>
  );
}
// Usage: [{label:'Home',href:'/'}, {label:'Settings',href:'/settings'}, {label:'Profile'}]
```

**DAVID checks:**
- Deep pages (3+ levels) always have breadcrumb
- Current page in breadcrumb is `aria-current="page"` and not a link
- Breadcrumb is `<nav>` with `aria-label`
- Mobile: breadcrumb collapses to `... / Current Page` to save space

---

### Pagination Patterns

```jsx
// ✅ Accessible pagination
function Pagination({ currentPage, totalPages, onPageChange }) {
  return (
    <nav aria-label="Pagination">
      <ul className="flex items-center gap-1">
        {/* Prev */}
        <li>
          <button
            onClick={() => onPageChange(currentPage - 1)}
            disabled={currentPage === 1}
            aria-label="Previous page"
          >
            <ChevronLeft size={16} />
          </button>
        </li>

        {/* Page numbers — show max 7, collapse middle with ellipsis */}
        {getPageRange(currentPage, totalPages).map((page, i) =>
          page === '...' ? (
            <li key={i}><span aria-hidden>…</span></li>
          ) : (
            <li key={i}>
              <button
                onClick={() => onPageChange(page)}
                aria-current={page === currentPage ? 'page' : undefined}
                aria-label={`Page ${page}`}
              >
                {page}
              </button>
            </li>
          )
        )}

        {/* Next */}
        <li>
          <button
            onClick={() => onPageChange(currentPage + 1)}
            disabled={currentPage === totalPages}
            aria-label="Next page"
          >
            <ChevronRight size={16} />
          </button>
        </li>
      </ul>
    </nav>
  );
}
```

---

### Back Navigation Patterns

```jsx
// ❌ Browser back — may go to unrelated page if user arrived externally
<button onClick={() => window.history.back()}>← Back</button>

// ✅ Explicit back with known destination
<button onClick={() => router.push('/dashboard')}>← Back to Dashboard</button>
// OR
<Link href="/dashboard">← Back to Dashboard</Link>

// ✅ Smart back — go to referrer if from same app, fallback if external
function BackButton({ fallback = '/dashboard' }) {
  const router = useRouter();
  return (
    <button onClick={() => {
      if (document.referrer.includes(window.location.origin)) {
        router.back();
      } else {
        router.push(fallback);
      }
    }}>
      ← Back
    </button>
  );
}
```

**DAVID checks:**
- No dead-end pages (every page has a path back)
- Back button labels the destination ("← Back to Projects", not just "← Back")
- Modal close buttons always present (X + Escape + backdrop click)
- Wizard back button never loses progress
- 404 page offers at least 2 navigation options (home + search)

---

### Deep Link & URL State

```jsx
// ❌ State in memory — back button loses filter/sort/page
const [page, setPage] = useState(1);
const [filter, setFilter] = useState('all');

// ✅ State in URL — shareable, bookmarkable, back/forward works
const [searchParams, setSearchParams] = useSearchParams();
const page = parseInt(searchParams.get('page') || '1');
const filter = searchParams.get('filter') || 'all';

function handleFilterChange(newFilter) {
  setSearchParams(prev => {
    prev.set('filter', newFilter);
    prev.set('page', '1'); // Reset to page 1 on filter change
    return prev;
  });
}
```

---

## Emotional Design & Delight Patterns (O14)

### Microinteraction Library

```css
/* Button press feedback */
.btn-primary {
  transition: transform 80ms ease-out, box-shadow 80ms ease-out;
}
.btn-primary:active {
  transform: scale(0.97);
  box-shadow: none; /* Pressed = flat */
}

/* Success checkmark animation */
@keyframes checkmark-draw {
  from { stroke-dashoffset: 24; }
  to   { stroke-dashoffset: 0; }
}
.checkmark-path {
  stroke-dasharray: 24;
  stroke-dashoffset: 24;
  animation: checkmark-draw 300ms ease-out forwards;
}

/* Item entrance — stagger for lists */
@keyframes fade-slide-up {
  from { opacity: 0; transform: translateY(8px); }
  to   { opacity: 1; transform: translateY(0); }
}
.list-item {
  animation: fade-slide-up 200ms ease-out both;
}
.list-item:nth-child(1) { animation-delay: 0ms; }
.list-item:nth-child(2) { animation-delay: 40ms; }
.list-item:nth-child(3) { animation-delay: 80ms; }
/* Cap stagger at ~400ms total regardless of list length */
```

---

### Emotional Milestone Moments

```jsx
// ✅ First completion — confetti moment
function TaskList({ tasks }) {
  const allDone = tasks.every(t => t.completed);

  useEffect(() => {
    if (allDone) {
      confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
    }
  }, [allDone]);

  return allDone ? (
    <EmptyState
      icon="🎉"
      headline="All done!"
      description="You've completed everything on your list."
    />
  ) : (
    <TaskItems tasks={tasks} />
  );
}

// ✅ Progress celebration — show when milestone hit
function ProgressBar({ value, total }) {
  const percent = (value / total) * 100;
  const milestones = [25, 50, 75, 100];
  const [celebrated, setCelebrated] = useState(new Set());

  useEffect(() => {
    milestones.forEach(m => {
      if (percent >= m && !celebrated.has(m)) {
        setCelebrated(prev => new Set(prev).add(m));
        if (m === 100) triggerConfetti();
        else triggerSubtleGlow();
      }
    });
  }, [percent]);

  return <progress value={value} max={total} />;
}
```

---

### Error State Personality

```jsx
// ❌ Generic, cold error
function ErrorPage() {
  return <div>404 — Page not found</div>;
}

// ✅ Branded, helpful, human
function NotFoundPage() {
  return (
    <div className="text-center py-24">
      <div className="text-8xl mb-6" role="img" aria-label="Confused emoji">🤔</div>
      <h1 className="text-3xl font-bold">Hmm, can't find that page</h1>
      <p className="mt-4 text-gray-500 max-w-md mx-auto">
        The page you're looking for may have moved, been deleted, or never existed.
        Let's get you back on track.
      </p>
      <div className="mt-8 flex flex-col sm:flex-row gap-3 justify-center">
        <Link href="/" className="btn-primary">Go to home</Link>
        <Link href="/search" className="btn-secondary">Search the site</Link>
      </div>
      <p className="mt-8 text-sm text-gray-400">
        Still lost? <a href="mailto:help@app.com">Contact support</a>
      </p>
    </div>
  );
}
```

---

### Loading State Personality

```jsx
// ❌ Generic spinner — no personality
<Spinner />

// ✅ Context-aware loading with brand voice
// For a data analytics dashboard:
<LoadingState
  messages={[
    "Crunching your numbers...",
    "Almost there...",
    "Building your report...",
  ]}
  // Cycles through messages every 2s for long loads
/>

// ✅ Skeleton that feels premium
function DashboardSkeleton() {
  return (
    <div className="animate-pulse">
      {/* Match EXACT layout of loaded content */}
      <div className="grid grid-cols-4 gap-4 mb-8">
        {[...Array(4)].map((_, i) => (
          <div key={i} className="bg-gray-100 dark:bg-gray-800 rounded-xl h-28" />
        ))}
      </div>
      <div className="bg-gray-100 dark:bg-gray-800 rounded-xl h-64 mb-4" />
      <div className="grid grid-cols-2 gap-4">
        <div className="bg-gray-100 dark:bg-gray-800 rounded-xl h-48" />
        <div className="bg-gray-100 dark:bg-gray-800 rounded-xl h-48" />
      </div>
    </div>
  );
}
```

---

