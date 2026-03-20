# Flexprice - Suggested Commands

## Local Development
```powershell
# Start all services
docker-compose up -d

# Run the app
go run main.go
```

## Build
```powershell
make build             # Build binary (see Makefile for targets)
go build -o flexprice . # Manual build
```

## Database / EntGo
```powershell
go generate ./ent/...         # Regenerate EntGo code from schema
# Migrations: check migrations/ directory
```

## Testing
```powershell
go test ./...          # Run all tests
go test ./internal/... # Test specific package
```

## Linting
```powershell
golangci-lint run      # Run linter (if installed)
```

## AWS Lambda deploy
```powershell
# Check scripts/ and lambda/ directories for deploy scripts
```
