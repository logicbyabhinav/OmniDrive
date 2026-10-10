# OmniDrive: Unified Multi-Account Cloud Storage Platform

> **A modern, mobile-first cloud aggregation system combining multiple Google Drive accounts into a single virtual 45+ GB storage workspace. Engineered as a Progressive Web App (PWA) with native Android packaging.**

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
| **Cross-Drive Transfer** | Download 2 GB file to local phone/laptop disk, then re-upload to Drive B. | **Direct Stream Piping**: Streams directly from Account A to Account B without touching local disk. |
| **Quota Blindness** | No aggregate view of available storage across accounts. | **Virtual Pooled Storage**: Single unified progress bar (e.g. *28 GB used of 45 GB*). |
| **API Latency & Limits** | Google Drive API is rate-limited and takes 500ms+ per folder view. | **Local Metadata Caching**: Folder trees cached in-memory (via MiniCache) for 60fps instant browsing. |

---

## 2. High-Level System Architecture

OmniDrive operates on a three-tier architecture:

`
┌────────────────────────────────────────────────────────────────────────┐
│             Client Layer: PWA & Android Native Shell                   │
│   • React / Next.js mobile-first UI                                    │
│   • Responsive Grid & Virtualized File Explorer                        │
│   • Android Native Shell (via Capacitor / Trusted Web Activity)        │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ HTTPS / WebSocket (JSON & Chunks)
┌───────────────────────────────────▼────────────────────────────────────┐
│      Application Gateway & BFF (Backend-For-Frontend Layer)            │
│   ┌───────────────────────────┐    ┌──────────────────────────────┐    │
│   │ Multi-Account Token Vault │    │ Scatter-Gather Search Engine │    │
│   │ (OAuth 2.0 Refresh Mgr)   │    │ (Parallel API Coordinator)   │    │
│   └─────────────┬─────────────┘    └──────────────┬───────────────┘    │
│                 │                                 │                    │
│   ┌─────────────▼─────────────┐    ┌──────────────▼───────────────┐    │
│   │ Direct Stream Piping      │    │ Smart Upload Auto-Router     │    │
│   │ (Zero-Disk Stream Pipe)   │    │ (Quota-Aware Placement)      │    │
│   └───────────────────────────┘    └──────────────────────────────┘    │
└───────────────────┬────────────────────────────────┬───────────────────┘
                    │                                │
                    ▼                                ▼
       ┌────────────────────────┐       ┌────────────────────────┐
       │   MiniCache Engine     │       │ Google Drive API v3    │
       │   (In-Memory Cache)    │       │ Account 1 (15 GB)      │
       │   Folder Trees, Quotas,│       │ Account 2 (15 GB)      │
       │   Session States       │       │ Account 3 (15 GB)      │
       └────────────────────────┘       └────────────────────────┘
`

---

## 3. Core Architectural Subsystems

### 3.1. Multi-Account OAuth 2.0 Token Vault
Managing multiple Google IDs requires robust multi-tenant authentication:
- **Authorization Flow**: Uses Google Identity Services with offline access to acquire long-lived efresh_tokens.
- **Token Encryption**: All refresh tokens are encrypted at rest using **AES-256-GCM** with keys derived from user credentials or Android Keystore.
- **Automatic Rotation**: A Token Lifecycle Manager checks token expiration and proactively refreshes short-lived ccess_tokens before API calls without interrupting the user.
- **Granular Scopes**: Uses https://www.googleapis.com/auth/drive.file (only files created/opened by the app) for secure mode, or https://www.googleapis.com/auth/drive for full existing drive aggregation.

### 3.2. Parallel Scatter-Gather Search Engine
When a user searches for tax_returns.pdf:
1. The search coordinator receives the query.
2. It simultaneously fires $ asynchronous HTTP requests across all connected Google Drive accounts using Google's q search parameter:
   `http
   GET https://www.googleapis.com/drive/v3/files?q=name contains 'tax_returns.pdf' and trashed = false
   `
3. A scatter-gather aggregator collects responses as they resolve, tags each item with its parent ccountId and ccountEmail, merges results, and streams them to the UI.

### 3.3. Direct Drive-to-Drive Stream Piping (Zero-Local-Disk Transfer)
Moving large files between accounts on mobile devices without exhausting device storage:
- **Traditional flow (Flawed)**: Account A $\rightarrow$ Download to Phone Disk (2 GB) $\rightarrow$ Upload to Account B. (Fails on phones with low storage).
- **OmniDrive Stream Pipeline**:
  `
  Account A API (Read Stream) ──▶ [ OmniDrive Server Pipe ] ──▶ Account B API (Resumable Upload Stream)
  `
  The data is streamed as a sequence of in-memory chunk buffers using HTTP 1.1 / HTTP 2 chunked transfer encoding. **Zero bytes are written to local disk**, achieving line-speed migration.

### 3.4. Local Metadata Caching via MiniCache
Calling Google Drive's REST API on every folder click introduces 400ms–800ms network round-trips and quickly exhausts Google's per-minute quota (20,000 requests/100 seconds).

**The MiniCache Integration**:
- Folder directory structures, file metadata, and storage statistics are cached in **MiniCache** with a 5-minute TTL.
- When navigating folders, MiniCache serves listings in $<1\text{ms}$.
- Write events (uploading, renaming, deleting) issue explicit invalidation (DEL user:drive:folder_id) to maintain freshness without serving stale trees.

### 3.5. Smart Quota-Aware Upload Auto-Router
When a user uploads a new file without picking an explicit destination:
1. OmniDrive queries the cached storage statistics across all accounts.
2. It filters out accounts where emaining_space < file.size + safety_margin.
3. It selects the account with the highest available free percentage, load-balancing the storage pools evenly.

### 3.6. Resumable Chunked Upload Engine
Mobile wireless networks are unreliable:
- Uploads larger than 5 MB utilize Google Drive's **Resumable Upload Protocol**.
- Files are sliced into 256 KB – 8 MB chunks.
- If a connection drops halfway through a 1 GB video upload, the client queries Google for the last acknowledged byte offset (Range: bytes 0-chunk_size/*) and resumes seamlessly without restarting from byte 0.

---

## 4. Technology Stack Justification

| Layer | Chosen Technology | Rationale |
| :--- | :--- | :--- |
| **Frontend UI** | **React / Next.js + Vanilla CSS / Tailwind** | Mobile-first responsiveness, component reusability, smooth animations, and PWA manifest support. |
| **Mobile Runtime** | **Capacitor / Android TWA (Trusted Web Activity)** | Wraps the high-performance web client into a native Android .apk / .aab with hardware access and Android file picker integration. |
| **Backend / BFF** | **Node.js (Express / Fastify) or Java** | Non-blocking asynchronous I/O optimized for streaming byte pipes and concurrent HTTP scatter-gather queries. |
| **Cache Layer** | **MiniCache** | Sub-millisecond in-memory cache for folder trees, user sessions, and Google API rate-limit avoidance. |
| **Storage APIs** | **Google Drive REST API v3** | Official, fully featured cloud storage API supporting chunked streaming and search queries. |

---

## 5. Security Architecture & Threat Model

1. **No Third-Party Cloud Data Storage**:
   - OmniDrive never permanently stores user files on an intermediate server. Files either stream directly between Google and the client, or pipe through transient memory buffers.
2. **Token Security at Rest**:
   - Encrypted with AES-256-GCM.
   - On Android devices, secret keys are stored within the hardware-backed **Android Keystore System**.
3. **Transport Security**:
   - 100% of network communication is strictly enforced over TLS 1.3 / HTTPS.

---

## 6. Phased Implementation Roadmap

- [ ] **Phase 1: PRD & UX Design**
  - Figma / interactive wireframes: Combined storage bar, multi-drive file explorer, and account manager.
- [ ] **Phase 2: Authentication & Multi-Account Vault**
  - Google OAuth 2.0 flow with offline refresh token rotation.
- [ ] **Phase 3: Unified Virtual Storage Dashboard**
  - Aggregated quota computation, per-account breakdown bar, and health monitoring.
- [ ] **Phase 4: Multi-Drive File Explorer & MiniCache Integration**
  - Virtual folder tree navigation backed by in-memory metadata caching.
- [ ] **Phase 5: Parallel Scatter-Gather Search**
  - Cross-drive concurrent search engine with debounced real-time UI filtering.
- [ ] **Phase 6: Direct Stream Piping & Resumable Uploads**
  - Drive-to-drive migration pipeline and chunked fault-tolerant upload manager.
- [ ] **Phase 7: Android APK Packaging & Testing**
  - Capacitor / TWA build, Android storage permission integration, and on-device performance validation.
