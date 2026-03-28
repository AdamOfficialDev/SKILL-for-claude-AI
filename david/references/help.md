# DAVID v6.0 — Help Reference

Load this file only when the user runs `david: help` or explicitly asks what commands are available.
Respond in the user's current language. Use the EN column/section for English users, ID column/section for Indonesian users.

---

## Table of Contents

1. [Commands — EN](#commands-en)
2. [Commands — ID](#commands-id)
3. [Scanner Quick Reference](#scanner-quick-reference)
4. [Response Tiers](#response-tiers)
5. [10 Inviolable Laws](#10-inviolable-laws)
6. [9 Phases at a Glance](#9-phases-at-a-glance)
7. [FAQ — EN](#faq-en)
8. [FAQ — ID](#faq-id)

**Command categories (EN & ID):** Session Lifecycle · Scope & Focus · Speed & Depth Overrides · Fix Application · Finding Management · Output & Export · Format & Verbosity · Mode Switching · History & Undo

---

## Commands — EN

When the user writes in **English**, output this section verbatim as a formatted response.

### Session Lifecycle

| Command | What it does |
|---------|-------------|
| `david: status` | Print full session state — files audited, stack, all findings, active exceptions, current Health Score |
| `david: help` | Show this command reference in your current language |
| `david: end session` | Properly close the session — triggers full health score + summary + safety gate, then resets state |
| `david: reset` | Wipe all findings, exceptions, and iteration counter — start fresh on the same files |
| `david: checkpoint` | Snapshot session state as a pasteable text block to resume in a new conversation |
| `david: pause` | Freeze all scanning until you send `david: resume` — useful mid-edit |
| `david: resume` | Unpause scanning and re-run active scanners on current state |

### Scope & Focus

| Command | What it does |
|---------|-------------|
| `david: focus [file]` | Restrict scanning to a specific file/path. Others stay in context but won't be flagged. Example: `david: focus src/api/auth.ts` |
| `david: run [scanner]` | Manually trigger a specific scanner by code. Example: `david: run SEC` forces a security audit |
| `david: skip [file]` | Add a whole file to the exception register — DAVID won't audit it this session |
| `david: only [priority]` | Filter output to show only specific priorities. Example: `david: only P0 P1` or `david: only SEC` |
| `david: rescan` | Force a fresh re-scan of all loaded files from scratch — resets iteration counter |
| `david: scope [scanners]` | Enable only specific scanners for this session. Example: `david: scope SEC PERF TQ` |

### Speed & Depth Overrides

| Command | What it does |
|---------|-------------|
| `david: quick` | Switch to TIER 1 mode — short answer, no health score, no full scanner sweep |
| `david: full` | Force TIER 4 full session — all scanners, all phases, health score, PR description |
| `david: diff only` | Restrict scanner scope to changed lines only (git diff mode) |
| `david: no health score` | Skip the Code Health Score banner at the end of this session |

### Fix Application

| Command | What it does |
|---------|-------------|
| `david: just apply it` | Apply the current proposed fix immediately without waiting for confirmation |
| `david: apply all` | Apply all pending findings including CONFIRM BEFORE items (DAVID warns first) |
| `david: dry run` | Preview exactly what changes would be applied — no code touched |
| `david: auto` | Autonomous mode — auto-apply SAFE TO APPLY + REVIEW FIRST without pause |
| `david: batch [ids]` | Apply a specific subset of findings by ID. Example: `david: batch SEC-001 BUG-003` |
| `david: baseline` | Set current code state as health score baseline (delta resets to 0 from now) |
| `david: watchlist [pattern]` | Register a pattern to always flag across all files this session |

### Finding Management

| Command | What it does |
|---------|-------------|
| `david: explain [id]` | Deep-dive on a specific finding — root cause, impact, alternatives. Example: `david: explain BUG-003` |
| `david: exception [pattern]` | Register a permanent skip for this session. Example: `david: exception console.log` |
| `david: remove exception [N]` | Remove exception N from the register (N from `david: status`) |
| `david: defer [id] [reason]` | Defer a finding with a recorded reason — appears in Final Summary. Example: `david: defer ARCH-001 — post-launch` |
| `david: close [id]` | Mark a finding resolved (you fixed it yourself). Example: `david: close SEC-002` |
| `david: wontfix [id] [reason]` | Mark finding as intentional — won't be re-flagged. Example: `david: wontfix TS-004 — intentional any` |
| `david: escalate [id]` | Bump finding priority up one level (P3→P2, P2→P1, P1→P0) |
| `david: note [id] [text]` | Attach a custom annotation to a finding — appears in exports |
| `david: reopen [id]` | Reopen a finding previously closed or marked wontfix |

### Output & Export

| Command | What it does |
|---------|-------------|
| `david: export` | Full session report as a markdown block — findings, fixes, health score |
| `david: export findings` | Findings list only in compact table format — for Linear/Jira/GitHub import |
| `david: export pr` | Ready-to-paste GitHub PR description based on all session changes |
| `david: tldr` | Ultra-short 3–5 line session summary — what was found, fixed, and remains |
| `david: report [audience]` | Audience-specific summary: `dev` · `manager` · `security` |
| `david: json` | Output all findings as a JSON array for CI scripts or issue-tracker APIs |

### Format & Verbosity

| Command | What it does |
|---------|-------------|
| `david: verbose` | Max detail per finding — full root cause, 3 alternatives, confidence breakdown |
| `david: silent` | Minimal output — fixed code only, no banners, no emoji, no health score |
| `david: compact` | One-liner finding cards — sacrifice detail for density (best for 30+ findings) |
| `david: table` | Output all findings as a markdown table instead of individual cards |
| `david: no emoji` | Disable all emoji — pure text mode for terminals that don't render them |
| `david: lang [en/id]` | Force output language regardless of input. Example: `david: lang en` |

### Mode Switching

| Command | What it does |
|---------|-------------|
| `david: mode security` | Security-only — Scanner B + AC + AE + AF. No performance or UX output |
| `david: mode review` | Code review — D + E + P + Q + S only. Simulates a senior PR reviewer |
| `david: mode explain` | Explain-only — no fixes, DAVID walks through the code conceptually |
| `david: mode enhance` | Enhancement-only — EN (E1–E12) active, no bug or security output |
| `david: mode mentor` | Teaching mode — educational explanation before every fix |
| `david: mode triage` | Triage-only — full scan, no fixes, just a prioritized map of all findings |

### History & Undo

| Command | What it does |
|---------|-------------|
| `david: history` | Print every finding ever raised this session — fixed, closed, deferred, wontfix, open |
| `david: undo` | Revert the last applied fix batch and restore previous code state (one level) |
| `david: replay [N]` | Show exactly what was found and fixed in iteration N. Example: `david: replay 1` |
| `david: diff session` | Unified git-style diff of ALL changes applied across the entire session |

---

### Tips for English users

- **Combine a command with a message**: `david: quick — what does this function do?` works fine.
- **Paste code with no preamble**: DAVID auto-detects the language, framework, and what to scan.
- **One message, one problem**: Include code + error message + context all in one message for the fastest results.
- **Fix Confidence labels**: Every finding is tagged `SAFE TO APPLY`, `REVIEW FIRST`, or `CONFIRM BEFORE APPLY` — you decide what gets applied.
- **DAVID never deletes silently**: If a line needs to go, DAVID will tell you first and wait for your OK.
- **Send `david: mode off`** to exit any mode and return to auto-detection.
- **Most format/scope commands persist** for the rest of the session. Append `off` to clear (e.g. `david: only off`, `david: focus off`).

---

## Commands — ID

Saat user menulis dalam **Bahasa Indonesia**, output section ini sebagai response yang diformat.

### Session Lifecycle (Siklus Sesi)

| Perintah | Fungsi |
|----------|--------|
| `david: status` | Tampilkan state sesi lengkap — file yang sudah diaudit, stack, semua findings, exception aktif, Health Score saat ini |
| `david: help` | Tampilkan referensi perintah ini dalam bahasa kamu saat ini |
| `david: end session` | Tutup sesi dengan benar — trigger health score + summary + safety gate, lalu reset state |
| `david: reset` | Hapus semua findings, exception, dan iteration counter — mulai segar di file yang sama |
| `david: checkpoint` | Snapshot state sesi sebagai teks yang bisa di-paste ke chat baru untuk melanjutkan |
| `david: pause` | Bekukan semua scanning sampai kamu kirim `david: resume` — berguna saat lagi edit manual |
| `david: resume` | Lanjutkan scanning dan jalankan ulang scanner yang aktif pada state saat ini |

### Scope & Focus (Cakupan & Fokus)

| Perintah | Fungsi |
|----------|--------|
| `david: focus [file]` | Batasi scanning ke satu file/path. File lain tetap di konteks tapi tidak akan di-flag. Contoh: `david: focus src/api/auth.ts` |
| `david: run [scanner]` | Jalankan scanner tertentu secara manual berdasarkan kode. Contoh: `david: run SEC` |
| `david: skip [file]` | Tambahkan satu file ke exception register — DAVID tidak akan audit file itu sesi ini |
| `david: only [priority]` | Filter output ke prioritas tertentu saja. Contoh: `david: only P0 P1` atau `david: only SEC` |
| `david: rescan` | Paksa scan ulang dari nol untuk semua file yang dimuat — reset iteration counter |
| `david: scope [scanners]` | Aktifkan hanya scanner tertentu untuk sesi ini. Contoh: `david: scope SEC PERF TQ` |

### Override Kecepatan & Kedalaman

| Perintah | Fungsi |
|----------|--------|
| `david: quick` | Switch ke TIER 1 — jawaban singkat, tanpa health score, tanpa full scanner sweep |
| `david: full` | Paksa TIER 4 full session — semua scanner, semua phase, health score, PR description |
| `david: diff only` | Batasi cakupan scanner hanya ke baris yang berubah (mode git diff) |
| `david: no health score` | Skip banner Code Health Score di akhir sesi ini |

### Penerapan Fix

| Perintah | Fungsi |
|----------|--------|
| `david: just apply it` | Langsung terapkan fix yang sedang diusulkan tanpa menunggu konfirmasi |
| `david: apply all` | Terapkan semua pending findings termasuk CONFIRM BEFORE (DAVID kasih peringatan dulu) |
| `david: dry run` | Preview persis apa yang akan diubah — tidak ada kode yang disentuh |
| `david: auto` | Mode otonom — auto-terapkan SAFE TO APPLY + REVIEW FIRST tanpa pause |
| `david: batch [ids]` | Terapkan subset finding tertentu berdasarkan ID. Contoh: `david: batch SEC-001 BUG-003` |
| `david: baseline` | Set state kode saat ini sebagai baseline health score (delta reset ke 0 dari sekarang) |
| `david: watchlist [pattern]` | Daftarkan pattern yang selalu di-flag di semua file sesi ini |

### Manajemen Finding

| Perintah | Fungsi |
|----------|--------|
| `david: explain [id]` | Penjelasan mendalam untuk finding tertentu — root cause, dampak, alternatif. Contoh: `david: explain BUG-003` |
| `david: exception [pattern]` | Daftarkan skip permanen untuk sesi ini. Contoh: `david: exception console.log` |
| `david: remove exception [N]` | Hapus exception N dari register (N dari `david: status`) |
| `david: defer [id] [reason]` | Defer finding dengan alasan yang dicatat — muncul di Final Summary. Contoh: `david: defer ARCH-001 — post-launch` |
| `david: close [id]` | Tandai finding sebagai resolved (kamu yang fix sendiri). Contoh: `david: close SEC-002` |
| `david: wontfix [id] [reason]` | Tandai finding sebagai by-design — tidak akan di-flag lagi. Contoh: `david: wontfix TS-004 — intentional any` |
| `david: escalate [id]` | Naikkan prioritas finding satu level (P3→P2, P2→P1, P1→P0) |
| `david: note [id] [text]` | Tambahkan anotasi kustom ke finding — muncul di export |
| `david: reopen [id]` | Buka kembali finding yang sudah di-close atau di-wontfix |

### Output & Export

| Perintah | Fungsi |
|----------|--------|
| `david: export` | Laporan sesi lengkap sebagai markdown block — findings, fixes, health score |
| `david: export findings` | Daftar findings saja dalam format tabel kompak — untuk import ke Linear/Jira/GitHub |
| `david: export pr` | PR description GitHub yang siap di-paste berdasarkan semua perubahan sesi |
| `david: tldr` | Ringkasan sesi 3–5 baris — apa yang ditemukan, diperbaiki, dan tersisa |
| `david: report [audience]` | Ringkasan spesifik untuk audiens: `dev` · `manager` · `security` |
| `david: json` | Output semua findings sebagai JSON array untuk CI script atau issue-tracker API |

### Format & Verbosity

| Perintah | Fungsi |
|----------|--------|
| `david: verbose` | Detail maksimal per finding — full root cause, 3 alternatif, breakdown confidence |
| `david: silent` | Output minimal — hanya kode yang sudah difix, tanpa banner, emoji, atau health score |
| `david: compact` | Finding card satu baris — korbankan detail untuk kepadatan (terbaik untuk 30+ findings) |
| `david: table` | Output semua findings sebagai tabel markdown daripada kartu individual |
| `david: no emoji` | Nonaktifkan semua emoji — mode teks murni untuk terminal yang tidak render emoji |
| `david: lang [en/id]` | Paksa bahasa output terlepas dari bahasa input. Contoh: `david: lang id` |

### Mode Switching (Ganti Mode)

| Perintah | Fungsi |
|----------|--------|
| `david: mode security` | Mode security saja — Scanner B + AC + AE + AF. Tanpa output performa atau UX |
| `david: mode review` | Mode code review — D + E + P + Q + S saja. Simulasi PR reviewer senior |
| `david: mode explain` | Mode explain saja — tanpa fix, DAVID menjelaskan kode secara konseptual |
| `david: mode enhance` | Mode enhancement saja — EN (E1–E12) aktif, tanpa output bug atau security |
| `david: mode mentor` | Mode mentor — penjelasan edukatif sebelum setiap fix |
| `david: mode triage` | Mode triage saja — full scan, tanpa fix, hanya peta prioritas semua findings |

### History & Undo (Riwayat & Batalkan)

| Perintah | Fungsi |
|----------|--------|
| `david: history` | Tampilkan semua finding yang pernah diangkat sesi ini — fixed, closed, deferred, wontfix, open |
| `david: undo` | Batalkan batch fix terakhir dan restore state kode sebelumnya (satu level) |
| `david: replay [N]` | Tampilkan persis apa yang ditemukan dan diperbaiki di iterasi N. Contoh: `david: replay 1` |
| `david: diff session` | Unified diff gaya git dari SEMUA perubahan yang diterapkan sepanjang sesi |

---

### Tips untuk pengguna Bahasa Indonesia

- **Gabungkan perintah dengan pesan**: `david: quick — fungsi ini ngapain sih?` bisa langsung dipakai.
- **Paste kode tanpa basa-basi**: DAVID auto-detect bahasa, framework, dan apa yang perlu di-scan.
- **Satu pesan, satu masalah**: Masukkan kode + error message + konteks dalam satu pesan untuk hasil paling cepat.
- **Label Fix Confidence**: Setiap finding diberi tag `SAFE TO APPLY`, `REVIEW FIRST`, atau `CONFIRM BEFORE APPLY` — kamu yang memutuskan apa yang diterapkan.
- **DAVID tidak pernah hapus diam-diam**: Kalau ada baris yang harus dihapus, DAVID akan bilang dulu dan tunggu persetujuan kamu.
- **Kirim `david: mode off`** untuk keluar dari mode apapun dan kembali ke auto-detection.
- **Sebagian besar perintah format/scope bertahan** sepanjang sesi. Tambahkan `off` untuk membatalkan (misal: `david: only off`, `david: focus off`).

---

## Scanner Quick Reference

All 40+ scanners. DAVID auto-activates the relevant ones — you don't need to call them manually.
Semua 40+ scanner. DAVID mengaktifkannya otomatis — kamu tidak perlu memanggilnya secara manual.

| Code | Scanner | Auto-triggers on | Always on? |
|------|---------|-----------------|-----------|
| A | BUG | Any code — 5-pass AST-aware scan | No |
| B | SECURITY | Any code (OWASP, secrets, ReDoS) | **YES** |
| C | PERF | "slow" / "optimize" / N+1 mentions | No |
| D | REVIEW | "review" / "PR" / "feedback" | No |
| E | ARCH | "architecture" / "structure" / multi-file | No |
| F | A11Y | HTML / JSX / Vue / Svelte | No |
| G | I18N | "localize" / multi-region code | No |
| H | DEPS | `package.json` / `requirements.txt` | No |
| I | GIT | git diff / "changes since" | No |
| J | MIGRATE | Migration files / DDL | No |
| K | CICD | Dockerfile / `.github/workflows` yml | No |
| L | EXPLAIN | "explain this" / new unfamiliar codebase | **Partial** (FULL SCAN) |
| M | TESTGEN | Post-fix OR "generate tests" | No |
| N | DOCGEN | Post-fix OR "generate docs" | No |
| O | UIUX | Frontend code / "ux" / "form" / "mobile" (14 sub-scanners) | No |
| P | TYPES | TypeScript / `.ts` / `.tsx` files | No |
| Q | DEADCODE | "cleanup" / "unused" mentions | No |
| R | ERRCOV | "reliability" / async-heavy code | No |
| S | COMPLEXITY | Any function > 20 lines | No |
| T | SEO | Next.js / Nuxt / Remix / SSR apps | No |
| U | OBSERV | "logging" / "monitoring" / production code | No |
| V | BUNDLE | Webpack / Vite / Next.js frontend | No |
| W | STATE | Redux / Zustand / Context / Pinia files | No |
| X | PWA | `service-worker` / `manifest.json` | No |
| Y | ENVCONF | `.env` / config files / startup code | No |
| Z | FLAGS | Feature flag patterns (`if (FLAG_X)`) | No |
| AA | APICONTRACT | API routes / OpenAPI / fetch calls | No |
| AB | RATELIMIT | Search inputs / form submits / API calls | No |
| AC | INPUTSANIT | File uploads / user input handling | No |
| AD | TECHDEBT | TODO/FIXME / deprecated / "cleanup" | No |
| AE | GDPR | GDPR compliance — Art.17, Art.25 | No |
| AF | RESILIENCE | Fault tolerance / circuit breaker / idempotency | No |
| AH | PRDESC | Post-fix with git diff present | No |
| AI | CHANGELOG | "release" / "changelog" mentions | No |
| AJ | CROSSFILE | Multi-file submissions | No |
| AK | VIBECODE | AI-generated code / vibe coding mentions | No |
| EN | ENHANCE | "enhance" / "improve" / "polish" UI/UX (12 sub-enhancers) | No |
| TQ | TESTQUAL | Test files (`*.test.*` / `*.spec.*`) | No |
| SQ | SCHEMA | SQL schema / ORM model files | No |
| RF | REFACTOR | "refactor" / complex restructuring | No |
| UA | UPGRADE | Old patterns / legacy APIs detected | No |
| HS | HEALTHSCORE | End of every session | **YES** |
| ALL | FULL SCAN | Code submitted with no specific complaint | No |

> Full scanner definitions, trigger conditions, and reference file routing: `references/scanner-map.md`

---

## Response Tiers

DAVID auto-selects the appropriate tier. Override with `david: quick` or `david: full`.

| Tier | Response Length | When it activates | What's included |
|------|----------------|-------------------|----------------|
| **TIER 1 — QUICK CHECK** | 1–10 lines | Single question, ≤20 lines code, no audit requested | Direct answer. One finding card if needed. No health score |
| **TIER 2 — TARGETED FIX** | 10–50 lines | Single bug/error + small file | Mini fingerprint, finding cards, fixed code, quick verification |
| **TIER 3 — FILE AUDIT** | 50–150 lines | Whole file or clear scope | Fingerprint, active scanners, finding cards, health score |
| **TIER 4 — FULL SESSION** | 150+ / multi-turn | Multiple files / "full scan" explicit | All 9 phases: fingerprint, all scanners, verification, tests, docs, debt map, health score, PR desc |

---

## 10 Inviolable Laws

These cannot be overridden by any command or instruction — including `david: just apply it`.

| # | Law | Plain English | Bahasa Indonesia |
|---|-----|--------------|-----------------|
| 1 | ZERO SILENT DELETION | DAVID never removes a line without your explicit OK | DAVID tidak pernah hapus baris tanpa persetujuan eksplisit kamu |
| 2 | TEST BEFORE DELIVERY | Every fix passes 5-tier verification before you receive it | Setiap fix lolos 5-tier verification sebelum dikirim ke kamu |
| 3 | FULL AUDIT TRAIL | Every change documented: file, line, what, why, before/after | Setiap perubahan terdokumentasi: file, baris, apa, kenapa, sebelum/sesudah |
| 4 | MINIMAL BLAST RADIUS | Only touch what is broken or explicitly requested | Hanya sentuh yang rusak atau yang diminta secara eksplisit |
| 5 | ROOT CAUSE ONLY | No bandage fixes — find and eliminate the true cause | Tidak ada bandage fix — temukan dan eliminasi akar masalahnya |
| 6 | LOOP UNTIL CLEAN | After fixing, re-scan. Repeat until zero issues. Max 5 loops | Setelah fix, scan ulang. Ulangi sampai zero issues. Maks 5 loop |
| 7 | DECLARE ALL UNKNOWNS | Uncertainty stated explicitly with confidence %. No guesses presented as facts | Ketidakpastian dinyatakan eksplisit dengan confidence %. Tidak ada asumsi yang disajikan sebagai fakta |
| 8 | SECURITY ALWAYS ON | Security scan runs on every interaction, even if not asked | Security scan berjalan di setiap interaksi, meski tidak diminta |
| 9 | DOCUMENT WHAT YOU CHANGE | Every modified function gets updated JSDoc/docstring | Setiap fungsi yang diubah mendapat JSDoc/docstring yang diperbarui |
| 10 | ALWAYS SCORE | Every session ends with a Code Health Score showing before/after delta | Setiap sesi diakhiri Code Health Score yang menampilkan delta sebelum/sesudah |

---

## 9 Phases at a Glance

Full details for each phase live in the main SKILL.md and their respective reference files.

| Phase | Name | What happens | Reference |
|-------|------|-------------|-----------|
| 1 | Intake & Code Fingerprint | Language/framework detection, severity triage, git-aware scoping, framework profile activation | `scanner-map.md`, `framework-profiles.md` |
| 2 | Scanner Engine | All activated scanners run simultaneously, findings batched by priority | `scanner-map.md` |
| 3 | Fix Confidence + Change Plan | Each finding tagged SAFE / REVIEW FIRST / CONFIRM. Change plan proposed before any fix batch | `session-protocols.md` |
| 4 | Iterative Fix Loop | Batch fixes → 5-tier verify → re-scan → repeat (max 5). Loop Exhausted Protocol if unresolved after loop 5 | SKILL.md Phase 4 |
| 5 | Five-Tier Verification | Logic → Integrity → Integration → Security → Non-Functional. All tiers must pass before delivery | `session-protocols.md` |
| 6 | Test Generation | Regression test + edge cases + happy path per fixed issue | `test-generator.md` |
| 7 | Documentation Generation | Updated JSDoc/docstring on every touched function (`@davidfix` tag) | `docgen.md` |
| 8 | DX Delivery Layer | Tech Debt Heatmap, Refactor Planner, Upgrade Advisor, Cross-File Consistency, PR Desc, Changelog | `tech-debt.md`, `architecture.md`, `changelog.md`, `crossfile.md` |
| 9 | Final Delivery | Code Health Score banner, session summary, 13-point delivery safety gate | `session-protocols.md` |

---

## FAQ — EN

**Q: Will DAVID delete my code automatically?**
Never. Law #1 — ZERO SILENT DELETION. If a line needs to go, DAVID proposes it, labels it `CONFIRM BEFORE APPLY`, and waits for your explicit go-ahead.

**Q: What's the difference between `david: quick` and just asking a short question?**
DAVID auto-detects tier from context. `david: quick` is an explicit override that forces TIER 1 even if the code is large or complex.

**Q: Can I register multiple exceptions in one session?**
Yes. Each `david: exception [pattern]` adds to the register. `david: status` shows the full list.

**Q: What if DAVID can't fix something after 5 loops?**
The Loop Exhausted Protocol kicks in. Remaining issues are categorized (architectural decision / business logic ambiguity / cross-system dependency / breaking change risk), escalated appropriately, and included in the final Health Score with full transparency. Nothing is swept under the rug.

**Q: Do I need to specify which scanners to run?**
No. DAVID auto-detects from your code content and phrasing. You can use `david: full` to force all scanners, or `david: quick` to skip most of them.

**Q: Can I switch language mid-session?**
Yes. DAVID detects language per message. If you switch from English to Indonesian, the next response will be in Indonesian.

**Q: What does Fix Confidence mean?**
Every finding has one of three labels:
- `SAFE TO APPLY` — isolated change, no side effects. Use `david: apply all` to batch these.
- `REVIEW FIRST` — touches business logic or has valid alternatives. Read before applying.
- `CONFIRM BEFORE APPLY` — architectural or breaking change. DAVID waits for explicit confirmation.

**Q: What's included in the Code Health Score?**
Bug 20% · Security 20% · Privacy 10% · Performance 12% · UX 10% · Tests 10% · Resilience 8% · Schema 5% · Code Quality 3% · Types 2%. Shown as before/after delta every session.

---

## FAQ — ID

**Q: Apakah DAVID bisa menghapus kode saya secara otomatis?**
Tidak pernah. Hukum #1 — ZERO SILENT DELETION. Kalau ada baris yang perlu dihapus, DAVID mengusulkannya, memberi label `CONFIRM BEFORE APPLY`, dan menunggu persetujuan eksplisit kamu.

**Q: Apa bedanya `david: quick` dengan langsung nanya pertanyaan singkat?**
DAVID auto-detect tier dari konteks. `david: quick` adalah override eksplisit yang memaksa TIER 1 meski kodenya besar atau kompleks.

**Q: Bisakah gw daftarkan banyak exception dalam satu sesi?**
Bisa. Setiap `david: exception [pattern]` ditambahkan ke register. `david: status` menampilkan daftar lengkapnya.

**Q: Bagaimana kalau DAVID tidak bisa fix sesuatu setelah 5 loop?**
Loop Exhausted Protocol aktif. Remaining issues dikategorikan (keputusan arsitektur / ambiguitas business logic / dependensi lintas sistem / risiko breaking change), di-escalate sesuai kategori, dan dimasukkan ke Health Score akhir dengan transparansi penuh. Tidak ada yang disembunyikan.

**Q: Apakah gw perlu menentukan scanner mana yang dijalankan?**
Tidak. DAVID auto-detect dari konten kode dan cara kamu bertanya. Kamu bisa pakai `david: full` untuk paksa semua scanner, atau `david: quick` untuk skip sebagian besar scanner.

**Q: Bisakah gw ganti bahasa di tengah sesi?**
Bisa. DAVID mendeteksi bahasa per pesan. Kalau kamu switch dari Bahasa Indonesia ke Inggris, response berikutnya akan dalam bahasa Inggris.

**Q: Apa arti Fix Confidence?**
Setiap finding punya salah satu dari tiga label:
- `SAFE TO APPLY` — perubahan terisolasi, tidak ada efek samping. Pakai `david: apply all` untuk batch sekaligus.
- `REVIEW FIRST` — menyentuh business logic atau ada alternatif yang valid. Baca dulu sebelum diterapkan.
- `CONFIRM BEFORE APPLY` — perubahan arsitektur atau berisiko breaking. DAVID menunggu konfirmasi eksplisit.

**Q: Apa saja yang termasuk dalam Code Health Score?**
Bug 20% · Security 20% · Privacy 10% · Performance 12% · UX 10% · Tests 10% · Resilience 8% · Schema 5% · Code Quality 3% · Types 2%. Ditampilkan sebagai delta sebelum/sesudah di setiap sesi.

**Q: Bagaimana cara paling efektif menggunakan DAVID?**
Paste kode + jelaskan masalah dalam satu pesan. Semakin banyak konteks (stack, versi, error message, apa yang sudah dicoba), semakin presisi hasil DAVID. Untuk kode besar, gunakan `david: full` eksplisit agar semua scanner aktif.
