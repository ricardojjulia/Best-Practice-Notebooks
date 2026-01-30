# Cluster Health Monitoring

> **Series:** K8S | **Notebook:** 4 of 12 | **Created:** January 2026 | **Last Updated:** 01/30/2026

## Deep-Dive into Kubernetes Cluster Metrics
Cluster health monitoring provides visibility into the infrastructure layer of Kubernetes: nodes, control plane, and cluster-wide resources. This notebook covers key metrics, thresholds, and DQL queries for proactive cluster management.

---

## Table of Contents

1. [Node Monitoring](#node-monitoring)
2. [Resource Capacity Planning](#resource-capacity-planning)
3. [Control Plane Health](#control-plane-health)
4. [Cluster-Wide Events](#cluster-wide-events)
5. [Cost Optimization Queries](#cost-optimization-queries)
6. [Alerting Strategies](#alerting-strategies)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Kubernetes monitoring |
| **DynaKube** | ActiveGate with `kubernetes-monitoring` capability |
| **Permissions** | `metrics.read`, `entities.read`, `logs.read` |
| **Data** | At least 24 hours of cluster data |

## 1. Cluster Health Overview

### Key Health Indicators

| Category | Metrics | Healthy State |
|----------|---------|---------------|
| **Node Status** | Ready/NotReady | All nodes Ready |
| **Pod Scheduling** | Pending pods | No long-pending pods |
| **Resource Pressure** | CPU/Memory pressure | No pressure conditions |
| **Disk Pressure** | Disk space, inode usage | >15% available |
| **Network** | CNI health, DNS latency | <100ms DNS resolution |

### Dynatrace Kubernetes Dashboard

The built-in Kubernetes dashboard provides:
- Cluster overview with node status
- Namespace resource usage
- Workload health summary
- Recent events and problems

Navigate to: **Infrastructure > Kubernetes**

```dql
// List all monitored Kubernetes clusters
fetch dt.entity.kubernetes_cluster
| fields entity.name, tags
| sort entity.name asc
```

<a id="node-monitoring"></a>
## 2. Node Monitoring
### Node Status and Conditions

| Condition | Description | Alert When |
|-----------|-------------|------------|
| **Ready** | Node can accept pods | False for >5 min |
| **MemoryPressure** | Low memory | True |
| **DiskPressure** | Low disk space | True |
| **PIDPressure** | Too many processes | True |
| **NetworkUnavailable** | Network not configured | True |

```dql
// List all Kubernetes nodes
fetch dt.entity.kubernetes_node
| fields entity.name, tags
| sort entity.name asc
```

```dql
// Node CPU utilization over time (top 10 by name)
timeseries nodeCpu = avg(dt.kubernetes.node.cpu_usage), by:{dt.entity.kubernetes_node}
| limit 10
```

```dql
// Node memory utilization - average over time period
fetch dt.metrics
| filter metric.key == "dt.kubernetes.node.memory_usage"
| summarize avgMemory = avg(value), by:{dt.entity.kubernetes_node}
| filter avgMemory > 80
| sort avgMemory desc
```

```dql
// Node filesystem usage - disk pressure detection
fetch dt.metrics
| filter metric.key == "dt.host.disk.used.percent"
| summarize avgDiskUsage = avg(value), by:{dt.entity.host}
| filter avgDiskUsage > 80
| sort avgDiskUsage desc
```

<a id="resource-capacity-planning"></a>
## 3. Resource Capacity Planning
### Capacity Metrics

| Metric | Description | Use Case |
|--------|-------------|----------|
| **Allocatable** | Resources available for pods | Scheduling decisions |
| **Requested** | Sum of pod requests | Capacity planning |
| **Used** | Actual consumption | Right-sizing |
| **Limits** | Maximum allowed | Burst capacity |

### Utilization vs. Allocation

![Node Resource Utilization Model](images/node-capacity-utilization.svg)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Layer | Description |
|-------|-------------|
| Node Capacity | Total hardware resources |
| System Reserved | kubelet, runtime, OS |
| Allocatable | Available for pods |
| Requested (sum) | Pod requests |
| Actually Used | Real-time usage |

**Key Insight:** Requested ≠ Used. Over-provisioning wastes resources. Monitor both to optimize cluster efficiency.
For environments where SVG doesn't render
-->

```dql
// CPU requests by namespace (average over time period)
fetch dt.metrics
| filter metric.key == "dt.kubernetes.workload.requests_cpu"
| summarize avgCpuRequests = avg(value), by:{k8s.namespace.name}
| sort avgCpuRequests desc
| limit 15
```

```dql
// Memory requests by namespace (average over time period)
fetch dt.metrics
| filter metric.key == "dt.kubernetes.workload.requests_memory"
| summarize avgMemRequests = avg(value), by:{k8s.namespace.name}
| sort avgMemRequests desc
| limit 15
```

```dql
// Find over-provisioned workloads (low CPU usage)
fetch dt.metrics
| filter metric.key == "dt.containers.cpu.usage_percent"
| summarize avgCpuUsage = avg(value), by:{dt.entity.cloud_application}
| filter avgCpuUsage < 20
| sort avgCpuUsage asc
| limit 20
```

<a id="control-plane-health"></a>
## 4. Control Plane Health
### Control Plane Components

| Component | Function | Key Metrics |
|-----------|----------|-------------|
| **API Server** | REST API for K8s | Request latency, error rate |
| **etcd** | Distributed KV store | Disk sync latency, leader elections |
| **Scheduler** | Pod placement | Scheduling latency, failures |
| **Controller Manager** | Reconciliation loops | Queue depth, sync latency |

### Managed Kubernetes Note

For managed Kubernetes (EKS, AKS, GKE), control plane metrics are limited. Focus on:
- API server response times (client-side)
- Kubernetes events for scheduling issues
- Cloud provider metrics for control plane health

```dql
// API server events and errors
fetch logs
| filter matchesPhrase(content, "kube-apiserver") or matchesPhrase(content, "api-server")
| filter matchesPhrase(content, "error") or matchesPhrase(content, "failed")
| fields timestamp, content
| sort timestamp desc
| limit 20
```

<a id="cluster-wide-events"></a>
## 5. Cluster-Wide Events
### Event Types to Monitor

| Event Type | Reason | Action |
|------------|--------|--------|
| **Warning** | FailedScheduling | Check resource constraints |
| **Warning** | FailedMount | Check PV/PVC configuration |
| **Warning** | OOMKilled | Increase memory limits |
| **Warning** | Evicted | Node under pressure |
| **Normal** | Pulling/Pulled | Image operations |
| **Normal** | Scheduled | Pod placement |

```dql
// Kubernetes warning events
fetch logs
| filter matchesPhrase(content, "Warning") and (matchesPhrase(log.source, "kubernetes") or matchesPhrase(log.source, "k8s"))
| fields timestamp, content
| sort timestamp desc
| limit 50
```

```dql
// Failed scheduling events
fetch logs
| filter matchesPhrase(content, "FailedScheduling") or matchesPhrase(content, "Insufficient")
| fields timestamp, content
| sort timestamp desc
| limit 30
```

```dql
// OOMKilled events - memory issues
fetch logs
| filter matchesPhrase(content, "OOMKilled") or matchesPhrase(content, "Out of memory")
| fields timestamp, content
| sort timestamp desc
| limit 30
```

```dql
// Event summary by type
fetch logs, from: now() - 24h
| filter matchesPhrase(log.source, "kubernetes") or matchesPhrase(log.source, "k8s")
| parse content, "LD:eventType ' ' LD"
| summarize count = count(), by:{eventType}
| sort count desc
| limit 20
```

<a id="cost-optimization-queries"></a>
## 6. Cost Optimization Queries
### Resource Efficiency Analysis

| Metric | Target | Action If Not Met |
|--------|--------|-------------------|
| **CPU Utilization** | >40% avg | Reduce requests |
| **Memory Utilization** | >50% avg | Reduce requests |
| **Node Utilization** | >60% | Scale down nodes |
| **Idle Pods** | 0 | Review necessity |

```dql
// Find workloads with very low CPU utilization (candidates for right-sizing)
fetch dt.metrics
| filter metric.key == "dt.containers.cpu.usage_percent"
| summarize avgCpuUsage = avg(value), by:{dt.entity.cloud_application}
| filter avgCpuUsage < 10
| sort avgCpuUsage asc
| limit 25
```

```dql
// Memory usage efficiency by workload (low usage = over-provisioned)
fetch dt.metrics
| filter metric.key == "dt.containers.memory.usage_percent"
| summarize avgMemUsage = avg(value), by:{dt.entity.cloud_application}
| filter avgMemUsage < 30
| sort avgMemUsage asc
| limit 25
```

<a id="alerting-strategies"></a>
## 7. Alerting Strategies
### Recommended Alerts

| Alert | Condition | Severity |
|-------|-----------|----------|
| **Node NotReady** | Node condition != Ready for 5 min | Critical |
| **High Node CPU** | CPU > 85% for 15 min | Warning |
| **High Node Memory** | Memory > 90% for 10 min | Critical |
| **Disk Pressure** | Disk > 85% | Warning |
| **Pod Scheduling Failed** | FailedScheduling events | Warning |
| **OOM Kills** | OOMKilled events | Warning |

### Alert Configuration in Dynatrace

Navigate to: **Settings > Anomaly detection > Kubernetes**

Configure:
- Node availability alerts
- Resource saturation thresholds
- Workload health anomalies

### Custom Metric Events

For advanced alerting, use custom metric events with DQL-derived thresholds.

## Next Steps

With cluster health monitoring in place, proceed to:

| Next Notebook | Topic |
|---------------|-------|
| **K8S-05: Workload Monitoring** | Application-level observability |
| **K8S-06: Namespace Organization** | Boundaries and access control |
| **K8S-07: Events and Logs** | Log ingestion and analysis |

---

## Summary

In this notebook, you learned:

- Cluster health overview and key indicators
- Node monitoring for CPU, memory, and disk
- Resource capacity planning with requests vs. usage analysis
- Control plane health considerations
- Cluster-wide event monitoring and analysis
- Cost optimization queries for right-sizing
- Alerting strategies for proactive cluster management

---

## References

- [Kubernetes Cluster Monitoring](https://docs.dynatrace.com/docs/observe/infrastructure-monitoring/kubernetes-and-openshift-monitoring/kubernetes-cluster-monitoring)
- [Kubernetes Metrics](https://docs.dynatrace.com/docs/observe/infrastructure-monitoring/kubernetes-and-openshift-monitoring/kubernetes-workload-and-node-monitoring)
- [Kubernetes Anomaly Detection](https://docs.dynatrace.com/docs/observe/infrastructure-monitoring/kubernetes-and-openshift-monitoring/kubernetes-anomaly-detection)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
