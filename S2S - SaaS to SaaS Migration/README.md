# S2S - SaaS to SaaS Migration

Comprehensive guide to migrating between Dynatrace SaaS environments, covering every phase from readiness assessment through post-migration optimization.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Migration Scenarios and Readiness Assessment](markdown/-[S2S]-01-migration-scenarios-readiness.md) — Identifying migration drivers and evaluating readiness
2. [Discovery and Configuration Export](markdown/-[S2S]-02-discovery-configuration-export.md) — Cataloging and exporting source environment configuration
3. [IAM, SSO, and User Migration](markdown/-[S2S]-03-iam-sso-user-migration.md) — Migrating identity, access management, and SSO
4. [OneAgent and ActiveGate Cutover](markdown/-[S2S]-04-oneagent-activegate-cutover.md) — Redirecting agents to the target environment
5. [Settings and Configuration Import](markdown/-[S2S]-05-settings-configuration-import.md) — Importing configuration into the target environment
6. [Cloud Integration Migration](markdown/-[S2S]-06-cloud-integration-migration.md) — Migrating cloud provider integrations
7. [Dashboard, Workflow, and Integration Migration](markdown/-[S2S]-07-dashboard-workflow-integration-migration.md) — Migrating dashboards, workflows, and third-party integrations
8. [OpenPipeline and Grail Bucket Migration](markdown/-[S2S]-08-openpipeline-grail-bucket-migration.md) — Migrating data pipelines and storage configuration
9. [Data Continuity and Parallel Operation](markdown/-[S2S]-09-data-continuity-parallel-operation.md) — Running source and target in parallel during transition
10. [SLO and Alerting Migration](markdown/-[S2S]-10-slo-alerting-migration.md) — Migrating SLOs and alerting configuration
11. [Cutover Execution and Validation](markdown/-[S2S]-11-cutover-execution-validation.md) — Executing the final cutover and validating completeness
12. [Post-Migration Optimization and Decommission](markdown/-[S2S]-12-post-migration-optimization-decommission.md) — Optimizing the target environment and decommissioning source

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Start with Migration Scenarios and Readiness Assessment for an overview, then follow the numbered sequence through each migration phase.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
