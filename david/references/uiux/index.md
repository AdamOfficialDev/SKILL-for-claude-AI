# DAVID — UI/UX Audit Reference (Router)

Scanner O master index. 14 sub-scanners across 4 reference files.
Load the file(s) that match the current audit scope.

## Which file to load?

| When auditing... | Load |
|------------------|------|
| Typography, spacing, color tokens, component states | `references/uiux/foundation.md` |
| Forms, mobile, animation, dark mode, skeleton/empty states, microcopy, responsive | `references/uiux/interaction.md` |
| Heuristics, design system consistency, navigation IA, emotional design | `references/uiux/heuristics.md` |
| Search, data tables, file upload, onboarding, wizards, real-time, offline, media, keyboard, data viz, drag-and-drop, trust | `references/uiux/patterns.md` |
| **Full UX audit / FULL SCAN** | **Load all 4 files** |

## Sub-file Summary

| File | Sections | Lines | Use Case |
|------|----------|-------|----------|
| `uiux/foundation.md` | Visual, Spacing, Color, States | ~232 | Every frontend audit |
| `uiux/interaction.md` | Forms, Mobile, Animation, Dark Mode, Skeleton, Empty, Microcopy, Perf, Responsive | ~716 | Interactive UI audit |
| `uiux/heuristics.md` | Framework-Specific, Nielsen x10, Cognitive Load, Design System, Nav IA, Emotional | ~1068 | Deep design review |
| `uiux/patterns.md` | Search, Tables, Upload, Onboarding, Wizard, Real-Time, Offline, Media, Keyboard, DataViz, Clipboard, DnD, Trust | ~891 | Specialized component audit |

## Scanner O Sub-Scanner Map

| Sub-Scanner | Primary file |
|-------------|-------------|
| O1 — Visual Hierarchy | `uiux/foundation.md` |
| O2 — Heuristics | `uiux/heuristics.md` |
| O3 — Interaction States | `uiux/foundation.md` |
| O4 — Form UX | `uiux/interaction.md` |
| O5 — Cognitive Load | `uiux/heuristics.md` |
| O6 — Mobile UX | `uiux/interaction.md` |
| O7 — Animation | `uiux/interaction.md` |
| O8 — Dark Mode | `uiux/interaction.md` |
| O9 — Skeleton / Empty States | `uiux/interaction.md` |
| O10 — Design System | `uiux/heuristics.md` |
| O11 — Microcopy | `uiux/interaction.md` |
| O12 — Navigation | `uiux/heuristics.md` |
| O13 — Responsive | `uiux/interaction.md` |
| O14 — Emotional / Delight | `uiux/heuristics.md` |
