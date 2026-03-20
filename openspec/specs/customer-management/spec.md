# Customer Management

> **Source**: `internal/domain/customer/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Customer entity management with external ID mapping, hierarchical customers (parent/child), and address handling.

## Domain Model

### Customer

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `external_id` | string | External system identifier |
| `name` | string | Customer name |
| `email` | string | Customer email |
| `parent_customer_id` | *string | Parent customer for hierarchy |
| `address_line1` | string | Address line 1 (≤255 chars) |
| `address_line2` | string | Address line 2 (≤255 chars) |
| `address_city` | string | City (≤100 chars) |
| `address_state` | string | State (≤100 chars) |
| `address_postal_code` | string | Postal code (≤20 chars) |
| `address_country` | string | ISO 3166-1 alpha-2 country code (2 chars) |
| `metadata` | map[string]string | Arbitrary metadata |

### Validation
- Country code: exactly 2 characters (ISO 3166-1 alpha-2)
- Postal code: max 20 characters
- Address lines: max 255 characters each
- City/State: max 100 characters each

## Relationships
- **Customer → Subscriptions**: One-to-many (customer can have multiple subscriptions)
- **Customer → Wallets**: One-to-many (customer can have multiple wallets)
- **Customer → Invoices**: One-to-many
- **Customer → Parent Customer**: Self-referencing hierarchy via `parent_customer_id`
- **Invoicing Override**: Subscriptions can set `invoicing_customer_id` to bill a different customer

## Key Design Patterns
1. **Dual ID**: Internal `id` + external system `external_id` for integration
2. **Customer Hierarchy**: Parent/child relationships for corporate billing
3. **Structured Address**: Full address fields for tax calculation and compliance
