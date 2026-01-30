# Troubleshooting Kubernetes Monitoring

> **Series:** K8S | **Notebook:** 9 of 12 | **Created:** January 2026 | **Last Updated:** 01/30/2026

## Debugging Dynatrace Monitoring in Kubernetes
When monitoring doesn't work as expected, systematic troubleshooting is essential. This notebook covers common issues, diagnostic procedures, and resolution steps for Dynatrace Kubernetes monitoring.

---

## Table of Contents

1. [Troubleshooting Methodology](#troubleshooting-methodology)
2. [Operator Issues](#operator-issues)
3. [OneAgent Issues](#oneagent-issues)
4. [ActiveGate Issues](#activegate-issues)
5. [Injection Issues](#injection-issues)
6. [Data Collection Issues](#data-collection-issues)
7. [Connectivity Issues](#connectivity-issues)
8. [Diagnostic Commands](#diagnostic-commands)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS/Managed access |
| **kubectl** | Configured for your cluster |
| **Permissions** | Cluster admin or debug access |
| **Knowledge** | K8S-01 through K8S-03 |

<a id="troubleshooting-methodology"></a>
## 1. Troubleshooting Methodology
### Systematic Approach

![Troubleshooting Methodology](images/troubleshooting-methodology.svg)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Step | Action | Purpose |
|------|--------|---------|
| 1 | Identify symptom | What's wrong? |
| 2 | Check status | Pods running? |
| 3 | Review logs/events | Error messages |
| 4 | Verify config | DynaKube YAML |
| 5 | Test connectivity | curl endpoints |
| 6 | Apply fix & validate | Fix, verify, document |
For environments where SVG doesn't render
-->

### Common Symptom Categories

| Symptom | Likely Component | Start Here |
|---------|------------------|------------|
| No cluster data | ActiveGate | Section 4 |
| No node/pod data | OneAgent | Section 3 |
| No app traces | Injection | Section 5 |
| Partial data | Data collection | Section 6 |
| Operator errors | Operator | Section 2 |

### First Checks

```bash
# DynaKube status
kubectl -n dynatrace get dynakube

# All Dynatrace pods
kubectl -n dynatrace get pods

# Recent events
kubectl -n dynatrace get events --sort-by='.lastTimestamp' | tail -20
```

<a id="operator-issues"></a>
## 2. Operator Issues
### Operator Not Running

**Symptoms:**
- DynaKube CR shows no status
- OneAgent pods not created

**Diagnostic:**
```bash
# Check operator deployment
kubectl -n dynatrace get deployment dynatrace-operator

# Check operator logs
kubectl -n dynatrace logs -l app.kubernetes.io/name=dynatrace-operator --tail=100
```

**Common Causes:**

| Cause | Resolution |
|-------|------------|
| Missing CRDs | Reinstall operator with Helm |
| RBAC issues | Check ServiceAccount permissions |
| Resource limits | Increase operator resources |
| Webhook failure | Check webhook certificates |

### DynaKube Stuck in Deploying

**Diagnostic:**
```bash
# Describe DynaKube
kubectl -n dynatrace describe dynakube dynakube

# Check conditions
kubectl -n dynatrace get dynakube -o jsonpath='{.items[*].status.conditions}' | jq .
```

**Common Causes:**

| Cause | Resolution |
|-------|------------|
| Invalid API token | Regenerate and update secret |
| Network policy blocking | Allow egress to Dynatrace |
| Image pull failure | Check pull secrets |

<a id="oneagent-issues"></a>
## 3. OneAgent Issues
### OneAgent Pods Not Starting

**Diagnostic:**
```bash
# Check DaemonSet
kubectl -n dynatrace get daemonset -l app.kubernetes.io/component=oneagent

# Check pods
kubectl -n dynatrace get pods -l app.kubernetes.io/component=oneagent -o wide

# Describe failing pod
kubectl -n dynatrace describe pod <oneagent-pod-name>
```

**Common Causes:**

| Cause | Resolution |
|-------|------------|
| Tolerations missing | Add tolerations to DynaKube |
| Node selector mismatch | Update node selectors |
| Resource unavailable | Check node resources |
| Security context denied | Configure PSP/PSS |

### OneAgent Not Reporting

**Diagnostic:**
```bash
# Check OneAgent logs
kubectl -n dynatrace logs <oneagent-pod> -c oneagent --tail=100

# Check connectivity
kubectl -n dynatrace exec <oneagent-pod> -c oneagent -- curl -v https://<tenant>.live.dynatrace.com/api/v1/deployment/installer/agent/connectioninfo
```

**Common Causes:**

| Cause | Resolution |
|-------|------------|
| Firewall blocking | Allow egress port 443 |
| Proxy misconfiguration | Set proxy in DynaKube |
| Certificate issues | Check custom CA config |

<a id="activegate-issues"></a>
## 4. ActiveGate Issues
### ActiveGate Not Starting

**Diagnostic:**
```bash
# Check StatefulSet
kubectl -n dynatrace get statefulset -l app.kubernetes.io/component=activegate

# Check pods
kubectl -n dynatrace get pods -l app.kubernetes.io/component=activegate

# Check logs
kubectl -n dynatrace logs <activegate-pod> --tail=100
```

**Common Causes:**

| Cause | Resolution |
|-------|------------|
| Memory too low | Increase resources in DynaKube |
| PVC issues | Check storage class |
| Token invalid | Regenerate API token |

### No Kubernetes Metrics

**Diagnostic:**
```bash
# Check capabilities
kubectl -n dynatrace get dynakube -o jsonpath='{.items[*].spec.activeGate.capabilities}'

# Should include: kubernetes-monitoring
```

**Resolution:** Ensure `kubernetes-monitoring` capability is enabled:

```yaml
spec:
  activeGate:
    capabilities:
      - kubernetes-monitoring
      - routing
```

<a id="injection-issues"></a>
## 5. Injection Issues
### Pods Not Getting Injected

**Diagnostic:**
```bash
# Check namespace labels
kubectl get namespace <namespace> -o jsonpath='{.metadata.labels}'

# Check pod annotations
kubectl get pod <pod> -o jsonpath='{.metadata.annotations}'

# Check webhook
kubectl get mutatingwebhookconfigurations
```

**Common Causes:**

| Cause | Resolution |
|-------|------------|
| Namespace not selected | Add matching labels |
| Pod has opt-out annotation | Remove annotation |
| Webhook not registered | Restart operator |
| CSI driver not mounted | Enable CSI in DynaKube |

### Check Injection Status

```bash
# Check if init container present
kubectl get pod <pod> -o jsonpath='{.spec.initContainers[*].name}'

# Check environment variables
kubectl exec <pod> -c <container> -- env | grep DT_
```

### Namespace Selector Debugging

If using `namespaceSelector` in DynaKube:

```bash
# DynaKube selector
kubectl -n dynatrace get dynakube -o jsonpath='{.items[*].spec.namespaceSelector}'

# Label a namespace for injection
kubectl label namespace <namespace> dynatrace-injection=enabled
```

<a id="data-collection-issues"></a>
## 6. Data Collection Issues
### Missing Metrics

**Diagnostic Steps:**

1. Verify entity exists in Dynatrace
2. Check metric availability for entity type
3. Verify time range includes data
4. Check for filtering issues

```dql
// Verify Kubernetes entities are being discovered
fetch dt.entity.kubernetes_cluster
| fields entity.name, lastSeenTms
| sort lastSeenTms desc
```

```dql
// Check if container metrics are flowing
timeseries cpu = avg(dt.containers.cpu.usage_percent)
| limit 1
```

```dql
// Check if logs are being ingested
fetch logs, from: now() - 1h
| filter isNotNull(k8s.namespace.name)
| summarize logCount = count()
```

### Missing Logs

**Diagnostic:**

```bash
# Check if log analytics is enabled
kubectl -n dynatrace get dynakube -o yaml | grep -A5 "env:"

# Verify OneAgent log collection
kubectl -n dynatrace exec <oneagent-pod> -c oneagent -- cat /var/lib/dynatrace/oneagent/log/oneagent.log | tail -50
```

**Common Causes:**

| Cause | Resolution |
|-------|------------|
| Log ingest disabled | Enable in tenant settings |
| Log volume too high | Configure sampling/filtering |
| Log format unsupported | Configure custom parsing |

<a id="connectivity-issues"></a>
## 7. Connectivity Issues
### Testing Connectivity

```bash
# From OneAgent pod
kubectl -n dynatrace exec <oneagent-pod> -c oneagent -- \
  curl -v https://<tenant>.live.dynatrace.com/api/v1/time

# From ActiveGate pod
kubectl -n dynatrace exec <activegate-pod> -- \
  curl -v https://<tenant>.live.dynatrace.com/api/v1/time
```

### Network Policy Issues

```yaml
# Allow Dynatrace egress
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dynatrace-egress
  namespace: dynatrace
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
      ports:
        - protocol: TCP
          port: 443
```

### Proxy Configuration

```yaml
spec:
  proxy:
    value: https://proxy.example.com:8080
  # Or use a secret
  # proxy:
  #   valueFrom:
  #     secretKeyRef:
  #       name: proxy-secret
  #       key: proxy
```

<a id="diagnostic-commands"></a>
## 8. Diagnostic Commands
### Quick Health Check

```bash
#!/bin/bash
# dynatrace-health-check.sh

echo "=== DynaKube Status ==="
kubectl -n dynatrace get dynakube

echo "\n=== Dynatrace Pods ==="
kubectl -n dynatrace get pods

echo "\n=== OneAgent DaemonSet ==="
kubectl -n dynatrace get daemonset -l app.kubernetes.io/component=oneagent

echo "\n=== ActiveGate StatefulSet ==="
kubectl -n dynatrace get statefulset -l app.kubernetes.io/component=activegate

echo "\n=== Recent Events ==="
kubectl -n dynatrace get events --sort-by='.lastTimestamp' | tail -10
```

### Collect Support Bundle

```bash
# Create support bundle
mkdir dynatrace-debug
cd dynatrace-debug

# DynaKube
kubectl -n dynatrace get dynakube -o yaml > dynakube.yaml

# Pods
kubectl -n dynatrace get pods -o yaml > pods.yaml

# Events
kubectl -n dynatrace get events > events.txt

# Operator logs
kubectl -n dynatrace logs -l app.kubernetes.io/name=dynatrace-operator > operator.log

# OneAgent logs (one pod)
kubectl -n dynatrace logs $(kubectl -n dynatrace get pods -l app.kubernetes.io/component=oneagent -o jsonpath='{.items[0].metadata.name}') -c oneagent > oneagent.log

# Package
cd ..
tar -czf dynatrace-debug.tar.gz dynatrace-debug/
```

### Common kubectl Commands

| Command | Purpose |
|---------|----------|
| `kubectl -n dynatrace get all` | All resources |
| `kubectl -n dynatrace describe dynakube` | DynaKube details |
| `kubectl -n dynatrace logs <pod>` | Pod logs |
| `kubectl -n dynatrace exec -it <pod> -- sh` | Shell access |
| `kubectl -n dynatrace port-forward <pod> 9999:9999` | Local port forward |

## Summary

In this notebook, you learned:

- Systematic troubleshooting methodology
- Operator debugging and common issues
- OneAgent troubleshooting steps
- ActiveGate diagnostics
- Code injection debugging
- Data collection issue resolution
- Connectivity testing and network policy configuration
- Diagnostic commands for support bundle collection

---

## References

- [Dynatrace Operator Troubleshooting](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/troubleshooting)
- [OneAgent Troubleshooting](https://docs.dynatrace.com/docs/ingest-from/setup-on-k8s/troubleshooting/oneagent-troubleshooting)
- [Kubernetes Monitoring FAQ](https://docs.dynatrace.com/docs/observe/infrastructure-monitoring/kubernetes-and-openshift-monitoring/kubernetes-monitoring-faq)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
