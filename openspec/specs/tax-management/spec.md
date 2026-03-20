# Tax Management

> **Source**: `internal/domain/tax/`, `internal/domain/taxapplied/`, `internal/domain/taxassociation/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Tax rate definitions with percentage and fixed-value support, scoped application, and association to entities.

## Domain Model

### TaxRate

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `name` | string | Tax name (e.g., "VAT", "GST") |
| `description` | string | Description |
| `code` | string | Tax code |
| `tax_rate_status` | TaxRateStatus | Rate lifecycle state |
| `tax_rate_type` | TaxRateType | Classification |
| `scope` | TaxRateScope | Application scope |
| `percentage_value` | *decimal | Percentage rate (e.g., 0.18 for 18%) |
| `fixed_value` | *decimal | Fixed amount |
| `metadata` | map[string]string | Arbitrary metadata |

### Related Domains
- **TaxApplied**: Instance of tax applied to a specific line item/invoice
- **TaxAssociation**: Links tax rates to entities (plans, subscriptions, etc.)

## Key Design Patterns
1. **Dual Calculation**: Both percentage-based and fixed-amount taxes
2. **Scoped Application**: Tax rates scoped to specific contexts
3. **Association Pattern**: Tax rates linked to entities through separate association table
