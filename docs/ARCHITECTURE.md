# Architecture & Stack Decisions

## Core Stack

- **Backend:** Laravel (REST API + **Laravel Reverb** for WebSockets + **Filament** for back-office admin/inventory/reporting)
- **Frontend:** Vue 3 SPA (Vite + Pinia + `vite-plugin-pwa`, optionally wrapped with **Capacitor** for native mobile Bluetooth/camera APIs)
  - *Note:* Use a plain **Vue 3 SPA**, **not** Nuxt SSR, to keep offline Service Worker caching simple and reliable.
- **Offline Storage:** IndexedDB via **Dexie.js** (never `localStorage` for POS catalog or transactions)
- **Database:** PostgreSQL or MySQL (with `BIGINT` for all monetary values in minor units / cents)
- **Infrastructure:** Docker & Docker Compose (consistent dev, cloud VPS, or local Linux mini-PC deployment)

---

## Repository Layout

```text
.
├── AGENTS.md                  # Mandatory AI agent rules & guardrails
├── docker-compose.yml         # Local/production container orchestration
├── docker/                    # Dockerfiles & service configs (php, nginx, etc.)
├── docs/                      # Architecture, features, money/sync spec, stack docs
├── shared/                    # Cross-stack contracts & golden test cases (pricing_cases.json)
├── backend/                   # Laravel 13 API + Reverb + Filament v5
└── frontend/                  # Vue 3 SPA + Pinia + Dexie.js + PWA/Capacitor
```

---

## Key Architectural Patterns

### 1. Offline-First POS & Sync (Outbox Pattern)
- The Vue PWA caches the product catalog, categories, tax rules, and active promotions in **IndexedDB** via Dexie.js.
- When offline (or during a power outage), completed sales, refunds, and shift events are written to a local **Outbox Queue** with client-generated **UUIDv4/UUIDv7** identifiers.
- When connectivity returns, the client pushes queued events to idempotent Laravel sync endpoints in chronological order.
- **Inventory uses deltas, not overwrites:** Offline sales record stock changes (`-2`), never absolute stock totals (`stock = 8`), preventing multi-device sync conflicts.
- **Offline-aware End-of-Day (EOD):** Each device tracks its own shift (`shift_id` UUID). Closing EOD warns managers if any registered device still has unsynced offline transactions.

### 2. Dual-Mode Mobile App (Phone as Scanner & Backup POS)
One Vue codebase serves desktop registers and mobile phones with a 1-tap mode switch:
1. **Scanner Mode (Online / Local Wi-Fi):**
   - Phone camera scans barcodes/QR codes (`html5-qrcode` or native Barcode Detection API / Capacitor ML Kit).
   - Sends scan events to Laravel Reverb WebSockets, which immediately injects the SKU into the paired desktop register's active cart (with audio beep + haptic vibrate feedback).
2. **Standalone POS Mode (Power Outage / Line-Busting):**
   - Operates 100% offline using cached IndexedDB data.
   - Supports cash/QR checkout, digital QR receipts (or portable Bluetooth ESC/POS printing), and local shift tracking until power/network returns.

### 3. Hardware & Printing
- **USB / LAN Thermal Printers (Desktop):** Use ESC/POS via a lightweight local print bridge (e.g., QZ Tray) or a Tauri desktop wrapper for silent printing and cash-drawer kick without browser print dialogs.
- **Bluetooth Thermal Printers (Mobile):** Use Capacitor Bluetooth plugins for outage-mode receipt printing.
- **Paperless Fallback:** Display a QR code on-screen containing/linking to the receipt so checkout never blocks if printers lose power or run out of paper.

---

## Stack Alternatives Considered (For Reference)

| Alternative | When It Would Make Sense | Why We Chose Laravel + Vue + Docker Instead |
| :--- | :--- | :--- |
| **Flutter** (frontend) | Heavy daily mobile POS use needing native SQLite (zero browser cache eviction) and native Bluetooth/ML Kit. | Vue 3 + PWA/Capacitor lets us keep one web/mobile codebase with faster iteration. |
| **Node/TypeScript** (backend) | Sharing exact TypeScript math/types between client and server. | Laravel gives us Reverb, Sanctum, queues, and Filament out of the box; we prevent math drift via integer cents + a shared JSON Golden Test Suite. |
| **PocketBase** (Go + SQLite) | Ultra-low-RAM single-shop Raspberry Pi server. | Laravel + Docker scales cleanly from local mini-PC to cloud multi-site deployments. |
