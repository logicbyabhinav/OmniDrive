# OmniDrive: Unified Multi-Account Cloud Storage Platform

> **A modern, mobile-first cloud aggregation system combining multiple Google Drive accounts into a single virtual 45+ GB storage workspace. Engineered as a Zero-Backend, Client-First Progressive Web App (PWA) with native Android packaging.**

---

## 1. Executive Summary & Problem Statement

Google provides 15 GB of free cloud storage per Google account. Most power users, students, and developers maintain between 2 and 4 Google accounts:
- Primary Personal Account
- College / University Academic Account
- Side Project / Developer Account
- Media / Archive Account

While this yields **45 GB to 60+ GB of free cloud storage**, using it today is fragmented and painful:

| Pain Point | Current Experience | OmniDrive Solution |
| :--- | :--- | :--- |
| **Account Switching** | Must log out, switch accounts, or use private tabs. | **Single Unified Dashboard**: All accounts linked simultaneously. |
| **Global File Search** | Must remember which account holds receipt.pdf and search each one. | **Parallel Cross-Drive Search**: One search query searches all accounts in parallel. |
| **Cross-Drive Transfer** | Download 2 GB file to local phone/laptop disk, then re-upload to Drive B. | **Direct In-Memory Streaming**: Pipes data directly between Google endpoints without intermediate disk storage. |
| **Quota Blindness** | No aggregate view of available storage across accounts. | **Virtual Pooled Storage**: Single unified progress bar (e.g. *28 GB used of 45 GB*). |
| **Infrastructure Costs** | Running backends costs hosting fees and introduces privacy risks. | **Zero-Backend Architecture**: 100% client-driven directly against Google Drive APIs. |

---

## 2. High-Level System Architecture: The Zero-Backend Model

OmniDrive is engineered with a **Zero-Backend / Client-First architecture**. The application runs entirely on the client (browser / PWA / Android shell) and communicates directly with Google's cloud APIs:

`
┌────────────────────────────────────────────────────────────────────────┐
│             Client Layer: PWA & Android Native Shell                   │
│   • React / Next.js mobile-first UI                                    │
│   • Responsive Grid & Virtualized File Explorer                        │
│   • Android Native Shell (via Capacitor / Trusted Web Activity)        │
│   • Local Storage Engine (IndexedDB & Encrypted Token Vault)           │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Direct Google OAuth 2.0 & REST API v3
                                    │ (Zero intermediate servers)
        ┌───────────────────────────┼───────────────────────────┐
        ▼                           ▼                           ▼
┌───────────────┐           ┌───────────────┐           ┌───────────────┐
│ Google Drive  │           │ Google Drive  │           │ Google Drive  │
│   Account 1   │           │   Account 2   │           │   Account 3   │
│   (15 GB)     │           │   (15 GB)     │           │   (15 GB)     │
└───────────────┘           └───────────────┘           └───────────────┘
`

### Architectural Advantages:
1. ** Infrastructure & Hosting Costs**: Can be hosted statically for free on Vercel, Cloudflare Pages, or bundled directly as an Android APK.
2. **Maximum Privacy & Security**: User credentials, files, and OAuth tokens **never touch a third-party server**. All data moves exclusively between the user's device and Google's servers.
3. **Infinite Scalability**: Since the client performs scatter-gather queries directly against Google, the system never bottlenecks on an application server.

---

## 3. Core Architectural Subsystems

### 3.1. Client-Side Multi-Account OAuth 2.0 Vault
- **Identity Provider**: Google Identity Services (GIS) token client.
- **Multi-Tenant Token Management**: Allows users to authorize Account 1, Account 2, and Account 3 sequentially.
- **Encrypted Local Storage**: Access tokens and metadata are stored in client-side IndexedDB encrypted via Web Crypto API (AES-GCM) or Android Keystore on mobile.

### 3.2. Parallel Scatter-Gather Search Engine
When a user searches for tax_returns.pdf:
1. The client fires $ concurrent etch() requests directly to Google's Drive API endpoint:
   `http
   GET https://www.googleapis.com/drive/v3/files?q=name contains 'tax_returns.pdf' and trashed = false
   `
2. Results are merged, deduplicated, and displayed in real-time in the unified UI.

### 3.3. In-Browser Direct Stream Piping
Transferring files between accounts without local disk downloads:
`
Account A (GET media stream) ──▶ [ In-Memory ReadableStream ] ──▶ Account B (POST Resumable Upload)
`
Browser ReadableStream and chunked etch pipe the bytes in memory, enabling direct migration between accounts.

### 3.4. Local Metadata Caching
To ensure instant, 60fps folder navigation and stay well clear of Google API rate limits, directory structures are cached locally in IndexedDB / memory with configurable TTLs.

---

## 4. Technology Stack Justification

| Layer | Technology | Rationale |
| :--- | :--- | :--- |
| **Frontend Framework** | **React / Next.js + Tailwind CSS** | Fast component reusability, PWA support, mobile-first responsive layout. |
| **Mobile Distribution** | **Capacitor / Android TWA** | Zero-friction packaging of the web codebase into a production Android .apk. |
| **API Integration** | **Google Drive REST API v3** | Official, CORS-enabled cloud storage API for file listing, streaming, and uploads. |
| **Local State & Cache**| **IndexedDB + Web Crypto API** | Persistent, client-side metadata caching and hardware-secured token encryption. |

---

## 5. Phased Implementation Roadmap

- [ ] **Phase 1: PRD & UI Wireframes** (Pooled storage bar, account manager, unified explorer).
- [ ] **Phase 2: Client-Side Google Identity Services Integration** (Multi-account token management).
- [ ] **Phase 3: Aggregated Virtual Storage Dashboard** (Visual breakdown of total pooled space).
- [ ] **Phase 4: Multi-Drive File Explorer with Local Caching** (Fast, rate-limit friendly browsing).
- [ ] **Phase 5: Concurrent Scatter-Gather Search** (Parallel queries across all accounts).
- [ ] **Phase 6: In-Memory Stream Piping & Resumable Uploads** (Direct drive-to-drive migration).
- [ ] **Phase 7: Android APK Packaging** (Testing on real hardware via Capacitor).
