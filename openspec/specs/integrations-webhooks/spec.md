# Integrations & Webhooks

> **Source**: `internal/domain/connection/`, `internal/domain/entityintegrationmapping/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

External system integrations via connection management with encrypted secrets, bi-directional sync configuration, and entity mapping.

## Domain Model

### Connection

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `name` | string | Connection name |
| `provider_type` | SecretProvider | Integration provider |
| `encrypted_secret_data` | ConnectionMetadata | Provider-specific encrypted credentials |
| `metadata` | map[string]interface{} | General metadata (e.g., invoice_sync_enable) |
| `sync_config` | *SyncConfig | Bi-directional sync settings per entity type |

### Supported Providers (8)
| Provider | Key Fields |
|----------|-----------|
| **Stripe** | publishable_key, secret_key, webhook_secret, account_id |
| **S3** | aws_access_key_id, aws_secret_access_key, aws_session_token |
| **HubSpot** | access_token, client_secret, app_id |
| **Razorpay** | key_id, secret_key, webhook_secret |
| **Chargebee** | site, api_key, webhook_secret, webhook_username, webhook_password |
| **QuickBooks** | client_id, client_secret, realm_id, OAuth tokens, income_account_id |
| **Nomod** | api_key, webhook_secret |
| **Moyasar** | secret_key, publishable_key, webhook_secret |
| **Generic** | Arbitrary data map (fallback) |

### SyncConfig (Bi-directional)
Entity-level inbound/outbound sync control:

| Entity | Inbound | Outbound |
|--------|---------|----------|
| Plan | ✅ Supported | ❌ Not supported |
| Subscription | ✅ Supported | ❌ Not supported |
| Invoice | ❌ Not supported | ✅ Supported |
| Deal | ❌ Not supported | ✅ Supported |
| Quote | ❌ Not supported | ✅ Supported |
| Payment | ✅ Supported | ✅ Supported |

### EntityIntegrationMapping
Maps FlexPrice entities to their external counterparts for sync tracking.

## Key Design Patterns
1. **Provider-Typed Metadata**: Structured credential storage per provider
2. **Encrypted Secrets**: Credentials stored encrypted
3. **Bi-Directional Sync**: Per-entity inbound/outbound control
4. **Entity Mapping**: Cross-system entity ID mapping for sync
5. **Webhook Support**: Provider-specific webhook secret management
