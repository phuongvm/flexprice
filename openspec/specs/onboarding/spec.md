# Onboarding

> **Source**: `internal/service/onboarding.go`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Guided onboarding service for new tenant setup — creates initial entities (meters, features, plans, prices) from configuration.

## Key Capabilities
1. **Templated Setup**: Pre-configured onboarding templates for common billing models
2. **Entity Creation**: Automated creation of meters → features → plans → prices pipeline
3. **Quick Start**: Reduces time-to-first-invoice for new tenants
