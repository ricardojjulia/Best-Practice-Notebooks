# ORGNZ-02 LAB: Understanding Grail Buckets - Hands-on Exercises

> **Series:** ORGNZ — Organize Data: Buckets, Segments, Security | **Notebook:** 2 of 10 | **Type:** LAB | **Created:** February 2026 | **Last Updated:** 04/25/2026

## Overview

This lab notebook contains 5 hands-on exercises extracted from **ORGNZ-02: Understanding Grail Buckets**. Complete the lecture notebook first, then work through these exercises to reinforce the concepts with real DQL queries against your Dynatrace environment.

---

## Table of Contents

1. [Exercise 1: Bucket Administration Permissions](#exercise-1)
2. [Exercise 2: Bucket Administration Permissions](#exercise-2)
3. [Exercise 3: Bucket Administration Permissions](#exercise-3)
4. [Exercise 4: Querying Data from Buckets](#exercise-4)
5. [Exercise 5: Querying Data from Buckets](#exercise-5)
6. [Lab Summary](#lab-summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Completed** | ORGNZ-02: Understanding Grail Buckets (lecture notebook) |
| **Dynatrace Environment** | SaaS tenant with Grail enabled |
| **Permissions** | `logs.read`, `metrics.read`, `entities.read`, `spans.read` |

<a id="exercise-1"></a>
## Exercise 1: Bucket Administration Permissions

# ORGNZ-02: Understanding Grail Buckets

> **Series:** ORGNZ — Organize Data: Buckets, Segments, Security | **Notebook:** 2 of 10 | **Created:** January 2026 | **Last Updated:** 04/25/2026


**Buckets** are logical storage containers in Dynatrace Grail where records are stored. Think of a bucket as a folder in a filesystem. Use buckets to group telemetry data that naturally belongs together, such as data from the same region, environment, or with the same sensitivity classification.


| Requirement | Details |
|---

---


1. What Are Buckets?
2. Supported Data Types (Tables)
3. Bucket Limits
4. Query Constraints
5. Default Buckets
6. Querying Bucket Information
7. Querying Data from Buckets
8. Bucket Characteristics
9. Bucket Naming Rules
10. When to Create Custom Buckets

---

-------------|----------|
| **Dynatrace Environment** | SaaS environment with Grail enabled |
| **Permissions** | `storage:buckets:read` for viewing, `storage:bucket-definitions:write` for creating |
| **Knowledge** | Completed ORGNZ-01 |


| Permission | Purpose |
|------------|----------|
| `storage:bucket-definitions:read` | View bucket configurations |
| `storage:bucket-definitions:write` | Create and modify buckets |
| `storage:bucket-definitions:delete` | Remove buckets |
| `storage:bucket-definitions:truncate` | Truncate bucket data |


By the end of this notebook, you will:
- Understand bucket fundamentals and supported data types
- Know bucket limits and query constraints
- Query bucket information using DQL
- Recognize when custom buckets are needed

Buckets organize data to meet performance, compliance, and retention requirements:

| Function | Description |
|----------|-------------|
| **Data Grouping** | Organize large datasets into meaningful segments |
| **Streamlined Organization** | Records that should be handled together are stored together |
| **Query Performance** | Reduced queried dataset improves query speed |
| **Retention Management** | Each bucket has its own retention policy |
| **Access Control** | Bucket-level IAM policies for team isolation |
| **Cost Attribution** | Enable data cost tracking by organizational unit |

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


| Limit | Value | Notes |
|-------|-------|-------|
| Maximum buckets per environment | 80 | Default limit; can request increase |
| Typical capacity | Up to 5 TB/day per table | With default bucket limit |


| Metric | Value | Impact |
|--------|-------|--------|
| Optimal ingest per bucket | ~1 TB/day | Best query performance |
| Acceptable ingest | 1-3 TB/day | Limited query window |
| Maximum ingest | 3 TB/day | Performance degrades above this |


| Limit | Value |
|-------|-------|
| Minimum retention | 1 day |
| Maximum retention | 3,657 days (~10 years + 1 week) |

Understanding query limits is critical for bucket planning:


| Limit | Value | Impact |
|-------|-------|--------|
| Maximum data scanned | 500 GB | Limits queryable time window |
| Maximum records returned | 1,000 | Use aggregations for larger datasets |
| Maximum response payload | 1 MB | Large result sets may be truncated |


The 500 GB scan limit determines how far back you can query:

| Daily Ingest | Queryable Window |
|--------------|------------------|
| 500 GB/day | ~24 hours |
| 1 TB/day | ~12 hours |
| 3 TB/day | ~4 hours |
| 10 TB/day | ~1 hour |

> **Implication**: High-volume buckets may only allow querying recent data. Use aggregations, time filters, and strategic bucket partitioning to work within these limits.

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

```dql
// List all buckets with retention and status
fetch dt.system.buckets
| fields name, display_name, dt.system.table, retention_days, estimated_uncompressed_bytes
| sort dt.system.table asc, name asc
```

<a id="exercise-2"></a>
## Exercise 2: Bucket Administration Permissions

```dql
// Query logs from default bucket (implicit)
fetch logs, from:-1h
| limit 10
```

<a id="exercise-3"></a>
## Exercise 3: Bucket Administration Permissions

```dql
// Query logs from a specific custom bucket
fetch logs, from:-1h, bucket:{"default_logs"}
| limit 10
```

<a id="exercise-4"></a>
## Exercise 4: Querying Data from Buckets

```dql
// Query from multiple buckets simultaneously
fetch logs, from:-1h, bucket:{"default_logs", "audit_logs", "security_logs"}
| limit 100
```

<a id="exercise-5"></a>
## Exercise 5: Querying Data from Buckets

```dql
// Filter by bucket within query
fetch logs, from:-1h
| filter dt.system.bucket == "default_logs"
| filter loglevel == "ERROR"
| limit 50
```

<a id="lab-summary"></a>
## Lab Summary

You have completed 5 hands-on exercises for **ORGNZ-02: Understanding Grail Buckets**.

### Exercises Completed

- [ ] Exercise 1: Bucket Administration Permissions
- [ ] Exercise 2: Bucket Administration Permissions
- [ ] Exercise 3: Bucket Administration Permissions
- [ ] Exercise 4: Querying Data from Buckets
- [ ] Exercise 5: Querying Data from Buckets

### Next Steps

Continue with **ORGNZ-03** for the next notebook.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
