# DBMON-02: SQL Database Monitoring

> **Series:** DBMON — Database Monitoring | **Notebook:** 2 of 7 | **Created:** March 2026 | **Last Updated:** 04/25/2026

## Overview

This notebook focuses on monitoring relational SQL databases with Dynatrace. You will learn how to analyze query performance for PostgreSQL, MySQL, Microsoft SQL Server, and Oracle databases. We cover slow query detection, operation-level breakdowns, connection monitoring, and response time analysis using distributed trace spans.

---

## Table of Contents

1. [SQL Database Landscape](#sql-database-landscape)
2. [Query Performance Analysis](#query-performance-analysis)
3. [Slow Query Detection](#slow-query-detection)
4. [Operation Breakdown by Table](#operation-breakdown-by-table)
5. [Connection Pool Monitoring](#connection-pool-monitoring)
6. [Database-Specific Patterns](#database-specific-patterns)
7. [Response Time Distribution](#response-time-distribution)
8. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace Environment** | SaaS or Managed with Grail enabled |
| **OneAgent** | Deployed on application hosts with SQL database clients |
| **Permissions** | `storage:spans:read`, `storage:entities:read` |
| **Data** | Application traffic generating SQL database calls (PostgreSQL, MySQL, MS SQL, or Oracle) |
| **Prior Reading** | DBMON-01: Database Monitoring Fundamentals |

<a id="sql-database-landscape"></a>

## 1. SQL Database Landscape

Relational databases share a common monitoring model: they all execute SQL statements against tables in schemas. Dynatrace captures these calls consistently across vendors using the OpenTelemetry `db.*` span attributes.

| Vendor | `db.system` Value | Common Operations | Default Port |
|--------|-------------------|-------------------|--------------|
| PostgreSQL | `postgresql` | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `COPY` | 5432 |
| MySQL / MariaDB | `mysql` | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CALL` | 3306 |
| Microsoft SQL Server | `mssql` | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `EXEC` | 1433 |
| Oracle | `oracle` | `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `MERGE` | 1521 |
| IBM Db2 | `db2` | `SELECT`, `INSERT`, `UPDATE`, `DELETE` | 50000 |

Let's start by identifying which SQL databases are active in your environment.

```dql
// Discover active SQL databases in the environment
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| summarize call_count = count(),
           avg_duration_ms = avg(duration) / 1000000.0,
           p95_duration_ms = percentile(duration, 95) / 1000000.0,
           unique_queries = countDistinct(db.statement),
           by:{db.system, db.namespace, server.address}
| sort call_count desc
```

<a id="query-performance-analysis"></a>

## 2. Query Performance Analysis

The most important aspect of SQL database monitoring is understanding query performance. Dynatrace normalizes SQL statements by replacing literal values with `?` placeholders, allowing you to group identical query patterns together.

```dql
// Top 20 SQL queries by total execution time (highest impact)
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| filter isNotNull(db.statement)
| summarize total_time_ms = sum(duration) / 1000000.0,
           call_count = count(),
           avg_ms = avg(duration) / 1000000.0,
           p95_ms = percentile(duration, 95) / 1000000.0,
           by:{db.system, db.statement}
| sort total_time_ms desc
| limit 20
```

```dql
// Query throughput over time by database system
fetch spans, from:-6h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| makeTimeseries queries_per_min = count(), by:{db.system}, interval:5m
```

<a id="slow-query-detection"></a>

## 3. Slow Query Detection

Slow queries are the most common cause of database-related performance problems. A query is considered "slow" relative to its own baseline or an absolute threshold. We use both approaches below.

```dql
// Detect slow queries — calls exceeding 500ms
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| filter duration > 500000000
| fields timestamp, db.system, db.namespace, db.operation,
        db.statement, server.address,
        duration_ms = duration / 1000000.0,
        dt.entity.service
| sort duration_ms desc
| limit 25
```

```dql
// Slow query frequency over time — how often do queries exceed 500ms?
fetch spans, from:-6h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| makeTimeseries total = count(),
                 slow = countIf(duration > 500000000),
                 interval:10m
```

```dql
// Identify query patterns with the highest P95 — potential optimization candidates
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| filter isNotNull(db.statement)
| summarize call_count = count(),
           avg_ms = avg(duration) / 1000000.0,
           p95_ms = percentile(duration, 95) / 1000000.0,
           max_ms = max(duration) / 1000000.0,
           by:{db.statement, db.system}
| filter call_count >= 10
| sort p95_ms desc
| limit 15
```

<a id="operation-breakdown-by-table"></a>

## 4. Operation Breakdown by Table

Understanding which tables receive the most read and write traffic helps identify hot spots. We parse the table name from the normalized SQL statement to group by table.

```dql
// Operation mix by database — read vs write ratio
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| filter isNotNull(db.operation)
| fieldsAdd op_type = if(db.operation == "SELECT", then:"READ", else:"WRITE")
| summarize op_count = count(), by:{db.system, db.namespace, op_type}
| sort db.system asc, op_count desc
```

```dql
// Average response time by operation type — are writes slower than reads?
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| filter isNotNull(db.operation)
| summarize call_count = count(),
           avg_ms = avg(duration) / 1000000.0,
           p95_ms = percentile(duration, 95) / 1000000.0,
           by:{db.operation, db.system}
| sort db.system asc, avg_ms desc
```

<a id="connection-pool-monitoring"></a>

## 5. Connection Pool Monitoring

Connection pool exhaustion is a common source of application errors. While Dynatrace does not directly expose connection pool counters through spans, you can infer connection pressure by analyzing concurrent database calls and error patterns.

```dql
// Database calls per service — identify which services make the most DB calls
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| summarize call_count = count(),
           avg_ms = avg(duration) / 1000000.0,
           error_count = countIf(otel.status_code == "ERROR"),
           by:{dt.entity.service, db.system, server.address}
| fieldsAdd service_name = entityName(dt.entity.service, type:"dt.entity.service")
| sort call_count desc
| limit 20
```

```dql
// Database error patterns — connection refused, timeout, deadlock
fetch spans, from:-6h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| filter otel.status_code == "ERROR"
| summarize error_count = count(),
           by:{db.system, server.address, otel.status_message}
| sort error_count desc
| limit 20
```

<a id="database-specific-patterns"></a>

## 6. Database-Specific Patterns

Each SQL database has unique characteristics worth monitoring. The following queries target vendor-specific patterns.

### PostgreSQL

PostgreSQL is commonly used in cloud-native applications. Key concerns: vacuum operations, index bloat, and lock contention.

```dql
// PostgreSQL — query performance breakdown by database
fetch spans, from:-1h
| filter db.system == "postgresql"
| summarize call_count = count(),
           avg_ms = avg(duration) / 1000000.0,
           p95_ms = percentile(duration, 95) / 1000000.0,
           errors = countIf(otel.status_code == "ERROR"),
           by:{db.namespace, db.operation}
| sort call_count desc
```

### MySQL / MariaDB

MySQL monitoring focuses on query cache effectiveness, InnoDB buffer pool usage, and replication lag.

```dql
// MySQL — response time trend over 6 hours
fetch spans, from:-6h
| filter db.system == "mysql"
| makeTimeseries avg_ms = avg(duration) / 1000000.0,
                 p95_ms = percentile(duration, 95) / 1000000.0,
                 call_count = count(),
                 interval:10m
```

<a id="response-time-distribution"></a>

## 7. Response Time Distribution

Understanding the distribution of response times helps set realistic SLOs and identify bimodal patterns (e.g., cached vs uncached queries).

```dql
// Response time distribution buckets — group queries into latency tiers
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| fieldsAdd duration_ms = duration / 1000000.0
| fieldsAdd latency_tier = if(duration_ms < 1, then:"<1ms",
    else:if(duration_ms < 10, then:"1-10ms",
    else:if(duration_ms < 100, then:"10-100ms",
    else:if(duration_ms < 1000, then:"100ms-1s",
    else:">1s"))))
| summarize query_count = count(), by:{latency_tier}
| sort latency_tier asc
```

```dql
// Percentile summary across all SQL databases
fetch spans, from:-1h
| filter in(db.system, {"postgresql", "mysql", "mssql", "oracle", "db2"})
| summarize p50_ms = percentile(duration, 50) / 1000000.0,
           p90_ms = percentile(duration, 90) / 1000000.0,
           p95_ms = percentile(duration, 95) / 1000000.0,
           p99_ms = percentile(duration, 99) / 1000000.0,
           max_ms = max(duration) / 1000000.0,
           total_calls = count(),
           by:{db.system}
| sort total_calls desc
```

<a id="summary"></a>

## 8. Summary and Next Steps

In this notebook you learned:

- How to identify and inventory SQL databases in your environment using span data
- Techniques for finding the highest-impact and slowest queries
- How to break down database operations by type and table to identify hot spots
- Connection pressure and error pattern analysis
- Vendor-specific monitoring patterns for PostgreSQL and MySQL
- Response time distribution analysis for setting realistic SLOs

### Next Steps

- **DBMON-03: NoSQL Database Monitoring** — MongoDB, Cassandra, DynamoDB, and Cosmos DB analysis
- **DBMON-05: Query Analysis** — Deep dive into N+1 detection, query frequency patterns, and optimization

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
