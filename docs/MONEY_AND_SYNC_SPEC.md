# Money Math & Offline Sync Specification

This document defines the strict rules for financial calculations and offline synchronization between **Vue (JavaScript)** and **Laravel (PHP)** so totals never drift by \$0.01 and offline receipts remain trustworthy.

---

## 1. Integer Minor Units (Divide by 100 Only at Display)

- **All monetary amounts MUST be stored, transmitted, and calculated as integers in minor units (cents/sen).**
  - Example: `\$13.50` is stored and calculated as `1350`.
- **Database:** Use `BIGINT` (signed where negative adjustments/refunds apply). Never use `FLOAT` or `DOUBLE`.
- **Backend (Laravel/PHP):** Use PHP `int` (or `brick/money` backed by integer minor units). Never use PHP `float` for money.
- **Frontend (Vue/JS):** Use JS integer `number` (`Number.isSafeInteger`). Never perform floating-point arithmetic on currency values.
- **Display Layer Only:** Divide by `100` (e.g., `(amountCents / 100).toFixed(2)` or `Intl.NumberFormat`) **strictly when formatting for UI or receipts**. Never pass formatted decimal strings back into calculations.

---

## 2. Deterministic Calculation Order & Rounding

Both Vue (offline) and Laravel (online) MUST follow the exact same order of operations and rounding rule:

1. **Line Subtotal:** `unit_price_cents * quantity` (exact integer).
2. **Line Discount:**
   - Fixed discount: subtracted directly in integer cents (clamped at `0`).
   - Percentage discount: computed in integer basis points (`bps`, where `100% = 10000`, `15% = 1500`) using **Half-Up Rounding** (`floor((amount * bps + 5000) / 10000)` for non-negative values).
3. **Cart Discount:** Applied after line discounts using the same basis-point Half-Up formula.
4. **Tax Calculation:** Pick **one** tax mode per shop configuration (tax-inclusive or tax-exclusive, per-line or on-subtotal) and use the exact same integer Half-Up formula on both sides.
5. **Grand Total:** Sum of net lines + tax (if tax-exclusive) + optional donation/round-up.

> **Important:** Do not rely on `Math.round()` in JS vs `round()` in PHP on negative numbers or floats, as their tie-breaking rules differ. Use an explicit integer helper `roundHalfUpDiv(numerator, denominator)` implemented identically in both PHP and JS.

---

## 3. Shared Golden Test Suite

- Maintain a single shared file: `shared/pricing_cases.json`.
- Include tricky edge cases:
  - Repeating decimals (e.g., 3 items at `333` cents with 6% tax and 15% discount)
  - Split tender payments
  - Partial refunds & restocking
  - Zero-priced / 100% discounted items
  - Rounding boundary ties (`.5` cent cases)
- Both **Pest/PHPUnit** (Laravel) and **Vitest** (Vue) MUST run against `shared/pricing_cases.json` in CI/local testing. Any 1-cent mismatch fails the build.

---

## 4. Offline Sync & Receipt Immutability

1. **Client-Generated UUIDs:** Every offline sale, sale line, payment, and shift uses a client-generated UUID so syncing never produces primary-key collisions.
2. **Idempotent Sync Endpoints:** Submitting the same `sale_uuid` twice MUST return the existing record without duplicating the sale or double-deducting inventory.
3. **Never Mutate a Synced Offline Receipt:**
   - Whatever total the POS charged the customer offline is the **financial truth of that receipt**.
   - On sync, Laravel stores the exact receipt snapshot (unit prices, discounts, tax, grand total, tender amounts).
   - Laravel may run a server-side verification calculation; if a discrepancy is detected, **do not overwrite the receipt total**—record a `sync_discrepancy` flag in the audit log for manager review.
4. **Inventory Deltas:** Sync stock changes as append-only ledger movements (`-qty` for sale, `+qty` for restock/return), never absolute stock overwrites.
