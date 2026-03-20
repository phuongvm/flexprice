# Flexprice - Task Completion Checklist

When completing a task in Flexprice:

1. **EntGo**: if schema changed, run `go generate ./ent/...` to regenerate ORM code
2. **Build**: `go build -o flexprice .` — must succeed without errors
3. **Tests**: `go test ./...` — all tests pass
4. **Migrations**: if DB schema changed, add migration file to `migrations/`
5. **OpenAPI**: if API changed, update spec via Speakeasy
6. **Docker**: `docker-compose up -d` for local integration testing
