# Requirement Analysis — Output Template & Worked Example

Copy this skeleton for the output. The worked example below (a fictional "apply loyalty points at checkout" change) shows the expected level of detail — replace with real file:line references from the actual codebase being analyzed.

---

## 1. Scope

**In-scope**
- Applying loyalty points as a discount at checkout for logged-in customers

**Out-of-scope**
- Earning/accruing loyalty points (handled by a separate requirement)
- Guest checkout (no account → no points to redeem)

**Assumptions** *(confirm with requester)*
- Assuming 1 point = 1,000 VND based on the existing `LoyaltyService` conversion rate — the requirement doc doesn't state this explicitly.

---

## 2. Functional Requirements

| ID | Actor | Trigger → Action → Outcome | Acceptance Criteria |
|---|---|---|---|
| FR-01 | Customer | At checkout, selects "use points" → system deducts available points as discount | Given a customer with ≥100 points, when they toggle "use points", then order total decreases by `points × 1000 VND`, capped at order subtotal |
| FR-02 | Customer | Has fewer points than order subtotal requires | Given points < subtotal, when applied, then only available points are deducted (partial redemption), remainder paid normally |

---

## 3. Non-Functional Requirements

| Category | Applicable? | Notes |
|---|---|---|
| Performance | Yes | Points balance check must not add >200ms to checkout (currently on critical path) |
| Security & authorization | Yes | Only the authenticated account owner can redeem their own points |
| Reliability | Yes | Redemption + order placement must be atomic — a failed order must not deduct points |
| Compliance/audit | Yes | Point deduction must be logged for finance reconciliation |
| Scalability | Not applicable | No expected volume change from this feature |

---

## 4. Business Logic — Old vs New Flow

| Step | Old (file:function) | New (file:function) | What changes |
|---|---|---|---|
| 1. Compute subtotal | `checkout/service.ts:88 — CheckoutService.computeSubtotal()` | unchanged | — |
| 2. Apply discount codes | `checkout/service.ts:102 — CheckoutService.applyDiscounts()` | same function, extended | Now also calls `LoyaltyService.reserve()` |
| 3. Reserve/deduct points | *(does not exist)* | `(new) loyalty/service.ts — LoyaltyService.reserve()` | New step: validates balance, holds points pending order confirmation |
| 4. Finalize order | `checkout/service.ts:140 — CheckoutService.placeOrder()` | same, extended | On success calls `LoyaltyService.commit()`; on failure calls `LoyaltyService.release()` |

---

## 5. Data Model — Old vs New

**`orders` table** (`db/models/order.ts`)
- Old: no field tracking points used
- New: add `points_redeemed INT NOT NULL DEFAULT 0`, `points_discount_amount DECIMAL NOT NULL DEFAULT 0`
- Backward compatibility: default `0` is safe for existing rows; no backfill needed

**`loyalty_points` table** (`db/models/loyalty.ts`)
- Old: `balance INT`
- New: add `reserved INT NOT NULL DEFAULT 0` to support the reserve/commit/release flow in Step 4 above

---

## 6. Glossary

| Term | Meaning |
|---|---|
| GMV | Gross Merchandise Value |
| Reserve/Commit/Release | Two-phase pattern: hold points during checkout, finalize on success, return on failure |

---

## 7. Business Rules & Edge Cases

| Rule | Example input → outcome | Edge case | Expected outcome |
|---|---|---|---|
| Redemption capped at subtotal | 500 points (500,000đ) on a 300,000đ order → only 300 points used, 200 remain | Order total is exactly 0 after discount | Order still requires a valid payment method call with amount 0, not skipped |
| Points must be non-negative and integer | 100 points → valid | Balance is 0 | "Use points" toggle disabled, not shown as an error |
| Points reserved must be released on payment failure | Payment fails after reserve | Reserve succeeds, order placement times out (unknown outcome) | Reservation must have a TTL/reconciliation job — otherwise points are stuck in limbo |
| Only the account owner can redeem | User A tries to redeem using User B's account context | Concurrent redemption from two devices on the same account | Second request must see the already-reserved balance (no double-spend) |
