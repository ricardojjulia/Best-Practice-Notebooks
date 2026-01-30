# DynaKube Configuration Review & Recommendations

**Cluster:** `usf-app-sit-panamax-eks-cluster`  
**Environment:** SIT (System Integration Testing)  
**Dynatrace Operator Version:** 1.6.1  
**API URL:** `https://cwt03869.live.dynatrace.com/api`  
**Review Date:** January 2026

---

## Executive Summary

Your current DynaKube configuration is **well-structured** and follows many best practices. However, there are several tuning opportunities that could help with your New Relic coexistence requirement, improve observability, and enhance operational control.

| Category                  | Current State                         | Recommendation Priority | Timeline                 |
| ------------------------- | ------------------------------------- | ----------------------- | ------------------------ |
| Opt-In/Opt-Out Monitoring | Not configured (default: monitor all) | 🔴 High                 | Pre-01/31                |
| Build Version Propagation | Not enabled                           | 🟡 Medium               | Enable now, labels later |
| NGINX Ingress Monitoring  | Not configured                        | 🟡 Medium               | Post-01/31 OK            |
| CSI Driver Resources      | Missing provisioner limits            | 🟡 Medium               | Pre-01/31                |
| OTel Collector Resources  | Not defined                           | 🟡 Medium               | Pre-01/31                |
| ActiveGate Configuration  | Good                                  | 🟢 Low                  | N/A                      |
| Feature Flags             | Minimal                               | 🟡 Medium               | Anytime                  |
| Log Monitoring            | Enabled (implicit)                    | 🟢 OK as-is             | N/A                      |

---

## Current Configuration Analysis

### What You Have

```yaml
apiVersion: dynatrace.com/v1beta5
kind: DynaKube
metadata:
  name: usf-app-sit-panamax-eks-cluster
  namespace: dynatrace
  labels:
    dynatrace.com/created-by: "dynatrace.kubernetes"
  annotations:
    feature.dynatrace.com/k8s-app-enabled: "true"
spec:
  apiUrl: https://cwt03869.live.dynatrace.com/api
  networkZone: usf.sit
  metadataEnrichment:
    enabled: true
  oneAgent:
    hostGroup: sit
    cloudNativeFullStack:
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
          operator: Exists
        - effect: NoSchedule
          key: node-role.kubernetes.io/control-plane
          operator: Exists
  activeGate:
    capabilities:
      - routing
      - kubernetes-monitoring
      - debugging
    resources:
      requests:
        cpu: 500m
        memory: 1.5Gi
      limits:
        cpu: 1000m
        memory: 2Gi
    replicas: 2
  templates:
    otelCollector:
      imageRef:
        repository: public.ecr.aws/dynatrace/dynatrace-otel-collector
        tag: latest
  logMonitoring: {}
  telemetryIngest:
    protocols:
      - jaeger
      - otlp
      - statsd
      - zipkin
    serviceName: telemetry-ingest
```

### ✅ What's Good

1. **API Version:** Using `v1beta5` (current stable version)
2. **Network Zone:** Properly configured (`usf.sit`) for traffic routing
3. **Metadata Enrichment:** Enabled for enhanced telemetry context
4. **Host Group:** Defined (`sit`) for logical grouping
5. **Tolerations:** Correctly configured for master/control-plane nodes
6. **ActiveGate:**
   - Proper capabilities enabled (routing, kubernetes-monitoring, debugging)
   - Resource limits defined
   - HA with 2 replicas
7. **Telemetry Ingest:** Multi-protocol support (Jaeger, OTLP, StatsD, Zipkin)
8. **OTel Collector:** Using ECR repository (good for EKS)

### ⚠️ Areas for Improvement

| Issue                                 | Impact                                              | Section                                                                               |
| ------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------- |
| No namespace selector                 | All namespaces monitored (conflicts with New Relic) | [Opt-In Mode](#1-enable-opt-in-mode-for-new-relic-coexistence)                        |
| No version detection                  | Missing build/release metadata                      | [Version Detection](#2-enable-build-version-propagation)                              |
| No injection failure policy           | Silent failures may go unnoticed                    | [Feature Flags](#3-add-recommended-feature-flags)                                     |
| OTel Collector using `latest` tag     | Non-deterministic deployments                       | [Image Tags](#4-pin-otel-collector-image-version)                                     |
| CSI Driver missing provisioner limits | Unbounded resource usage                            | [Resource Configuration](#7-resource-configuration-for-csi-driver-and-otel-collector) |
| OTel Collector no resource limits     | Potential memory growth under load                  | [Resource Configuration](#7-resource-configuration-for-csi-driver-and-otel-collector) |
| No NGINX ingress setup                | Ingress traffic not traced                          | [NGINX Monitoring](#6-nginx-ingress-controller-monitoring)                            |

### ✅ Already Working (No Changes Needed)

| Item           | Status     | Notes                                                                         |
| -------------- | ---------- | ----------------------------------------------------------------------------- |
| Log Monitoring | ✅ Enabled | `logMonitoring: {}` enables with defaults — logs already flowing to Dynatrace |

---

## Recommended Changes

### 1. Enable Opt-In Mode for New Relic Coexistence

**Priority:** 🔴 High

Since you're running New Relic NRI bundle on the same cluster, you have two valid approaches:

---

#### Option A: Infrastructure-Only Monitoring (Omit cloudNativeFullStack)

**This is what you're currently doing with your minimal installation.**

If you omit `oneAgent.cloudNativeFullStack` entirely, Dynatrace provides:

| ✅ Included               | ❌ Not Included       |
| ------------------------- | --------------------- |
| Kubernetes cluster health | Application-level APM |
| Workload metrics          | Code-level tracing    |
| Pod/container metrics     | Distributed traces    |
| Kubernetes events         | OneAgent injection    |
| ActiveGate capabilities   |                       |

**This approach is valid if:**

- You only need K8s infrastructure observability from Dynatrace
- New Relic handles all application-level tracing/APM
- You don't need Dynatrace code-level insights on this cluster

**No changes needed** — your current minimal installation without `cloudNativeFullStack` achieves this.

---

#### Option B: Selective APM with Opt-In Mode

If you want Dynatrace APM on _some_ workloads while New Relic monitors others, switch to opt-in mode:

**Add to metadata.annotations:**

```yaml
metadata:
  annotations:
    feature.dynatrace.com/k8s-app-enabled: "true"
    feature.dynatrace.com/automatic-injection: "false" # ADD THIS
```

**Add namespace selector to oneAgent spec:**

```yaml
oneAgent:
  hostGroup: sit
  cloudNativeFullStack:
    namespaceSelector:
      matchLabels:
        dt-monitoring: "true" # Only monitor labeled namespaces
    tolerations:
      # ... existing tolerations
```

**Then label namespaces you want Dynatrace to monitor:**

```bash
kubectl label namespace my-app dt-monitoring=true
kubectl label namespace another-app dt-monitoring=true
```

**To explicitly exclude New Relic namespace (belt-and-suspenders approach):**

```yaml
namespaceSelector:
  matchLabels:
    dt-monitoring: "true"
  matchExpressions:
    - key: kubernetes.io/metadata.name
      operator: NotIn
      values:
        - newrelic
        - nr-infra
```

---

### 2. Enable Build Version Propagation

**Priority:** 🟡 Medium  
**Timeline:** Enable now — labels can be added later

This allows Dynatrace to automatically detect version information from your Kubernetes labels.

> **Important:** Labels are **not a prerequisite**. You can enable the feature flag now, and Dynatrace will simply not have version metadata until labels are added. When you add labels later (post-01/31 is fine), Dynatrace will automatically start picking them up. This is purely additive — zero risk.

**Add to metadata.annotations:**

```yaml
metadata:
  annotations:
    feature.dynatrace.com/k8s-app-enabled: "true"
    feature.dynatrace.com/label-version-detection: "true" # ADD THIS
```

**Add labels to your application deployments later (whenever ready):**

```yaml
# In your application deployments
metadata:
  labels:
    app.kubernetes.io/name: "my-service"
    app.kubernetes.io/version: "1.2.3"
    app.kubernetes.io/component: "api"
```

---

### 3. Add Recommended Feature Flags

**Priority:** 🟡 Medium

**Add these annotations for better operational control:**

```yaml
metadata:
  annotations:
    feature.dynatrace.com/k8s-app-enabled: "true"
    feature.dynatrace.com/automatic-injection: "false"
    feature.dynatrace.com/label-version-detection: "true"
    # Fail pod startup if injection fails (instead of silent failure)
    feature.dynatrace.com/injection-failure-policy: "fail"
```

---

### 4. Pin OTel Collector Image Version

**Priority:** 🟡 Medium

Using `latest` tag can cause unexpected behavior during updates.

**Change from:**

```yaml
templates:
  otelCollector:
    imageRef:
      repository: public.ecr.aws/dynatrace/dynatrace-otel-collector
      tag: latest # ❌ Non-deterministic
```

**Change to:**

```yaml
templates:
  otelCollector:
    imageRef:
      repository: public.ecr.aws/dynatrace/dynatrace-otel-collector
      tag: "0.16.0" # ✅ Pin to specific version
```

> **Note:** Check the [Dynatrace OTel Collector releases](https://github.com/Dynatrace/dynatrace-otel-collector/releases) for the current stable version.

---

### 5. Log Monitoring Configuration

**Priority:** 🟢 OK as-is  
**Status:** ✅ Already working

Your current `logMonitoring: {}` **already enables log monitoring with defaults** — this is why logs are flowing to the Dynatrace UI in SIT.

| Config              | Behavior                                 |
| ------------------- | ---------------------------------------- |
| `logMonitoring: {}` | ✅ Enabled with defaults (what you have) |
| Omitted entirely    | ❌ Log monitoring disabled               |

**No changes required to enable log monitoring.**

#### Configuring Log Monitoring Resources

> ⚠️ **Important:** Resource limits for log monitoring are configured under `spec.templates.logMonitoring`, NOT under `spec.logMonitoring`. The `spec.logMonitoring` field only accepts an empty object `{}` to enable the feature.

**Incorrect (will cause validation errors):**

```yaml
# ❌ WRONG - these will fail with "unknown field" errors
spec:
  logMonitoring:
    enabled: true           # Not allowed
  logMonitoring:
    resources: ...          # Not allowed here
```

**Correct structure:**

```yaml
spec:
  # Enable log monitoring (empty object only)
  logMonitoring: {}

  # Resource configuration goes under templates
  templates:
    logMonitoring:
      resources:
        requests:
          cpu: 100m
          memory: 256Mi
        limits:
          cpu: 500m
          memory: 512Mi
      # Optional: tolerations
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
          operator: Exists
        - effect: NoSchedule
          key: node-role.kubernetes.io/control-plane
          operator: Exists
```

**Optional future enhancement:** If you want to exclude certain namespaces from log collection (e.g., to avoid duplication with New Relic), configure log ingest rules in the Dynatrace UI:

1. Go to **Settings > Log Monitoring > Log ingest rules**
2. Create rules to include/exclude specific namespaces
3. This is done in the UI, not in the DynaKube YAML

---

### 6. NGINX Ingress Controller Monitoring

**Priority:** 🟡 Medium  
**Timeline:** Can be enabled post-01/31 — no impact to release

> **Note:** This feature is independent of your DynaKube deployment. It's configured via a ConfigMap change on the ingress controller itself, so it can be added whenever convenient after go-live.

#### Prerequisites

- OneAgent version 1.227+
- Pod name must contain "ingress-nginx-"
- x86-64 architecture

#### Configuration Steps (When Ready)

Edit the ingress-nginx-controller ConfigMap:

```bash
kubectl edit configmap ingress-nginx-controller -n ingress-nginx
```

**Add to the ConfigMap data:**

```yaml
data:
  main-snippet: |
    load_module /opt/dynatrace/oneagent-paas/agent/bin/current/linux-musl-x86-64/liboneagentnginx.so;
```

**Optional: Add trace context to access logs:**

```yaml
data:
  main-snippet: |
    load_module /opt/dynatrace/oneagent-paas/agent/bin/current/linux-musl-x86-64/liboneagentnginx.so;
  log-format-upstream: >-
    $remote_addr - $remote_user [$time_local] "$request"
    [!dt dt.trace_id=$dt_trace_id,dt.span_id=$dt_span_id,dt.trace_sampled=$dt_trace_sampled]
    $status $body_bytes_sent "$http_referer" "$http_user_agent"
```

**No DynaKube changes required** — just the ConfigMap edit and a rolling restart of ingress pods.

---

### 7. Resource Configuration for CSI Driver and OTel Collector

**Priority:** 🟡 Medium

> ⚠️ **Important:** In DynaKube v1beta5, resource limits for OTel Collector and Log Monitoring are configured under `spec.templates`, not directly under their respective spec sections.

#### CSI Driver Resources

The CSI driver DaemonSet runs **5 containers** (sidecar pattern), each requiring individual resource definitions. The per-container approach in your `values.yaml` is correct — simplified single-resource blocks in some documentation are outdated.

**Recommended `values.yaml` configuration:**

```yaml
csidriver:
  enabled: true
  csiInit:
    resources:
      requests:
        cpu: 50m
        memory: 100Mi
      limits:
        cpu: 100m
        memory: 128Mi
  server:
    resources:
      requests:
        cpu: 50m
        memory: 100Mi
      limits:
        cpu: 100m
        memory: 128Mi
  provisioner:
    resources:
      requests:
        cpu: 300m
        memory: 100Mi
      limits:
        cpu: 500m # Add limit (missing in default)
        memory: 256Mi # Add limit (missing in default)
  registrar:
    resources:
      requests:
        cpu: 20m
        memory: 30Mi
      limits:
        cpu: 50m
        memory: 64Mi
  livenessprobe:
    resources:
      requests:
        cpu: 20m
        memory: 30Mi
      limits:
        cpu: 50m
        memory: 64Mi
```

**CSI Driver Container Resource Summary:**

| Container       | CPU Request | CPU Limit | Memory Request | Memory Limit | Notes                                              |
| --------------- | ----------- | --------- | -------------- | ------------ | -------------------------------------------------- |
| `csiInit`       | 50m         | 100m      | 100Mi          | 128Mi        | Init container, runs once                          |
| `server`        | 50m         | 100m      | 100Mi          | 128Mi        | Main CSI plugin                                    |
| `provisioner`   | 300m        | 500m      | 100Mi          | 256Mi        | Handles volume provisioning — needs more resources |
| `registrar`     | 20m         | 50m       | 30Mi           | 64Mi         | Node registration sidecar                          |
| `livenessprobe` | 20m         | 50m       | 30Mi           | 64Mi         | Health check sidecar                               |

> ⚠️ **Note:** Your default `values.yaml` is missing limits for the `provisioner` container. Add them to ensure predictable resource usage.

#### OTel Collector and Log Monitoring Resources

Both OTel Collector and Log Monitoring resources are configured under `spec.templates` in the DynaKube CR.

**Add to your DynaKube spec:**

```yaml
templates:
  otelCollector:
    imageRef:
      repository: public.ecr.aws/dynatrace/dynatrace-otel-collector
      tag: "0.16.0"
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi
  logMonitoring:
    resources:
      requests:
        cpu: 100m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi
    tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/master
        operator: Exists
      - effect: NoSchedule
        key: node-role.kubernetes.io/control-plane
        operator: Exists
```

**OTel Collector Sizing Guide:**

| Telemetry Volume | CPU Limit | Memory Limit | Environment |
| ---------------- | --------- | ------------ | ----------- |
| Low              | 200m      | 256Mi        | Dev/Test    |
| Medium           | 500m      | 512Mi        | SIT         |
| High             | 1000m     | 1Gi          | Production  |

**Log Monitoring Sizing Guide:**

| Log Volume | CPU Limit | Memory Limit | Environment |
| ---------- | --------- | ------------ | ----------- |
| Low        | 200m      | 256Mi        | Dev/Test    |
| Medium     | 500m      | 512Mi        | SIT         |
| High       | 1000m     | 1Gi          | Production  |

---

## Complete Recommended DynaKube Configuration

Here's the full recommended configuration incorporating all changes:

```yaml
apiVersion: dynatrace.com/v1beta5
kind: DynaKube
metadata:
  name: usf-app-sit-panamax-eks-cluster
  namespace: dynatrace
  labels:
    dynatrace.com/created-by: "dynatrace.kubernetes"
  annotations:
    # Kubernetes app monitoring
    feature.dynatrace.com/k8s-app-enabled: "true"
    # Opt-in mode for New Relic coexistence
    feature.dynatrace.com/automatic-injection: "false"
    # Enable version detection from K8s labels
    feature.dynatrace.com/label-version-detection: "true"
    # Fail pod if injection fails
    feature.dynatrace.com/injection-failure-policy: "fail"
spec:
  apiUrl: https://cwt03869.live.dynatrace.com/api
  networkZone: usf.sit

  metadataEnrichment:
    enabled: true

  oneAgent:
    hostGroup: sit
    cloudNativeFullStack:
      # Opt-in: Only monitor namespaces with this label
      namespaceSelector:
        matchLabels:
          dt-monitoring: "true"
        matchExpressions:
          - key: kubernetes.io/metadata.name
            operator: NotIn
            values:
              - newrelic
              - kube-system
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
          operator: Exists
        - effect: NoSchedule
          key: node-role.kubernetes.io/control-plane
          operator: Exists

  activeGate:
    capabilities:
      - routing
      - kubernetes-monitoring
      - debugging
    resources:
      requests:
        cpu: 500m
        memory: 1.5Gi
      limits:
        cpu: 1000m
        memory: 2Gi
    replicas: 2

  templates:
    otelCollector:
      imageRef:
        repository: public.ecr.aws/dynatrace/dynatrace-otel-collector
        tag: "0.16.0" # Pin to specific version
      resources:
        requests:
          cpu: 100m
          memory: 256Mi
        limits:
          cpu: 500m
          memory: 512Mi
    logMonitoring:
      resources:
        requests:
          cpu: 100m
          memory: 256Mi
        limits:
          cpu: 500m
          memory: 512Mi
      tolerations:
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
          operator: Exists
        - effect: NoSchedule
          key: node-role.kubernetes.io/control-plane
          operator: Exists

  # Enable log monitoring (empty object - resources configured above in templates)
  logMonitoring: {}

  telemetryIngest:
    protocols:
      - jaeger
      - otlp
      - statsd
      - zipkin
    serviceName: telemetry-ingest
```

---

## Operator values.yaml Review

Your Helm values are well-configured with good security practices:

### ✅ Strengths

- **Security contexts** properly configured (non-root, read-only filesystem, dropped capabilities)
- **Seccomp profiles** enabled (RuntimeDefault)
- **Webhook HA** enabled
- **CSI driver** enabled with proper tolerations
- **Resource limits** defined for all components

### ⚠️ Suggestions

| Setting                                 | Current   | Recommendation                              |
| --------------------------------------- | --------- | ------------------------------------------- |
| `webhook.mutatingWebhook.failurePolicy` | `Ignore`  | Consider `Fail` in non-prod to catch issues |
| `csidriver.priorityClassValue`          | `1000000` | Verify this is appropriate for your cluster |

---

## ArgoCD Application Review

Your ArgoCD Applications (`usf-dynatrace-operator.yaml` and `usf-dynatrace-dynakube.yaml`) look standard.

**Suggestion:** Consider adding automated sync and self-heal:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
  syncOptions:
    - CreateNamespace=true
```

---

## Implementation Checklist

### Pre-01/31 Release (Optional Enhancements)

- [ ] **Feature Flags** (if using cloudNativeFullStack)
  - [ ] Add `automatic-injection: "false"` annotation
  - [ ] Add `label-version-detection: "true"` annotation
  - [ ] Add `injection-failure-policy: "fail"` annotation

- [ ] **Namespace Selection** (if using cloudNativeFullStack)
  - [ ] Add `namespaceSelector` to DynaKube spec
  - [ ] Label target namespaces with `dt-monitoring=true`
  - [ ] Verify New Relic namespaces are excluded

- [ ] **Resource Configuration**
  - [ ] Add missing `provisioner` limits in CSI driver `values.yaml`
  - [ ] Add OTel Collector resource limits in DynaKube spec
  - [ ] Pin OTel Collector image version

### Post-01/31 Release (When Convenient)

- [ ] **Build Version Propagation**
  - [ ] Add `app.kubernetes.io/version` labels to application deployments
  - [ ] Add `app.kubernetes.io/name` labels to application deployments
  - [ ] Verify version info appears in Dynatrace

- [ ] **NGINX Ingress Monitoring** (if applicable)
  - [ ] Edit `ingress-nginx-controller` ConfigMap
  - [ ] Add `main-snippet` to load OneAgent module
  - [ ] Rolling restart ingress pods
  - [ ] Validate traces flow through ingress

- [ ] **Log Management** (if needed)
  - [ ] Configure log ingest rules in Dynatrace UI to exclude New Relic namespaces

### Validation

- [ ] Verify Kubernetes metrics flowing to Dynatrace
- [ ] Verify logs appearing in Dynatrace UI
- [ ] If using APM: Verify pods in labeled namespaces are instrumented
- [ ] If using APM: Verify pods in unlabeled namespaces are NOT instrumented

---

## Frequently Asked Questions

### Q1: Can I omit cloudNativeFullStack to avoid monitoring specific namespaces?

**Yes.** Omitting `oneAgent.cloudNativeFullStack` entirely is a valid approach for infrastructure-only monitoring. You'll get:

| ✅ Included               | ❌ Not Included       |
| ------------------------- | --------------------- |
| Kubernetes cluster health | Application-level APM |
| Workload metrics          | Code-level tracing    |
| Pod/container metrics     | Distributed traces    |
| Kubernetes events         | OneAgent injection    |

This is appropriate when New Relic handles all APM and you only need Dynatrace for K8s platform observability.

---

### Q2: Do I need application labels before enabling Build Version Propagation?

**No.** You can enable `label-version-detection: "true"` immediately. If labels are missing, Dynatrace simply won't have version metadata — no errors or failures occur. When you add labels later, Dynatrace automatically picks them up. This is purely additive with zero risk.

---

### Q3: Can NGINX Ingress Monitoring be enabled after the 01/31 release?

**Yes.** NGINX instrumentation is configured via a ConfigMap change on the ingress controller itself — it's independent of the DynaKube CR. You can add it at any time post-release with no impact to your timeline.

---

### Q4: Is explicit log monitoring configuration required? How do I set resource limits?

**Enabling:** Your current `logMonitoring: {}` already enables log monitoring with defaults — this is why logs are flowing to Dynatrace in SIT.

**Resource Limits:** To configure resources for log monitoring, use `spec.templates.logMonitoring`, NOT `spec.logMonitoring`:

```yaml
spec:
  # Just enables the feature (no nested properties allowed)
  logMonitoring: {}

  # Resources go here
  templates:
    logMonitoring:
      resources:
        requests:
          cpu: 100m
          memory: 256Mi
        limits:
          cpu: 500m
          memory: 512Mi
```

> ⚠️ **Common Error:** `spec.logMonitoring.enabled` and `spec.logMonitoring.resources` will fail with "unknown field" validation errors. The `spec.logMonitoring` field only accepts an empty object `{}`.

---

## Additional Resources

- [DynaKube Feature Flags Reference](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/reference/dynakube-feature-flags)
- [Configure Monitoring for Namespaces and Pods](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/guides/deployment-and-configuration/monitoring-and-instrumentation/annotate)
- [NGINX Instrumentation Guide](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/guides/deployment-and-configuration/monitoring-and-instrumentation/instrument-nginx)
- [Dynatrace Operator GitHub](https://github.com/Dynatrace/dynatrace-operator)

---

_Generated by Claude | Review your specific requirements before implementing changes_
