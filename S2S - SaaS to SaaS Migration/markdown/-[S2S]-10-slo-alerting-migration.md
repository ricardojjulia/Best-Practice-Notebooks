# S2S-10: SLO and Alerting Migration

> **Series:** S2S | **Notebook:** 10 of 12 | **Created:** March 2026 | **Last Updated:** 03/23/2026

## Overview

SLOs and alerting are the core of production monitoring. This notebook covers migrating SLO definitions, notification rules, and alerting configurations while handling entity ID references and evaluation window gaps.

---

## Table of Contents

1. [SLO Migration Strategy](#slo-strategy)
2. [Notification Rule Migration](#notification-rules)
3. [Alerting Profile Migration](#alerting-profiles)
4. [Maintenance Window Migration](#maintenance-windows)
5. [Validation Queries](#validation-queries)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Target Tenant** | Agents reporting, data flowing |
| **Exported Config** | SLOs and alerting from S2S-02 |
| **Tags** | Auto-tags and enrichment configured in target |

<a id="slo-strategy"></a>
## 1. SLO Migration Strategy

### Converting Entity ID Filters to Tags

Most SLOs reference entities by ID. Convert to tags before migration:

| Source SLO Filter | Target SLO Filter | Why |
|-------------------|-------------------|-----|
| `type(SERVICE),entityId(SERVICE-xxx)` | `type(SERVICE),tag(app:checkout)` | Portable across tenants |
| `type(HOST),entityId(HOST-xxx)` | `type(HOST),tag(env:production)` | Works in any tenant |
| `type(SERVICE),mzId(123)` | `type(SERVICE),mzName("Production")` | Management zone IDs change |

### Terraform SLO Import

```hcl
resource "dynatrace_slo_v2" "checkout_availability" {
  name               = "Checkout Availability"
  enabled            = true
  evaluation_type    = "AGGREGATE"
  filter             = "type(SERVICE),tag(app:checkout),tag(env:production)"
  target             = 99.5
  warning            = 99.9
  evaluation_window  = "ROLLING_WEEK"
  metric_expression  = "100*(builtin:service.errors.server.successCount:splitBy())/(builtin:service.requestCount.server:splitBy())"
}
```

<a id="notification-rules"></a>
## 2. Notification Rule Migration

Export from source and apply to target:

```hcl
resource "dynatrace_alerting" "critical" {
  name = "Critical Alerts"
  rules {
    severity_level   = "AVAILABILITY"
    delay_in_minutes = 0
    tags {
      filter {
        context = "CONTEXTLESS"
        key     = "env"
        value   = "production"
      }
    }
  }
}
```

<a id="alerting-profiles"></a>
## 3. Alerting Profile Migration

| Configuration | Migration Notes |
|--------------|-----------------|
| Severity filters | Port as-is |
| Tag filters | Ensure tags exist in target |
| Delay settings | Port as-is |
| Event filters | Port as-is |

<a id="maintenance-windows"></a>
## 4. Maintenance Window Migration

Maintenance windows are timezone-sensitive:

| Setting | Verify |
|---------|--------|
| Schedule (cron) | Same timezone in target |
| Scope | Update entity references to target |
| Suppression type | Port as-is |

<a id="validation-queries"></a>
## 5. Validation Queries

Verify SLOs are evaluating in the target tenant:

```dql
// Check SLO status in target tenant
fetch logs, from:-1h
| filter matchesPhrase(log.source, "audit")
| filter contains(content, "slo")
| fields timestamp, content
| sort timestamp desc
| limit 20
```

---

## Next Steps

Continue to **S2S-11: Cutover Execution and Validation** for the final migration checklist.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
