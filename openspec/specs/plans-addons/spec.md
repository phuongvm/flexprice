# Plans & Addons

> **Source**: `internal/domain/plan/`, `internal/domain/addon/`, `internal/types/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Product catalog structure — Plans define subscription packages, Addons extend them with additional features. Both serve as containers for Prices.

## Domain Model

### Plan

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID identifier |
| `name` | string | Display name |
| `lookup_key` | string | External lookup identifier |
| `description` | string | Plan description |
| `environment_id` | string | Environment scoping |
| `metadata` | types.Metadata | Arbitrary key-value metadata |
| `display_order` | *int | Ordering for display (optional) |
| `BaseModel` | embedded | tenant_id, status, timestamps, created_by/updated_by |

### Addon

| Field | Type | Description |
|-------|------|-------------|
| `id` | string | UUID identifier |
| `environment_id` | string | Environment scoping |
| `lookup_key` | string | External lookup identifier |
| `name` | string | Display name |
| `description` | string | Addon description |
| `type` | AddonType | Addon classification type |
| `metadata` | map[string]interface{} | Arbitrary metadata |
| `BaseModel` | embedded | tenant_id, status, timestamps, created_by/updated_by |

## Repositories

### Plan Repository
- `Create` / `Get` / `Update` / `Delete` — standard CRUD
- `List` / `ListAll` / `Count` — filtered queries via `PlanFilter`
- `GetByLookupKey` — external system lookup

### Addon Repository
- `Create` / `GetByID` / `Update` / `Delete` — standard CRUD
- `GetByLookupKey` — external system lookup
- `List` / `Count` — filtered queries via `AddonFilter`

## Relationships
- **Plan → Prices**: One-to-many via `price.entity_type = PLAN` and `price.entity_id = plan.id`
- **Addon → Prices**: One-to-many via `price.entity_type = ADDON` and `price.entity_id = addon.id`
- Plans and Addons are **catalog entities** — they define what can be subscribed to
- Prices are scoped to either a Plan or Addon (never standalone)

## Key Design Patterns
1. **Lightweight Catalog**: Plans and Addons are simple containers; pricing complexity lives in the Price model
2. **LookupKey**: Both support external system lookups for integration
3. **DisplayOrder**: Plans support explicit ordering for UI presentation
4. **Environment Scoping**: Both are scoped to environments for multi-environment support
