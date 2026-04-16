# Analysis — Microsoft Store Submission Workflow

## Files

| # | File | Focus |
|---|------|-------|
| 01 | `01-context-actors.md` | Who's involved, system boundaries, trust/identity flow |
| 02 | `02-workflow-msix-path.md` | MSIX submission end-to-end (recommended path) |
| 03 | `03-workflow-exe-path.md` | EXE submission end-to-end (URL-based path) |
| 04 | `04-ci-cd-architecture.md` | GitHub Actions pipeline structure, both workflows |
| 05 | `05-decision-framework.md` | Which path to pick, decision matrix, scoring |
| 06 | `06-current-state-autoclickkey.md` | AutoClickKey's specific situation and gaps |

## Summary

The workflow spans 4 zones: Developer Local → GitHub CI/CD → Microsoft Ecosystem → End User.

Two submission paths exist (MSIX vs EXE), differing primarily in:
- Who signs (MS vs Developer)
- Where artifact lives (Partner Center upload vs developer-hosted URL)
- Cost profile ($0 vs $120-700/year cert)

For AutoClickKey specifically, MSIX is strictly better — no cost, existing CI supports it, and none of its features require Win32 privileges that MSIX restricts.

Current blocker: Partner Center reservation is the wrong type (EXE), and manifest has placeholder identity.
