# OmniDrive

> **Unified Multi-Account Cloud Storage Hub (PWA to Android Native)**

## Overview
OmniDrive is a modern cloud storage aggregation platform that seamlessly pools and manages multiple Google Drive accounts under a single, unified interface. Turn multiple 15 GB free Google accounts into a single, continuous 45+ GB storage workspace.

## Problem Statement
Power users and students frequently maintain 2–4 Google accounts (college, personal, archive, projects). Managing files across these accounts requires constant account switching, duplicate searches, and manual download-and-reupload workflows. 

OmniDrive solves this by aggregating storage quotas, unifying search across all accounts, and enabling direct drive-to-drive transfers.

## Key Features
1. **Unified Multi-Account OAuth 2.0**: Securely link multiple Google accounts and manage refresh tokens.
2. **Aggregated Virtual Storage Bar**: Visual breakdown of total pooled space and per-account usage (e.g., *28 GB used of 45 GB pooled*).
3. **Global Multi-Drive Search**: Search by filename, extension, or date across all connected accounts in parallel.
4. **Direct Drive-to-Drive Migration**: Move files from Account A to Account B via stream piping without downloading to the local device.
5. **Smart Auto-Routing on Upload**: Automatically place uploads into the account with the most available quota.
6. **Local Metadata Caching**: Fast UI navigation by caching folder structures locally (using MiniCache / local storage).

## Target Tech Stack
- **Frontend**: Next.js / React (PWA) with responsive mobile-first UI.
- **Mobile Packaging**: Capacitor / TWA (Trusted Web Activity) for native Android APK distribution.
- **Backend / APIs**: Google Drive REST API v3, OAuth 2.0 multi-tenancy.
- **Cache / Metadata**: MiniCache / in-memory cache for fast folder lookups.

## Project Roadmap
- [ ] Phase 1: Product Requirements Document (PRD) & UI Wireframes.
- [ ] Phase 2: Multi-Account OAuth 2.0 flow & token vault.
- [ ] Phase 3: Aggregated storage & file explorer view.
- [ ] Phase 4: Global cross-drive search.
- [ ] Phase 5: Direct streaming transfer between accounts.
- [ ] Phase 6: PWA to Android APK bundling.
