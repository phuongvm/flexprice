# Customer Portal

> **Source**: `internal/service/customer_portal.go`, `internal/domain/auth/model.go` (SessionClaims)
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Customer-facing self-service portal with JWT-based session authentication for viewing subscriptions, invoices, usage, and wallets.

## Domain Model

### SessionClaims (JWT Token)
| Field | Type | Description |
|-------|------|-------------|
| `external_customer_id` | string | External ID from tenant system |
| `customer_id` | string | Internal FlexPrice customer ID |
| `tenant_id` | string | Tenant context |
| `environment_id` | string | Environment context |

## Key Capabilities
1. **Session-based Auth**: JWT tokens with customer+tenant+environment context
2. **Self-service Access**: View subscriptions, invoices, usage, and wallet balances
3. **Dual ID**: Both external and internal customer IDs for cross-system mapping
