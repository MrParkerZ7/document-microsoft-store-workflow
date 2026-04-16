# Microsoft Store Submission Workflow

This doc explains the full workflow for publishing a .NET WPF desktop app (like **AutoClickKey**) to the Microsoft Store.

**Source app reference**: `D:\Programing\_app\app-auto-key-click-x-claude`

---

## 1. Big Picture — Two Possible Paths

```mermaid
flowchart TD
    Start([You want to publish to Microsoft Store]) --> Choice{Which submission<br/>type did you pick<br/>when reserving?}

    Choice -->|EXE or MSI app| ExePath[EXE/MSI Path<br/>URL-based submission]
    Choice -->|MSIX or PWA app| MsixPath[MSIX Path<br/>Upload-based submission]

    ExePath --> ExeNeeds[Requirements:<br/>- Public HTTPS URL<br/>- Signed .exe<br/>- Code signing cert<br/>- GitHub Releases or CDN]

    MsixPath --> MsixNeeds[Requirements:<br/>- Store-assigned identity<br/>- Package.appxmanifest match<br/>- No cert needed from you<br/>- Microsoft re-signs]

    ExeNeeds --> ExeCost[Cost: ~$10-700/year<br/>for code signing cert]
    MsixNeeds --> MsixCost[Cost: $0 extra<br/>just Partner Center $19 one-time]

    style MsixPath fill:#c8e6c9
    style MsixNeeds fill:#c8e6c9
    style MsixCost fill:#c8e6c9
    style ExePath fill:#ffe0b2
    style ExeNeeds fill:#ffe0b2
    style ExeCost fill:#ffe0b2
```

**Recommendation for AutoClickKey**: MSIX path (green). Cheaper, simpler, already mostly built.

---

## 2. Current State — Where You Are Now

```mermaid
flowchart LR
    subgraph Repo[AutoClickKey Repo]
        Code[Source code]
        CI[.github/workflows/ci.yml<br/>Builds MSIX already ✅]
        Release[.github/workflows/release.yml<br/>Builds signed EXE ✅<br/>Not yet used]
        Manifest[Package.appxmanifest<br/>⚠️ Placeholder identity]
    end

    subgraph PartnerCenter[Microsoft Partner Center]
        ExeReserve[EXE/MSI app reservation<br/>❌ Wrong type]
        MsixReserve[MSIX/PWA app reservation<br/>❌ Not yet created]
    end

    Code --> CI
    Code --> Release
    CI -.builds.-> MsixArtifact[.msix artifact]
    Release -.builds.-> ExeArtifact[signed .exe<br/>requires cert]

    ExeArtifact -.blocked by.-> NoCertYet[❌ No cert configured]
    MsixArtifact -.needs.-> RealIdentity[Real identity from<br/>Partner Center]

    style Manifest fill:#ffccbc
    style ExeReserve fill:#ffccbc
    style MsixReserve fill:#fff9c4
    style NoCertYet fill:#ffccbc
```

---

## 3. MSIX Path — Full Workflow (Recommended)

```mermaid
flowchart TD
    Step1[1. Discard current EXE reservation<br/>in Partner Center]
    Step2[2. Reserve new app as<br/>'MSIX or PWA app']
    Step3[3. Copy identity values from<br/>Partner Center → Product identity]
    Step4[4. Update Package.appxmanifest<br/>with real identity]
    Step5[5. Commit + push to trigger CI]
    Step6[6. CI builds .msix artifact]
    Step7[7. Download .msix from<br/>GitHub Actions artifacts]
    Step8[8. Upload .msix to Partner Center<br/>Packages section]
    Step9[9. Submit for review]
    Step10[10. Microsoft re-signs + publishes]

    Step1 --> Step2 --> Step3 --> Step4 --> Step5 --> Step6 --> Step7 --> Step8 --> Step9 --> Step10

    Step3 -.provides.-> IdentityValues[Identity Name<br/>Publisher CN<br/>Publisher Display Name]
    IdentityValues -.plug into.-> Step4

    style Step1 fill:#e3f2fd
    style Step2 fill:#e3f2fd
    style Step3 fill:#fff9c4
    style Step4 fill:#fff9c4
    style Step10 fill:#c8e6c9
```

### MSIX Path — Component View

```mermaid
flowchart LR
    Dev([You]) -->|git push tag| GitHub[GitHub Repo]
    GitHub --> Actions[GitHub Actions CI<br/>ci.yml]
    Actions --> Build[dotnet build]
    Build --> Publish[dotnet publish<br/>WindowsPackageType=MSIX]
    Publish --> Wack[WACK Validation]
    Wack --> Artifact[.msix artifact]
    Artifact --> Dev2([You download])
    Dev2 -->|manual upload| Partner[Partner Center<br/>Packages]
    Partner --> Review[Microsoft Review<br/>Certification]
    Review --> Sign[MS re-signs with<br/>Store cert]
    Sign --> Store[Microsoft Store<br/>Published ✅]

    style Actions fill:#fff9c4
    style Store fill:#c8e6c9
```

---

## 4. EXE Path — Full Workflow (Alternative)

Only use this if you want to distribute OUTSIDE the Store too (e.g., direct download from your website).

```mermaid
flowchart TD
    A[1. Buy code signing cert<br/>$10-700/year]
    B[2. Add to GitHub secrets<br/>CERTIFICATE_PFX + CERTIFICATE_PASSWORD]
    C[3. git tag v1.0.0]
    D[4. git push origin v1.0.0]
    E[5. release.yml triggers]
    F[6. Build single-file .exe]
    G[7. Sign .exe with cert]
    H[8. Publish GitHub Release]
    I[9. Copy release download URL]
    J[10. Paste into Partner Center<br/>Package URL field]
    K[11. Microsoft validates<br/>no re-signing]
    L[12. Published ✅]

    A --> B --> C --> D --> E --> F --> G --> H --> I --> J --> K --> L

    style A fill:#ffccbc
    style L fill:#c8e6c9
```

### EXE Path — Component View

```mermaid
flowchart LR
    Dev([You]) -->|git push tag| GitHub[GitHub Repo]
    GitHub --> Actions[GitHub Actions<br/>release.yml]
    Actions --> Build[dotnet publish<br/>PublishSingleFile=true]
    Build --> Sign[signtool sign<br/>with your cert]
    Sign --> Release[GitHub Release<br/>+ signed .exe]
    Release --> URL[Public HTTPS URL]
    URL -->|paste URL| Partner[Partner Center<br/>Package URL field]
    Partner --> Review[Microsoft Review]
    Review --> Download[Customers download<br/>from your URL]

    style Actions fill:#fff9c4
    style Review fill:#fff9c4
```

---

## 5. Side-by-Side Comparison

| Aspect | MSIX Path ✅ | EXE Path |
|--------|-------------|----------|
| **Cost** | $0 | $10–700/year (cert) |
| **Signing** | Microsoft signs for you | You must sign |
| **Cert required** | No | Yes (trusted CA) |
| **Hosting** | Microsoft hosts | You host (GitHub Releases) |
| **Update flow** | Upload new .msix | Tag → auto-release → update URL |
| **SmartScreen** | Clean | Warnings until reputation builds |
| **WACK validation** | Required (already in CI) | Not required |
| **Auto-updates** | Store handles | Store handles |
| **Customer install** | From Store app | Store downloads your .exe |
| **Your CI** | ✅ Already works | Needs cert in secrets |

---

## 6. Decision Tree — What To Do RIGHT NOW

```mermaid
flowchart TD
    Now([Right now]) --> Q1{Do you need to<br/>distribute OUTSIDE<br/>the Store too?}

    Q1 -->|No, only Store| MsixGo[Go MSIX path]
    Q1 -->|Yes, also direct download| Q2{Willing to pay<br/>for code cert?}

    Q2 -->|Yes| ExeGo[Go EXE path<br/>Set up cert]
    Q2 -->|No| MsixGo

    MsixGo --> NextAction[NEXT:<br/>1. Delete EXE reservation<br/>in Partner Center<br/>2. Reserve new MSIX app<br/>3. Paste identity values back]

    ExeGo --> NextActionB[NEXT:<br/>1. Buy cert from<br/>SSL.com or Sectigo<br/>2. Configure secrets<br/>3. Tag v1.0.0]

    style MsixGo fill:#c8e6c9
    style NextAction fill:#c8e6c9
    style ExeGo fill:#ffe0b2
    style NextActionB fill:#ffe0b2
```

---

## 7. Your Immediate Next Steps (Assuming MSIX Path)

1. **In Partner Center browser tab**:
   - Close/discard the current EXE submission draft
   - Delete the EXE product (or keep it parked and use a new name)
   - Click **+ New product → MSIX or PWA app**
   - Reserve your app name

2. **After reservation, navigate to**:
   - Your new MSIX app → **Product management → Product identity**
   - Copy three values:
     - Package/Identity Name
     - Publisher (starts with `CN=`)
     - Publisher display name

3. **Paste them back to me** — I'll:
   - Update `Package.appxmanifest` in AutoClickKey repo
   - Remove the `release.yml` workflow (no longer needed for MSIX)
   - Push to trigger CI → MSIX artifact
   - Show you how to download + upload the artifact

---

## Glossary

- **MSIX**: Modern Windows app package format. Sandboxed. Auto-updates via Store.
- **WACK**: Windows App Certification Kit. Tests MSIX for Store compliance.
- **Partner Center**: Microsoft's portal where developers submit apps.
- **Package identity**: Unique Publisher+Name combo that identifies your app in the Store.
- **SmartScreen**: Windows safety filter. Unsigned/low-rep EXEs trigger warnings.
- **Code signing cert**: Cryptographic cert used to sign .exe files. Proves publisher identity.
- **EV cert**: Extended Validation cert. Instant SmartScreen trust. More expensive.
