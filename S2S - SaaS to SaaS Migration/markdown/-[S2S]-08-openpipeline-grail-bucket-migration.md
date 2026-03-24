# S2S-08: OpenPipeline and Grail Bucket Migration

> **Series:** S2S | **Notebook:** 8 of 12 | **Created:** March 2026 | **Last Updated:** 03/23/2026

## Overview

OpenPipeline processing rules and Grail bucket definitions control how data is routed, enriched, and retained. These must be migrated as a coordinated unit — deploying routing rules without the target bucket causes data loss.

---

## Table of Contents

1. [Bucket Migration](#bucket-migration)
2. [OpenPipeline Rule Migration](#openpipeline-migration)
3. [Retention Policy Alignment](#retention-alignment)
4. [Coordinated Deployment](#coordinated-deployment)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Target Tenant** | Provisioned with appropriate license for custom buckets |
| **Permissions** | `storage:bucket-definitions:write`, `openpipeline:configurations:write` |

<a id="bucket-migration"></a>
## 1. Bucket Migration

### Export Source Buckets

```bash
# Monaco export
monaco download manifest.yaml --environment source \
  --specific-settings builtin:bucket-definitions

# Or via Terraform
terraform import dynatrace_platform_bucket.my_bucket "<bucket-name>"
```

### Bucket Naming for Consolidation

When consolidating tenants, bucket names may collide:

| Source Tenant | Bucket Name | Target Bucket Name |
|--------------|------------|-------------------|
| US-East | `app_logs` | `us-east_app_logs` |
| EU-West | `app_logs` | `eu-west_app_logs` |
| APAC | `app_logs` | `apac_app_logs` |

> **Alternative:** If consolidating into a single logical bucket, merge routing rules to target one bucket with enrichment tags for filtering.

<a id="openpipeline-migration"></a>
## 2. OpenPipeline Rule Migration

OpenPipeline rules define how logs, metrics, and events are processed:

| Rule Type | What It Does | Migration Notes |
|-----------|-------------|-----------------|
| **Processing rules** | Parse, transform, enrich | Port as-is if field names match |
| **Routing rules** | Route to specific buckets | Update bucket references to target names |
| **Filtering rules** | Drop or sample data | Port as-is |
| **Extraction rules** | Extract fields from content | Port as-is |

<a id="retention-alignment"></a>
## 3. Retention Policy Alignment

| Environment | Source Retention | Target Retention | Action |
|-------------|-----------------|-----------------|--------|
| Production | 360 days (PCI) | 360 days | Match exactly |
| Staging | 35 days | 35 days | Match or reduce |
| Development | 7 days | 7 days | Match or reduce |

> **Compliance Note:** For regulated industries, verify that retention policies in the target tenant meet all compliance requirements before cutover.

<a id="coordinated-deployment"></a>
## 4. Coordinated Deployment

Deploy in this order:

1. **Buckets first** — create all target buckets
2. **Enrichment rules** — ensure tags are populating
3. **OpenPipeline rules** — route data to new buckets
4. **Verify** — confirm data flows to correct buckets

```bash
# Deploy all three in one Monaco command
monaco deploy manifest.yaml --environment target
```

---

## Next Steps

Continue to **S2S-09: Data Continuity and Parallel Operation** to plan the dual-tenant transition period.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
