# Feature Management

> **Source**: `internal/domain/feature/`, `internal/domain/entitlement/` (if exists)
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Feature catalog linking meters to billable features with units, reporting conversions, grouping, and alert thresholds.

## Domain Model

### Feature

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID |
| `name` | string | Display name |
| `lookup_key` | string | External lookup identifier |
| `description` | string | Feature description |
| `meter_id` | string | Linked meter for usage tracking |
| `type` | FeatureType | Feature classification |
| `unit_singular` | string | Display unit (e.g., "API Call") |
| `unit_plural` | string | Plural unit (e.g., "API Calls") |
| `reporting_unit` | *ReportingUnit | Alternative display unit with conversion |
| `alert_settings` | *AlertSettings | Usage alert thresholds (critical/info) |
| `group_id` | string | Feature group association |
| `group` | *Group | Populated by service layer (not persisted) |
| `metadata` | Metadata | Arbitrary key-value data |

### ReportingUnit
Converts base meter units to customer-facing display units.

| Field | Type | Description |
|-------|------|-------------|
| `unit_singular` | string | Display unit name |
| `unit_plural` | string | Plural display unit |
| `conversion_rate` | *decimal | Base unit → reporting unit conversion |

**Formula**: `reporting_value = unit_value / conversion_rate` (rounded to 2 decimal places)

## Key Design Patterns
1. **Meter Binding**: Features link to meters for usage tracking
2. **Unit Conversion**: Reporting units allow displaying usage in customer-friendly terms (e.g., "minutes" instead of raw "seconds")
3. **Feature Groups**: Organize related features via `group_id`
4. **Alert Thresholds**: Multi-level usage alerts (critical/info) per feature
