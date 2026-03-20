# Subscription Lifecycle

> **Source**: `internal/domain/subscription/` (12 files), `internal/types/subscription*.go`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Full subscription lifecycle management — creation, billing cycles, plan changes (upgrades/downgrades with proration), pauses, phases, cancellation, trials, and commitments.

## Domain Model

### Subscription

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID identifier |
| `lookup_key` | string | External lookup |
| `customer_id` | string | Owning customer |
| `plan_id` | string | Active plan |
| `subscription_status` | SubscriptionStatus | Lifecycle state |
| `currency` | string | ISO currency (lowercase) |
| `billing_anchor` | time.Time | Reference point for billing cycle alignment |
| `billing_cycle` | BillingCycle | `anniversary` (from start date) or `calendar` (aligned to 1st) |
| `start_date` | time.Time | Subscription start |
| `end_date` | *time.Time | Subscription end (optional) |
| `current_period_start/end` | time.Time | Current billable period |
| `cancelled_at` | *time.Time | When cancellation occurred |
| `cancel_at` | *time.Time | Scheduled cancellation date |
| `cancel_at_period_end` | bool | Cancel at end of current period |
| `trial_start/end` | *time.Time | Trial period window |
| `billing_cadence` | BillingCadence | RECURRING or ONETIME |
| `billing_period` | BillingPeriod | Period type (month, year, etc.) |
| `billing_period_count` | int | Period multiplier (default: 1) |
| `version` | int | Optimistic locking |
| `pause_status` | PauseStatus | Current pause state |
| `active_pause_id` | *string | Active pause reference |
| `commitment_amount` | *decimal | Minimum committed spend per period |
| `commitment_duration` | *BillingPeriod | Commitment time frame (e.g., ANNUAL on MONTHLY sub) |
| `overage_factor` | *decimal | Multiplier for usage beyond commitment |
| `payment_behavior` | string | Payment handling mode |
| `collection_method` | string | Invoice collection method |
| `gateway_payment_method_id` | *string | Payment gateway reference |
| `proration_behavior` | ProrationBehavior | How mid-cycle changes are prorated |
| `enable_true_up` | bool | True-up reconciliation enabled |
| `invoicing_customer_id` | *string | Override customer for invoicing (parent company billing) |
| `parent_subscription_id` | *string | Hierarchy support (child subscriptions) |
| `payment_terms` | *PaymentTerms | NET payment terms (e.g., NET 15, NET 30) |

**Relationships**: LineItems[], Pauses[], Phases[], CouponAssociations[]

### SubscriptionLineItem (~30 fields)

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `subscription_id` | string | Parent subscription |
| `customer_id` | string | Customer |
| `entity_id/type` | string/EntityType | Plan or Addon reference |
| `price_id` | string | Price reference |
| `price_type` | PriceType | USAGE or FIXED |
| `meter_id` | string | Meter for usage prices |
| `quantity` | decimal | Fixed quantity |
| `currency` | string | Line item currency |
| `billing_period/count` | BillingPeriod/int | From price at creation |
| `invoice_cadence` | InvoiceCadence | When to invoice this line |
| `trial_period` | int | Trial days |
| `start_date/end_date` | time.Time | Line item validity window |
| `subscription_phase_id` | *string | Phase association |
| **Commitment fields** | | Per-line-item commitments |
| `commitment_amount` | *decimal | Min spend commitment |
| `commitment_quantity` | *decimal | Min quantity commitment |
| `commitment_type` | CommitmentType | Type of commitment |
| `commitment_overage_factor` | *decimal | Overage multiplier |
| `commitment_true_up_enabled` | bool | True-up per line item |
| `commitment_windowed` | bool | Windowed commitment calculation |
| `commitment_duration` | *BillingPeriod | Commitment time frame |

### SubscriptionPhase
Represents lifecycle phases within a subscription (e.g., trial → active → renewal).

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `subscription_id` | string | Parent subscription |
| `start_date` | time.Time | Phase start |
| `end_date` | *time.Time | Phase end (nil = indefinite) |

### SubscriptionPause
Pause/resume functionality with mode configuration.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `subscription_id` | string | Parent subscription |
| `pause_status` | PauseStatus | Pause state |
| `pause_mode` | PauseMode | How the pause behaves |
| `resume_mode` | ResumeMode | How resumption works |
| `pause_start` | time.Time | Actual pause start |
| `pause_end` | *time.Time | Scheduled end (nil = indefinite) |
| `resumed_at` | *time.Time | Actual resumption time |
| `original_period_start/end` | time.Time | Billing period when pause was created |
| `reason` | string | Why the subscription was paused |

### SubscriptionChange
Plan change operations (upgrade/downgrade) with proration.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `subscription_id` | string | Subscription being changed |
| `target_plan_id` | string | New plan |
| `proration_behavior` | ProrationBehavior | How proration is handled |
| `effective_date` | time.Time | When change takes effect |
| `billing_cycle_anchor` | BillingCycleAnchor | How billing cycle adjusts |

**Preview**: `SubscriptionChangePreview` provides pre-execution impact analysis including: change type (upgrade/downgrade/lateral), proration details (credit/charge/net amounts, days used/remaining), new billing cycle, immediate + next invoice amounts, warnings.

**Result**: `SubscriptionChangeResult` captures: old/new subscription IDs, change type, immediate invoice ID, proration applied, effective date.

## Key Design Patterns
1. **Optimistic Locking**: `version` field prevents concurrent modification
2. **Billing Anchor**: Anniversary vs calendar billing cycles with custom anchor dates
3. **Commitment Model**: Both subscription-level and line-item-level commitments with amount, quantity, overage factor, true-up, and windowed options
4. **Pause/Resume**: Configurable modes for both pause and resume behavior
5. **Plan Change Preview**: Full impact analysis before executing changes
6. **Subscription Hierarchy**: Parent/child subscriptions via `parent_subscription_id`
7. **Invoice Customer Override**: Invoicing can target a different customer (parent company billing for child)
8. **Mixed Billing Periods**: Subscription detects when line items have different billing periods
