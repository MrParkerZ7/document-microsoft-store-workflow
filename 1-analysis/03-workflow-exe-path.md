# Analysis: EXE/MSI Submission Path (For Store + Direct Distribution)

## Overview

The EXE path provides a public HTTPS URL to a signed `.exe` or `.msi` installer. Microsoft Partner Center downloads from that URL to distribute. Unlike MSIX, Microsoft does NOT re-sign — developer must provide a signed installer.

## Pre-Submission Setup (One-time)

### Partner Center Setup
1. Developer registers Partner Center account ($19 one-time)
2. Reserves app name as **"EXE or MSI app"** type
3. No package identity assigned (URL-based, not manifest-based)

### Code Signing Certificate — CRITICAL
Must obtain a publicly trusted code signing certificate:

| Option | Cost/Year | Export to .pfx? | Notes |
|--------|-----------|-----------------|-------|
| Azure Trusted Signing | ~$120 | Managed (no .pfx needed) | Recommended, Microsoft-managed |
| SSL.com OV | $129 | HSM token required | Traditional route |
| Sectigo OV | $199 | HSM token required | Alternative CA |
| DigiCert OV | $399 | HSM token required | Enterprise CA |
| SSL.com EV | $399 | USB token only | Instant SmartScreen trust |
| Self-signed | $0 | Exportable | REJECTED by Store |

**Note**: Since 2023, OV certs cannot be issued as exportable .pfx — they ship on hardware tokens. GitHub Actions secret-based signing requires either Azure Trusted Signing or a pre-2023 cert.

### GitHub Secrets Setup
Add to repo settings → Secrets:
- `CERTIFICATE_PFX` — base64-encoded .pfx file
- `CERTIFICATE_PASSWORD` — password to decrypt .pfx

### Hosting Setup
- GitHub Releases (free, versioned URLs) — recommended
- Azure Blob Storage (paid, custom domain)
- AWS S3 (paid, custom domain)
- Own domain + CDN

## Build Pipeline (Automated via GitHub Actions)

### Phase A: Trigger
1. Developer: `git tag v1.0.0 && git push origin v1.0.0`
2. Release workflow triggers on tag pattern `v*.*.*`

### Phase B: Build + Test
3. Checkout + setup .NET
4. `dotnet restore` + `dotnet build -c Release`
5. Run tests (xUnit) with coverage gates

### Phase C: Publish Single-File EXE
6. `dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true -p:Version=<ver>`
7. Output: `AutoClickKey.exe` (self-contained, ~60MB)

### Phase D: Signing
8. Decode base64 `CERTIFICATE_PFX` secret → temp .pfx file
9. Locate `signtool.exe` (Windows SDK)
10. `signtool sign /fd SHA256 /tr <timestamp-url> /td SHA256 /f <.pfx> /p <password> <.exe>`
11. `signtool verify /pa /v <.exe>` — confirm signature valid
12. Delete temp .pfx

### Phase E: Release Publishing
13. Generate SHA256 checksum of signed .exe
14. Create GitHub Release with tag `v1.0.0`
15. Upload .exe + .sha256 as release assets
16. Auto-generate release notes from commits
17. Output stable URL: `https://github.com/{owner}/{repo}/releases/download/v1.0.0/AutoClickKey-1.0.0-win-x64.exe`

## Submission Flow (Manual)

1. **Copy URL** from GitHub Release page
2. **Partner Center** → your app → **Packages** tab
3. Paste URL into **"Package URL"** field
4. Select architecture (x64)
5. Fill in **Store listing**
6. **Submit for certification**

## Microsoft Review Phase

1. **Download**: Partner Center HTTPS GETs the URL
2. **Signature verification**
   - Confirm cert is publicly trusted
   - Validate timestamp
   - Check cert matches publisher name in listing
3. **Malware scan**
4. **Content review** (manual)
5. **Approve** → Store listing goes live

## Update Flow

For each new version:
1. Update code + commit
2. `git tag v1.0.1 && git push origin v1.0.1`
3. Workflow auto-builds signed .exe → new Release
4. **Partner Center** → app → **+ Create new submission**
5. Update **Package URL** to new version's download URL
6. Submit for certification
7. Customers see update notification in Store

## SmartScreen Reputation

- Fresh standard cert → warnings appear on download
- ~3000 installs → reputation established, warnings disappear
- EV cert → instant reputation, no warnings from day 1

## Pros & Cons

**Pros:**
- Traditional installer experience
- Distribute anywhere (Store + direct download + other stores)
- Full Win32 capabilities (no sandbox)
- Works on older Windows without MSIX runtime
- Offline install (self-contained)

**Cons:**
- Requires paid code signing cert ($120-700/year)
- HSM token management complicates CI signing
- SmartScreen warnings until reputation builds (standard cert)
- Customer must explicitly accept downloads
- Larger file size (self-contained ~60MB vs MSIX ~30MB)
