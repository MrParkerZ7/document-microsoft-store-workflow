# Analysis: Decision Framework — Which Path to Pick

## Primary Decision Axis

The fundamental question: **Will you distribute OUTSIDE the Microsoft Store?**

- **No, only Store** → MSIX path
- **Yes, Store + direct downloads** → EXE path (or dual: MSIX to Store + EXE direct)

## Secondary Decision Axes

### Budget Axis
- **$0 available** → MSIX path (no cert needed)
- **$120+/year available** → Either path viable

### Timeline Axis
- **Ship this week** → MSIX path (no cert procurement delay)
- **Willing to wait 2-4 weeks** → Either path viable (cert ordering + validation takes time)

### Complexity Tolerance Axis
- **Keep it simple** → MSIX path
- **Willing to manage certs + HSM tokens + reputation** → EXE path

### User Base Axis
- **Consumer apps (SmartScreen-sensitive)** → MSIX or EV cert
- **Enterprise apps (controlled environments)** → Either path

### Feature Requirements Axis
- **Need full Win32 capabilities** → EXE path (sandbox constraints in MSIX)
- **Standard desktop app** → MSIX path fine

## Decision Matrix

| Scenario | Recommended Path |
|----------|-----------------|
| Indie dev, first Store app, Store-only | MSIX |
| Indie dev, existing direct distribution | EXE |
| Enterprise app for specific customers | EXE |
| Freeware consumer app, Store-only | MSIX |
| Commercial app, Store + website | Dual (MSIX + EXE) |
| App requires kernel-mode access | EXE (MSIX may restrict) |
| App uses unrestricted filesystem | EXE or MSIX with capabilities |
| Startup tool for rapid iteration | MSIX (simpler updates) |

## AutoClickKey-Specific Recommendation

### Given:
- WPF desktop app with standard features
- Mouse/keyboard automation (uses user32.dll via P/Invoke)
- No existing direct distribution channel
- Single developer, cost-sensitive
- MSIX pipeline already built in CI

### Risk Assessment for MSIX:
- Mouse/keyboard P/Invoke: works in MSIX with `runFullTrust` capability (already declared in manifest)
- Global hotkeys: work in MSIX full-trust apps
- File system access to `%APPDATA%`: works normally
- Win32 API usage: fine with `packagedClassicApp` runtime behavior

### Conclusion:
**MSIX path is strictly better for AutoClickKey.**
- Zero additional cost
- Existing CI produces the artifact
- No SmartScreen warnings
- Auto-updates for users
- Only work needed: update manifest identity from Partner Center values

## Decision Process Flow

```
Q1: Only Store distribution?
├── Yes → Q2: MSIX compatible features?
│         ├── Yes → MSIX PATH ✓
│         └── No  → EXE PATH (necessity)
└── No  → Q3: Budget for cert?
          ├── Yes → EXE PATH or DUAL
          └── No  → Reconsider: drop direct or get MSIX-only
```

## Switch Costs

### EXE → MSIX
- Discard Partner Center EXE reservation (3-day name hold if deleted)
- Create new MSIX reservation
- Update Package.appxmanifest identity
- Test MSIX build locally
- Upload MSIX to Partner Center
- **Total effort**: ~1-2 hours

### MSIX → EXE
- Reserve EXE app in Partner Center
- Procure cert (2-4 weeks for OV, 1-4 weeks for EV)
- Set up signing in CI
- Build signed EXE
- Submit URL
- **Total effort**: ~3-5 days wall-clock (most waiting on cert)

## Recommendation Weights

For AutoClickKey, weighing all factors:

| Factor | Weight | MSIX Score | EXE Score |
|--------|--------|------------|-----------|
| Cost | 30% | 10/10 | 3/10 |
| Setup time | 20% | 9/10 | 2/10 |
| SmartScreen trust | 15% | 10/10 | 4/10 |
| Update UX | 15% | 9/10 | 8/10 |
| Feature fit | 10% | 9/10 | 10/10 |
| Future flexibility | 10% | 6/10 | 9/10 |

**MSIX weighted score: 9.0/10**
**EXE weighted score: 5.2/10**

**Recommendation: MSIX path.**
