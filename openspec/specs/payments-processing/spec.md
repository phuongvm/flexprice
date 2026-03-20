# Payments Processing

> **Source**: `internal/domain/payment/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Payment processing with gateway abstraction, multiple method types, retry tracking, and idempotent operations.

## Domain Model

### Payment (~20 fields)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `idempotency_key` | string | Dedup key |
| `destination_type` | PaymentDestinationType | Target entity (invoice, subscription) |
| `destination_id` | string | Target entity ID |
| `payment_method_type` | PaymentMethodType | `card`, `bank_transfer`, `offline`, `payment_link` |
| `payment_method_id` | string | Specific payment method |
| `payment_gateway` | *string | Gateway name |
| `gateway_payment_id` | *string | Gateway transaction ID |
| `gateway_tracking_id` | *string | Gateway tracking ID |
| `gateway_metadata` | Metadata | Gateway-specific data |
| `amount` | decimal | Payment amount (must be >0) |
| `currency` | string | ISO currency |
| `payment_status` | PaymentStatus | pending/succeeded/failed/refunded |
| `track_attempts` | bool | Whether to monitor attempts |
| `succeeded_at/failed_at/refunded_at/recorded_at` | *time.Time | Lifecycle timestamps |
| `error_message` | *string | Failure details |
| `attempts` | []PaymentAttempt | Processing attempt log |

### PaymentAttempt
| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `payment_id` | string | Parent payment |
| `attempt_number` | int | Sequential (must be >0) |
| `payment_status` | PaymentStatus | This attempt's outcome |
| `gateway_attempt_id` | *string | Gateway-specific attempt ID |
| `error_message` | *string | Attempt-specific error |

### Validation
- Amount MUST be >0
- Offline payments: `payment_method_id` MUST be empty
- Payment links: `payment_method_id` MUST be empty
- Card payments: `payment_method_id` is optional (auto-fetched)
- Other types: `payment_method_id` is required

## Key Design Patterns
1. **Gateway Abstraction**: Gateway details stored as metadata, supporting multiple processors
2. **Idempotency**: Dedup via idempotency_key
3. **Attempt Tracking**: Full history of payment processing attempts
4. **Method-Type Rules**: Different validation for offline/card/payment-link
