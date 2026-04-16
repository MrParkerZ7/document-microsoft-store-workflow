# History — 2-generated/

## 2026-04-17 02:50
- **Updated**: All 9 .drawio files — cycle-review iter 2 fixes applied
  - Border hierarchy enforced via `_apply_border_hierarchy()` post-processing (all containers now 4/3/2 per topology)
  - Legends added to diagrams 04, 05, 07, 09

## 2026-04-17 02:28
- **Updated**: `08-trust-identity-flow.drawio` — cycle-review fixes applied
  - m6 color: warn (amber) → microsoft (blue) — now semantically consistent
  - Added Arrow Legend (explains 3 arrow colors)
  - Added Component Legend (explains 6 box color categories)
  - Summary upgraded from plain text to highlight box
  - Page height: 960 → 1220

## 2026-04-17 01:38
- **Updated**: All 9 .drawio files regenerated — arrows changed from orthogonal to straight by default
  - `arrow()` default style_type: "orthogonal" → "straight"
  - `arrow_free()` now delegates to `arrow()` with straight style
  - Removed forced entry/exit points so drawio picks natural connection points

## 2026-04-17 01:34
- **Updated**: 9 .drawio files regenerated after iteration 1 fixes
  - D3 s17 y-position: 1560 → 1470
  - D8 containers height: 720 → 780; page 900 → 960
  - D2 s13→s14 arrow: forced-vertical → free-routing with "queue" label

## 2026-04-17 01:29
- **Created**: 9 .drawio files generated via `🚫generate-diagrams.py`
  - `01-c4-context.drawio` — System context, actors, zones
  - `02-workflow-msix-path.drawio` — MSIX submission flow (5-lane swim)
  - `03-workflow-exe-path.drawio` — EXE submission flow (5-lane swim)
  - `04-decision-tree.drawio` — Path selection decision tree
  - `05-comparison-matrix.drawio` — MSIX vs EXE comparison table
  - `06-cicd-architecture.drawio` — GitHub Actions workflow architecture
  - `07-current-state-autoclickkey.drawio` — AutoClickKey current state (working/blocked/missing)
  - `08-trust-identity-flow.drawio` — Trust models (declarative vs cryptographic)
  - `09-update-flow.drawio` — Update propagation comparison

## 2026-04-17 01:28
- **Created**: `_discover-diagram.md` — Catalog of 9 applicable diagrams (5 high, 3 medium, 1 low)
