# Project Agent Rules — Conservation Areas Souvenirs POS

These rules are mandatory for all AI agents working in this repository. Follow them strictly so work stays predictable, safe, and aligned with the project architecture.

---

## 1. Anti-Rogue & Scope Guardrails

1. **Stay strictly within scope:** Only modify files and implement features requested by the user. Do **not** perform unsolicited refactors, rename unrelated files, or add unrequested features.
2. **Ask before big decisions:** Always ask the user before:
   - Changing the core stack or adding major packages/services.
   - Making destructive changes (dropping tables, wiping Docker volumes, resetting migrations, deleting files).
   - Changing financial calculation rules, tax behavior, or offline sync contracts.
3. **No silent workarounds:** If a test fails, a tool breaks, or a requirement is ambiguous, stop and report it clearly instead of faking data, skipping tests, or guessing silently.
4. **Keep docs in sync:** Whenever an architectural decision, feature status, stack version, or money/sync rule changes, update the corresponding file in `docs/` (`ARCHITECTURE.md`, `FEATURES.md`, `MONEY_AND_SYNC_SPEC.md`, `STACK_DOCS.md`).
5. **Verify before claiming done:** Run relevant tests/linters/builds after code changes and report the actual results.
6. **Latest Official Documentation Only:** Before installing packages, configuring frameworks, or using version-sensitive APIs, ALWAYS search and verify against the **latest official documentation** (never rely on outdated memory or deprecated patterns). Keep `docs/STACK_DOCS.md` updated when adopting new versions or packages.

---

## 2. Hard Technical Constraints

### Stack Lock
- **Backend:** Laravel (REST API + Laravel Reverb + Filament for back-office).
- **Frontend:** Vue 3 SPA with Vite, Pinia, and `vite-plugin-pwa` (plus Capacitor if native mobile APIs are added). **Do NOT introduce Nuxt SSR.**
- **Containerization:** Docker & Docker Compose.

### Money & Math (Zero-Drift Policy)
- **NEVER use floats (`float`, `double`, floating-point JS math) for money.**
- **ALWAYS store, transmit, and calculate money in integer minor units (cents/sen)** (`BIGINT` in DB, `int` in PHP/JS).
- **Divide by `100` ONLY at the display/formatting layer** (UI and printed/digital receipts).
- Any change to pricing, discount, tax, or refund math MUST be updated in **both** Laravel and Vue and verified against the shared `shared/pricing_cases.json` golden test suite.

### Offline-First & Sync Integrity
- **NEVER use `localStorage`** for POS catalog, shifts, or transaction queues—use **IndexedDB (Dexie.js)**.
- **ALWAYS use client-generated UUIDs** for offline-created sales, payments, and shifts.
- **Sync endpoints MUST be idempotent** (retrying the same payload must never duplicate sales or double-deduct stock).
- **NEVER mutate a synced offline receipt's total.** Store the exact snapshot charged to the customer; if server-side recalculation differs, log an audit discrepancy instead of altering the receipt.
- **ALWAYS track inventory via append-only deltas** (`-qty` / `+qty` stock movements), never absolute stock overwrites from offline devices.

### Security & Auditability
- Sensitive POS actions (voids, refunds, manual discounts, price overrides, "No Sale" drawer opens) MUST require role checks / supervisor PIN override and write to an immutable audit log.
- Never commit secrets, `.env` credentials, or production database dumps.

### File & Folder Organization
- Keep the repository root clean—only root config files (`AGENTS.md`, `README.md`, `docker-compose.yml`, `.gitignore`, etc.) belong at the top level.
- Group files strictly by purpose:
  - `backend/` — Laravel 13 API, Reverb, and Filament back-office.
  - `frontend/` — Vue 3 SPA (Vite, Pinia, Dexie.js, PWA/Capacitor).
  - `docker/` — Dockerfiles and service configs (`nginx/`, `php/`, `postgres/`, etc.).
  - `docs/` — All project documentation and specs.
  - `shared/` — Cross-stack shared assets such as `pricing_cases.json` golden tests.
- Inside `backend/` and `frontend/`, follow clean feature/domain grouping (e.g., `pos/`, `inventory/`, `shifts/`, `sync/`), consistent naming conventions, and never leave stray temp, scratch, or duplicate files in the repo.
