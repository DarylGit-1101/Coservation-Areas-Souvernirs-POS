# POS System Feature Specification

## 1. Core POS Features

### Checkout & Cart
- [ ] Fast barcode/QR scanning, instant search by name/SKU, and touch-friendly color-coded product grid
- [ ] Product variants (size, color, etc.) and bundles/kits (selling 1 gift pack deducts component SKUs)
- [ ] Split & mixed tender payments (Cash, Card, QR/E-Wallet, combined payments)
- [ ] Item-level and cart-level discounts (% or fixed), promo codes, custom/open-price items
- [ ] Optional round-up / conservation donation prompt at checkout
- [ ] Hold / Park Sale (suspend a cart, serve the next customer, resume later)
- [ ] Returns, partial refunds, exchanges, and voids with mandatory reason codes and restock toggle

### Cash Management & End-of-Day (EOD)
- [ ] Shift open/close with starting cash float
- [ ] Cash-in / Cash-out logs (petty cash, safe drops)
- [ ] Blind cash count (cashier enters counted cash before seeing expected cash; system calculates over/short variance)
- [ ] X-Report (mid-day snapshot) and Z-Report (final EOD closing summary by payment method, tax, discounts, refunds)
- [ ] Offline-aware EOD warning when mobile/offline devices still have unsynced sales

### Inventory & Catalog
- [ ] Real-time stock tracking with low-stock threshold alerts
- [ ] Mobile stocktake / cycle counting using the phone camera scanner
- [ ] Stock adjustments (damaged, lost, expired, internal use)
- [ ] Barcode/QR label generation & printing for un-barcoded local/handmade souvenirs
- [ ] Suppliers & Purchase Orders (receiving stock deliveries & tracking unit cost / COGS)

### Hardware & Receipts
- [ ] Thermal receipt printing (ESC/POS via USB/LAN on desktop, Bluetooth on mobile)
- [ ] Digital receipts (Email, SMS, or on-screen QR code for paperless/power-outage checkout)
- [ ] Automatic cash drawer kick on cash sales (manual "No Sale" opens logged in audit trail)

### Users, Security & Audit Trail
- [ ] Quick 4–6 digit PIN switching between cashiers
- [ ] Role-Based Access Control (Cashier, Supervisor, Admin)
- [ ] Manager PIN override for voids, refunds, manual discounts, and price overrides
- [ ] Full immutable audit log (voids, drawer opens, price edits, offline sync discrepancies)

### Reporting & Analytics
- [ ] Sales breakdown by hour, day, category, item, cashier, and payment method
- [ ] Profit & margin reports (Revenue vs. COGS)
- [ ] Tax summaries & one-click CSV/Excel exports

---

## 2. Quality-of-Life (QoL) Features

- [ ] **1-Tap Phone Mode Switcher:** Switch between *Scanner Mode* (beams scans to desktop via WebSocket with beep/vibrate) and *Standalone POS Mode* (offline checkout).
- [ ] **Line-Busting / Queue-Busting:** Staff scan a waiting customer's items on a phone, generate a temporary cart QR code, and the main register scans it to load the cart instantly.
- [ ] **Always-Visible Sync Status Pill:** Displays `Online` or `Offline • N sales queued` with a manual **Sync Now** button and a view of pending/failed items.
- [ ] **Smart Quick-Cash Buttons:** Auto-suggest tender buttons based on cart total (e.g., for \$13.50: `Exact`, `\$15`, `\$20`, `\$50`).
- [ ] **Favorites / Fast-Movers Tab:** Pin top 15–20 small/frequent items (magnets, postcards, water, stickers) to the first screen.
- [ ] **Zero-Click Hardware Scanner & Keyboard Shortcuts:** Capture USB/Bluetooth HID scanner input globally without focusing a search field; add shortcuts (`F2` search, `F4` pay, `Esc` clear).
- [ ] **Reprint Last Receipt & Quick-Fix Payment:** 1-click reprint for paper jams, and admin option to fix a mis-tapped payment method without a full refund.
- [ ] **Battery-Saver / High-Contrast Mode:** Low-power dark/high-contrast UI for mobile power-outage shifts and outdoor sunlight readability.
- [ ] **Customer-Facing Display:** Secondary browser tab/tablet showing live cart items, total, and payment QR code.
- [ ] **Automated Docker Backups:** Scheduled Laravel job dumping the database daily to local/cloud encrypted storage.

---

## 3. Suggested Build Phases

1. **Phase 1 (MVP):** Docker setup, Laravel API + Filament catalog, Vue 3 offline-capable checkout (IndexedDB + integer cents), Phone Scanner Mode (Reverb), basic receipts & sync queue.
2. **Phase 2 (Operations):** Shifts & Blind EOD, Offline-aware EOD checks, Refunds/Voids with Manager PIN override, Inventory adjustments & Mobile Stocktake, Thermal printer integration.
3. **Phase 3 (QoL & Scale):** Bundles/Kits, Line-Busting QR carts, Smart Quick-Cash, Favorites tab, Customer-Facing Display, automated backups, advanced profit/tax reporting.
