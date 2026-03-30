# 🛠️ SKILL-CREATOR
### *The meta-skill — bikin, test, dan improve skill baru*

> *"Skill terbaik adalah yang lu lupa bahwa lu pake — karena dia udah jadi bagian dari workflow lu."*

[![Skill Type](https://img.shields.io/badge/type-Meta%20Skill-orange?style=flat-square)](#)
[![Language](https://img.shields.io/badge/language-EN%20%2F%20ID-blue?style=flat-square)](#)
[![Author](https://img.shields.io/badge/author-AdamOfficialDev-purple?style=flat-square)](https://github.com/AdamOfficialDev)

---

## 🤔 Tentang Skill-Creator

Skill-Creator adalah **meta-skill** — skill yang digunakan untuk membuat skill baru. Ini adalah satu-satunya skill di arsenal yang tugasnya bukan menyelesaikan problem teknis, melainkan **memperluas kemampuan Claude itu sendiri**.

Dengan Skill-Creator, lo bisa:
- 🏗️ Bikin skill baru dari ide mentah
- ✏️ Edit dan improve skill yang sudah ada
- 🧪 Test skill dengan prompt realistis
- 📊 Ukur performa skill secara kuantitatif
- 🎯 Optimasi trigger description supaya skill aktif di waktu yang tepat

---

## ⚡ Cara Aktifkan

```
# Bikin skill baru
/skill-creator Gw mau bikin skill untuk [deskripsi]

# Update skill yang ada
/skill-creator Update skill DAVID untuk tambah scanner baru

# Optimize trigger skill
/skill-creator Optimize trigger description untuk skill REY
```

Atau cukup mention kata kunci:
- `"bikin skill"` / `"create skill"`
- `"improve skill"` / `"update skill"`
- `"test skill"` / `"eval skill"`
- `"optimize skill description"`

---

## 🔄 Alur Kerja Skill-Creator

```
┌─────────────────────────────────────────────────────────────┐
│                    SKILL CREATION LOOP                       │
└─────────────────────────────────────────────────────────────┘

    1. CAPTURE INTENT
       └─ Apa yang skill ini harus lakukan?
       └─ Kapan skill ini harus trigger?
       └─ Format output yang diharapkan?
              │
              ▼
    2. INTERVIEW & RESEARCH
       └─ Edge cases, input/output format
       └─ Success criteria, dependencies
       └─ Contoh prompt yang akan trigger skill
              │
              ▼
    3. WRITE SKILL.md DRAFT
       └─ YAML frontmatter (name + description)
       └─ Instruksi + output format
       └─ Reference files jika diperlukan
              │
              ▼
    4. TEST (2–3 prompt realistis)
       └─ Jalankan with-skill vs baseline
       └─ Save output per iterasi
              │
              ▼
    5. EVALUATE
       └─ Review output secara kualitatif (user)
       └─ Benchmark kuantitatif (assertions)
              │
              ▼
    6. ITERATE
       └─ Revisi SKILL.md berdasarkan feedback
       └─ Ulangi test + eval
              │
              ▼
    7. PACKAGE & DELIVER
       └─ .skill file siap install
```

---

## 📐 Anatomy of a Skill

### Struktur Folder
```
nama-skill/
├── SKILL.md                ← WAJIB. Otak skill.
└── resources/              ← Optional
    ├── scripts/            ← Executable code
    ├── references/         ← Docs tambahan (diload on-demand)
    └── assets/             ← Template, font, icon, dll
```

### Struktur SKILL.md
```markdown
---
name: nama-skill
description: >
  Kapan skill ini aktif dan apa yang dilakukannya.
  Semakin detail dan "pushy", semakin akurat trigger-nya.
---

# Nama Skill

## Apa yang dilakukan

## Step-by-step instruksi

## Output format
```

### 3-Level Loading System (Progressive Disclosure)

| Level | Konten | Ukuran | Kapan Diload |
|:---:|---|---|---|
| **1** | Metadata (name + description) | ~100 kata | ⚡ Selalu — untuk trigger detection |
| **2** | SKILL.md body | <500 baris | 🟢 Saat skill aktif |
| **3** | Bundled resources | Unlimited | 🔵 On-demand saat dibutuhkan |

---

## ✏️ Nulis Skill yang Efektif

### Tips Description (Trigger Mechanism Utama)

Description adalah satu-satunya hal yang Claude baca sebelum memutuskan apakah skill perlu diaktifkan. Tulis dengan baik!

```
❌ BURUK (terlalu generik):
"Skill untuk membantu debugging"

✅ BAGUS (specific + pushy):
"Aktifkan ketika user: paste code/error/diff, sebut 'debug/fix/audit/review',
minta security scan, GDPR audit, atau TypeScript error. Jangan aktifkan untuk:
nulis kode dari nol, penjelasan konsep."
```

**Prinsip description yang baik:**
- Sebutkan kata kunci trigger secara eksplisit
- Sebutkan juga apa yang TIDAK mengaktifkan skill
- Sedikit "pushy" — lebih baik over-trigger daripada under-trigger
- Maksimal ~100 kata untuk nama + description (yang selalu di context)

### Prinsip Menulis Instruksi

| ✅ DO | ❌ DON'T |
|---|---|
| Pakai imperative form ("Load X", "Run Y") | Pakai passive voice |
| Jelaskan KENAPA sesuatu penting | Hanya bilang HARUS tanpa alasan |
| Sertakan contoh input → output | Instruksi abstrak tanpa contoh |
| Organisir dengan hierarchy yang jelas | Dump semua info flat |
| Keep SKILL.md < 500 baris | Taruh semua di satu file |
| Reference file eksternal untuk detail | Repeat info di banyak tempat |

---

## 🧪 Cara Test Skill

Skill-Creator membantu lo test dengan membandingkan output with-skill vs baseline (tanpa skill):

```
Workspace structure:
skill-workspace/
└── iteration-1/
    ├── eval-basic-case/
    │   ├── with_skill/outputs/     ← Output dengan skill aktif
    │   ├── without_skill/outputs/  ← Output tanpa skill (baseline)
    │   └── eval_metadata.json      ← Prompt + assertions
    └── eval-edge-case/
        └── ...
```

### Menulis Assertion yang Baik

Setiap test case punya assertions — kriteria kuantitatif untuk menilai output:

```json
{
  "eval_id": 1,
  "eval_name": "debug-typescript-error",
  "prompt": "Fix error ini: Type 'string' is not assignable to type 'number'",
  "assertions": [
    {
      "id": "A1",
      "description": "Output berisi fixed code",
      "type": "contains_code"
    },
    {
      "id": "A2", 
      "description": "Menjelaskan root cause, bukan hanya fix",
      "type": "explains_root_cause"
    },
    {
      "id": "A3",
      "description": "Code Health Score ada di output",
      "type": "contains_health_score"
    }
  ]
}
```

### Kapan Skill Butuh Test Cases?

| Tipe Skill | Perlu Test Cases? |
|---|:---:|
| File transforms (PDF, DOCX, XLSX) | ✅ Ya |
| Data extraction | ✅ Ya |
| Code generation dengan format fixed | ✅ Ya |
| Workflow steps yang deterministic | ✅ Ya |
| Writing style / tone | ⚠️ Opsional |
| Creative output | ⚠️ Opsional |

---

## 🎯 Optimasi Trigger Description

Skill-Creator bisa optimize description agar skill trigger di waktu yang tepat (hanya di Claude Code / Cowork):

```bash
python -m scripts.run_loop \
  --eval-set trigger-eval.json \
  --skill-path path/to/skill \
  --model claude-sonnet-4-20250514 \
  --max-iterations 5
```

Proses otomatis ini:
1. Split eval set: 60% train / 40% held-out test
2. Evaluasi description saat ini (3x per query untuk reliabilitas)
3. Claude propose perbaikan berdasarkan yang gagal
4. Re-evaluasi, iterasi sampai max 5x
5. Return `best_description` berdasarkan test score (bukan train score)

> 💡 Tersedia hanya di Claude Code. Di Claude.ai, skip step ini.

---

## 📦 Cara Package Skill

Setelah skill selesai, package menjadi `.skill` file untuk distribusi:

```bash
python -m scripts.package_skill path/to/skill-folder
```

Output: `nama-skill.skill` — file yang bisa langsung di-install di Claude.

---

## 🛠️ Updating Skill yang Ada

Kalau mau update skill existing (bukan bikin dari nol):

1. **Preserve nama** — Jangan ubah `name` di frontmatter dan nama folder
2. **Copy dulu** sebelum edit — path skills mungkin read-only
   ```bash
   cp -r /mnt/skills/user/david/ /tmp/david/
   ```
3. Edit di `/tmp/`
4. Package dari copy tersebut
5. Install untuk replace versi lama

---

## 📋 Contoh: Bikin Skill dari Nol

```
User: /skill-creator Gw mau bikin skill untuk generate commit messages
      yang mengikuti Conventional Commits format

Skill-Creator:
1. Interview:
   - "Kamu mau skill ini trigger dari: (a) tiap kali ngomongin git,
     (b) cuma saat minta commit message, atau (c) selalu saat ada diff?"
   - "Ada format spesifik yang mau diikuti selain Conventional Commits?"

2. Draft SKILL.md:
   ---
   name: commit-message-generator
   description: >
     Generate commit messages mengikuti Conventional Commits spec.
     Aktifkan ketika user: paste git diff, minta commit message,
     sebut "conventional commits", atau mau push/commit kode.
   ---
   
   # Commit Message Generator
   ...

3. Test Cases:
   - "Buatin commit message untuk diff ini: [paste diff]"
   - "Gw baru tambahin auth feature, commit message-nya apa?"
   - "Fix typo di docs, commitnya gimana?"

4. Iterate berdasarkan hasil → Package → Done!
```

---

## 📁 Struktur Referensi

```
skill-creator/
├── SKILL.md                       ← Instruksi utama (yang lu sedang baca ini)
├── agents/
│   ├── grader.md                  ← Cara evaluate assertion vs output
│   ├── comparator.md              ← A/B comparison dua output
│   └── analyzer.md                ← Analisis kenapa satu versi lebih baik
└── references/
    └── schemas.md                 ← JSON structure untuk evals.json, grading.json, dll
```

---

## 🌍 Platform Notes

Behavior Skill-Creator berbeda tergantung platform:

| Feature | Claude.ai | Claude Code | Cowork |
|---|:---:|:---:|:---:|
| Draft + test skill | ✅ | ✅ | ✅ |
| Subagent parallel testing | ❌ | ✅ | ✅ |
| Browser-based eval viewer | ❌ | ✅ | ❌ (static HTML) |
| Description optimization loop | ❌ | ✅ | ✅ |
| Package .skill file | ✅ | ✅ | ✅ |
| Blind A/B comparison | ❌ | ✅ | ✅ |

---

## 💡 Tips & Best Practices

```
✅ Mulai kecil — 1 skill untuk 1 use case yang jelas
✅ Description yang "pushy" lebih baik dari yang malu-malu
✅ Test dengan prompt REALISTIS — bukan edge case artificial
✅ Pisahkan info besar ke reference files
✅ Keep SKILL.md < 500 baris
✅ Nama skill pakai kebab-case (commit-generator, not commitGenerator)

❌ Jangan bikin 1 skill yang coba handle segalanya
❌ Jangan taruh semua info di SKILL.md body
❌ Jangan skip test cases untuk skill kompleks
❌ Jangan pakai skill untuk sesuatu yang Claude bisa handle sendiri
```

---

<div align="center">

*Skill-Creator — The meta-skill that powers all other skills.*

**[← Kembali ke README](../README.md)**

</div>
