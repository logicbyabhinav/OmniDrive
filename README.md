# OmniDrive: Client-Side Encrypted Storage Aggregator

<p align="center">
  <img src="https://img.shields.io/badge/Architecture-Zero--Backend%20Headless%20Engine-0969da?style=flat-square&logo=blueprint&logoColor=white" alt="Zero-Backend Architecture" />
  &nbsp;
  <img src="https://img.shields.io/badge/Language-TypeScript%20Strict-3178c6?style=flat-square&logo=typescript&logoColor=white" alt="TypeScript Strict" />
  &nbsp;
  <img src="https://img.shields.io/badge/Security-Client--Side%20AES--256--GCM-2ea44f?style=flat-square&logo=security&logoColor=white" alt="AES-256-GCM" />
  &nbsp;
  <img src="https://img.shields.io/badge/Storage-Google%20Drive%20REST%20v3-e53935?style=flat-square&logo=googledrive&logoColor=white" alt="Google Drive API" />
  &nbsp;
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-gray?style=flat-square" alt="MIT License" /></a>
</p>

> A zero-backend, client-first cloud storage aggregation engine built in TypeScript. Unifies the free storage quotas of multiple Google Drive accounts (3 $\times$ 15 GB = 45+ GB) into a single virtual pooled workspace with client-side AES-256-GCM chunked encryption, parallel scatter-gather search, and constant-memory stream piping.

---

## 1. Executive Summary & Problem Statement

Google provides 15 GB of free cloud storage per account. Most power users, students, and engineers maintain between 2 and 4 distinct Google accounts (Personal, Academic/College, Developer, Archive), yielding **45 GB to 60+ GB of free cloud storage**. 

However, using this storage today is fragmented and painful:

| Friction Point | Current User Experience | OmniDrive Solution |
| :--- | :--- | :--- |
| **Account Switching** | Constant logout/login loops and incognito windows. | **Unified Workspace**: All accounts authenticated and managed simultaneously. |
| **Quota Blindness** | No aggregate visibility into free space across drives. | **Virtual Pooled Storage**: Single unified progress meter (e.g. *38 GB used of 45 GB*). |
| **Cross-Drive Search** | Must remember which account holds a file and search each one. | **Scatter-Gather Parallel Search**: Queries all linked accounts concurrently in parallel. |
| **Cross-Drive Migration** | Download 2 GB file to local disk, then manually re-upload. | **In-Memory Stream Piping**: Pipes bytes directly between Google endpoints in RAM. |
| **Privacy Liability** | Third-party aggregation services store credentials on servers. | **Zero-Backend Architecture**: 100% client-driven directly against Google's cloud APIs. |

---

## 2. High-Level Architectural Topology

OmniDrive is engineered with a **Zero-Backend / Headless Engine Architecture**. All computation, cryptographic key generation, and network orchestration execute strictly inside the user's browser runtime:

```
┌────────────────────────────────────────────────────────────────────────┐
│                        UI Presentation Layer                           │
│     • Clean visual dashboard, drag-and-drop uploader, storage meters   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ Method invocation
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   OmniDrive Core Engine (TypeScript)                   │
│   • AccountManager : Multi-tenant OAuth 2.0 lifecycle & token vault    │
│   • DriveClient    : Direct Google Drive REST API v3 integration       │
│   • CryptoStream   : Client-side AES-256-GCM chunked encryption        │
│   • StoragePool    : Scatter-gather parallel queries & quota balancer  │
│   • MetadataCache  : IndexedDB directory tree persistence              │
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
```

### Architectural Advantages:
1. **Zero Infrastructure Costs**: Can be deployed statically for free on GitHub Pages, Cloudflare Pages, or bundled into a native mobile app via Capacitor.
2. **Absolute Privacy**: User files, OAuth tokens, and encryption keys **never touch an intermediate server**. All data moves exclusively between the client device and Google's endpoints.
3. **Infinite Client Scalability**: The application never bottlenecks on a central server because network requests scatter-gather directly from client hardware.

---

## 3. Core Subsystems

### 3.1. Multi-Tenant OAuth 2.0 Token Vault (`AccountManager`)
* **Identity Provider**: Google Identity Services (GIS) token client.
* **Multi-Tenant State**: Maintains independent, concurrent OAuth access tokens for Account A, Account B, and Account C.
* **Token Persistence**: Scoped access tokens and account metadata are securely persisted in browser `IndexedDB`.

### 3.2. Scatter-Gather Parallel Query Engine (`StoragePool`)
* When querying files or searching for documents:
  1. The engine fires concurrent asynchronous `fetch()` requests across all linked accounts using `Promise.allSettled()`.
  2. Results are dynamically coalesced, sorted, and presented as a single unified virtual filesystem tree.
  3. Eliminates account boundaries from the user experience.

### 3.3. Constant-Memory Stream Piping (`CryptoStream`)
* **The Memory Problem**: Loading a 500 MB video into JavaScript memory (`response.blob()`) crashes mobile devices and browser tabs with `OutOfMemoryError`.
* **The Solution**: Files are sliced into fixed-size chunks (e.g., 5 MB) via `Blob.slice()` and piped using the browser's native **Streams API** (`ReadableStream`) directly into Google Drive's **Resumable Upload Protocol** (`Content-Range: bytes 0-5242879/20971520`). Memory consumption remains strictly constant ($O(1)$ RAM).

### 3.4. Client-Side Zero-Knowledge Encryption (`Web Crypto API`)
* Before any chunk leaves the browser, it is encrypted in memory using the browser's hardware-accelerated **Web Crypto API** (`crypto.subtle`) with **AES-256-GCM**.
* Each chunk receives a cryptographically random 96-bit Initialization Vector (IV).
* Encryption keys remain exclusively on the user's local device; Google stores only ciphertext.

---

## 4. Technology Stack Justification

| Layer | Technology | Rationale |
| :--- | :--- | :--- |
| **Language** | **TypeScript (Strict Mode)** | Compile-time type safety; clean data contracts for complex Google Drive API responses; seamless progression from Java OOP. |
| **Build & Dev Tool** | **Vite (Vanilla-TS)** | Sub-millisecond hot-reloading; zero framework overhead; produces a clean static bundle with zero build bloat. |
| **Cloud Storage** | **Google Drive REST API v3** | Official, CORS-enabled cloud endpoints for multi-part uploads, chunked streaming, and metadata queries. |
| **Authentication** | **Google Identity Services (GIS)** | Lightweight OAuth 2.0 token client for Single Page Applications without backend secrets. |
| **Cryptography** | **Web Crypto API (`SubtleCrypto`)** | Browser-native, hardware-accelerated AES-256-GCM. Zero third-party cryptographic dependencies. |
| **Local State** | **IndexedDB** | High-capacity, transactional asynchronous browser database for token vaulting and directory tree caching. |

---

## 5. Phased Implementation Roadmap

- [ ] **Phase 1: Project Scaffolding & Google Cloud Setup**
  - Initialize Vite TypeScript project (`npm create vite@latest . -- --template vanilla-ts`).
  - Configure Google Cloud OAuth 2.0 Client ID and authorized JavaScript origins (`http://localhost:5173`).
- [ ] **Phase 2: Multi-Account Token Vault (`AccountManager`)**
  - Implement GIS token client authorization flows.
  - Manage concurrent access tokens for multiple Google accounts.
  - Handle token expiration and silent re-authorization.
- [ ] **Phase 3: Virtual Pooled Storage Dashboard (`StoragePool`)**
  - Query `drive.about.get` across all linked accounts.
  - Calculate total pooled storage, consumed bytes, and available space.
- [ ] **Phase 4: Scatter-Gather File Explorer**
  - Parallel cross-drive file search and virtual directory navigation.
  - Local metadata caching in `IndexedDB` for instant 60fps rendering.
- [ ] **Phase 5: Resumable Uploads & In-Memory Stream Piping**
  - Chunked upload pipeline via Google Drive Resumable Upload Protocol.
  - Constant-memory stream piping for cross-account transfers.
- [ ] **Phase 6: Web Crypto AES-256-GCM Layer**
  - Client-side master key derivation and in-place chunk encryption.

---

## 6. Quickstart & Local Development

### Prerequisites
* **Node.js**: v20+ or v22+
* **Package Manager**: `npm`

### Installation
```bash
# Clone the repository
git clone https://github.com/logicbyabhinav/OmniDrive.git
cd OmniDrive

# Install dependencies
npm install

# Start local development server
npm run dev
```

Navigate to `http://localhost:5173` to view the local application.

---

## License

Released under the **[MIT License](LICENSE)**. Copyright (c) 2026 Abhinav Kishore.
