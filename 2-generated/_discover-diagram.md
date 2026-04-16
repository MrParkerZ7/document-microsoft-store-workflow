# Diagram Discovery Catalog

## Inventory

Based on analysis files in `1-analysis/`, the following `.drawio` diagrams are recommended.

## Catalog

### HIGH Priority (Core understanding)

| # | Filename | Type | Purpose | Source |
|---|----------|------|---------|--------|
| 1 | `01-c4-context.drawio` | C4 Context | Show all actors (Developer, GitHub, Partner Center, MS Review, End User) and their top-level interactions | 01-context-actors.md |
| 2 | `02-workflow-msix-path.drawio` | Workflow/Process | End-to-end MSIX pipeline (tag → CI → artifact → Partner Center → Review → Store → User) | 02-workflow-msix-path.md |
| 3 | `03-workflow-exe-path.drawio` | Workflow/Process | End-to-end EXE pipeline (tag → CI → sign → Release → URL → Partner Center → Review → User) | 03-workflow-exe-path.md |
| 4 | `04-decision-tree.drawio` | Flowchart | Decision flow: which path to pick given constraints | 05-decision-framework.md |
| 5 | `05-comparison-matrix.drawio` | Comparison | Side-by-side MSIX vs EXE across cost, effort, trust, flexibility | 05-decision-framework.md |

### MEDIUM Priority (Deep structure)

| # | Filename | Type | Purpose | Source |
|---|----------|------|---------|--------|
| 6 | `06-cicd-architecture.drawio` | Architecture | GitHub Actions jobs structure (ci.yml + release.yml), dependencies, secrets flow | 04-ci-cd-architecture.md |
| 7 | `07-current-state-autoclickkey.drawio` | Current State | AutoClickKey today: what works, what's blocked, what's missing | 06-current-state-autoclickkey.md |
| 8 | `08-trust-identity-flow.drawio` | Flow Diagram | How trust propagates: cert chain (EXE) vs Store identity (MSIX) | 01-context-actors.md |

### LOW Priority (Supplementary)

| # | Filename | Type | Purpose | Source |
|---|----------|------|---------|--------|
| 9 | `09-update-flow.drawio` | Workflow | How app updates propagate to users (both paths compared) | 02 & 03 |

## Rationale

- **C4 Context (#1)**: Foundation — establishes who's who before showing flows
- **Workflow diagrams (#2, #3)**: Temporal flow is the core of "how to publish"
- **Decision tree (#4)**: Directly answers user's confusion
- **Comparison matrix (#5)**: Quick reference for trade-offs
- **CI/CD architecture (#6)**: Technical depth on automation
- **Current state (#7)**: Personalized to AutoClickKey's situation
- **Trust/identity flow (#8)**: Explains subtle but important signing differences
- **Update flow (#9)**: Post-publication concern (low priority for initial decision)

## Generation Plan

- All 9 diagrams will be generated (cycle-diagram runs ALL priorities)
- Will use Python script following master DrawIO standards
- Light background + dark text (per feedback memory on contrast)
- Straight or orthogonal arrows with legends
- 10px grid, inside-out container sizing

## Notes

- No ERD needed (no data model in this topic)
- No infrastructure diagram needed (no deployed infra — it's a documentation topic)
- Project structure diagram not needed (this IS the documentation project)
- Sequence diagrams absorbed into workflow diagrams (simpler for this audience)
