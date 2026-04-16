# Requirement: Microsoft Store Submission Workflow Documentation

## Goal

Create a comprehensive visual documentation package that explains how to publish a .NET WPF desktop application to the Microsoft Store. The audience is a developer who is mid-submission and confused about the two possible paths.

## Context

Reference app: **AutoClickKey** (`D:\Programing\_app\app-auto-key-click-x-claude`)
- .NET 8.0 WPF desktop app
- Modular monorepo (Core library + WPF app)
- Existing CI/CD that builds MSIX + WACK validation
- Facing decision: EXE/MSI path vs MSIX path

## Scope

Document the END-TO-END workflow:
1. Source code → Build pipeline → Signing → Release → Partner Center → Store publication
2. Both submission paths (EXE/MSI URL-based vs MSIX upload-based)
3. Decision criteria between paths
4. Roles of each actor (Developer, GitHub Actions, Partner Center, Microsoft Review, End User)
5. Current state vs target state for AutoClickKey

## Key Questions to Answer Visually

1. **Big picture** — What are the two paths and how do they differ at a high level?
2. **Context** — Who are the actors and systems involved? How do they interact?
3. **MSIX path flow** — Step-by-step sequence from code to Store
4. **EXE path flow** — Step-by-step sequence from code to Store
5. **Decision tree** — Which path should I pick, and why?
6. **Current state** — Where is AutoClickKey today in this pipeline?
7. **CI/CD architecture** — How does GitHub Actions fit into both paths?
8. **Cost comparison** — Side-by-side what each path requires

## Deliverables

- Architecture/context diagrams showing system boundaries
- Workflow/sequence diagrams showing temporal flow
- Decision tree diagrams for path selection
- Component diagrams for CI/CD pipeline
- Current-state vs target-state snapshots

## Constraints

- Must be understandable to a developer who knows .NET but is new to Microsoft Store
- Must map to real tools the dev will use (Partner Center, GitHub Actions, signtool, etc.)
- Must highlight the decision points and tradeoffs clearly

## Success Criteria

- Developer can look at the diagrams and immediately understand:
  1. Which path applies to them
  2. What they need to do next
  3. What artifacts/secrets/accounts they need
  4. Where Microsoft takes over vs where they're responsible
