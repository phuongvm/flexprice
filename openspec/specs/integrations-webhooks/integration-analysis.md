# FlexPrice Integration Architecture — Deep Analysis

> **Generated**: 2026-03-20 (reverse-engineered from source code)
> **Spec**: `integrations-webhooks/spec.md`
> **Purpose**: Cross-referenced inventory of all integrations with code line references

## Integration Layers Overview

FlexPrice has **4 integration layers** that determine how external providers are connected:

```
Layer 1: SecretProvider enum        → Who CAN be connected (credential registry)
Layer 2: ConnectionMetadata structs → What credentials each provider needs
Layer 3: PaymentGatewayType enum    → Who CAN process payments
Layer 4: Integration Factory        → Who HAS full service implementations
```

---

## Layer 1: SecretProvider Enum (9 providers)

**File**: [`types/secret.go`](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L27-L39)

| # | Provider | Enum Value | Line |
|---|----------|-----------|------|
| 1 | FlexPrice (internal) | `"flexprice"` | [L31](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L31) |
| 2 | **Stripe** | `"stripe"` | [L32](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L32) |
| 3 | **S3** | `"s3"` | [L33](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L33) |
| 4 | **HubSpot** | `"hubspot"` | [L34](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L34) |
| 5 | **Razorpay** | `"razorpay"` | [L35](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L35) |
| 6 | **Chargebee** | `"chargebee"` | [L36](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L36) |
| 7 | **QuickBooks** | `"quickbooks"` | [L37](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L37) |
| 8 | **Nomod** | `"nomod"` | [L38](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L38) |
| 9 | **Moyasar** | `"moyasar"` | [L39](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L39) |

Validation: [`Validate()`](file:///O:/workspaces/oss/flexprice/internal/types/secret.go#L42-L59) rejects any provider not in this list.

---

## Layer 2: ConnectionMetadata Structs (8 typed + 1 generic)

**File**: [`types/connection.go`](file:///O:/workspaces/oss/flexprice/internal/types/connection.go)

| # | Provider | Struct | Key Credential Fields | Lines |
|---|----------|--------|----------------------|-------|
| 1 | Stripe | `StripeConnectionMetadata` | publishable_key, secret_key, webhook_secret, account_id | [L42-47](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L42-L47) |
| 2 | S3 | `S3ConnectionMetadata` | aws_access_key_id, aws_secret_access_key, aws_session_token | [L51-55](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L51-L55) |
| 3 | HubSpot | `HubSpotConnectionMetadata` | access_token, client_secret, app_id | [L73-77](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L73-L77) |
| 4 | Razorpay | `RazorpayConnectionMetadata` | key_id, secret_key, webhook_secret | [L95-99](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L95-L99) |
| 5 | Chargebee | `ChargebeeConnectionMetadata` | site, api_key, webhook_secret, webhook_username, webhook_password | [L117-123](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L117-L123) |
| 6 | QuickBooks | `QuickBooksConnectionMetadata` | client_id, client_secret, realm_id, environment, OAuth tokens, webhook_verifier_token | [L141-164](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L141-L164) |
| 7 | Nomod | `NomodConnectionMetadata` | api_key, webhook_secret | [L194-197](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L194-L197) |
| 8 | Moyasar | `MoyasarConnectionMetadata` | publishable_key, secret_key, webhook_secret | [L211-215](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L211-L215) |
| 9 | Generic | `GenericConnectionMetadata` | data (arbitrary map) — fallback | [L254-256](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L254-L256) |

Union struct: [`ConnectionMetadata`](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L269-L280) — holds one-of the above.
Validation switch: [`Validate(providerType)`](file:///O:/workspaces/oss/flexprice/internal/types/connection.go#L283-L349) dispatches per provider.

---

## Layer 3: PaymentGatewayType Enum (4 payment gateways)

**File**: [`types/payment_gateway.go`](file:///O:/workspaces/oss/flexprice/internal/types/payment_gateway.go#L10-L14)

| # | Gateway | Enum Value | Line | Payment Link Support |
|---|---------|-----------|------|---------------------|
| 1 | **Stripe** | `"stripe"` | [L11](file:///O:/workspaces/oss/flexprice/internal/types/payment_gateway.go#L11) | ✅ Yes |
| 2 | **Razorpay** | `"razorpay"` | [L12](file:///O:/workspaces/oss/flexprice/internal/types/payment_gateway.go#L12) | ✅ Yes |
| 3 | **Nomod** | `"nomod"` | [L13](file:///O:/workspaces/oss/flexprice/internal/types/payment_gateway.go#L13) | ✅ Yes |
| 4 | **Moyasar** | `"moyasar"` | [L14](file:///O:/workspaces/oss/flexprice/internal/types/payment_gateway.go#L14) | ✅ Yes (payment link only) |

> **Note**: Only 4 of the 8 external providers are payment gateways. The others (HubSpot, Chargebee, QuickBooks, S3) are sync/utility integrations.

Gateway dispatch in payment processor: [`payment_processor.go:L242-257`](file:///O:/workspaces/oss/flexprice/internal/service/payment_processor.go#L242-L257)
Default gateway: Stripe ([L232](file:///O:/workspaces/oss/flexprice/internal/service/payment_processor.go#L232))

---

## Layer 4: Integration Factory (7 full implementations)

**File**: [`integration/factory.go`](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go)

| # | Provider | Factory Method | Lines | Services |
|---|----------|---------------|-------|----------|
| 1 | **Stripe** | `GetStripeIntegration()` | [L88-154](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L88-L154) | Client, Customer, Payment, InvoiceSync, Plan, Subscription, Webhook |
| 2 | **HubSpot** | `GetHubSpotIntegration()` | [L157-215](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L157-L215) | Client, Customer, InvoiceSync, DealSync, QuoteSync, Webhook |
| 3 | **Razorpay** | `GetRazorpayIntegration()` | [L218-266](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L218-L266) | Client, Customer, Payment, InvoiceSync, Webhook |
| 4 | **Chargebee** | `GetChargebeeIntegration()` | [L269-339](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L269-L339) | Client, ItemFamily, Item, ItemPrice, Customer, Invoice, PlanSync, Webhook |
| 5 | **QuickBooks** | `GetQuickBooksIntegration()` | [L342-411](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L342-L411) | Client, Customer, ItemSync, Invoice, Payment, Webhook |
| 6 | **Nomod** | `GetNomodIntegration()` | [L414-463](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L414-L463) | Client, Customer, Payment, InvoiceSync, Webhook |
| 7 | **Moyasar** | `GetMoyasarIntegration()` | [L466-512](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L466-L512) | Client, Customer, Payment, InvoiceSync, Webhook |

Generic dispatcher: [`GetIntegrationByProvider()`](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L515-L538) — routes to above 7.
S3 Storage: [`GetStorageProvider()`](file:///O:/workspaces/oss/flexprice/internal/integration/factory.go#L769-L779) — separate from payment/sync.

---

## Integration Source Code Directories

**Path**: `internal/integration/`

| # | Directory | Files | Type |
|---|-----------|-------|------|
| 1 | `stripe/` | client, customer, dto, invoice_sync, payment (66KB!), plan, subscription, webhook/ | Payment Gateway + Full Sync |
| 2 | `razorpay/` | client, customer, dto, invoice, payment, webhook/ | Payment Gateway |
| 3 | `moyasar/` | client, client_test, customer, dto, integration_test, invoice, payment, webhook/ | Payment Gateway |
| 4 | `nomod/` | (similar structure) | Payment Gateway |
| 5 | `hubspot/` | (CRM sync) | Sync Only (Deals, Quotes, Invoices) |
| 6 | `chargebee/` | (billing platform sync) | Sync Only (Items, Plans, Invoices) |
| 7 | `quickbooks/` | (accounting sync) | Sync Only (Customers, Items, Invoices, Payments) |
| 8 | `s3/` | (file export) | Storage Only |

---

## Temporal Sync Workflows

**Path**: `internal/temporal/workflows/` and `internal/temporal/activities/`

| Provider | Sync Workflows | Activities |
|----------|---------------|------------|
| **HubSpot** | deal_sync, invoice_sync, quote_sync | deal_sync_activities, invoice_sync_activities, quote_sync_activities |
| **Moyasar** | invoice_sync | invoice_sync_activities |
| **Nomod** | invoice_sync | invoice_sync_activities |
| **QuickBooks** | price_sync | price_sync_activities |
| **Stripe** | (via `invoice_sync.go` service, not Temporal) | N/A |

---

## Paddle Status: ❌ NOT PRESENT

**Confirmed across all 4 layers**:
- ❌ No `SecretProviderPaddle` in `types/secret.go`
- ❌ No `PaddleConnectionMetadata` in `types/connection.go`  
- ❌ No `PaymentGatewayTypePaddle` in `types/payment_gateway.go`
- ❌ No `GetPaddleIntegration()` in `integration/factory.go`
- ❌ No `integration/paddle/` directory
- ❌ No Paddle Temporal workflows
- ❌ Zero grep matches for "paddle" (case-insensitive) across entire codebase

---

## Provider Capability Matrix

| Provider | L1: Credential | L2: Metadata Struct | L3: Payment Gateway | L4: Factory | Webhook | Temporal Sync |
|----------|:-:|:-:|:-:|:-:|:-:|:-:|
| **Stripe** | ✅ | ✅ | ✅ | ✅ (7 services) | ✅ | ❌ (direct) |
| **Razorpay** | ✅ | ✅ | ✅ | ✅ (5 services) | ✅ | ❌ |
| **Nomod** | ✅ | ✅ | ✅ | ✅ (5 services) | ✅ | ✅ |
| **Moyasar** | ✅ | ✅ | ✅ | ✅ (5 services) | ✅ | ✅ |
| **HubSpot** | ✅ | ✅ | ❌ | ✅ (6 services) | ✅ | ✅ |
| **Chargebee** | ✅ | ✅ | ❌ | ✅ (8 services) | ✅ | ❌ |
| **QuickBooks** | ✅ | ✅ | ❌ | ✅ (6 services) | ✅ | ✅ |
| **S3** | ✅ | ✅ | ❌ | ✅ (storage only) | ❌ | ❌ |
| **Paddle** | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

---

## Integration Categories

### 1. Payment Gateways (4) — Process real payments
- Stripe, Razorpay, Nomod, Moyasar

### 2. Billing Platform Sync (1) — Import plans/products
- Chargebee (item families, items, item prices, plan sync)

### 3. CRM Sync (1) — Outbound deals/quotes/invoices
- HubSpot (deal sync, quote sync, invoice sync)

### 4. Accounting Sync (1) — Financial record keeping
- QuickBooks (customers, items, invoices, payments; OAuth-based)

### 5. Storage (1) — File export
- S3 (AWS credentials, bucket operations)
