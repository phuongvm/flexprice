# Pricing Engine

> **Source**: `internal/domain/price/`, `internal/types/price.go`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Flexible pricing model supporting flat-fee, usage-based, tiered, and package billing with multi-currency and custom price units.

## Domain Model

### Price
Core pricing entity with ~35 fields.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID identifier |
| `amount` | decimal | Base amount in main currency units (e.g., $12.50, not cents) |
| `display_amount` | string | Formatted with currency symbol |
| `currency` | string | 3-digit ISO code (lowercase: usd, eur, gbp) |
| `type` | PriceType | `USAGE` or `FIXED` |
| `billing_model` | BillingModel | `FLAT_FEE`, `PACKAGE`, or `TIERED` |
| `billing_period` | BillingPeriod | Subscription billing period |
| `billing_period_count` | int | Period multiplier (default: 1) |
| `billing_cadence` | BillingCadence | `RECURRING` or `ONETIME` |
| `invoice_cadence` | InvoiceCadence | When invoices are generated |
| `tier_mode` | BillingTier | Tier calculation mode |
| `tiers` | []PriceTier | Tier definitions (for TIERED model) |
| `meter_id` | string | Linked meter (for USAGE type) |
| `display_name` | string | Human-readable name |
| `min_quantity` | *decimal | Minimum billable quantity |
| `trial_period` | int | Trial days (recurring fixed prices only) |
| `transform_quantity` | TransformQuantity | Package quantity transformation |
| `metadata` | map[string]string | Arbitrary key-value metadata |
| `entity_type` | PriceEntityType | `PLAN` or `ADDON` — what this price belongs to |
| `entity_id` | string | Plan/Addon ID |
| `parent_price_id` | string | Root price reference for lineage/overrides |
| `group_id` | string | Group association |
| `start_date` / `end_date` | *time.Time | Price validity window |
| `lookup_key` | string | External lookup identifier |

#### Custom Price Units
| Field | Type | Description |
|-------|------|-------------|
| `price_unit_type` | PriceUnitType | `FIAT` or `CUSTOM` |
| `price_unit_id` | *string | Custom unit ID |
| `price_unit` | *string | Unit code (e.g., 'btc', 'eth') |
| `price_unit_amount` | *decimal | Amount in custom unit |
| `conversion_rate` | *decimal | Custom unit → fiat conversion rate |
| `price_unit_tiers` | []PriceTier | Tiered pricing in custom units |

### PriceTier
| Field | Type | Description |
|-------|------|-------------|
| `up_to` | *uint64 | Upper bound (INCLUSIVE). `nil` = last tier (∞) |
| `unit_amount` | decimal | Per-unit cost within tier |
| `flat_amount` | *decimal | Optional flat fee on top of unit_amount × qty |

### TransformQuantity (for PACKAGE model)
| Field | Type | Description |
|-------|------|-------------|
| `divide_by` | int | Divide raw quantity by this (≥1) |
| `round` | RoundType | `up` or `down` after division |

## Enums

### PriceType
- `USAGE` — usage-based, linked to a meter (`meter_id` required)
- `FIXED` — fixed amount (default quantity = 1)

### BillingModel
- `FLAT_FEE` — simple per-unit pricing: `amount × quantity`
- `PACKAGE` — quantity transformed via `divide_by` + `round`, then priced
- `TIERED` — quantity matched against tier boundaries

### BillingCadence
- `RECURRING` — charged every billing period
- `ONETIME` — charged once

## Business Rules

### Validation
- Amount MUST be non-negative
- Trial period MUST be non-negative; only allowed for `RECURRING` + `FIXED` prices
- `transform_quantity.divide_by` MUST be ≥ 1
- Entity type MUST be valid (`PLAN` or `ADDON`)

### Price Calculation
- **Flat fee**: `price.amount × quantity`
- **Tiered**: Each tier calculates `tier.unit_amount × tier_quantity + tier.flat_amount`
- **Package**: Quantity transformed first (`divide_by` + `round`), then priced

### Price Lineage
- `parent_price_id` always references the root/original plan price
- Cloned prices clear `parent_price_id` to avoid retaining source lineage
- `GetRootPriceID()` returns `parent_price_id` if set, else self

### Eligibility
- Price is eligible for subscription when: `currency` matches AND `billing_period` matches AND `billing_period_count` matches
- Price is active when: `status = published` AND current time is within `[start_date, end_date]`

### Currency
- Amounts stored in main units (dollars, not cents)
- Precision rounding based on currency config (`GetCurrencyPrecision()`)
- Display format: `{symbol}{amount}` (e.g., `$12.50`)

### Limits
- Maximum 3000 active prices per entity

## Key Design Patterns
1. **Dual Currency**: Fiat + custom price units (crypto, tokens) with conversion rates
2. **Tiered with Flat Fee**: Each tier supports both per-unit AND flat-amount components
3. **Price Lineage**: `parent_price_id` tracks price overrides back to original plan price
4. **Entity Scoping**: Prices belong to either plans or addons via `entity_type` + `entity_id`
5. **CopyWith Pattern**: Functional clone with selective overrides for price versioning
