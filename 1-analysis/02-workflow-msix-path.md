# Analysis: MSIX Submission Path (Recommended for Store-only)

## Overview

The MSIX path uploads a `.msix` (or `.msixupload`) package directly to Microsoft Partner Center. Microsoft handles hosting, re-signing, and distribution. No code signing certificate needed from the developer.

## Pre-Submission Setup (One-time)

### Partner Center Setup
1. Developer registers Microsoft Partner Center account ($19 one-time)
2. Developer reserves app name as **"MSIX or PWA app"** type
3. Partner Center assigns unique identity values:
   - Package/Identity Name
   - Publisher (CN=<GUID>)
   - Publisher Display Name

### Manifest Configuration
Update `Package.appxmanifest`:
```xml
<Identity
  Name="<Store-assigned-name>"
  Publisher="CN=<Store-assigned-CN>"
  Version="1.0.0.0" />
<Properties>
  <PublisherDisplayName><Your-seller-name></PublisherDisplayName>
</Properties>
```

### Project Configuration
`.csproj` must have:
- `WindowsPackageType=MSIX` (during publish)
- Proper asset images (Square44x44Logo, Square150x150Logo, StoreLogo)
- `TargetDeviceFamily` = Windows.Desktop

## Build Pipeline (Automated via GitHub Actions)

### Phase A: Source → Build
1. Developer pushes code (or tags release)
2. GitHub Actions checkout + setup .NET 8
3. `dotnet restore` → `dotnet build -c Release`

### Phase B: Test + Quality Gates
4. Run all tests (xUnit) with 100% coverage enforcement
5. StyleCop analyzers
6. Semgrep security scan (SAST)
7. `dotnet format --verify-no-changes`

### Phase C: Package
8. `dotnet publish -c Release -r win-x64 -p:WindowsPackageType=MSIX -p:GenerateAppxPackageOnBuild=true`
9. Output: `.msix` file
10. (Optional) Sign with dev cert for local testing — NOT required for Store

### Phase D: WACK Validation
11. Run Windows App Certification Kit against .msix
12. Upload WACK report as artifact

### Phase E: Artifact Storage
13. Upload .msix to GitHub Actions artifacts (retention 30-90 days)

## Submission Flow (Manual)

1. **Download** .msix from GitHub Actions artifacts
2. **Partner Center** → your app → **Packages** tab
3. **Drag & drop** or browse to upload .msix
4. Partner Center auto-validates:
   - Identity matches reservation
   - Manifest conforms to schema
   - Assets present
   - WACK results (re-run server-side)
5. Fill in **Store listing** (description, screenshots, pricing)
6. **Submit for certification**

## Microsoft Review Phase

1. **Pre-processing** (automated, minutes)
   - Signature strip (if dev-signed)
   - Re-sign with Microsoft Store cert
   - Package integrity verification
2. **Certification** (automated + manual, hours to days)
   - Content review (policy compliance)
   - Malware scan
   - Binary analysis
3. **Publishing**
   - Push to Microsoft Store CDN
   - Update Store listing page
   - Notify developer

## Update Flow

For each new version:
1. Bump `Version` in `Package.appxmanifest`
2. Build new .msix via CI
3. **Partner Center** → app → **+ Create new submission**
4. Upload new .msix
5. Submit for certification
6. Existing customers auto-update via Store

## Pros & Cons

**Pros:**
- No cost for code signing
- Microsoft handles signature management
- Automatic updates for users
- Sandboxed execution (more secure)
- Clean SmartScreen (no warnings)

**Cons:**
- Cannot distribute outside Microsoft Store
- Manifest identity locked to Store
- Some Win32 features restricted (sandbox)
- MSIX runtime required on older systems
