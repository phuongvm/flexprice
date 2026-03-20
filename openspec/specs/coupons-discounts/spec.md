# Coupons & Discounts

> **Source**: `internal/domain/coupon/`, `internal/domain/coupon_association/`, `internal/domain/coupon_application/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Discount management with fixed/percentage coupons, redemption limits, duration control, and application tracking.

## Domain Model

### Coupon

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `name` | string | Coupon name |
| `type` | CouponType | `fixed` or `percentage` |
| `amount_off` | *decimal | Fixed discount amount |
| `percentage_off` | *decimal | Percentage discount |
| `currency` | string | Currency (for fixed coupons) |
| `cadence` | CouponCadence | Coupon cadence |
| `duration_in_periods` | *int | How many billing periods the discount lasts |
| `redeem_after` | *time.Time | Earliest redemption date |
| `redeem_before` | *time.Time | Latest redemption date |
| `max_redemptions` | *int | Maximum total uses |
| `total_redemptions` | int | Current usage count |
| `rules` | *map[string]interface{} | Custom eligibility rules |
| `metadata` | *map[string]string | Arbitrary metadata |

### Validation (IsValid)
- Status must be `published`
- Current time must be within `[redeem_after, redeem_before]`
- `total_redemptions < max_redemptions` (if max set)

### Discount Calculation (ApplyDiscount)
- **Fixed**: discount = `amount_off`
- **Percentage**: discount = `original_price × percentage_off / 100`
- Discount rounded to currency precision
- Final price cannot go below zero (discount capped at original price)

### Related Domains
- **CouponAssociation**: Links coupons to subscriptions
- **CouponApplication**: Records coupon usage on specific invoices

## Key Design Patterns
1. **Dual Discount**: Fixed amount or percentage off
2. **Redemption Window**: Time-bounded validity
3. **Usage Limits**: Max redemptions with counter tracking
4. **Currency Precision**: Discount rounded at source to currency precision
5. **Floor at Zero**: Final price never goes negative
