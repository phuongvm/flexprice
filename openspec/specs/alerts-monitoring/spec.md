# Alerts & Monitoring

> **Source**: `internal/domain/alertlogs/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Alert logging and monitoring for entity state changes — tracks balance alerts, usage thresholds, and system events.

## Domain Model

### AlertLog

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `entity_type` | AlertEntityType | What entity triggered the alert |
| `entity_id` | string | Entity ID |
| `parent_entity_type` | *string | Parent entity type (optional) |
| `parent_entity_id` | *string | Parent entity ID (optional) |
| `customer_id` | *string | Related customer (optional) |
| `alert_type` | AlertType | Alert classification |
| `alert_status` | AlertState | Alert state |
| `alert_info` | AlertInfo | Additional alert details |

## Integration Points
- **Wallet Alerts**: Balance threshold alerts (critical/warning/info)
- **Feature Alerts**: Usage threshold alerts per feature
- **Balance Checks**: Triggered by wallet credits/debits, manual triggers, or cron

## Key Design Patterns
1. **Entity-Agnostic**: Alert system works across wallets, features, and other entities
2. **Parent-Child Tracking**: Alerts link to parent entities for context
3. **Multi-Level Severity**: Critical/Warning/Info threshold support
4. **Event-Driven**: Alerts triggered by async events (Kafka)
