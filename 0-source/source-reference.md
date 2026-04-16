# Source References

## Primary source materials

### Internal (already in this project)
- `../README.md` — Existing Mermaid overview (6 diagrams covering big picture, current state, MSIX path, EXE path, comparison, decision tree)

### External (AutoClickKey repo)
- `D:\Programing\_app\app-auto-key-click-x-claude\CLAUDE.md` — Project build/test/CI instructions
- `D:\Programing\_app\app-auto-key-click-x-claude\README.md` — App description and existing release docs
- `D:\Programing\_app\app-auto-key-click-x-claude\.github\workflows\ci.yml` — Existing 4-job CI pipeline (security, build/test+MSIX, WACK, format)
- `D:\Programing\_app\app-auto-key-click-x-claude\.github\workflows\release.yml` — New EXE release workflow (created during recent session)
- `D:\Programing\_app\app-auto-key-click-x-claude\src\AutoClickKey\Package.appxmanifest` — MSIX manifest (currently has placeholder identity)
- `D:\Programing\_app\app-auto-key-click-x-claude\src\AutoClickKey\AutoClickKey.csproj` — Project file with MSIX packaging conditional

## Key facts extracted

### Microsoft Store Submission Paths
- **EXE/MSI path**: URL-based; developer hosts signed .exe publicly; MS validates but doesn't re-sign
- **MSIX path**: Upload-based; developer uploads .msix directly; MS re-signs with Store cert

### Certificate reality (2026)
- Standard OV code signing certs: $100–400/year, hardware USB token required (non-exportable)
- EV certs: $300–700/year, USB token, instant SmartScreen trust
- Azure Trusted Signing: ~$10/month managed service (EV-equivalent)
- Self-signed: free but MS Store rejects for public distribution

### AutoClickKey Current State
- Has working MSIX build pipeline in CI
- `release.yml` exists but needs cert to run
- `Package.appxmanifest` has placeholder `Publisher="CN=AutoClickKey"` — not Store-compliant
- Partner Center reservation currently EXE-type (per user's "Package URL" form)

### Decision drivers
- If distributing ONLY to Store → MSIX is simpler, free (no cert needed)
- If distributing both Store + direct download → EXE path needed
- MSIX benefits: sandboxed, auto-update via Store, Microsoft signs
- EXE benefits: traditional installer experience, works outside Store
