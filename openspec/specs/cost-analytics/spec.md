# Cost Analytics

> **Source**: `internal/domain/costsheet/`, `internal/service/costsheet*.go`, `internal/service/revenue_analytics.go`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Cost tracking and revenue analytics — maps internal costs to customer-facing prices for margin analysis.

## Domain Model

### CostSheet
Links internal cost rates to meters/features for margin calculations.

### Revenue Analytics
Aggregated revenue reporting across subscriptions, invoices, and usage.

## Key Capabilities
1. **Cost-to-Revenue Mapping**: Internal costs vs customer-facing prices
2. **Margin Analysis**: Per-customer and per-feature profitability
3. **Time-Series Analytics**: Revenue trends over billing periods
4. **Usage-Cost Correlation**: Correlate metered usage with cost and revenue
