# Wallets & Credits

> **Source**: `internal/domain/wallet/`, `internal/domain/creditgrant/`, `internal/types/wallet.go`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Credit-based billing system with prepaid wallets, transactions, credit grants (one-time/recurring), auto-topup, and balance alerts.

## Domain Model

### Wallet

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `customer_id` | string | Owning customer |
| `currency` | string | ISO currency |
| `balance` | decimal | Balance in fiat currency (= credit_balance × conversion_rate) |
| `credit_balance` | decimal | Balance in credits |
| `wallet_status` | WalletStatus | Wallet state |
| `name` | string | Display name |
| `wallet_type` | WalletType | Wallet classification |
| `config` | WalletConfig | Wallet configuration |
| `conversion_rate` | decimal | Credits → fiat rate (must be >0). 1 credit = conversion_rate units of currency |
| `topup_conversion_rate` | decimal | Rate for topup operations |
| `auto_topup` | *AutoTopup | Auto-topup configuration |
| `alert_settings` | *AlertSettings | Balance alert thresholds (critical/warning/info) |
| `alert_state` | AlertState | Current alert state |

**Validation**: `balance` MUST equal `credit_balance × conversion_rate`. Both rates must be >0.

### Transaction

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `wallet_id` | string | Parent wallet |
| `customer_id` | string | Customer |
| `type` | TransactionType | `credit` or `debit` |
| `amount` | decimal | Amount in fiat currency |
| `credit_amount` | decimal | Amount in credits |
| `credit_balance_before/after` | decimal | Balance snapshots |
| `tx_status` | TransactionStatus | Transaction state |
| `reference_type` | WalletTxReferenceType | What triggered this transaction |
| `reference_id` | string | Reference entity ID |
| `description` | string | Human-readable description |
| `expiry_date` | *time.Time | Credit expiration |
| `credits_available` | decimal | Remaining available credits |
| `transaction_reason` | TransactionReason | Reason classification |
| `priority` | *int | Deduction priority |
| `idempotency_key` | string | Deduplication key |
| `conversion_rate` | *decimal | Rate at time of transaction |
| `topup_conversion_rate` | *decimal | Topup rate at time of transaction |

**ComputeCreditsAvailable**: For credits, if balance was negative before, available = max(0, balance_after); otherwise available = credit_amount. Debits always = 0.

### CreditGrant

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `name` | string | Grant name |
| `scope` | CreditGrantScope | `PLAN` or `SUBSCRIPTION` |
| `plan_id` | *string | Required for PLAN scope |
| `subscription_id` | *string | Required for SUBSCRIPTION scope |
| `credits` | decimal | Credit amount (must be >0) |
| `cadence` | CreditGrantCadence | `ONETIME` or `RECURRING` |
| `period` | *CreditGrantPeriod | Required for RECURRING (e.g., MONTHLY, YEARLY) |
| `period_count` | *int | Period multiplier |
| `expiration_type` | CreditGrantExpiryType | How credits expire |
| `expiration_duration` | *int | Expiry duration |
| `expiration_duration_unit` | *CreditGrantExpiryDurationUnit | Expiry unit |
| `priority` | *int | Credit consumption order |
| `start_date` | *time.Time | Grant start (required for SUBSCRIPTION scope) |
| `end_date` | *time.Time | Grant end |
| `credit_grant_anchor` | *time.Time | Anchor for recurring grants (must be ≥ start_date) |
| `conversion_rate` | *decimal | Credits → fiat rate |
| `topup_conversion_rate` | *decimal | Topup rate |

**CopyWith**: Functional clone with selective overrides for grant versioning.

### WalletBalanceAlertEvent
Async event published for wallet balance checks.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | Event ID |
| `customer_id` | string | Customer |
| `wallet_id` | string | Specific wallet (optional) |
| `force_calculate_balance` | bool | Force fresh calculation |
| `get_from_cache` | bool | Use cached balance (max 1 min old) |
| `source` | string | Trigger source (wallet_credit, wallet_debit, manual, cron) |

### Export Data Types
- **CreditTopupsExportData**: Topup reporting (topup ID, external ID, customer name, amounts, reason)
- **CreditUsageExportData**: Usage reporting (customer, current+realtime balance, wallet count)

## Key Design Patterns
1. **Dual Balance**: Credits (internal unit) + fiat currency (external unit) with conversion rates
2. **Separate Topup Rate**: Different conversion rates for topup vs usage
3. **Balance Invariant**: `balance == credit_balance × conversion_rate` (enforced by validation)
4. **Credit Expiration**: Transaction-level expiry dates with available credits tracking
5. **Priority Deduction**: Credits consumed in priority order
6. **Idempotency**: Transaction deduplication via idempotency_key
7. **Alert System**: Multi-level threshold alerts (critical/warning/info) with cached balance checks
8. **Auto-Topup**: Configurable automatic replenishment
9. **Scoped Credit Grants**: Plan-level (catalog) vs subscription-level (instance) grants with recurring/onetime cadence
