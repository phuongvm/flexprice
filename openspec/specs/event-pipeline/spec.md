# Event Pipeline

> **Source**: `internal/service/event.go`, `internal/domain/events/`, Kafka/ClickHouse infrastructure
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Async event ingestion and processing infrastructure: events → Kafka → ClickHouse, with multi-tenant routing, deduplication, and monitoring.

## Architecture

```
API → Event Service → Kafka Publisher (fire-and-forget)
                         ↓
              Kafka Consumer (per-tenant or shared)
                         ↓
              Event Processing → ClickHouse (ProcessedEvent)
                         ↓
              Post-Processing → Feature Usage / Cost Calculation
```

### Key Components
1. **Publisher**: Events published to Kafka topics (never blocks API response)
2. **Multi-Tenant Routing**: "Lazy" tenants get separate Kafka topics/consumer groups for isolation (`config.Kafka.RouteTenantsOnLazyMode`)
3. **Event Consumption**: Kafka consumer processes raw events → enriches with billing context → stores as ProcessedEvents
4. **Post-Processing**: Feature usage calculation, cost sheet association
5. **Deduplication**: ClickHouse ReplacingMergeTree with `version` + `sign` fields
6. **Monitoring**: Sentry spans for Kafka consumer lag tracking

### Storage
- **ClickHouse**: Primary event storage (events, processed_events, raw_events)
- **ReplacingMergeTree**: Mutable event data with version-based dedup
- **CollapsingMergeTree**: Sign-based row deletion for corrections

## Key Design Patterns
1. **Fire-and-Forget**: API never blocks on event processing
2. **Tenant Isolation**: Configurable per-tenant Kafka routing
3. **Lag Monitoring**: Consumer lag tracked via Sentry
4. **Batch Processing**: Bulk event operations with worker pools
