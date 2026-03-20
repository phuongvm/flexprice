# Workflow Orchestration

> **Source**: `internal/domain/scheduledtask/`, `internal/domain/workflowexecution/`, `internal/domain/task/`
> **Generated**: 2026-03-20 (reverse-engineered from FlexPrice source code)

## Purpose

Temporal-based workflow execution for scheduled billing operations, subscription lifecycle events, and background task management.

## Domain Model

### ScheduledTask
Defines recurring or one-time scheduled operations.

### WorkflowExecution
Tracks Temporal workflow execution state and results.

### Task
Generic task entity for background operations.

## Architecture
- **Temporal**: Workflow orchestration engine for reliable long-running processes
- **Use Cases**: Subscription renewals, invoice generation, payment retries, usage aggregation
- **Reliability**: Temporal handles retries, timeouts, and failure recovery

## Key Design Patterns
1. **Temporal Workflows**: Durable, reliable execution for billing operations
2. **Scheduled Tasks**: Cron-like scheduling for recurring operations
3. **Execution Tracking**: Full workflow execution history
