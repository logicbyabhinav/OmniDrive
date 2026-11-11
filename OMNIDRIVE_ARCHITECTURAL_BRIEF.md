# OmniDrive: Architectural Brief & Project Kickoff Log

> **Context**: Documented summary of the systems-engineering kickoff for **OmniDrive**. Ready to copy-paste directly into your dedicated OmniDrive chat session.

---

## 1. Executive Summary & Vision

Google provides 15 GB of free cloud storage per account. Most power users, students, and engineers maintain 3 to 4 distinct Google accounts (Personal, Academic/College, Developer, Archive), yielding **45 GB to 60+ GB of free storage**. However, using this storage is currently crippled by account switching friction, quota blindness, and lack of cross-account search.

**OmniDrive** solves this by unifying multiple Google Drive accounts into a single virtual pooled workspace (e.g. *38 GB used of 45 GB*).

---

## 2. The Core Technical Paradigm: Zero-Backend & Headless Engine

Unlike traditional storage aggregators that route files through an intermediate proxy server (which incurs costly hosting bills, bandwidth bottlenecks, and severe privacy liabilities), OmniDrive uses a **Zero-Backend / Headless Architecture**:

```
┌────────────────────────────────────────────────────────┐
│                   UI Presentation Layer                │
│       • Clean visual dashboard, Stitch / HTML Shell    │
│       • Drag-and-drop file uploader, storage meters    │
└───────────────────────────┬────────────────────────────┘
                            │ Method calls: engine.uploadFile(file)
                            ▼
┌────────────────────────────────────────────────────────┐
│             OmniDrive Core Engine (TypeScript)         │
│   • AccountManager: Multi-tenant OAuth 2.0 lifecycle   │
│   • DriveClient   : Direct Google Drive REST API v3    │
│   • CryptoStream  : Client-side AES-256-GCM encryption │
│   • StoragePool   : Scatter-gather quota & search      │
└───────────────────────────┬────────────────────────────┘
                            │ Direct CORS fetch()
                            ▼
         ┌──────────────────┼──────────────────┐
         ▼                  ▼                  ▼
  [ Google Drive 1 ] [ Google Drive 2 ] [ Google Drive 3 ]
```

### Why a "Headless Core Engine" (No React)?
* **Separation of Concerns**: The engine contains 100% of the logic, state, networking, and cryptography with zero UI framework dependencies.
* **Mastery Over JavaScript/TypeScript**: Eliminates the cognitive overload of React component lifecycles (`useState`, `useEffect`, stale closures) so the developer focuses purely on browser APIs, network protocols, and cryptographic algorithms.
* **Portability**: The exact same TypeScript engine can run in a browser tab, be wrapped in an Android shell via Capacitor, or be driven by a CLI.

---

## 3. The 4 Deep Systems Challenges

Our architectural discussion identified four non-trivial engineering problems that elevate OmniDrive beyond a simple frontend dashboard:

### Challenge 1: Multi-Tenant OAuth 2.0 Without a Backend
* **Problem**: Google Identity Services (GIS) assumes a user authenticates one account. OmniDrive must maintain simultaneous active access tokens for Account A, Account B, and Account C.
* **Solution**: Client-side Token Vault utilizing Google's Token Client (Implicit/PKCE flow), securely persisting scoped tokens in browser storage (`IndexedDB` / encrypted local storage) and orchestrating per-account refresh handshakes.

### Challenge 2: Browser Memory Protection via the Streams API
* **Problem**: Loading a 500 MB video into JavaScript memory (`response.blob()`) will instantly crash browser tabs and mobile devices with `OutOfMemoryError`.
* **Solution**: In-memory byte streaming using the browser's native **Streams API** (`ReadableStream` and `WritableStream`) coupled with Google Drive's **Resumable Upload Protocol** (`Content-Range: bytes 0-5242879/20971520`). Files are piped directly in fixed-size chunks (e.g. 5 MB) with constant memory footprint ($O(1)$ RAM).

### Challenge 3: Scatter-Gather Parallel Networking
* **Problem**: Searching for a file across 3 independent drives sequentially takes $3 \times \text{latency}$.
* **Solution**: Concurrent asynchronous querying using `Promise.allSettled`. The engine queries all linked drives in parallel, dynamically coalesces results, deduplicates cross-drive entries, and presents a single virtual folder hierarchy.

### Challenge 4: Client-Side Zero-Knowledge Encryption (Web Crypto API)
* **Problem**: Sensitive personal or academic documents stored across cloud accounts should never be readable by cloud providers.
* **Solution**: Before any chunk leaves the browser, it is encrypted in-place using the browser's hardware-accelerated **Web Crypto API** (`crypto.subtle`) with **AES-256-GCM** and a cryptographically random 96-bit Initialization Vector (IV) per chunk. Keys remain strictly on the client device.

---

## 4. Technology Stack Specification

| Component | Selected Technology | Architectural Rationale |
| :--- | :--- | :--- |
| **Language** | **TypeScript (Strict Mode)** | Compile-time type safety; natural progression from Java 21 OOP; eliminates runtime typos when dealing with complex Google API schemas. |
| **Bundler & Server** | **Vite (Vanilla-TS)** | Instant sub-millisecond hot-reloading; zero framework bloat; builds a static bundle that can be hosted for free on Cloudflare Pages or GitHub Pages. |
| **Cloud Storage** | **Google Drive REST API v3** | Official CORS-enabled REST endpoints for file metadata, multi-part uploads, and chunked streaming. |
| **Authentication** | **Google Identity Services (GIS)** | Modern, lightweight OAuth 2.0 token client for single-page applications without server secrets. |
| **Cryptography** | **Web Crypto API (`SubtleCrypto`)** | Hardware-accelerated browser-native AES-256-GCM encryption. Zero third-party crypto libraries required. |
| **Local Storage** | **IndexedDB** | Asynchronous, high-capacity client-side database for directory metadata caching and token storage. |

---

## 5. Phased Implementation Roadmap

* **Phase 1: Project Scaffolding & Google Cloud Setup**
  * Scaffold Vite Vanilla-TS in `d:\Dev\OmniDrive`.
  * Configure Google Cloud Console project, OAuth 2.0 Client ID, and authorized JavaScript origins (`http://localhost:5173`).
* **Phase 2: Multi-Account Token Vault (`AccountManager`)**
  * Implement GIS token client integration.
  * Build multi-account authorization flow (Add Account A, Add Account B).
  * Secure token persistence and expiry detection.
* **Phase 3: Aggregated Virtual Storage Dashboard (`StoragePool`)**
  * Fetch `drive.about.get` quota numbers for all linked accounts.
  * Aggregate total pooled capacity and compute available workspace bytes.
* **Phase 4: Multi-Drive File Explorer & Scatter-Gather Search**
  * Parallel file listing across all connected drives via `Promise.allSettled`.
  * Unified virtual directory tree rendering.
* **Phase 5: Resumable Uploads & In-Memory Stream Piping**
  * Slice files into 5 MB chunks via `Blob.slice()`.
  * Upload directly to Google Drive endpoints with progress tracking.
* **Phase 6: Web Crypto AES-256-GCM Layer**
  * Generate client-side keys via `crypto.subtle.generateKey`.
  * Encrypt/decrypt chunks transparently during upload/download.

---

## 6. Developer Guidelines
* **Zero Vibe-Coding**: Every class, interface, and network call must be understood from first principles.
* **Document Every Architecture Decision**: Maintain authoritative Architectural Decision Records (`ENGINEERING_DECISIONS.md`).
