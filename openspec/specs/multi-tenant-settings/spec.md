# Multi-Tenant & Settings

> **Source**: `internal/domain/tenant/`, `internal/domain/settings/`, `internal/domain/environment/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Multi-tenant isolation with tenant-scoped billing details, environment management, and configurable settings.

## Domain Model

### Tenant

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `name` | string | Organization name |
| `status` | Status | Tenant state |
| `billing_details` | TenantBillingDetails | Email, help email, phone, address |
| `metadata` | Metadata | Arbitrary data |

### TenantBillingDetails

| Field | Type | Description |
|-------|------|-------------|
| `email` | string | Billing contact email |
| `help_email` | string | Support email |
| `phone` | string | Contact phone |
| `address` | TenantAddress | Full postal address (line1/2, city, state, postal_code, country) |

### Environment
Environments provide multi-environment isolation within a tenant (e.g., production, staging, development).

### Settings
Tenant/environment-scoped configuration values.

## Key Design Patterns
1. **Tenant Isolation**: Every entity scoped to tenant_id via BaseModel
2. **Environment Scoping**: Further isolation within tenant (prod/staging/dev)
3. **Billing Contact**: Tenant-level billing details for invoice headers
4. **Address Formatting**: Helper methods for address line concatenation
