# Flexprice - Style & Conventions

## Go Conventions
- Go 1.23, idiomatic Go patterns
- EntGo schema-first ORM (define schema in `ent/schema/`, generate with `go generate`)
- Repository pattern for data access (`internal/`)
- API-first: OpenAPI spec maintained via Speakeasy (`.speakeasy/`)

## Error Handling
- `cockroachdb/errors` for structured error wrapping

## Code Organization
- `cmd/` — entry points (separate binaries)
- `internal/` — private packages (domain logic, services, repositories)
- `api/` — HTTP handlers, middleware, routing
- `ent/` — generated ORM code (do not manually edit generated files)

## Docker / local dev
- `docker-compose.yml` for local environment (Postgres, ClickHouse, Kafka, Temporal)
