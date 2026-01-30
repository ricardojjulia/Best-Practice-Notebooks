# DQL Queries for Kubernetes

> **Series:** K8S | **Notebook:** 8 of 12 | **Created:** January 2026 | **Last Updated:** 01/30/2026

## Advanced Query Patterns for Kubernetes Data
This notebook provides a comprehensive reference of DQL queries for Kubernetes monitoring. From basic entity queries to complex performance analysis, these patterns help you extract insights from your Kubernetes data.

---

## Table of Contents

1. [Entity Queries](#entity-queries)
2. [Metric Queries](#metric-queries)
3. [Log and Event Queries](#log-and-event-queries)
4. [Trace Queries](#trace-queries)
5. [Correlation Queries](#correlation-queries)
6. [Dashboard Queries](#dashboard-queries)
7. [Alerting Queries](#alerting-queries)
8. [Query Templates](#query-templates)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Kubernetes monitoring |
| **Permissions** | `metrics.read`, `entities.read`, `logs.read` |
| **Knowledge** | K8S-01 through K8S-07 |
| **Data** | Active Kubernetes cluster monitored |

<a id="entity-queries"></a>
## 1. Entity Queries
### Kubernetes Entity Types

| Entity Type | DQL Table | Description |
|-------------|-----------|-------------|
| Cluster | `dt.entity.kubernetes_cluster` | K8s clusters |
| Node | `dt.entity.kubernetes_node` | Worker nodes |
| Namespace | `dt.entity.cloud_application_namespace` | Namespaces |
| Workload | `dt.entity.cloud_application` | Deployments, StatefulSets |
| Container | `dt.entity.container_group_instance` | Container instances |
| Service | `dt.entity.service` | Detected services |

```dql
// List all Kubernetes clusters
fetch dt.entity.kubernetes_cluster
| fields entity.name, tags
| sort entity.name asc
```

```dql
// List all Kubernetes nodes
fetch dt.entity.kubernetes_node
| fields entity.name, tags
| sort entity.name asc
```

```dql
// List all namespaces
fetch dt.entity.cloud_application_namespace
| fields entity.name, tags
| sort entity.name asc
```

```dql
// List all workloads (deployments, statefulsets)
fetch dt.entity.cloud_application
| fields entity.name, tags
| sort entity.name asc
| limit 50
```

```dql
// Count Kubernetes nodes
fetch dt.entity.kubernetes_node
| summarize nodeCount = count()
```

<a id="metric-queries"></a>
## 2. Metric Queries
### Common Kubernetes Metrics

| Metric | Description | Unit |
|--------|-------------|------|
| `dt.containers.cpu.usage_percent` | Container CPU vs limit | Percent |
| `dt.containers.memory.usage_percent` | Container memory vs limit | Percent |
| `dt.containers.cpu.throttled_time` | CPU throttle time | Nanoseconds |
| `dt.containers.memory.working_set_bytes` | Working memory | Bytes |
| `dt.kubernetes.workload.requests_cpu` | CPU requests | Millicores |
| `dt.kubernetes.workload.requests_memory` | Memory requests | Bytes |

```dql
// Container CPU usage - top consumers
fetch dt.metrics
| filter metric.key == "dt.containers.cpu.usage_percent"
| summarize avgCpu = avg(value), by:{dt.entity.container_group_instance}
| sort avgCpu desc
| limit 15
```

```dql
// Container memory usage approaching limits
fetch dt.metrics
| filter metric.key == "dt.containers.memory.usage_percent"
| summarize avgMem = avg(value), by:{dt.entity.container_group_instance}
| filter avgMem > 80
| sort avgMem desc
```

```dql
// CPU throttling detection
fetch dt.metrics
| filter metric.key == "dt.containers.cpu.throttled_time"
| summarize totalThrottle = sum(value), by:{dt.entity.container_group_instance}
| filter totalThrottle > 0
| sort totalThrottle desc
| limit 15
```

```dql
// Namespace-level resource usage
fetch dt.metrics
| filter metric.key == "dt.containers.cpu.usage_percent"
| summarize avgCpuUsage = avg(value), by:{k8s.namespace.name}
| sort avgCpuUsage desc
| limit 15
```

```dql
// Resource requests by namespace (capacity planning)
fetch dt.metrics
| filter metric.key == "dt.kubernetes.workload.requests_cpu"
| summarize avgCpuRequests = avg(value), by:{k8s.namespace.name}
| sort avgCpuRequests desc
| limit 15
```

<a id="log-and-event-queries"></a>
## 3. Log and Event Queries
### Log Query Patterns

```dql
// Error logs by namespace
fetch logs
| filter loglevel == "ERROR"
| filter isNotNull(k8s.namespace.name)
| summarize errorCount = count(), by:{k8s.namespace.name}
| sort errorCount desc
| limit 15
```

```dql
// Kubernetes events - warnings only
fetch logs
| filter matchesPhrase(content, "Warning")
| filter matchesPhrase(log.source, "kubernetes") or matchesPhrase(log.source, "k8s")
| fields timestamp, content
| sort timestamp desc
| limit 30
```

```dql
// Pod crashes and restarts
fetch logs
| filter matchesPhrase(content, "CrashLoopBackOff") or matchesPhrase(content, "BackOff")
| fields timestamp, content
| sort timestamp desc
| limit 20
```

```dql
// OOM events
fetch logs
| filter matchesPhrase(content, "OOMKilled") or matchesPhrase(content, "Out of memory")
| fields timestamp, content
| sort timestamp desc
| limit 20
```

```dql
// Log volume trend by namespace
fetch logs, from: now() - 24h
| filter isNotNull(k8s.namespace.name)
| summarize log_count = count(), by:{k8s.namespace.name, time_bucket = bin(timestamp, 1h)}
| sort time_bucket asc
| limit 200
```

<a id="trace-queries"></a>
## 4. Trace Queries
### Span Queries for Kubernetes Services

```dql
// Service response times in K8s
fetch spans
| filter span.kind == "server"
| filter isNotNull(k8s.namespace.name)
| summarize 
    p50 = percentile(duration, 50),
    p95 = percentile(duration, 95),
    p99 = percentile(duration, 99),
    by:{k8s.namespace.name, dt.entity.service}
| sort p99 desc
| limit 15
```

```dql
// Error rate by service
fetch spans
| filter span.kind == "server"
| filter isNotNull(k8s.namespace.name)
| summarize 
    total = count(),
    errors = countIf(otel.status_code == "ERROR"),
    by:{k8s.namespace.name, dt.entity.service}
| fieldsAdd errorRate = 100.0 * toDouble(errors) / toDouble(total)
| filter errors > 0
| sort errorRate desc
| limit 15
```

```dql
// Slow traces (>1 second)
fetch spans
| filter span.kind == "server" and duration > 1000000000
| filter isNotNull(k8s.namespace.name)
| fields timestamp, trace.id, k8s.namespace.name, span.name, duration
| sort duration desc
| limit 20
```

```dql
// Request throughput by namespace
fetch spans
| filter span.kind == "server"
| filter isNotNull(k8s.namespace.name)
| summarize requestCount = count(), by:{k8s.namespace.name}
| sort requestCount desc
| limit 10
```

<a id="correlation-queries"></a>
## 5. Correlation Queries
### Joining Multiple Data Sources

```dql
// High CPU workloads with their names
fetch dt.metrics
| filter metric.key == "dt.containers.cpu.usage_percent"
| summarize avgCpu = avg(value), by:{dt.entity.cloud_application}
| filter avgCpu > 50
| lookup [fetch dt.entity.cloud_application | fields id, entity.name], sourceField:dt.entity.cloud_application, lookupField:id
| fields entity.name, avgCpu
| sort avgCpu desc
```

```dql
// Nodes with high utilization
fetch dt.metrics
| filter metric.key == "dt.kubernetes.node.cpu_usage"
| summarize avgNodeCpu = avg(value), by:{dt.entity.kubernetes_node}
| filter avgNodeCpu > 70
| lookup [fetch dt.entity.kubernetes_node | fields id, entity.name], sourceField:dt.entity.kubernetes_node, lookupField:id
| fields entity.name, avgNodeCpu
| sort avgNodeCpu desc
```

<a id="dashboard-queries"></a>
## 6. Dashboard Queries
### Queries Optimized for Dashboards

```dql
// Cluster summary tile
fetch dt.entity.kubernetes_cluster
| summarize clusterCount = count()
```

```dql
// Node count tile
fetch dt.entity.kubernetes_node
| summarize nodeCount = count()
```

```dql
// Error rate trend (chart)
fetch logs, from: now() - 24h
| filter loglevel == "ERROR" and isNotNull(k8s.namespace.name)
| summarize errors = count(), by:{time_bucket = bin(timestamp, 1h)}
| sort time_bucket asc
```

```dql
// Top namespaces by CPU (bar chart)
fetch dt.metrics
| filter metric.key == "dt.containers.cpu.usage_percent"
| summarize avgCpu = avg(value), by:{k8s.namespace.name}
| sort avgCpu desc
| limit 10
```

<a id="alerting-queries"></a>
## 7. Alerting Queries
### Threshold-Based Alerting Patterns

```dql
// High memory usage alert query
fetch dt.metrics
| filter metric.key == "dt.containers.memory.usage_percent"
| summarize avgMem = avg(value), by:{dt.entity.container_group_instance}
| filter avgMem > 90
| summarize alertCount = count()
```

```dql
// CrashLoop detection (last hour)
fetch logs, from: now() - 1h
| filter matchesPhrase(content, "CrashLoopBackOff")
| summarize crashCount = count()
```

```dql
// Error rate spike detection
fetch logs, from: now() - 1h
| filter loglevel == "ERROR" and isNotNull(k8s.namespace.name)
| summarize errors = count(), by:{k8s.namespace.name}
| filter errors > 100
| sort errors desc
```

<a id="query-templates"></a>
## 8. Query Templates
### Reusable Query Patterns

**Filter by Namespace:**
```dql
| filter k8s.namespace.name == "NAMESPACE_NAME"
```

**Filter by Cluster:**
```dql
| filter kubernetesClusterName == "CLUSTER_NAME"
```

**Time-based Analysis:**
```dql
fetch logs, from: now() - DURATION
| summarize log_count = count(), by:{time_bucket = bin(timestamp, INTERVAL)}
| sort time_bucket asc
```

**Top N Pattern:**
```dql
| sort METRIC desc
| limit N
```

---

## Summary

This notebook provided DQL query patterns for:

- Entity discovery and relationships
- Container and node metrics
- Log and event analysis
- Distributed tracing
- Multi-source correlation
- Dashboard visualizations
- Alerting conditions

---

## References

- [DQL Documentation](https://docs.dynatrace.com/docs/observe-and-explore/query-data/dynatrace-query-language)
- [Kubernetes Metrics Reference](https://docs.dynatrace.com/docs/observe/infrastructure-monitoring/kubernetes-and-openshift-monitoring/kubernetes-metrics)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
