# Analysis: CI/CD Architecture

## Overview

Two GitHub Actions workflows serve the two submission paths:
- **ci.yml** — runs on every push/PR, produces MSIX artifact on main
- **release.yml** — runs on version tag, produces signed EXE in GitHub Release

## ci.yml — Continuous Integration

### Triggers
- `push` to main/master
- `pull_request` to main/master
- `workflow_dispatch` (manual)

### Jobs

**Job 1: security-scan** (ubuntu-latest, Semgrep container)
- Semgrep SAST scan with C# + default rules
- Output → SARIF uploaded to GitHub Security tab

**Job 2: build** (windows-latest, depends on security-scan)
- Setup .NET 8
- Restore + Build Release
- Test Core (business logic, net8.0)
- Test App (WPF helpers, net8.0-windows)
- Upload coverage reports (artifact)
- If push to main:
  - Publish MSIX (`WindowsPackageType=MSIX`)
  - Decode `CERTIFICATE_PFX` secret
  - Sign with signtool (SHA256 + timestamp)
  - Clean up cert file
  - Upload MSIX as artifact

**Job 3: wack-validation** (windows-latest, main push only, depends on build)
- Download MSIX artifact
- Run Windows App Certification Kit
- Upload wack-report.xml

**Job 4: format-check** (windows-latest, parallel)
- `dotnet format --verify-no-changes --verbosity diagnostic`

### Dependencies
```
security-scan ──► build ──► wack-validation
                  │
                  └─── (parallel) format-check
```

## release.yml — Release Publishing (EXE path)

### Triggers
- `push` of tag matching `v*.*.*`
- `workflow_dispatch` with version input

### Single Job: publish-exe (windows-latest)

**Step 1: Setup**
- Checkout
- Setup .NET 8

**Step 2: Version resolution**
- From tag: strip `v` prefix
- From dispatch: use input value

**Step 3: Build + test**
- Restore + Build Release
- Run both test projects

**Step 4: Publish single-file EXE**
- `dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true`
- Include native libraries for self-extract
- Inject version into assembly info

**Step 5: Rename**
- `AutoClickKey.exe` → `AutoClickKey-{version}-win-x64.exe`

**Step 6: Sign**
- Decode `CERTIFICATE_PFX` secret
- Locate signtool in Windows SDK
- Sign with SHA256 + DigiCert timestamp
- Verify signature (`signtool verify /pa`)
- Clean up cert

**Step 7: Checksum**
- SHA256 hash of signed EXE
- Write to `.sha256` file

**Step 8: GitHub Release**
- Create release with tag name
- Upload signed EXE + checksum
- Auto-generate release notes
- Include Microsoft Store URL in description

### Outputs
- GitHub Release with signed .exe + .sha256
- Stable versioned download URL

## Security Model

### Secret Management
- `CERTIFICATE_PFX` — base64 blob, decoded to temp file, always deleted
- `CERTIFICATE_PASSWORD` — passed inline to signtool, never logged
- Both scoped to GitHub Actions only (repo secrets)

### Code Signing Chain
```
Developer → PFX → signtool → Windows-trusted CA chain → User's Windows trusts signature
```

### Isolation
- Each workflow run: fresh runner VM
- Temp files: auto-cleaned after run
- No persistent state between runs

## Monitoring & Observability

- **GitHub Actions logs** — full build output, retained 90 days
- **Coverage reports** — artifact, retained 30 days
- **WACK reports** — artifact, retained 90 days
- **Semgrep SARIF** — GitHub Security tab (permanent)
- **Release assets** — permanent (until manual deletion)

## Failure Modes

| Failure | Cause | Recovery |
|---------|-------|----------|
| Build fails | Code error | Fix code, re-push |
| Test fails | Regression | Fix test or code |
| Coverage < 100% | Untested code | Add tests |
| Sign fails (cert missing) | Secret not set | Add cert to secrets |
| Sign fails (password wrong) | Secret mismatch | Update password secret |
| WACK fails | MSIX compliance issue | Check manifest, assets |
| Release upload fails | Permissions | Check GITHUB_TOKEN scope |
