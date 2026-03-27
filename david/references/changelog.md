# DAVID — PR Description & Changelog Reference (Scanner AH + AI)

Covers two closely related output scanners:
- **AH — PRDESC**: Generates structured PR descriptions from a git diff + findings
- **AI — CHANGELOG**: Generates versioned changelogs from commit history or fix sessions

Load this file when the user shares a git diff and wants a PR description, or when
they ask for a release changelog / release notes.

## Table of Contents
1. [When These Scanners Activate](#when-these-scanners-activate)
2. [Scanner AH — PR Description Generator](#scanner-ah--pr-description-generator)
3. [PR Description Templates](#pr-description-templates)
4. [PR Quality Scoring](#pr-quality-scoring)
5. [Scanner AI — Changelog Generator](#scanner-ai--changelog-generator)
6. [Conventional Commits Parsing](#conventional-commits-parsing)
7. [Changelog Templates](#changelog-templates)
8. [Semver Bump Logic](#semver-bump-logic)
9. [What Makes a Bad PR / Changelog](#what-makes-a-bad-pr--changelog)

---

## When These Scanners Activate

| Scanner | Trigger |
|---------|---------|
| AH — PRDESC | Git diff shared + post-fix session · `"buatkan PR desc"` · `"generate PR description"` · `"buat deskripsi PR"` · `"write PR"` |
| AI — CHANGELOG | `"changelog"` · `"release notes"` · `"what changed"` · `"buat changelog"` · `"release v"` · list of commits shared |

Both scanners can run together when the user shares commits AND asks for a PR.

---

## Scanner AH — PR Description Generator

### Input DAVID needs

To generate a quality PR description, DAVID needs at minimum:
- The git diff (or a summary of changed files)
- The original task/ticket context (if available)
- Whether it is a feature, bugfix, hotfix, refactor, or chore

If these are not provided, DAVID asks for exactly one targeted question:
> "What was this PR trying to solve? (link a ticket or one sentence is fine)"

### Generation process

```
1. Read the diff → build change map (files, +/- lines, type of change)
2. Identify the PRIMARY change (the main thing this PR does)
3. Identify SECONDARY changes (supporting changes, cleanup)
4. Detect breaking changes (signature change, removed export, DB migration)
5. Detect risk areas (auth, payments, data deletion, config changes)
6. Fill the PR template → score it → deliver
```

---

## PR Description Templates

### Standard Template (all PR types)

```markdown
## What

[1–3 sentences describing WHAT changed. Focus on the outcome, not the implementation.
Avoid "I changed X" — write "X now does Y".]

## Why

[1–2 sentences explaining WHY this change was needed.
Link to ticket/issue if available: Closes #123 or Fixes PROJ-456.]

## How

[Optional: 2–4 bullet points of the key implementation decisions, only if non-obvious.
Skip if the What section is self-explanatory.]

## Testing

[How the reviewer can verify this works. Be specific:]
- [ ] Unit: `npm test -- auth.spec.ts`
- [ ] Manual: Log in as a test user → navigate to /profile → verify email updates
- [ ] Edge case: Try with a deleted user ID → confirm 404 response

## Breaking Changes

[List any breaking changes explicitly. If none, write: None.]

## Screenshots / Recordings

[For UI changes: before/after screenshots. For API changes: request/response examples.
Skip if not applicable.]

## Checklist

- [ ] Tests added or updated
- [ ] Docs updated (README / JSDoc / OpenAPI)
- [ ] No `console.log` left in production code
- [ ] No hardcoded secrets or tokens
- [ ] Migration is reversible (if applicable)
- [ ] Feature flag added (if applicable)
```

### Hotfix Template (urgent production fix)

```markdown
## 🚨 Hotfix: [one-line summary]

**Severity:** P0 / P1
**Affected:** [which users/endpoints are impacted]
**Root cause:** [what was wrong]

## Fix

[What was changed to fix it. Be specific about the line/function.]

## Verification

- [ ] Reproduced the bug on staging
- [ ] Fix confirmed on staging
- [ ] Rollback plan: [how to undo if this makes things worse]

## Follow-up

[Any tech debt or deeper fix that should come after this hotfix stabilizes.]
```

### Refactor Template

```markdown
## Refactor: [what was refactored]

**Motivation:** [Why this refactor was needed — CC score, maintainability, etc.]
**Scope:** [Files/modules affected]
**Behavior change:** None (pure refactor)

## Changes

- [Specific structural change 1]
- [Specific structural change 2]

## Testing

All existing tests pass without modification:
```
npm test
```
[Add any new tests written to cover previously untested paths exposed by refactor.]
```

---

## PR Quality Scoring

After generating a PR description, DAVID scores it and shows which elements are present:

```
📋 PR DESCRIPTION QUALITY SCORE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[✅] What — outcome clearly described
[✅] Why — motivation / ticket linked
[✅] How — key decisions noted
[✅] Testing steps — specific and runnable
[✅] Breaking changes — explicitly stated (None)
[✅] Screenshots — included for UI change
[✅] Checklist — all items addressed
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Score: 7/7 — Ready to merge
```

Score thresholds:
- **7/7** — Ready to merge
- **5–6/7** — Acceptable, flag missing items
- **3–4/7** — Weak — reviewer will struggle to understand intent
- **<3/7** — Reject — missing critical context

---

## Scanner AI — Changelog Generator

### Input formats DAVID accepts

1. **Git log** — `git log --oneline v1.2.0..HEAD`
2. **Fix session summary** — DAVID's own finding list from the current session
3. **Manual list** — user pastes a bullet list of changes
4. **Mixed** — git log + additional context

### Generation process

```
1. Parse commits → classify each by type (feat/fix/perf/refactor/chore/docs/test/security)
2. Detect scope from path/files touched (auth, api, ui, db, infra, etc.)
3. Identify breaking changes (BREAKING CHANGE in commit body, ! in type)
4. Determine semver bump (see Semver Bump Logic section)
5. Group entries → sort by impact within group → render changelog
6. Flag any vague commits that need manual review
```

---

## Conventional Commits Parsing

DAVID parses [Conventional Commits](https://www.conventionalcommits.org/) format.

### Commit format
```
<type>(<scope>): <description>

[optional body]

[optional footer: BREAKING CHANGE: <description>]
```

### Type → Changelog group mapping

| Commit type | Changelog group | Semver impact |
|-------------|----------------|---------------|
| `feat` | ✨ New Features | MINOR |
| `fix` | 🐛 Bug Fixes | PATCH |
| `security` / `sec` | 🔐 Security | PATCH (or MINOR if new hardening) |
| `perf` | ⚡ Performance | PATCH |
| `refactor` | 🔨 Refactoring | PATCH (no user-facing change) |
| `docs` | 📝 Documentation | PATCH |
| `test` | 🧪 Tests | PATCH (usually omit from user changelog) |
| `chore` | 🔧 Maintenance | PATCH (usually omit from user changelog) |
| `BREAKING CHANGE` footer or `type!` | 💥 Breaking Changes | MAJOR |
| `deps` / `build` | 📦 Dependencies | PATCH |
| `ci` | omit from user changelog | — |

### Non-conventional commits — DAVID's fallback

When commits don't follow conventional format, DAVID infers from the message:

```
"Fixed login bug" → 🐛 Bug Fixes
"Add user export feature" → ✨ New Features  
"Update README" → 📝 Documentation
"Refactor auth service" → 🔨 Refactoring
"Upgrade lodash to 4.17.21" → 📦 Dependencies
"Remove deprecated getUserById" → 💥 Breaking Changes (if it was public API)
```

Vague commits get flagged:
```
⚠️ VAGUE COMMIT — needs manual review: "fix stuff" (abc1234)
```

---

## Changelog Templates

### Standard Release Changelog

```markdown
# Changelog

## [2.4.0] — 2025-03-26

### 💥 Breaking Changes

- **API**: `getUserProfile()` now requires `includeDeleted` parameter explicitly.
  Previously defaulted to true; now defaults to false.
  **Migration:** Add `includeDeleted: true` to existing callers if needed.

### ✨ New Features

- **Auth**: Added OAuth2 Google login support (#234)
- **Orders**: Export order history as CSV (max 10,000 rows) (#241)
- **API**: New `GET /users/{id}/activity` endpoint for user audit log (#238)

### 🐛 Bug Fixes

- **Payments**: Fixed race condition causing duplicate charge on double-click submit (#245)
- **UI**: Cart total no longer shows NaN when discount code is removed (#243)
- **Auth**: Password reset email now sends correctly for users with + in email address (#239)

### 🔐 Security

- Upgraded `jsonwebtoken` to 9.0.2 (fixes CVE-2022-23529)
- Added rate limiting on `/login` endpoint (10 req/min per IP)
- Removed internal user IDs from error response bodies

### ⚡ Performance

- Order list page load time reduced by ~60% (added DB indexes on user_id, created_at)
- Implemented Redis caching for product catalog (1-hour TTL)

### 📦 Dependencies

- `next`: 14.1.0 → 14.2.3
- `prisma`: 5.8.1 → 5.11.0
- Removed unused `moment` (replaced with native Date + `date-fns`)

---

## [2.3.1] — 2025-03-10

### 🐛 Bug Fixes

- **Checkout**: Fixed order confirmation email not sending for guest users (#231)
```

### Keep a Changelog format (alternative)

```markdown
# Changelog

All notable changes to this project will be documented in this file.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).
This project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- OAuth2 Google login support
- CSV export for order history

### Changed
- `getUserProfile()` now defaults `includeDeleted` to false (was true)

### Fixed
- Race condition on payment submit

### Security
- Upgraded jsonwebtoken to 9.0.2

## [2.3.1] — 2025-03-10

### Fixed
- Order confirmation email not sending for guest users
```

### Internal / Developer Changelog (for team, not users)

```markdown
## Sprint 24 — Internal Release Notes (2025-03-26)

### What shipped
- Google OAuth (#234) — @developer — tested on staging ✅
- CSV export (#241) — @developer — QA pending ⏳

### Tech debt addressed
- Removed 3 deprecated API endpoints (v1/users/*, v1/orders/old)
- Reduced test suite runtime by 40s (parallelised integration tests)
- Upgraded 6 dependencies with known vulnerabilities

### Known issues in this release
- CSV export fails for users with >10,000 orders (ticket: PROJ-301)
- Google OAuth not working on Safari iOS 16 (ticket: PROJ-298)

### Rollback procedure
git revert abc1234  # reverts the auth service changes
# OR
kubectl rollout undo deployment/api-server
```

---

## Semver Bump Logic

```
Current version: X.Y.Z

Has any BREAKING CHANGE?
  YES → X+1.0.0  (MAJOR bump, reset minor and patch to 0)
  NO  ↓

Has any feat / new feature?
  YES → X.Y+1.0  (MINOR bump, reset patch to 0)
  NO  ↓

Has any fix / security / perf / refactor / docs?
  YES → X.Y.Z+1  (PATCH bump)
  NO  → no version bump needed (ci, chore only)
```

### Version bump output

```
🏷️  SEMVER RECOMMENDATION
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Current  : v2.3.1
Bump     : MINOR (new features detected)
Suggested: v2.4.0

Reason:
  [MAJOR] No breaking changes detected
  [MINOR] 3 new features (OAuth, CSV export, activity endpoint)
  [PATCH] 4 bug fixes, 1 security patch, 2 perf improvements
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## What Makes a Bad PR / Changelog

### Bad PR descriptions DAVID will flag

```
❌ Title: "fix"
   → No context whatsoever. Reviewer cannot understand intent.

❌ What: "Changed some stuff in the auth module"
   → Vague. What changed? What does it do now?

❌ Testing: "tested locally"
   → Not reproducible. How did you test? What inputs?

❌ Breaking changes: [empty / not mentioned]
   → When there IS a breaking change, omission = trap for reviewers.
```

### Bad changelog entries DAVID will flag

```
❌ "Fixed bugs"
   → Which bugs? Who was affected?

❌ "Performance improvements"
   → What was slow? How much faster?

❌ "Minor updates"
   → Meaningless. Everything is an update.

❌ Including internal refactors in user-facing changelog
   → Users don't care that you renamed a variable.
```

### DAVID's fix for vague commits

When asked to generate a changelog from vague commits, DAVID:
1. Groups them by inferred type
2. Rewrites each entry to be user-facing
3. Flags the originals as "needs better commit discipline"
4. Appends a note recommending Conventional Commits adoption
