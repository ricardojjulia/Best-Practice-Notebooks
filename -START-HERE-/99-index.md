# Series Catalog & Cross-Reference

> **Purpose:** Full inventory of all 28 Dynatrace Best Practice Topic series with cross-references between them. Use this as a quick lookup when you need to find which series covers a specific topic, or to see related material across series.
> **Last Updated:** 05/07/2026

![Series by Category](images/99-series-by-category.svg)

---

## Table of Contents

1. [Alphabetical Catalog](#alphabetical-catalog) — All 28 series with one-line descriptions
2. [By Category](#by-category) — Series grouped by their role in the journey
3. [By Entry Point](#by-entry-point) — Which doorway uses each series
4. [Cross-Reference Matrix](#cross-reference-matrix) — For each series, related series to read
5. [Reading-Order Presets](#reading-order-presets) — "I have 1 hour / 1 week / 1 month / 1 quarter"

---

## Alphabetical Catalog

| Code | Series | Notebooks | Focus |
|---|---|---|---|
| **ADOPT** | [Observability Adoption & Maturity](../adopt/) | 6 | Maturity model, success metrics, optimization roadmap, FinOps |
| **AIOPS** | [Dynatrace Intelligence](../aiops/) | 8 | Davis problems, anomaly detection, generative AI, agentic workflows |
| **AUTOM** | [Dynatrace Automation](../autom/) | 11 | Settings API, Monaco, Terraform, workflows-as-code, GitOps, CI/CD |
| **BIZEV** | [Business Events & Funnel Analysis](../bizev/) | 7 | Business event ingestion, conversion funnels, revenue impact, executive reporting |
| **CLOUD** | [Cloud Provider Integrations](../cloud/) | 9 | AWS, Azure, GCP integrations; Lambda, EKS, multi-cloud patterns |
| **DASH** | [Dashboard Design & Building](../dash/) | 8 | Dashboard hierarchy, executive/operations/engineering audiences, sharing and reporting |
| **DBMON** | [Database Monitoring](../dbmon/) | 7 | SQL, NoSQL, cache, messaging, query analysis |
| **FAQ** | [Frequently Asked Questions](../faq/) | 2+ | Standalone single-page reference docs (host-group naming, tagging) — growing |
| **IAM** | [IAM Administration](../iam/) | 13 | Policies, boundaries, groups, SSO, audit, parameterized assignments |
| **K8S** | [Kubernetes Monitoring](../k8s/) | 15 | DynaKube, GitOps deployment, cluster + workload monitoring, troubleshooting |
| **M2S** | [Managed to SaaS Migration](../m2s/) | 10 | 9-step procedural runbook for Managed → SaaS deployment migration |
| **MOBL** | [Mobile Monitoring](../mobl/) | 13 | iOS, Android, cross-platform SDKs; crash reporting, session replay, privacy |
| **MZ2POL** | [Management Zone to Policy Migration](../mz2pol/) | 10 | MZ analysis, Gen2 → Gen3 access control migration |
| **NR2DT** | [New Relic to Dynatrace Migration Steps](../nr2dt/) | 11 | Procedural runbook (00 prereqs + 9 steps + summary); refers to NRLC for component depth |
| **NRLC** | [New Relic to Dynatrace Migration Deep Dives](../nrlc/) | 9 | Standalone component deep dives (NRQL→DQL, dashboards, alerts, synthetics, SLOs, logs) |
| **ONBRD** | [Dynatrace Onboarding](../onbrd/) | 11 | First steps, IAM, ActiveGate, OneAgent, basic data org and dashboards |
| **OPIPE** | [OpenPipeline Beyond Logs](../opipe/) | 7 | Spans, metrics, business and security event pipelines; cross-scope design patterns |
| **OPLOGS** | [OpenPipeline Logs](../oplogs/) | 9 | Log fundamentals, processing, buckets, parsing, security |
| **OPMIG** | [OpenPipeline Migration](../opmig/) | 10 | Classic Logs → OpenPipeline migration runbook |
| **ORGNZ** | [Organize Data: Buckets, Segments, Security](../orgnz/) | 11 | Buckets, security_context, permissions, segments, enterprise patterns |
| **OTEL** | [OpenTelemetry Integration](../otel/) | 9 | OTel collector deployment, trace/metric/log instrumentation, Dynatrace integration |
| **S2D** | [Splunk to Dynatrace Migration](../s2d/) | 10 | SPL → DQL, anomaly detectors, dashboards, naming standards |
| **S2S** | [SaaS to SaaS Migration](../s2s/) | 11 | 9-step runbook for SaaS → SaaS tenant consolidation; includes migration scripts |
| **SL2DT** | [Sumo Logic to Dynatrace](../sl2dt/) | 10 | Sumo procedural runbook; logs/dashboards/monitors; Gen3-first |
| **SPANS** | [Distributed Tracing and Spans](../spans/) | 9 | Span fundamentals, querying, topology, analytics, cost optimization |
| **SYNTH** | [Synthetic Monitoring](../synth/) | 7 | Browser monitors, HTTP monitors, private locations, network monitoring |
| **WEBRUM** | [Web Real User Monitoring](../webrum/) | 9 | RUM fundamentals, SPA, Core Web Vitals, session replay |
| **WFLOW** | [Workflows and Alert Notifications](../wflow/) | 10 | Workflow triggers, notification routing, incident management, JS/HTTP actions, governance |

---

## By Category

### Foundation (Universal)

Every customer needs these regardless of entry point.

- [ONBRD](../onbrd/) — Tenant orientation, OneAgent, ActiveGate
- [ORGNZ](../orgnz/) — Buckets, security_context, segments
- [IAM](../iam/) — Policies, groups, boundaries

### Migration (Pick Based on Source)

These are entry points for customers leaving another tool or migrating their Dynatrace deployment.

**Tool migration (typically with a net-new Dynatrace tenant):**

- [NRLC](../nrlc/) + [NR2DT](../nr2dt/) — From New Relic
- [S2D](../s2d/) — From Splunk
- [SL2DT](../sl2dt/) — From Sumo Logic
- [OPMIG](../opmig/) — Classic Logs → OpenPipeline (internal Dynatrace migration)
- [MZ2POL](../mz2pol/) — Management Zones → Policies (internal Dynatrace migration)

**Deployment migration (existing Dynatrace customer):**

- [M2S](../m2s/) — Managed → SaaS
- [S2S](../s2s/) — SaaS → SaaS

### Domain (Pick Based on What You Monitor)

- [K8S](../k8s/) — Kubernetes
- [CLOUD](../cloud/) — AWS, Azure, GCP
- [SPANS](../spans/) — Distributed tracing
- [WEBRUM](../webrum/) — Web Real User Monitoring
- [MOBL](../mobl/) — Mobile (iOS, Android)
- [DBMON](../dbmon/) — Databases
- [BIZEV](../bizev/) — Business events
- [SYNTH](../synth/) — Synthetic monitoring

### Ingestion (Cross-Cutting Mechanism)

- [OTEL](../otel/) — OpenTelemetry collector and instrumentation
- [OPLOGS](../oplogs/) — Log processing in OpenPipeline
- [OPIPE](../opipe/) — OpenPipeline for spans, metrics, business and security events

### Operationalize (Day-2 Operations)

Read in order; each builds on the previous.

- [DASH](../dash/) — Dashboards
- [WFLOW](../wflow/) — Workflows and alerting
- [AUTOM](../autom/) — Configuration automation
- [AIOPS](../aiops/) — Davis intelligence and AI workflows

### Maturity & Reference

- [ADOPT](../adopt/) — Continuous improvement framework
- [FAQ](../faq/) — Standalone reference docs

---

## By Entry Point

This table shows which doorway in the playbook uses each series and how it appears (Primary = first-class entry; Refresh = revisit only where Gen3 or scope changed; — = not used).

| Series | Net New | Expanding / Consolidating | Deployment Migration |
|---|---|---|---|
| ONBRD | Primary | Refresh as needed | Refresh as needed |
| ORGNZ | Primary | Primary (Gen3 changes) | Primary (Gen3 changes) |
| IAM | Primary | Primary (Gen3 changes) | Primary (Gen2 → Gen3) |
| NRLC | If from New Relic | If consolidating APM from NR | — |
| NR2DT | If from New Relic | — | — |
| S2D | If from Splunk | If consolidating logs from Splunk | — |
| SL2DT | If from Sumo Logic | If consolidating logs from Sumo | — |
| M2S | — | — | Primary (Managed → SaaS) |
| S2S | — | — | Primary (SaaS → SaaS) |
| OPMIG | — | If on Classic Logs | — |
| MZ2POL | — | If migrating Gen2 access control | If migrating Gen2 MZs |
| K8S, CLOUD, SPANS, WEBRUM, MOBL, DBMON, BIZEV, SYNTH | After Foundation | Primary for "adding a domain" | After deployment migration |
| OTEL, OPLOGS, OPIPE | As ingestion needs arise | As ingestion needs arise | As ingestion needs arise |
| DASH, WFLOW, AUTOM, AIOPS | After first domain | Primary for "maturing operations" | After deployment migration |
| ADOPT | Ongoing | Ongoing | Ongoing |
| FAQ | Reference | Reference | Reference |

---

## Cross-Reference Matrix

For each series, the related series most often read alongside it:

| If you are reading… | Also see… |
|---|---|
| [ADOPT](../adopt/) | [DASH](../dash/), [ORGNZ](../orgnz/) (cost optimization), [BIZEV](../bizev/) (success metrics) |
| [AIOPS](../aiops/) | [WFLOW](../wflow/) (alert routing), [DASH](../dash/) (visualizing problems), [AUTOM](../autom/) (workflow-as-code) |
| [AUTOM](../autom/) | [WFLOW](../wflow/) (workflow-as-code overlap), [IAM](../iam/) (provisioning), [K8S](../k8s/) (GitOps), [ONBRD](../onbrd/) (Settings API basics) |
| [BIZEV](../bizev/) | [OPIPE](../opipe/) (event pipelines), [DASH](../dash/) (executive reporting), [WEBRUM](../webrum/) (frontend events) |
| [CLOUD](../cloud/) | [OTEL](../otel/) (collectors), [K8S](../k8s/) (managed K8s), [OPLOGS](../oplogs/) (CloudWatch ingestion) |
| [DASH](../dash/) | [WFLOW](../wflow/), [BIZEV](../bizev/) (executive dashboards), [ORGNZ](../orgnz/) (segments and filters) |
| [DBMON](../dbmon/) | [SPANS](../spans/) (database call tracing), [DASH](../dash/) (database dashboards) |
| [FAQ](../faq/) | [ORGNZ](../orgnz/) (tagging), [ONBRD](../onbrd/) (host groups) |
| [IAM](../iam/) | [ORGNZ](../orgnz/) (security_context for boundaries), [MZ2POL](../mz2pol/) (Gen2 → Gen3), [ONBRD](../onbrd/) (initial setup) |
| [K8S](../k8s/) | [OTEL](../otel/) (collector deployment), [CLOUD](../cloud/) (managed K8s), [AUTOM](../autom/) (GitOps), [OPLOGS](../oplogs/) (K8s logs) |
| [M2S](../m2s/) | [ORGNZ](../orgnz/), [IAM](../iam/), [MZ2POL](../mz2pol/) (Gen2 → Gen3) |
| [MOBL](../mobl/) | [SPANS](../spans/) (mobile-to-backend tracing), [DASH](../dash/) (mobile dashboards) |
| [MZ2POL](../mz2pol/) | [IAM](../iam/), [ORGNZ](../orgnz/) |
| [NR2DT](../nr2dt/) | [NRLC](../nrlc/) (component depth), [ONBRD](../onbrd/), [OPLOGS](../oplogs/) (logs migration) |
| [NRLC](../nrlc/) | [NR2DT](../nr2dt/) (procedural runbook), [SPANS](../spans/), [DASH](../dash/), [WFLOW](../wflow/), [SYNTH](../synth/) |
| [ONBRD](../onbrd/) | [ORGNZ](../orgnz/), [IAM](../iam/), [DASH](../dash/), [WFLOW](../wflow/) |
| [OPIPE](../opipe/) | [SPANS](../spans/), [BIZEV](../bizev/), [OPLOGS](../oplogs/), [ORGNZ](../orgnz/) (buckets) |
| [OPLOGS](../oplogs/) | [OPMIG](../opmig/) (migration), [OPIPE](../opipe/), [ORGNZ](../orgnz/) (buckets) |
| [OPMIG](../opmig/) | [OPLOGS](../oplogs/), [OPIPE](../opipe/), [ORGNZ](../orgnz/) |
| [ORGNZ](../orgnz/) | [IAM](../iam/), [FAQ](../faq/) (tagging), [MZ2POL](../mz2pol/) |
| [OTEL](../otel/) | [K8S](../k8s/), [CLOUD](../cloud/), [SPANS](../spans/), [NRLC](../nrlc/) |
| [S2D](../s2d/) | [OPLOGS](../oplogs/), [OPIPE](../opipe/), [DASH](../dash/), [WFLOW](../wflow/), [ORGNZ](../orgnz/) |
| [S2S](../s2s/) | [ORGNZ](../orgnz/), [IAM](../iam/), [MZ2POL](../mz2pol/) |
| [SL2DT](../sl2dt/) | [OPLOGS](../oplogs/), [OPIPE](../opipe/), [DASH](../dash/), [WFLOW](../wflow/), [ORGNZ](../orgnz/) |
| [SPANS](../spans/) | [OTEL](../otel/), [OPIPE](../opipe/) (span pipelines), [DBMON](../dbmon/) (DB tracing) |
| [SYNTH](../synth/) | [DASH](../dash/), [WFLOW](../wflow/) (alerting on synthetic) |
| [WEBRUM](../webrum/) | [SPANS](../spans/) (frontend-to-backend tracing), [BIZEV](../bizev/), [DASH](../dash/) |
| [WFLOW](../wflow/) | [AIOPS](../aiops/) (problem routing), [AUTOM](../autom/) (workflow-as-code), [DASH](../dash/) |

---

## Reading-Order Presets

Quick lists for time-boxed reading:

### "I have 1 hour" — fastest orientation

1. This document — sections [Alphabetical Catalog](#alphabetical-catalog) and [By Category](#by-category)
2. [README front door](README.md) — pick a doorway based on your situation
3. [ONBRD](../onbrd/) — first 1–2 notebooks for orientation

### "I have 1 week" — Foundation only

1. [ONBRD](../onbrd/) — full series (11 notebooks)
2. [ORGNZ](../orgnz/) — first 5 notebooks (introduction, buckets, bucket strategy, Grail permissions, bucket-level access)
3. [IAM](../iam/) — first 5 notebooks (governance, SSO, group architecture, policies, boundaries)

Aligned with [Foundation Module](04-foundation.md).

### "I have 1 month" — Foundation + first domain + light operationalize

1. [Foundation Module](04-foundation.md) reading order
2. One domain — typically [K8S](../k8s/) or [CLOUD](../cloud/) depending on environment
3. [DASH](../dash/) — first 4 notebooks (fundamentals, hierarchy, executive, operations)
4. [WFLOW](../wflow/) — first 4 notebooks (fundamentals, triggers, notifications basics, notification routing)

### "I have 1 quarter" — full Net New track

1. [Net New Doorway](01-net-new.md) — full reading order across all phases
2. [Domain Enablement Module](05-domain-enablement.md) — at least 2 domains
3. [Operationalize Module](06-operationalize.md) — DASH → WFLOW → AUTOM
4. [Maturity Module](07-maturity.md) — kick off

---

> *This playbook was AI-generated from community-submitted and publicly available sources. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*
