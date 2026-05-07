# Overlap Map

> **Purpose:** Catalog of where multiple series cover the same ground, with recommendations for which is canonical and the suggested reading order. Use this when you find yourself reading similar material across two or more series and want to dedupe.
> **Last Updated:** 05/07/2026

---

## Table of Contents

1. [How to Use This Map](#how-to-use-this-map)
2. [Foundation Overlaps](#foundation-overlaps)
3. [Migration Overlaps](#migration-overlaps)
4. [Ingestion Overlaps](#ingestion-overlaps)
5. [Operationalize Overlaps](#operationalize-overlaps)
6. [Domain Overlaps](#domain-overlaps)
7. [Cross-Category Overlaps](#cross-category-overlaps)
8. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)

---

## How to Use This Map

When you encounter the same topic in multiple series, the table below tells you:

- **Canonical** — the series that goes deepest on the topic; treat as the source of truth
- **Also covers** — series that touch on the topic with lighter coverage; useful for context but not depth
- **Recommended reading order** — when more than one applies to your situation

This is reference material, not a curriculum. Use it as a lookup when reading rather than a sequence to follow end-to-end.

---

## Foundation Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| First-user setup and authentication | [IAM](../iam/) — notebooks 02 (SSO), 03 (groups) | [ONBRD](../onbrd/) — notebook 02 (light intro) | ONBRD-02 first for orientation; IAM-02..03 for depth |
| Tagging strategy | [FAQ](../faq/) — entry 02 (sources, standards, strategy) for the framework; [ORGNZ](../orgnz/) — notebook 01 for organizational concepts | [ONBRD](../onbrd/) — notebook 06 (basic tags); host-group naming in [FAQ](../faq/) entry 01 | FAQ-02 first for the framework; then ORGNZ-01; reference ONBRD-06 only if you need the basics |
| Bucket basics | [ORGNZ](../orgnz/) — notebooks 02 (understanding), 03 (strategy) | [ONBRD](../onbrd/) — notebook 06 (mention only) | ORGNZ-02..03 always — ONBRD's coverage is too light |
| Bucket access control | [ORGNZ](../orgnz/) — notebook 05 (bucket-level access control) | [IAM](../iam/) — notebook 05 (boundary design references it) | Read ORGNZ-05 only if scenario applies (compliance, retention, hard cost separation) |
| security_context | [ORGNZ](../orgnz/) — notebook 06 | [IAM](../iam/) — notebooks 04, 05 use it for boundaries | ORGNZ-06 first — IAM boundary work depends on it |
| Policy authoring | [IAM](../iam/) — notebooks 04 (policy authoring), 05 (boundary design), 10 (parameterized) | [ONBRD](../onbrd/) — notebook 02 (basic policy intro) | IAM-04..05 always; ONBRD-02 only for orientation |
| Gen2 → Gen3 access control | [MZ2POL](../mz2pol/) — full series | (overview only in [ONBRD](../onbrd/), [IAM](../iam/)) | MZ2POL only if migrating from Gen2; otherwise skip |
| Audit log queries | [IAM](../iam/) — notebook 07 (audit and compliance) | [ORGNZ](../orgnz/) — notebook 07 (advanced patterns) | IAM-07 for compliance lens; ORGNZ-07 for general patterns |

---

## Migration Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| NR component-level translation | [NRLC](../nrlc/) — full series | [NR2DT](../nr2dt/) references NRLC at each step | NRLC for depth; NR2DT for procedural sequencing |
| NR procedural runbook | [NR2DT](../nr2dt/) — 9-step runbook | (none) | NR2DT is the only procedural source |
| NRQL → DQL translation | [NRLC](../nrlc/) — notebook 02 | [NR2DT](../nr2dt/) — notebook 04 (translate) | NRLC-02 for the mappings; NR2DT-04 for engagement-level translation work |
| Splunk SPL → DQL | [S2D](../s2d/) — notebook 03 | (none) | S2D-03 only |
| SumoQL → DQL | [SL2DT](../sl2dt/) — notebook 04 | (none) | SL2DT-04 only |
| Logs migration architecture | [OPLOGS](../oplogs/) — notebooks 02 (migration), 03 (pipeline); [OPIPE](../opipe/) — notebook 01 | [S2D](../s2d/) — notebook 02 (Splunk locating); [SL2DT](../sl2dt/) — notebook 03 (Sumo ingest) | Source-tool series first for inventory; OPLOGS for ingestion design |
| Classic Logs → OpenPipeline | [OPMIG](../opmig/) — full series | [OPLOGS](../oplogs/) — notebook 02 (migration) | OPMIG for migration-specific; OPLOGS-02 if general |
| Static-threshold to anomaly detection | [AIOPS](../aiops/) — notebook 02 (anomaly detection) | [S2D](../s2d/) — notebook 04 (anomaly detectors); [SL2DT](../sl2dt/) — notebook 05 | AIOPS-02 for the framework; S2D and SL2DT for tool-specific translation patterns |

---

## Ingestion Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| OTel collector deployment | [OTEL](../otel/) — notebooks 02 (architecture), 03 (deployment) | [K8S](../k8s/) — chapters in 04, 05; [CLOUD](../cloud/) — chapters in cloud-specific notebooks | OTEL-02..03 first; domain-specific chapters for context |
| OTel trace instrumentation | [OTEL](../otel/) — notebook 04 | [SPANS](../spans/) — notebook 01 mentions OTel; [NRLC](../nrlc/) — for migration context | OTEL-04 always; SPANS-01 for the data model afterwards |
| Log pipeline processing | [OPLOGS](../oplogs/) — notebook 03 | [OPMIG](../opmig/) — notebook 06 (processing/parsing); [OPIPE](../opipe/) — notebook 01 | OPLOGS-03 first; OPMIG-06 for migration-specific transformations |
| Bucket-routing decisions | [ORGNZ](../orgnz/) — notebooks 02, 03 | [OPLOGS](../oplogs/) — notebook 04 (buckets governance); [OPMIG](../opmig/) — notebook 05 (routing buckets) | ORGNZ-02..03 first; OPLOGS-04 / OPMIG-05 for ingestion-specific routing |
| Cardinality management | [OPIPE](../opipe/) — notebook 04 | [SPANS](../spans/) — notebook 08 (cost optimization) | OPIPE-04 for the mechanism; SPANS-08 for span-specific cost work |
| Span pipelines | [OPIPE](../opipe/) — notebook 02 | [SPANS](../spans/) — notebook 07 (buckets and pipeline) | OPIPE-02 first; SPANS-07 for span-specific patterns |

---

## Operationalize Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| Davis problems and root cause | [AIOPS](../aiops/) — notebook 03 | [WFLOW](../wflow/) — notebook 04 (alert routing on problems) | AIOPS-03 first; WFLOW-04 for routing |
| Anomaly detection | [AIOPS](../aiops/) — notebook 02 | [SYNTH](../synth/) — notebook 02 (anomaly on synthetics); [DBMON](../dbmon/) — notebook 06 (anomaly on DB metrics) | AIOPS-02 for the model; domain series for domain-specific applications |
| Workflow as code | [WFLOW](../wflow/) — full series for concepts | [AUTOM](../autom/) — notebook 05 (workflows-as-code) | WFLOW for design; AUTOM-05 for IaC delivery |
| Dashboard automation | [DASH](../dash/) — notebook 07 (sharing and reporting) | [AUTOM](../autom/) — notebooks 02–04 (Settings API, Monaco, Terraform with dashboard examples) | DASH first; AUTOM-02..04 once dashboards stabilize |
| Settings API | [AUTOM](../autom/) — notebook 02 | [IAM](../iam/) — notebook 12 (API provisioning) | AUTOM-02 for general; IAM-12 for IAM-specific |
| GitOps for Dynatrace | [AUTOM](../autom/) — notebooks 03 (Monaco), 04 (Terraform), 07 (CI/CD) | [K8S](../k8s/) — notebook 03 (GitOps DynaKube) | AUTOM for general patterns; K8S-03 for DynaKube-specific |

---

## Domain Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| Frontend-to-backend tracing | [SPANS](../spans/) — full series for span concepts | [WEBRUM](../webrum/) — notebook 04 (session analysis); [MOBL](../mobl/) — notebook 07 (network requests) | SPANS-01..02 first; frontend domain for domain-specific |
| K8s logs | [K8S](../k8s/) — notebook 07 (events and logs) | [OPLOGS](../oplogs/) — for log processing | K8S-07 for K8s-specific; OPLOGS for ingestion patterns |
| Kubernetes monitoring | [K8S](../k8s/) — full series | [CLOUD](../cloud/) — notebook 03 (AWS EKS) | K8S full series first; CLOUD-03 for managed-K8s context |
| Database call tracing | [DBMON](../dbmon/) — notebook 05 (query analysis) | [SPANS](../spans/) — notebook 02 (querying) | DBMON-05 for DB-specific; SPANS-02 for the underlying span queries |
| Synthetic alerting | [WFLOW](../wflow/) — notebook 04 | [SYNTH](../synth/) — notebook 02 (browser monitors) and 06 (analytics) | SYNTH for what to alert on; WFLOW for routing |
| RUM dashboards | [DASH](../dash/) — notebook 04 (operations) | [WEBRUM](../webrum/) — notebook 08 (dashboards and alerting) | DASH for general dashboard design; WEBRUM-08 for RUM-specific layouts |

---

## Cross-Category Overlaps

| Topic | Canonical | Also covers | Recommended order |
|---|---|---|---|
| Cost optimization | [ADOPT](../adopt/) — notebook 05 (optimization roadmap) for the framework | [SPANS](../spans/) — notebook 08 (span cost); [ORGNZ](../orgnz/) — notebooks 02, 03 (bucket strategy affects cost); [OPIPE](../opipe/) — notebook 04 (cardinality) | ADOPT-05 for framework; signal-specific series for tactics |
| Tagging governance | [FAQ](../faq/) — entry 02 (tagging sources, standards, strategy) | [ORGNZ](../orgnz/) — notebook 01 (introduction); [ONBRD](../onbrd/) — notebook 06 | FAQ-02 first for the framework; ORGNZ-01 for organizational concepts |
| Multi-tenant patterns | [IAM](../iam/) — notebook 08 (multi-environment) | [ORGNZ](../orgnz/) — notebook 09 (enterprise patterns) | IAM-08 for access; ORGNZ-09 for organizational |
| FinOps and cost allocation | [ADOPT](../adopt/) — notebook 05 (optimization roadmap) | [ORGNZ](../orgnz/) — notebook 09 (enterprise patterns) | ADOPT-05 first; ORGNZ-09 for tagging that supports cost allocation |
| Migration validation | [NRLC](../nrlc/) — notebook 08; [NR2DT](../nr2dt/) — notebook 08 (validate) | [SL2DT](../sl2dt/) — notebook 09 (cutover validation); [S2D](../s2d/) — implicit in notebook 09 | Tool-specific validation per migration series |

---

## Anti-Patterns to Avoid

| Anti-Pattern | What's wrong | What to do instead |
|---|---|---|
| Reading [ONBRD](../onbrd/) notebook 09 (alerts) AND [WFLOW](../wflow/) full series | ONBRD-09 is an intro that doesn't add to WFLOW | Skip ONBRD-09 if you'll read WFLOW |
| Reading [ONBRD](../onbrd/) notebook 10 (dashboards) AND [DASH](../dash/) full series | ONBRD-10 is an intro that doesn't add to DASH | Skip ONBRD-10 if you'll read DASH |
| Treating bucket-level access control as the default | Buckets are immutable, capped at 80, not universal — they're for specific scenarios | Default to security_context (ORGNZ-06); use buckets only for compliance, retention, or hard cost separation |
| Reading [NRLC](../nrlc/) and [NR2DT](../nr2dt/) sequentially as if they're a single series | They're paired references — NRLC for component depth, NR2DT for procedural sequencing | Read in parallel: open NR2DT for the step you're on, dive into NRLC for the components touched in that step |
| Migrating multiple source tools simultaneously | Halves your team's focus, doubles the risk of either failing | Migrate in waves; one source tool at a time |
| Using [ONBRD](../onbrd/) as the definitive source for IAM, tagging, or buckets | ONBRD's coverage is intentionally light to keep the onboarding scope narrow | Use ONBRD for orientation; dive into IAM, ORGNZ, or FAQ for depth |

---

> *This playbook was AI-generated from community-submitted and publicly available sources. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*
