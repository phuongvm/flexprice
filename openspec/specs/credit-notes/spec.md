# Credit Notes

> **Source**: `internal/domain/creditnote/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Invoice corrections and refunds via credit notes, with line items, finalization, and voiding lifecycle.

## Domain Model

### CreditNote

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `credit_note_number` | string | Human-readable number |
| `invoice_id` | string | Linked invoice |
| `customer_id` | string | Customer |
| `subscription_id` | *string | Optional subscription link |
| `credit_note_status` | CreditNoteStatus | draft/finalized/voided |
| `credit_note_type` | CreditNoteType | `refund` or `adjustment` |
| `refund_status` | *PaymentStatus | Refund processing status |
| `reason` | CreditNoteReason | duplicate/fraudulent/order_change/product_unsatisfactory |
| `memo` | string | Optional memo |
| `currency` | string | ISO currency |
| `total_amount` | decimal | Total including discounts and tax |
| `voided_at` | *time.Time | Void timestamp |
| `finalized_at` | *time.Time | Finalization timestamp |
| `idempotency_key` | *string | Dedup key |
| `line_items` | []CreditNoteLineItem | Individual credit items |

## Key Design Patterns
1. **Type Distinction**: `refund` (actual money back) vs `adjustment` (billing correction, non-cash)
2. **Invoice Link**: Always tied to a specific invoice
3. **Reason Classification**: Structured reasons for audit trail
4. **Lifecycle**: Draft → Finalized → (optionally) Voided
