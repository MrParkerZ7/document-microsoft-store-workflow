# Analysis: Context & Actors

## System Boundaries

The end-to-end Microsoft Store publishing workflow spans FOUR distinct zones:

### Zone 1: Developer Local Environment
- **Developer** (human) — writes code, manages secrets, tags releases
- **Local workstation** — runs Visual Studio, dotnet CLI, git
- **Code signing certificate** (if EXE path) — stored as .pfx or on hardware token

### Zone 2: Source Control & CI/CD
- **GitHub Repository** — stores source code, workflows, secrets
- **GitHub Actions** — builds, tests, packages, signs (CI and Release workflows)
- **GitHub Releases** — hosts signed .exe artifacts (EXE path only)
- **GitHub Secrets** — stores CERTIFICATE_PFX, CERTIFICATE_PASSWORD

### Zone 3: Microsoft Ecosystem
- **Microsoft Partner Center** — dev portal for app submission/management
- **Microsoft Review System** — certification and validation pipeline
- **Windows App Certification Kit (WACK)** — automated test suite for MSIX
- **Microsoft Store CDN** — distribution infrastructure
- **Microsoft Code Signing Service** — re-signs MSIX with Store identity

### Zone 4: End User
- **Customer device** — Windows 10/11 x64
- **Microsoft Store app** — discovery and installation UI
- **Windows Installer / MSIX runtime** — on-device install execution
- **SmartScreen** — reputation-based safety filter

## Actor Responsibilities per Path

| Actor | MSIX Path | EXE Path |
|-------|-----------|----------|
| Developer | Writes manifest, builds MSIX, uploads | Buys cert, signs .exe, hosts URL |
| GitHub Actions | Builds + WACK-validates .msix | Builds + signs + publishes release |
| Partner Center | Accepts .msix upload, validates | Accepts URL, downloads, validates |
| Microsoft Review | WACK + content review + re-sign | Content review + signature verify |
| Microsoft Store | Hosts + distributes + auto-updates | Hosts metadata + pulls from URL |
| End User | Install from Store (MSIX) | Install from Store (downloads .exe) |

## Trust & Identity Flow

**MSIX path**: Identity is declarative (Package.appxmanifest) → Partner Center validates identity matches reservation → MS re-signs with Store cert → user trusts because Store publishes it.

**EXE path**: Identity is cryptographic (code signing cert) → cert CA vouches for publisher → SmartScreen reputation builds over downloads → user trusts based on signature + reputation.

## Key Interactions

1. **Developer → GitHub**: `git push` + `git tag`
2. **GitHub Actions → Artifacts**: Build outputs (.msix or .exe)
3. **Developer → Partner Center**: Upload .msix OR paste URL
4. **Partner Center → GitHub Releases** (EXE path): HTTPS GET to download .exe
5. **Partner Center → Microsoft Review**: Queue submission
6. **Microsoft Review → Store**: Approve → publish
7. **Customer → Store**: Browse → install
8. **Store → Customer Device**: Deliver package

## Stakeholders

- **Primary**: Developer (main audience)
- **Secondary**: End customers (downstream beneficiaries)
- **Gatekeeper**: Microsoft Review team (human + automated)
- **Infrastructure**: GitHub (CI/CD) + Microsoft (hosting/review)
