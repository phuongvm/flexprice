# Auth & Identity

> **Source**: `internal/domain/auth/`, `internal/domain/user/`, `internal/domain/secret/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Authentication and user identity management with provider-based auth, JWT claims, and customer portal sessions.

## Domain Model

### Auth

| Field | Type | Description |
|-------|------|-------------|
| `user_id` | string | User identifier |
| `provider` | AuthProvider | Auth provider type |
| `token` | string | Hashed credential (e.g., hashed password) |
| `status` | Status | Auth status |

### Claims (Admin JWT)
| Field | Type | Description |
|-------|------|-------------|
| `user_id` | string | Authenticated user |
| `tenant_id` | string | Tenant context |
| `email` | string | User email |

### SessionClaims (Customer Portal JWT)
| Field | Type | Description |
|-------|------|-------------|
| `external_customer_id` | string | External customer ID |
| `customer_id` | string | Internal customer ID |
| `tenant_id` | string | Tenant context |
| `environment_id` | string | Environment context |

## Key Design Patterns
1. **Provider-Based Auth**: Extensible auth provider system
2. **Dual JWT Types**: Admin claims (user/tenant/email) vs Customer sessions (customer/tenant/environment)
3. **Hashed Tokens**: Credentials stored as hashed values
