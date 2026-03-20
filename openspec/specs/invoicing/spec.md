# Invoicing

> **Source**: `internal/domain/invoice/` (4 files)
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Full invoice lifecycle — generation, finalization, payment tracking, voiding, credit notes, and PDF delivery.

## Domain Model

### Invoice (~30 fields)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `customer_id` | string | Recipient customer |
| `subscription_id` | *string | Linked subscription (optional) |
| `invoice_type` | InvoiceType | Subscription or one-time |
| `invoice_status` | InvoiceStatus | Draft, open, paid, void, etc. |
| `payment_status` | PaymentStatus | Pending, paid, failed, overpaid |
| `currency` | string | ISO currency |
| `amount_due` | decimal | Total amount owed |
| `amount_paid` | decimal | Amount already paid |
| `amount_remaining` | decimal | Outstanding balance |
| `subtotal` | decimal | Sum of line items |
| `total` | decimal | Final amount (incl. tax/discounts) |
| `total_discount` | decimal | Coupon discounts applied |
| `total_tax` | decimal | Sum of all taxes |
| `total_prepaid_credits_applied` | decimal | Prepaid credits used |
| `adjustment_amount` | decimal | Credit notes (type: adjustment) |
| `refunded_amount` | decimal | Credit notes (type: refund) |
| `invoice_number` | *string | Human-readable number (e.g., INV-2024-001) |
| `idempotency_key` | *string | Dedup for creation |
| `billing_sequence` | *int | Sequential cycle number |
| `due_date` | *time.Time | Payment due date |
| `paid_at` | *time.Time | Full payment timestamp |
| `voided_at` | *time.Time | Void timestamp |
| `finalized_at` | *time.Time | Finalization timestamp |
| `period_start/end` | *time.Time | Billing period covered |
| `invoice_pdf_url` | *string | PDF download URL |
| `billing_reason` | string | Why invoice was generated |
| `version` | int | Optimistic locking |
| `recalculated_invoice_id` | *string | Void → replacement invoice link |
| `line_items` | []InvoiceLineItem | Individual charges |
| `coupon_applications` | []CouponApplication | Discounts applied |

### Validation Rules
- `amount_due` and `amount_paid` MUST be non-negative
- `amount_paid ≤ amount_due` (unless PaymentStatus = OVERPAID)
- `amount_remaining = amount_due - amount_paid` (OVERPAID → remaining = 0)
- `period_end > period_start` when both set
- Subscription invoices MUST have `billing_period`
- Line item currency MUST match invoice currency

## Key Design Patterns
1. **Amount Invariant**: `amount_remaining = amount_due - amount_paid` (enforced)
2. **Overpayment Support**: `OVERPAID` status allows `amount_paid > amount_due`
3. **Void & Recalculate**: `recalculated_invoice_id` links voided → replacement invoice
4. **Prepaid Credits**: Applied as payment against invoice total
5. **Sequential Billing**: `billing_sequence` tracks cycle number
6. **Coupon Integration**: CouponApplications as separate line-level entities
