# DBMON-06: Dashboards and Alerting

> **Series:** DBMON | **Notebook:** 6 of 6 | **Created:** March 2026 | **Last Updated:** 03/12/2026

## Overview

This notebook covers building database monitoring dashboards and configuring alerting for database health. You will learn dashboard design patterns for database KPIs, how to create alert-ready queries for slow query detection, connection pool thresholds, error rate monitoring, and database-specific SLO definitions. These queries are designed to be directly usable in Dynatrace dashboards and metric events.


---

## Table of Contents

1. [Database KPIs](#database-kpis)
2. [Health Overview Dashboard](#health-overview)
3. [Response Time Monitoring](#response-time-monitoring)
4. [Error Rate Alerting](#error-rate-alerting)
5. [Throughput and Capacity](#throughput-capacity)
6. [Slow Query Alerting](#slow-query-alerting)
7. [Database SLO Definitions](#database-slos)
8. [Summary](#summary)

---


## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Dynatrace Environment** | SaaS or Managed with Grail enabled |
| **OneAgent** | Deployed on application hosts with database clients |
| **Permissions** | `storage:spans:read`, `storage:metrics:read`, `storage:entities:read` |
| **Data** | Active database traffic across monitored services |
| **Prior Reading** | DBMON-01 through DBMON-05 for context on database monitoring techniques |


<a id="database-kpis"></a>

## 1. Database KPIs

Effective database monitoring revolves around a small set of key performance indicators. Each KPI maps to a specific dashboard tile and potential alert condition.

| KPI | What It Measures | Dashboard Tile | Alert Threshold |
|-----|-----------------|----------------|----------------|
| **Response Time (P95)** | Query latency at the 95th percentile | Timeseries chart | > 500ms sustained |
| **Error Rate** | Percentage of failed database calls | Single value + trend | > 1% of total calls |
| **Throughput** | Queries per second | Timeseries chart | Sudden drop > 50% |
| **Slow Query Count** | Queries exceeding threshold | Counter + list | > 10 per 5min window |
| **Connection Errors** | Timeout and refused connections | Counter | Any occurrence |
| **Top Queries by Time** | Highest-impact query patterns | Table | Total time > SLO budget |


<a id="health-overview"></a>

## 2. Health Overview Dashboard

The health overview provides a single-pane summary of all database systems. Use these queries as dashboard tiles for an executive-level view.


```dql
// Dashboard tile: Database health summary — one row per database system
fetch spans, from:-1h
| filter isNotNull(db.system)
| summarize total_calls = count(),
           avg_ms = avg(duration) / 1000000.0,
           p95_ms = percentile(duration, 95) / 1000000.0,
           error_count = countIf(otel.status_code == "ERROR"),
           slow_count = countIf(duration > 500000000),
           unique_databases = countDistinct(db.namespace),
           by:{db.system}
| fieldsAdd error_rate_pct = round((toDouble(error_count) / toDouble(total_calls)) * 100, decimals:2)
| fieldsAdd slow_rate_pct = round((toDouble(slow_count) / toDouble(total_calls)) * 100, decimals:2)
| sort total_calls desc
```

```dql
// Dashboard tile: Total database calls — single value
fetch spans, from:-1h
| filter isNotNull(db.system)
| summarize total_db_calls = count()
```

```dql
// Dashboard tile: Database calls per service — identify heaviest consumers
fetch spans, from:-1h
| filter isNotNull(db.system)
| summarize call_count = count(),
           avg_ms = avg(duration) / 1000000.0,
           error_count = countIf(otel.status_code == "ERROR"),
           by:{dt.entity.service}
| fieldsAdd service_name = entityName(dt.entity.service, type:"dt.entity.service")
| sort call_count desc
| limit 10
```

<a id="response-time-monitoring"></a>

## 3. Response Time Monitoring

Response time monitoring is the most critical aspect of database dashboards. These queries provide both real-time and trend views.


```dql
// Dashboard tile: Database response time trend by system (6-hour view)
fetch spans, from:-6h
| filter isNotNull(db.system)
| makeTimeseries p95_ms = percentile(duration, 95) / 1000000.0,
                 by:{db.system},
                 interval:5m
```

```dql
// Dashboard tile: Response time by database instance (server.address)
fetch spans, from:-1h
| filter isNotNull(db.system) and isNotNull(server.address)
| summarize call_count = count(),
           avg_ms = avg(duration) / 1000000.0,
           p95_ms = percentile(duration, 95) / 1000000.0,
           p99_ms = percentile(duration, 99) / 1000000.0,
           by:{db.system, server.address, db.namespace}
| sort p95_ms desc
| limit 15
```

```dql
// Dashboard tile: P50 vs P95 vs P99 comparison — all databases combined
fetch spans, from:-6h
| filter isNotNull(db.system)
| makeTimeseries p50_ms = percentile(duration, 50) / 1000000.0,
                 p95_ms = percentile(duration, 95) / 1000000.0,
                 p99_ms = percentile(duration, 99) / 1000000.0,
                 interval:5m
```

<a id="error-rate-alerting"></a>

## 4. Error Rate Alerting

Database errors (connection timeouts, deadlocks, constraint violations) should trigger alerts when they exceed normal baseline levels.


```dql
// Alert query: Database error rate per 5-minute window
fetch spans, from:-1h
| filter isNotNull(db.system)
| makeTimeseries total = count(),
                 errors = countIf(otel.status_code == "ERROR"),
                 by:{db.system},
                 interval:5m
```

```dql
// Alert query: Error breakdown by type — classify error categories
fetch spans, from:-1h
| filter isNotNull(db.system) and otel.status_code == "ERROR"
| summarize error_count = count(),
           by:{db.system, server.address, otel.status_message}
| sort error_count desc
| limit 20
```

```dql
// Alert query: Error rate trend over 24 hours — detect escalating problems
fetch spans, from:-24h
| filter isNotNull(db.system)
| makeTimeseries total = count(),
                 errors = countIf(otel.status_code == "ERROR"),
                 interval:30m
```

### Recommended Alert Thresholds

| Condition | Severity | Alert When |
|-----------|----------|------------|
| Error rate > 5% for 5 minutes | Critical | Immediate page |
| Error rate > 1% for 15 minutes | Warning | Notification |
| Any connection refused error | Critical | Immediate page |
| Deadlock detected | Warning | Notification |


<a id="throughput-capacity"></a>

## 5. Throughput and Capacity

Monitoring query throughput helps detect traffic anomalies and plan for capacity. A sudden drop in throughput may indicate an upstream failure, while a spike may indicate a runaway process.


```dql
// Dashboard tile: Queries per minute by database system
fetch spans, from:-6h
| filter isNotNull(db.system)
| makeTimeseries qpm = count(), by:{db.system}, interval:1m
```

```dql
// Dashboard tile: Operation mix over time — read vs write trend
fetch spans, from:-6h
| filter isNotNull(db.system) and isNotNull(db.operation)
| fieldsAdd op_type = if(
    in(db.operation, {"SELECT", "find", "Query", "GetItem", "ReadItem", "GET", "HGET", "get", "search"}),
    then:"READ",
    else:"WRITE")
| makeTimeseries op_count = count(), by:{op_type}, interval:5m
```

```dql
// Dashboard tile: Hourly query volume comparison — today vs yesterday
fetch spans, from:-24h
| filter isNotNull(db.system)
| summarize today_count = count()
| append [
    fetch spans, from:-48h, to:-24h
    | filter isNotNull(db.system)
    | summarize yesterday_count = count()
  ]
```

<a id="slow-query-alerting"></a>

## 6. Slow Query Alerting

Slow query alerts detect when query performance degrades beyond acceptable thresholds. Configure these as metric events in Dynatrace.


```dql
// Alert query: Slow query count per 5-minute window (> 500ms threshold)
fetch spans, from:-1h
| filter isNotNull(db.system)
| filter duration > 500000000
| makeTimeseries slow_count = count(), by:{db.system}, interval:5m
```

```dql
// Alert query: Current slow query detail — for investigation
fetch spans, from:-15m
| filter isNotNull(db.system)
| filter duration > 500000000
| fields timestamp, db.system, db.namespace, db.operation,
        db.statement, server.address,
        duration_ms = duration / 1000000.0,
        dt.entity.service
| fieldsAdd service_name = entityName(dt.entity.service, type:"dt.entity.service")
| sort duration_ms desc
| limit 20
```

```dql
// Alert query: P95 response time exceeding SLO threshold
fetch spans, from:-6h
| filter isNotNull(db.system)
| makeTimeseries p95_ms = percentile(duration, 95) / 1000000.0,
                 by:{db.system, server.address},
                 interval:5m
```

### Slow Query Alert Thresholds

| Database Type | Warning Threshold | Critical Threshold | Window |
|--------------|-------------------|-------------------|---------|
| SQL (PostgreSQL, MySQL, etc.) | P95 > 200ms | P95 > 500ms | 5 minutes |
| NoSQL (MongoDB, DynamoDB) | P95 > 100ms | P95 > 300ms | 5 minutes |
| Cache (Redis, Memcached) | P95 > 5ms | P95 > 20ms | 5 minutes |
| Search (Elasticsearch) | P95 > 500ms | P95 > 2000ms | 5 minutes |


<a id="database-slos"></a>

## 7. Database SLO Definitions

Service Level Objectives (SLOs) for databases define the expected performance contract. Use these queries to measure SLO compliance.

### Recommended Database SLOs

| SLO | Target | Measurement |
|-----|--------|-------------|
| **Availability** | 99.9% of calls succeed | Error rate < 0.1% |
| **Latency** | 95% of calls under threshold | P95 < vendor-specific threshold |
| **Throughput** | No more than 50% drop from baseline | Queries/min vs 7-day average |


```dql
// SLO measurement: Database availability — success rate over 24 hours
fetch spans, from:-24h
| filter isNotNull(db.system)
| summarize total = count(),
           successful = countIf(otel.status_code != "ERROR"),
           by:{db.system}
| fieldsAdd availability_pct = round((toDouble(successful) / toDouble(total)) * 100, decimals:3)
| sort availability_pct asc
```

```dql
// SLO measurement: Latency compliance — percentage of calls under threshold
fetch spans, from:-24h
| filter isNotNull(db.system)
| summarize total = count(),
           under_threshold = countIf(duration < 500000000),
           by:{db.system}
| fieldsAdd latency_slo_pct = round((toDouble(under_threshold) / toDouble(total)) * 100, decimals:2)
| sort latency_slo_pct asc
```

```dql
// SLO trend: Hourly availability over 24 hours
fetch spans, from:-24h
| filter isNotNull(db.system)
| makeTimeseries total = count(),
                 errors = countIf(otel.status_code == "ERROR"),
                 by:{db.system},
                 interval:1h
```

<a id="summary"></a>

## 8. Summary

In this notebook you learned:

- The key database KPIs to monitor: response time, error rate, throughput, slow queries, and connection health
- Dashboard-ready queries for a unified database health overview
- Response time monitoring with P50/P95/P99 trend analysis
- Error rate alerting queries and recommended thresholds
- Throughput monitoring including read/write ratio trends and day-over-day comparison
- Slow query alerting with vendor-specific thresholds
- Database SLO definitions and compliance measurement queries

### Series Complete

This is the final notebook in the DBMON series. For a complete database monitoring implementation:

1. **DBMON-01** — Understand fundamentals and establish baselines
2. **DBMON-02** — Deep dive into SQL database monitoring
3. **DBMON-03** — NoSQL database monitoring patterns
4. **DBMON-04** — Cache and messaging system monitoring
5. **DBMON-05** — Advanced query analysis and optimization
6. **DBMON-06** — (This notebook) Dashboards, alerting, and SLOs


---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>

