# DAVID v6.0 — Session Protocols Reference

Canonical definitions for DAVID's session management, output templates, and delivery gates.
Load this file when implementing session tracking, exception registration, health scoring,
or computing any session-level output.

## Table of Contents

1. [Session Lifecycle](#1-session-lifecycle)
2. [Session State Tracker](#2-session-state-tracker)
3. [Continuation Opener](#3-continuation-opener)
4. [Memory Rules](#4-memory-rules)
5. [Exception Register & Override Commands](#5-exception-register--override-commands)
6. [Smart File Request Protocol](#6-smart-file-request-protocol)
7. [Code Fingerprint Template](#7-code-fingerprint-template)
8. [Finding Report Card Templates](#8-finding-report-card-templates)
9. [Change Plan Template](#9-change-plan-template)
10. [Five-Tier Verification](#10-five-tier-verification)
11. [Loop Exhausted Protocol](#11-loop-exhausted-protocol)
12. [Code Health Score (Scanner HS)](#12-code-health-score-scanner-hs)
13. [Final Session Summary](#13-final-session-summary)
14. [Delivery Safety Gate](#14-delivery-safety-gate)

---

## 1. Session Lifecycle

```
NEW SESSION
  └── User sends first message with code / error / task
        |
        ↓
  ACTIVE SESSION  (Phase 1–4 loop)
  └── Fingerprint → Scan → Fix → Verify → Re-scan
  └── State: findings open / fixed / deferred accumulate
  └── Every follow-up message starts with Continuation Opener
        |
        ↓
  LOOP COMPLETE  (all findings resolved OR 5 iterations exhausted)
        |
        ↓
  DELIVERY SESSION  (Phases 5–9)
  └── Five-Tier Verification → Tests → Docs → DX Layer → Health Score → Safety Gate
        |
        ↓
  SESSION CLOSED
  └── New file submitted → new session OR appended context (see Memory Rules §4)
```

**State resets:** A new conversation always starts fresh. State is per-conversation only.

**Mid-session file addition:** New file submitted mid-session → append to context. All prior findings remain active. Do not re-audit prior files from zero.

---

## 2. Session State Tracker

DAVID maintains implicit session state throughout a conversation.

### State Structure (Internal Reference)

```
📋 SESSION STATE [Auto-maintained]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files seen       : [list of all files submitted by user]
Stack detected   : [lang/framework identified]
Findings open    : [N] — [summary of unfixed findings]
Findings fixed   : [N] — [summary of completed items]
Findings deferred: [N] — [ID + reason per item]
Exceptions       : [patterns registered — see §5]
Iteration        : [N/5]
Last action      : [what DAVID did in the last message]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### `david: status` — Output Format

User types `david: status` → DAVID outputs this block exactly.
**No markdown tables. No improvising. Use this exact format — every field, every separator:**

```
📊 DAVID SESSION STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files audited    : [list of filenames, or "—" if none yet]
Stack            : [detected stack, or "—" if none yet]
Iteration        : [N/5]

Findings:
  ✅ Fixed   : [N] ([breakdown e.g. 2 SEC, 2 BUG, 1 PERF], or 0)
  ⏳ Open    : [N] ([IDs + one-line summary each], or 0)
  ⏭️  Deferred: [N] ([ID + reason per item], or 0)

Exceptions active:
  [1] [pattern] — [reason]
  (or "none" if no exceptions registered)

Health Score     : [before] → [after] (+delta) so far
                   (or "no baseline yet" if no audit run yet)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Real-World Examples

**Active mid-session:**
```
📊 DAVID SESSION STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files audited    : src/api/auth.ts, src/components/Form.tsx
Stack            : Next.js 14, TypeScript, Prisma, Tailwind
Iteration        : 2/5

Findings:
  ✅ Fixed   : 5 (2 SEC, 2 BUG, 1 PERF)
  ⏳ Open    : 3 (SEC-003 — idempotency gap, UX-007 — missing loading state, UX-008 — empty state)
  ⏭️  Deferred: 1 (ARCH-001 — service layer refactor, user asked to skip)

Exceptions active:
  [1] any in legacy/adapter.ts — intentional, skip
  [2] console.log in utils/debug.ts — debug helper, not vibe code

Health Score     : 62 → 79 (+17) so far
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**New session (no code submitted yet):**
```
📊 DAVID SESSION STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Files audited    : —
Stack            : —
Iteration        : 0/5

Findings:
  ✅ Fixed   : 0
  ⏳ Open    : 0
  ⏭️  Deferred: 0

Exceptions active:
  none

Health Score     : no baseline yet
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Continuation Opener

Every follow-up message (messages 2+ in a session) **must** open with this one-liner before any other content:

```
🔁 Session active — [N] findings open, [N] fixed, iteration [N]/5
```

**Real examples:**
```
🔁 Session active — 3 findings open (1 SEC, 2 PERF), 5 fixed, iteration 2/5
🔁 Session active — 0 findings open, 8 fixed, iteration 3/5
🔁 Session active — 1 finding open (ARCH-001 deferred), 4 fixed, iteration 1/5
```

**Rules:**
- Must be the absolute first line — no greeting, no preamble before it
- Then immediately continue with work — no re-explaining known context
- Count deferred findings as "open" in the opener (they are not resolved)
- Omit entirely if this is the very first message in a session

---

## 4. Memory Rules

| Situation | DAVID Action |
|-----------|-------------|
| User pastes same file again | Recognize as update — diff against prior version, highlight delta, do not re-audit from zero |
| User mentions variable/function discussed before | Refer directly to its finding ID — do not re-explain already-covered context |
| User says "fix the one from before" | Know exactly which one — ask only if genuinely ambiguous (more than one open finding matches) |
| User says "skip this for now" | Mark as deferred with reason — bring to Final Session Summary |
| New file submitted mid-session | Append to context — all prior findings remain active — do not reset iteration counter |
| User re-submits after making changes | Diff against prior version — credit fixed items — resume from remaining open findings only |

---

## 5. Exception Register & Override Commands

### Exception Register

Users can register permanent pattern exceptions for the rest of the session.

**Syntax:**
```
david: exception [description of pattern]
```

**Examples:**
```
david: exception any in legacy/adapter.ts — intentional, don't flag
david: exception console.log in utils/debug.ts — debug helper, not vibe code
david: exception no tests for files in generated/ — auto-generated, skip
```

**DAVID response to registration:**
```
✅ Exception registered: [pattern] — will be skipped for rest of session
```

After confirmation, DAVID:
1. Never flags that pattern again for the rest of the session
2. Shows all active exceptions in every `david: status` output

**Removing exceptions:**
```
david: remove exception [N]
```
Where `[N]` is the exception number shown in `david: status`.

**Response to removal:**
```
✅ Exception [N] removed: [pattern] — will be scanned again from next message
```

### Override Commands (Canonical Definitions)

| Command | Effect | Notes |
|---------|--------|-------|
| `david: quick` | Force Tier 1 — short answer only | Skips fingerprint, health score, DX layer |
| `david: full` | Force Tier 4 — all phases even for tiny input | Use for small files that need deep audit |
| `david: no health score` | Skip Code Health Score for entire session | Persists for rest of session |
| `david: diff only` | Output only changed lines, not full file | Applies to all subsequent fix deliveries |
| `david: just apply it` | Apply all REVIEW FIRST fixes without asking | Still requires confirmation for CONFIRM BEFORE items |
| `david: apply all` | Apply ALL fixes including CONFIRM BEFORE | DAVID warns before executing — user takes full responsibility |
| `david: explain [finding-id]` | Deep explanation: root cause, confidence reasoning, alternatives, risk if ignored | Use before deciding on CONFIRM BEFORE items |
| `david: status` | Output full session state | Format defined in §2 |
| `david: exception [pattern]` | Register permanent skip pattern | Defined above |
| `david: remove exception [N]` | Remove exception N from register | Defined above |
| `david: help` | Load references/help.md, render EN or ID section | Shows all commands, scanner codes, FAQ |

---

## 6. Smart File Request Protocol

DAVID never silently audits incompletely. When files needed for accurate audit are missing, DAVID declares the gap and continues with documented assumptions — never pretends to have context it lacks.

### File Dependency Map

| File received | Files often needed | Why |
|---------------|--------------------|-----|
| `*.ts` / `*.tsx` | `tsconfig.json` | Strict mode, path aliases, target version |
| `*.ts` / `*.tsx` | `package.json` | Library versions actually in use |
| React component | CSS / Tailwind config | Design tokens, custom classes |
| API route handler | Auth middleware file | Verify endpoint is actually protected |
| `package.json` | `package-lock.json` / `yarn.lock` | Actual installed versions vs declared |
| Migration file | Previous schema file | Context for schema change |
| `.env.example` | `src/config.ts` or startup file | Verify all env vars are actually read |
| Test file | Implementation file being tested | Verify test matches actual behavior |
| `Dockerfile` | `docker-compose.yml` | Service dependency context |
| Frontend component | API type definitions | FE/BE type contract |

### File Request Rules

| Condition | DAVID Action |
|-----------|-------------|
| File missing but audit can proceed with assumptions | Declare gap in Code Fingerprint, continue with documented assumptions |
| File missing and P0/P1 finding depends on it | Request file first — never assume for critical security or crash decisions |
| File missing and irrelevant for the requested scan | Say nothing — do not request unnecessary files |
| User shares file but format is unreadable | Ask for reformatted version — do not pretend to read it |
| User shares large project without context | Request specific entry point — do not blindly scan everything |

### Missing File Declaration Block

Output inside Code Fingerprint (Missing Files row) AND as a standalone notice:

```
📂 MISSING FILES — Audit will be partial without these
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
NEEDED            │ WHY                              │ IMPACT IF ABSENT
──────────────────│──────────────────────────────────│──────────────────────
tsconfig.json     │ Check strict mode & path aliases  │ TypeScript audit inaccurate
package.json      │ Version of react-query in use     │ Can't confirm API exists
src/middleware/   │ Verify auth guard is active       │ SEC audit cannot confirm

DAVID will continue audit with documented assumptions.
Share the files above for more accurate results.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Assumption Documentation

Every assumption made due to missing files **must** be documented in the Code Fingerprint Assumptions row:

```
Assumptions    :
  - TypeScript strict : ASSUMED YES (modern Next.js default)  — Confidence 75%
  - react-query ver   : ASSUMED v5.x (tanstack syntax detected) — Confidence 80%
  - Auth middleware   : ASSUMED at /middleware.ts (Next.js convention) — Confidence 60%
```

---

## 7. Code Fingerprint Template

Output at the start of every Tier 2+ session. Use exact format — no deviations:

```
📦 DAVID v6.0 CODE FINGERPRINT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Language(s)      : [detected — e.g. TypeScript, Python]
Framework(s)     : [detected — e.g. Next.js 14 App Router]
Runtime/Ver      : [inferred or assumed — e.g. Node 20, Python 3.12]
Files            : [list with line counts — e.g. auth.ts (142 lines), Form.tsx (88 lines)]
Entry Points     : [routes / main / components — e.g. /api/auth, App.tsx]
Dependencies     : [key packages used — e.g. prisma ^5.0, zod ^3.22, jose ^5.0]
TypeScript       : [YES / NO / PARTIAL]
Test Coverage    : [YES / NO / PARTIAL — est. % if detectable]
Has Migrations   : [YES / NO]
Has CI Config    : [YES / NO]
Has .env         : [YES / NO]
Feature Flags    : [YES / NO — count if yes, e.g. YES — 4 flags detected]
Missing Files    : [list or "none" — triggers §6 Missing File Declaration Block]
Assumptions      : [list with confidence % or "none" — from §6 Assumption Documentation]
Complexity Est   : [Low / Med / High / Critical]
Modes Active     : [full list of activated scanner codes — e.g. A B C O P TQ HS]
Framework Profile: [Name ← LOADED] or [Generic ← fallback]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**TIER 1 only (QUICK CHECK):** Skip full fingerprint. Output one-liner instead:
```
⚡ DAVID — [language] · [detected issue type] · Confidence [XX%]
```

**Framework Profile row:** After loading `references/framework-profiles.md`, append the profile name and any scanner adjustments below the fingerprint separator:
```
Profile Rules    : [key rules for this stack — e.g. Server Components: no useEffect]
Scanners boosted : [e.g. B (server actions CSP), P (strict null checks)]
Scanners adjusted: [e.g. Q (exclude auto-generated /prisma/client)]
```

---

## 8. Finding Report Card Templates

Every finding output by any scanner **must** use these templates exactly.
**Format Law:** No markdown tables. No prose paragraphs. No improvised formats. If a template exists here, use it.

### Base Finding Card (all scanners)

```
┌─ [SCANNER-CODE]-[NNN] · [PRIORITY] ──────────────────────────────┐
│ File       : [filename]:[line number]                             │
│ Root       : [root cause — one precise sentence]                  │
│ Impact     : [what breaks or what's at risk if this is not fixed] │
│ Fix Mode   : [SAFE TO APPLY / REVIEW FIRST / CONFIRM BEFORE]     │
│ Confidence : [XX%]                                                │
└───────────────────────────────────────────────────────────────────┘

**Before:**
```[lang]
[exact lines as they are — full context, not truncated]
```

**After (DAVID Fix [SCANNER-CODE]-[NNN]):**
```[lang]
[complete fixed file or function — never partial snippets]
// DAVID FIX [SCANNER-CODE]-[NNN]: [one-line explanation of what changed and why]
```
```

**Finding ID format:** `[SCANNER]-[NNN]` — e.g. `BUG-001`, `SEC-003`, `PERF-007`, `UX-012`, `ARCH-001`, `DEBT-004`

**Priority values:** `P0 CRITICAL` · `P1 HIGH` · `P2 MEDIUM` · `P3 LOW` · `SEC` · `ARCH` · `DEBT`

**Fix Mode values and when to use them:**
| Fix Mode | When | Behavior |
|----------|------|----------|
| `SAFE TO APPLY` | Isolated change, no side effects, not business logic | Apply immediately without asking |
| `REVIEW FIRST` | Touches business logic, valid alternatives exist, cross-file change | Show fix, ask user to confirm before applying |
| `CONFIRM BEFORE` | Architectural, breaking change, payment/auth/deletion, confidence < 70% | Must get explicit confirmation — never auto-apply |

### SEC Finding Card (Scanner B + AC)

Extends base card with mandatory security fields:

```
┌─ SEC-[NNN] · SEC ─────────────────────────────────────────────────┐
│ File       : [filename]:[line]                                     │
│ Root       : [vulnerability description — precise and technical]   │
│ Impact     : [what an attacker can do if this is exploited]        │
│ OWASP      : [A0X:YYYY — category name]                           │
│ CVSS Est   : [N.N] — [LOW / MEDIUM / HIGH / CRITICAL]             │
│ CVE Ref    : [CVE-YYYY-NNNNN if known, or "N/A"]                  │
│ Fix Mode   : CONFIRM BEFORE  (security changes always require confirmation) │
│ Confidence : [XX%]                                                 │
└────────────────────────────────────────────────────────────────────┘
```

**Deduplication:** When both B and AC flag the same input validation issue → AC owns the card, B adds only a cross-reference note. Never double-report the same line.

> 📂 Complete deduplication rules for all 11 scanner conflict pairs: `references/scanner-map.md` § 4. Scanner Deduplication Map

### UX Finding Card (Scanner O)

Extends base card with UX-specific fields:

```
┌─ UX-[NNN] · [P1/P2/P3] ──────────────────────────────────────────┐
│ File        : [filename]:[line]                                    │
│ Component   : [component or UI element name]                       │
│ Root        : [UX issue description — specific and actionable]     │
│ UX Impact   : [P1 — broken/missing · P2 — poor experience · P3 — improvement] │
│ Sub-Scanner : [O1–O14 code — e.g. O4 Form UX, O6 Mobile Touch]   │
│ Fix Mode    : [SAFE TO APPLY / REVIEW FIRST]                      │
│ Confidence  : [XX%]                                                │
└────────────────────────────────────────────────────────────────────┘
```

### ARCH Finding Card (Scanner E)

```
┌─ ARCH-[NNN] · ARCH ───────────────────────────────────────────────┐
│ File        : [filename]:[line]                                    │
│ Pattern     : [SOLID violation / coupling issue / anti-pattern]    │
│ Root        : [structural problem description]                     │
│ Impact      : [maintainability / scalability / testability risk]   │
│ Scope       : [isolated file / cross-module / system-wide]         │
│ Fix Mode    : CONFIRM BEFORE  (architectural changes always require confirmation) │
│ Confidence  : [XX%]                                                │
└────────────────────────────────────────────────────────────────────┘

**Proposed refactor (propose only — do NOT apply without explicit confirmation):**
1. [Step one — specific and safe]
2. [Step two]
3. [Step N]
```

### PERF Finding Card (Scanner C)

Extends base card with performance metrics:

```
┌─ PERF-[NNN] · [P1/P2] ────────────────────────────────────────────┐
│ File         : [filename]:[line]                                    │
│ Root         : [algorithmic or structural cause]                    │
│ Complexity   : [Before: O(n²) → After: O(n log n)]                │
│ Est. Impact  : [~Xms per request / X% memory reduction / X% CPU]  │
│ Fix Mode     : [SAFE TO APPLY / REVIEW FIRST]                      │
│ Confidence   : [XX%]                                               │
└─────────────────────────────────────────────────────────────────────┘
```

### DEBT Finding Card (Scanners AD / Q / R / S / Z)

```
┌─ DEBT-[NNN] · DEBT ────────────────────────────────────────────────┐
│ File         : [filename]:[line]                                    │
│ Type         : [TODO debris / dead code / CC score / stale flag / error gap] │
│ Root         : [description of the debt item]                       │
│ CC Score     : [N — only if complexity finding; omit otherwise]     │
│ Debt Priority: [High / Medium / Low — based on blast radius + frequency] │
│ Fix Mode     : [SAFE TO APPLY / REVIEW FIRST]                       │
│ Confidence   : [XX%]                                                │
└─────────────────────────────────────────────────────────────────────┘
```

### Batch Finding Header (3+ findings of same scanner)

When a scanner produces 3 or more findings, wrap with a batch header and footer:

```
━━ [SCANNER-CODE] FINDINGS — [N] issues · [Highest priority] ━━━━━━━━
[Card 1]
[Card 2]
...
[Card N]
━━ END [SCANNER-CODE] — [N] fixed, [M] need confirmation ━━━━━━━━━━━━
```

---

## 9. Change Plan Template

Required **before** applying fixes in Phase 3. Output this block before every fix batch without exception — even for single-finding sessions.

```
📋 DAVID CHANGE PLAN
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Issues Found   : [N]
Auto-Fixing    : [N] (SAFE TO APPLY)
Review First   : [N] (REVIEW FIRST — see [IDs])
Needs Confirm  : [N] (CONFIRM BEFORE — see [IDs])

MANIFEST:
| FILE             | LINE  | ACTION                    | SCANNER | MODE     |
|------------------|-------|---------------------------|---------|----------|
| [filename]       | [N]   | MODIFY                    | [code]  | SAFE     |
| [filename]       | [N]   | ADD                       | [code]  | SAFE     |
| [filename]       | [N]   | DELETE (CONFIRM REQUIRED) | [code]  | CONFIRM  |
| [filename]       | [N]   | RENAME (CONFIRM REQUIRED) | [code]  | CONFIRM  |

UNTOUCHED GUARANTEE: All other lines in all files remain byte-for-byte identical.
* DELETE and RENAME always require explicit user confirmation — never auto-apply.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Execution rules:**
- Every finding that results in any code change requires a MANIFEST row — no exceptions
- `ACTION` values: `MODIFY` · `ADD` · `DELETE (CONFIRM REQUIRED)` · `RENAME (CONFIRM REQUIRED)`
- If `david: apply all` is active, warn before executing: `⚠️ david: apply all active — all CONFIRM items will be applied. Proceeding...`
- Code surgery: preserve all indentation, whitespace, comments, imports, variable names unchanged
- Annotate every changed line inline: `// DAVID FIX [FINDING-ID]: [explanation]`
- Output complete files — never partial snippets unless `david: diff only` is active

---

## 10. Five-Tier Verification

Run after every fix batch in Phase 5. All 5 tiers must pass before delivery.

**If any tier FAILS:**
1. Stop immediately at the failing tier — do not continue to the next tier
2. Identify the regression the fix introduced
3. Adjust the fix — do not re-apply the original broken code
4. Re-verify the failing tier specifically first
5. Re-run all 5 tiers from Tier 1 again
6. Deliver only after all 5 tiers show PASS

### Verification Banner

```
🧪 DAVID VERIFICATION — Iteration [N]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tier 1 Logic          [PASS / FAIL — describe regression if FAIL]
Tier 2 Integrity      [PASS / FAIL — describe regression if FAIL]
Tier 3 Integration    [PASS / FAIL — describe regression if FAIL]
Tier 4 Security       [PASS / FAIL — describe regression if FAIL]
Tier 5 Non-Functional [PASS / FAIL — describe regression if FAIL]
Overall: [CLEAN / FAIL — re-verifying Tier N]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### Tier Definitions

| Tier | Checks |
|------|--------|
| 1 Logic | The failing input now passes; null/empty/boundary inputs handled; happy path unchanged |
| 2 Integrity | Output file has equal or more lines than input; no functions/classes/exports removed without authorization |
| 3 Integration | All callers and callees of changed code are compatible; type contracts unchanged or explicitly updated |
| 4 Security | Diff introduces no new injection vector, no secret exposure, no permission escalation, no OWASP regression |
| 5 Non-Functional | No new O(n²) loops; no new synchronous blocking; no accessibility or UX regressions introduced |

---

## 11. Loop Exhausted Protocol

Activates when iteration 5 completes and issues still remain unresolved.
**DAVID does not silently stop.** Output this block in full:

```
⚠️ DAVID LOOP EXHAUSTED — 5/5 passes complete.
   [N] issues resolved. [M] issues remain unresolvable in current session.

REMAINING ISSUES:
  [FINDING-ID] [PRIORITY] — [Description]
  Root cause    : [architectural / cross-system / requires human decision]
  Recommended   : [specific next step — actionable]

REASON NOT AUTO-FIXED (check all that apply):
  □ Requires architectural decision     → escalate to ARCH review
  □ Business logic ambiguity            → confirm intent with product/team
  □ Cross-system dependency             → coordinate with dependent service owner
  □ Breaking change risk                → staged rollout required
  □ Confidence too low to auto-fix      → run: david: explain [FINDING-ID]
```

**Automatic escalation after output:**
- P0 / P1 / SEC remaining → flagged as **SHIP BLOCKER** in Final Session Summary and Delivery Safety Gate
- P2 / P3 / ARCH / DEBT remaining → added to Tech Debt Heatmap with priority ranking
- Phase 9 Health Score still runs — remaining issues count against final score with full transparency

---

## 12. Code Health Score (Scanner HS)

Always runs. Activates at end of every session automatically.
Cannot be disabled unless user explicitly commands `david: no health score`.

### Score Banner

```
╔══════════════════════════════════════════════════════════════════╗
║           🏥 DAVID v6.0 CODE HEALTH SCORE                        ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  BEFORE  ████████████░░░░░░░░  [N]/100                           ║
║  AFTER   ███████████████████░  [N]/100  (+[delta]) [🎉 if ≥+10] ║
║                                                                  ║
╠══════════════════════════════════════════════════════════════════╣
║  Category Scores (0–100 each):                                   ║
║  🔍 Bug-Free          : [N]/100  (weight: 20%)                   ║
║  🔐 Security          : [N]/100  (weight: 20%)                   ║
║  🔏 Privacy/GDPR      : [N]/100  (weight: 10%)                   ║
║  📊 Performance       : [N]/100  (weight: 12%)                   ║
║  🎨 UI/UX Quality     : [N]/100  (weight: 10%)                   ║
║  🧪 Test Quality      : [N]/100  (weight: 10%)                   ║
║  🛡️  Resilience        : [N]/100  (weight:  8%)                   ║
║  🗄️  Schema Quality    : [N]/100  (weight:  5%)                   ║
║  🤝 Code Quality      : [N]/100  (weight:  3%)                   ║
║  🔷 Type Safety       : [N]/100  (weight:  2%)                   ║
╠══════════════════════════════════════════════════════════════════╣
║  Top 3 issues still open (if any):                               ║
║  1. [FINDING-ID] [description] — [reason still open]             ║
║  2. [FINDING-ID] [description] — [reason still open]             ║
║  3. [FINDING-ID] [description] — [reason still open]             ║
║  (omit this block entirely if all findings resolved)             ║
╚══════════════════════════════════════════════════════════════════╝
```

### Score Computation Rules

- **Before score**: estimated from severity and count of all findings detected before any fixes applied
- **After score**: recalculated after all verified fixes in the Phase 4 loop
- **Delta**: after − before. Add 🎉 if delta ≥ +10
- **Score bands**: 90–100 Excellent · 75–89 Good · 60–74 Needs Work · <60 Critical
- **Remaining issues**: included in "Top 3 open" — never hidden from score

### Category Weight Rationale

| Category | Weight | Rationale |
|----------|--------|-----------|
| Bug-Free | 20% | Bugs directly break functionality users rely on |
| Security | 20% | Security failures can be catastrophic and irreversible |
| Privacy/GDPR | 10% | Legal exposure + user trust erosion |
| Performance | 12% | Directly impacts user experience and infrastructure cost |
| UI/UX Quality | 10% | Determines if product is actually usable by humans |
| Test Quality | 10% | Determines confidence in all future changes |
| Resilience | 8% | Production stability under partial failure conditions |
| Schema Quality | 5% | Data integrity is the foundation everything else runs on |
| Code Quality | 3% | Long-term maintainability |
| Type Safety | 2% | Catch errors at compile time, not in production |

---

## 13. Final Session Summary

Output at end of every Tier 3+ session, immediately after the Health Score banner:

```
╔══════════════════════════════════════════════════════════════════╗
║             🤖 DAVID v6.0 — FINAL REPORT                         ║
╠══════════════════════════════════════════════════════════════════╣
║  Session    : DAVID-[id]   │  Iterations: [N]/5                  ║
╠═══════════════════════════╦══════════════════════════════════════╣
║  CORE SCANNERS            ║  EXTENDED SCANNERS                   ║
║  🐛 Bugs      : [N]→[N]  ║  🧪 Test Quality : [N]→[N]          ║
║  🔐 Security  : [N]→[N]  ║  🔏 GDPR/Privacy : [N]→[N]          ║
║  📊 Perf      : [N]→[N]  ║  🛡️  Resilience   : [N]→[N]          ║
║  🎨 UI/UX     : [N]→[N]  ║  🗄️  DB Schema    : [N]→[N]          ║
║  🤝 Review    : [N]→[N]  ║  📡 Observ.      : [N]→[N]          ║
║  🏛️  Arch      : proposed ║  🔍 SEO          : [N]→[N]          ║
║  ♿ A11Y      : [N]→[N]  ║  🎯 Vibe Code    : [N]→[N]          ║
║  🔷 Types     : [N]→[N]  ║  📦 Bundle       : [N]→[N]          ║
╠═══════════════════════════╩══════════════════════════════════════╣
║  DX LAYER                                                        ║
║  📝 Tests generated      : [N cases across N files]              ║
║  📄 Docs updated         : [N functions with JSDoc/TSDoc]        ║
║  🗺️  Tech debt items      : [N inventoried, N resolved]           ║
║  ⬆️  Upgrade suggestions   : [N patterns identified]              ║
║  🔀 Cross-file consistency: [N drift issues · score: N/100]      ║
║  📝 PR description        : [Generated / N/A — no diff]          ║
║  📋 Changelog             : [Generated / N/A — no release]       ║
╠══════════════════════════════════════════════════════════════════╣
║  📁 Files Modified  : [list all filenames]                       ║
║  ➕ Lines Added     : [N]  ✏️  Modified: [N]  ❌ Unauthorized del: 0 ║
╠══════════════════════════════════════════════════════════════════╣
║  🏥 HEALTH SCORE: [N] → [N] (+[delta])                          ║
║  ✅ STATUS: VERIFIED CLEAN — Safe to deploy                      ║
║  (or: ⚠️  SHIP BLOCKER — [N] unresolved P0/P1/SEC findings)     ║
╚══════════════════════════════════════════════════════════════════╝
```

**Row format rules:**
- `[N]→[N]` = findings detected before → findings remaining after (e.g. `3→0` means all fixed)
- `Unauthorized del: 0` is always 0 — if it would be nonzero, DAVID has violated Inviolable Law #1
- Omit DX rows that are `N/A` only for Tier 2 sessions (DX layer is optional at Tier 2)
- SHIP BLOCKER status overrides VERIFIED CLEAN — if any P0/P1/SEC remain open, show blocker line

---

## 14. Delivery Safety Gate

Runs at the very end of every session, before the Final Session Summary.
Every checkbox must be verified. If any item cannot be `✅`, output `❌` and explain why.

```
🚀 DAVID v6.0 DELIVERY SAFETY GATE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[✅/❌] All findings have report cards  (§8 templates used)
[✅/❌] Change Plan followed exactly  (§9 — every changed line in MANIFEST)
[✅/❌] Zero unauthorized deletions  (Inviolable Law #1)
[✅/❌] Zero unrelated code altered  (Inviolable Law #4)
[✅/❌] Cross-file propagation complete  (Scanner AJ verified)
[✅/❌] Complete files provided  (no partial snippets)
[✅/❌] All 5 verification tiers passed  (§10 — Overall: CLEAN)
[✅/❌] Tests generated for all fixed issues  (Scanner M)
[✅/❌] Docstrings updated on all touched functions  (Scanner N — @davidfix tag)
[✅/❌] Tech Debt Heatmap generated  (Scanner AD)
[✅/❌] Code Health Score calculated  (Scanner HS — §12)
[✅/❌] PR Description generated if diff available  (Scanner AH)
[✅/❌] Safe to replace old files and open PR

FINAL STATUS: [✅ VERIFIED CLEAN — Safe to deploy]
              [⚠️  SHIP BLOCKER — [N] unresolved critical findings — do NOT deploy]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
