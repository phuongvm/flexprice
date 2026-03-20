# Usage Metering

> **Source**: `internal/domain/events/`, `internal/domain/meter/`, `internal/service/event.go`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Real-time event ingestion, aggregation, and usage tracking. Foundation for all billing calculations.

## Domain Model

### Event
Core event structure stored in ClickHouse.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID with `evt_` prefix |
| `tenant_id` | string | Multi-tenant isolation |
| `environment_id` | string | Environment scoping |
| `event_name` | string | Event type identifier, used for meter matching |
| `properties` | map[string]interface{} | Arbitrary key-value properties for filtering & aggregation |
| `source` | string | Origin of the event |
| `timestamp` | time.Time | Event occurrence time (UTC) |
| `ingested_at` | time.Time | Server-side ingestion time (auto-set by ClickHouse) |
| `customer_id` | string | Internal customer ID |
| `external_customer_id` | string | External system customer ID |

**Validation**: At least one of `customer_id` or `external_customer_id` is required.

### ProcessedEvent
Billing-enriched event after processing pipeline.

| Field | Type | Description |
|-------|------|-------------|
| (all Event fields) | — | Inherits from Event |
| `subscription_id` | string | Linked subscription |
| `sub_line_item_id` | string | Specific subscription line item |
| `price_id` | string | Price used for calculation |
| `feature_id` | string | Feature being tracked |
| `meter_id` | string | Meter that matched |
| `period_id` | uint64 | Billing period identifier |
| `currency` | string | Billing currency |
| `unique_hash` | string | Deduplication key |
| `qty_total` | decimal | Total quantity |
| `qty_billable` | decimal | Billable quantity (after free units) |
| `qty_free_applied` | decimal | Free units consumed |
| `tier_snapshot` | decimal | Tier position at time of processing |
| `unit_cost` | decimal | Per-unit cost |
| `cost` | decimal | Total cost for this event |
| `version` | uint64 | ClickHouse ReplacingMergeTree version |
| `sign` | int8 | ClickHouse CollapsingMergeTree sign |
| `final_lag_ms` | uint32 | Processing latency |

### RawEvent
Raw event with indexed fields for fast unprocessed discovery.

| Field | Type | Description |
|-------|------|-------------|
| `field1`–`field10` | *string | Pre-indexed property fields |
| `payload` | string | Full event payload as string |

### FeatureUsage & CostUsage
Specialized processed events for feature tracking and cost sheet association.

### Meter
Defines how events are aggregated for billing.

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID with `mtr_` prefix |
| `event_name` | string | Binds to events by name. Multiple meters can track same event |
| `name` | string | Display name |
| `aggregation.type` | AggregationType | One of: COUNT, SUM, AVG, COUNT_UNIQUE, LATEST, SUM_WITH_MULTIPLIER, MAX, WEIGHTED_SUM |
| `aggregation.field` | string | Property key for aggregation (e.g., `duration_ms`) |
| `aggregation.expression` | string | CEL expression replacing field-based extraction (e.g., `token * duration`) |
| `aggregation.multiplier` | *decimal | Scale factor (required for SUM_WITH_MULTIPLIER, must be >0) |
| `aggregation.bucket_size` | WindowSize | Time window for bucketed MAX/SUM aggregation |
| `aggregation.group_by` | string | Property to group by within buckets (MAX with bucket_size only) |
| `filters` | []Filter | Pre-aggregation filters on event.properties |
| `reset_usage` | ResetUsage | `billing_period` (reset each cycle) or `never` (cumulative, e.g. storage) |

**Filter**: `{ key: string, values: []string }` — matches events where `properties[key]` is in `values`.

## Repositories

### Repository (Raw Events — ClickHouse)
- `InsertEvent` / `BulkInsertEvents` — direct ClickHouse writes
- `GetUsage` — aggregated usage with filters, window size, billing anchor
- `GetUsageWithFilters` — prioritized filter groups for multi-price dedup
- `GetEvents` — paginated raw event query (cursor + offset)
- `FindUnprocessedEvents` — for reprocessing pipeline
- `GetTotalEventCount` — monitoring with optional windowed time-series

### ProcessedEventRepository (Billing Events — ClickHouse)
- `InsertProcessedEvent` / `BulkInsertProcessedEvents`
- `IsDuplicate` — unique_hash dedup check
- `GetLineItemUsage` — qty + free units for a subscription line item in period
- `GetPeriodCost` — total cost for customer/subscription in billing period
- `GetPeriodFeatureTotals` — per-feature usage totals for invoicing
- `GetDetailedUsageAnalytics` — comprehensive analytics with filtering, grouping, time-series

### RawEventRepository
- `FindRawEvents` — with keyset pagination
- `FindUnprocessedRawEvents` — ANTI JOIN with feature_usage for discovery

## Service Layer

### Event Ingestion
- **CreateEvent**: Validate → publish to Kafka (fire-and-forget, log error but never fail request)
- **BulkCreateEvents**: Sequential publish per event

### Usage Queries
- **GetUsage**: Direct ClickHouse aggregation query via repository
- **GetUsageByMeter**: Resolves meter → builds usage params from meter config (aggregation type, field, multiplier, bucket_size, group_by) → for `never`-reset meters, combines historic (all-time before period) + current period
- **BulkGetUsageByMeter**: Parallel processing with goroutine pool (3 workers, batches of 10, 10s per-meter timeout, 5ms sleep between batches). Partial failure tolerated — returns partial results + error count
- **GetUsageByMeterWithFilters**: Prioritized filter groups with deterministic ordering (deprecated — moving to 1 meter per price)

### Multi-Tenant Kafka Routing
- "Lazy" tenants route to separate Kafka topics/consumer groups for isolation
- Configured via `config.Kafka.RouteTenantsOnLazyMode`
- Affects both event consumption and post-processing pipelines

### Monitoring
- **MonitorKafkaLag**: Creates Sentry monitoring spans for event consumption + post-processing lag

## Key Design Patterns
1. **ClickHouse ReplacingMergeTree**: `version` + `sign` fields for mutable event data
2. **Fire-and-Forget Ingestion**: Events published to Kafka, never block API response
3. **Keyset Pagination**: Efficient cursor-based pagination for high-volume event queries
4. **Billing Anchor**: Custom monthly periods (e.g., 5th-to-5th) for non-calendar billing cycles
5. **CEL Expressions**: Computed quantity from multiple properties (e.g., `token * duration * pixel`)
