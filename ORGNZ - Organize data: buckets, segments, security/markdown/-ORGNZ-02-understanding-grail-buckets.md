# ORGNZ-02: Understanding Grail Buckets

> **Series:** ORGNZ | **Notebook:** 2 of 9 | **Created:** January 2026 | **Last Updated:** 01/28/2026

## Overview

**Buckets** are logical storage containers in Dynatrace Grail where records are stored. Think of a bucket as a folder in a filesystem. Use buckets to group telemetry data that naturally belongs together, such as data from the same region, environment, or with the same sensitivity classification.

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS environment with Grail enabled |
| **Permissions** | `storage:buckets:read` for viewing, `storage:bucket-definitions:write` for creating |
| **Knowledge** | Completed ORGNZ-01 |

### Bucket Administration Permissions

| Permission | Purpose |
|------------|----------|
| `storage:bucket-definitions:read` | View bucket configurations |
| `storage:bucket-definitions:write` | Create and modify buckets |
| `storage:bucket-definitions:delete` | Remove buckets |
| `storage:bucket-definitions:truncate` | Truncate bucket data |

## Learning Objectives

By the end of this notebook, you will:
- Understand bucket fundamentals and supported data types
- Know bucket limits and query constraints
- Query bucket information using DQL
- Recognize when custom buckets are needed

## What Are Buckets?

Buckets organize data to meet performance, compliance, and retention requirements:

| Function | Description |
|----------|-------------|
| **Data Grouping** | Organize large datasets into meaningful segments |
| **Streamlined Organization** | Records that should be handled together are stored together |
| **Query Performance** | Reduced queried dataset improves query speed |
| **Retention Management** | Each bucket has its own retention policy |
| **Access Control** | Bucket-level IAM policies for team isolation |
| **Cost Attribution** | Enable data cost tracking by organizational unit |

## Supported Data Types (Tables)

Each bucket is associated with exactly **one data type**:

| Data Type | Default Bucket | Description |
|-----------|----------------|-------------|
| **logs** | `default_logs` | Log records from all sources |
| **metrics** | `default_metrics` | Metric data points |
| **spans** | `default_spans` | Distributed trace spans |
| **events** | `default_events` | Platform events |
| **bizevents** | `default_bizevents` | Business events |
| **security_events** | `default_security_events` | Security-related events |

> **Important**: You cannot mix data types in the same bucket. A logs bucket only stores logs.

## Bucket Limits

### Environment Limits

| Limit | Value | Notes |
|-------|-------|-------|
| Maximum buckets per environment | 80 | Default limit; can request increase |
| Typical capacity | Up to 5 TB/day per table | With default bucket limit |

### Bucket Size Guidelines

| Metric | Value | Impact |
|--------|-------|--------|
| Optimal ingest per bucket | ~1 TB/day | Best query performance |
| Acceptable ingest | 1-3 TB/day | Limited query window |
| Maximum ingest | 3 TB/day | Performance degrades above this |

### Retention Limits

| Limit | Value |
|-------|-------|
| Minimum retention | 1 day |
| Maximum retention | 3,657 days (~10 years + 1 week) |

## Query Constraints

Understanding query limits is critical for bucket planning:

### Query Limits

| Limit | Value | Impact |
|-------|-------|--------|
| Maximum data scanned | 500 GB | Limits queryable time window |
| Maximum records returned | 1,000 | Use aggregations for larger datasets |
| Maximum response payload | 1 MB | Large result sets may be truncated |

### Queryable Window by Ingest Volume

The 500 GB scan limit determines how far back you can query:

| Daily Ingest | Queryable Window |
|--------------|------------------|
| 500 GB/day | ~24 hours |
| 1 TB/day | ~12 hours |
| 3 TB/day | ~4 hours |
| 10 TB/day | ~1 hour |

> **Implication**: High-volume buckets may only allow querying recent data. Use aggregations, time filters, and strategic bucket partitioning to work within these limits.

## Default Buckets

Dynatrace provides default buckets with varying retention:

| Default Bucket | Data Type | Default Retention | Notes |
|----------------|-----------|-------------------|-------|
| `default_logs` | logs | 35 days | Standard log retention |
| `default_metrics` | metrics | 15 months | Extended for trend analysis |
| `default_spans` | spans | 10 days | Short-term APM data |
| `default_events` | events | 35 days | Platform events |
| `default_bizevents` | bizevents | 35 days | Business events |
| `default_security_events` | security_events | 35 days | Security events |

> **Note**: Query your environment's `dt.system.buckets` to verify current retention settings.

## Querying Bucket Information

```dql
// List all buckets with retention and status
fetch dt.system.buckets
| fields name, display_name, dt.system.table, retention_days, estimated_uncompressed_bytes
| sort dt.system.table asc, name asc
```

```dql
// Get bucket details including display names
fetch dt.system.buckets
| fields bucket.name, bucket.displayName, bucket.table, bucket.retentionDays
| filter bucket.table == "logs"
| sort bucket.retentionDays desc
```

## Querying Data from Buckets

```dql
// Query logs from default bucket (implicit)
fetch logs
| limit 10
```

```dql
// Query logs from a specific custom bucket
fetch logs, from: "custom_logs_bucket"
| limit 10
```

```dql
// Query from multiple buckets simultaneously
fetch logs, from: {"default_logs", "audit_logs", "security_logs"}
| limit 100
```

```dql
// Filter by bucket within query
fetch logs
| filter dt.system.bucket == "prod_infra_logs"
| filter loglevel == "ERROR"
| limit 50
```

## Bucket Characteristics

### Immutability Rules

| Characteristic | Mutable? | Notes |
|----------------|----------|-------|
| Bucket name | No | Cannot be changed after creation |
| Display name | Yes | Can be updated anytime |
| Retention period | Yes | **Caution**: Lowering deletes data |
| Data table type | No | Fixed at creation |

### Data Handling

| Operation | Supported? | Alternative |
|-----------|------------|-------------|
| Move data between buckets | No | Route new data via OpenPipeline |
| Selective record deletion | No | Wait for retention to expire |
| Bucket deletion | Yes | Deletes ALL data permanently |

## Bucket Naming Rules

| Rule | Requirement |
|------|-------------|
| First character | Must be a letter |
| Allowed characters | Lowercase alphanumeric, underscores, hyphens |
| Case | Lowercase only |
| Reserved names | Cannot use `default_*` prefix |

### Valid Examples

```
team_logs_30d
finance-audit-logs
prod_metrics_90d
app123_spans
```

### Invalid Examples

```
123_logs          # Starts with number
Team_Logs         # Contains uppercase
default_custom    # Uses reserved prefix
my logs           # Contains space
```

## When to Create Custom Buckets

| Need | Solution |
|------|----------|
| Different retention periods | Create bucket with specific retention |
| Cost attribution by team/LOB | Separate buckets per cost center |
| Simple team isolation | Bucket-level IAM policies |
| Regulatory compliance | Dedicated buckets for compliance data |
| Query performance | Isolate high-volume data sources |
| Debug/verbose logs | Short-retention bucket to reduce costs |

## Next Steps

Continue with the ORGNZ series:
- **ORGNZ-03**: Bucket Strategy and Design

## References

- [Grail Buckets](https://docs.dynatrace.com/docs/platform/grail/data-management/buckets)
- [Data Retention](https://docs.dynatrace.com/docs/platform/grail/data-management/data-retention)
- [DQL Reference](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)

---

<sub>*This notebook was AI-generated from Dynatrace documentation and enterprise best practices. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
