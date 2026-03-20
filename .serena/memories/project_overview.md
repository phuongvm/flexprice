# Flexprice - Project Overview

## Purpose
Flexprice is an open-source monetization infrastructure for AI-native and SaaS companies. It provides usage-based, credit-based, and hybrid pricing with real-time metering and reporting.

## Key Features
- **Real-time metering** — process usage events at scale
- **Flexible pricing models** — usage-based, seat-based, credit bundles, hybrid, free tiers with overage
- **Open-source & self-hostable** — full transparency, no vendor lock-in
- **API-first / SDK-first** — Go, Python, JavaScript SDKs
- **Integrations** — Stripe, Chargebee, CRM, CPQ, accounting tools

## Tech Stack
- **Language**: Go 1.23 (primary backend)
- **ORM**: EntGo (`entgo.io/ent`)
- **Databases**: PostgreSQL (via Ent), ClickHouse (analytics/metering)
- **Event streaming**: Apache Kafka (`Shopify/sarama`, `watermill`)
- **Workflow engine**: Temporal
- **Cloud**: AWS (DynamoDB, S3, lambda)
- **Container**: Docker, Docker Compose
- **API spec**: OpenAPI (`openapitools.json`, `.speakeasy/`)
- **License**: Open-source

## Repository Structure
```
cmd/              — CLI entry points
internal/         — core business logic
api/              — API handlers and routing  
ent/              — EntGo schema definitions (ORM)
migrations/       — database migrations
lambda/           — AWS Lambda functions
scripts/          — utility scripts
docs/             — documentation
main.go           — application entry point
Makefile          — build and development commands
docker-compose.yml — local dev environment
```

## SDKs
- Go: https://pkg.go.dev/github.com/flexprice/go-sdk
- Python: https://pypi.org/project/flexprice
- JavaScript: https://www.npmjs.com/package/@flexprice/sdk
