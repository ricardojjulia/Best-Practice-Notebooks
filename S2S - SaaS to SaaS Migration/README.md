# S2S - SaaS to SaaS Migration

Comprehensive guide to migrating between Dynatrace SaaS environments, covering every phase from readiness assessment through post-migration optimization.

> **Recommended:** Import the JSON files from notebooks/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Structure
- notebooks/ — Dynatrace notebook JSON files
- pdfs/ — Printable versions of each notebook
- markdown/ — Markdown exports of the notebooks

## Notebook Lineup
1. [Step 1 — Discover: Migration Scenarios and Inventory](markdown/-[S2S]-01-step-1-discover.md) — Understanding why you're migrating, inventorying your source environment, and identifying what migrates automatically
2. [Step 2 — Strategize: Define Your Migration Approach](markdown/-[S2S]-02-step-2-strategize.md) — Planning your migration strategy, phasing, and execution approach
3. [Step 3 — Design: Target Tenant Architecture](markdown/-[S2S]-03-step-3-design.md) — Designing your target SaaS architecture and tenant structure
4. [Step 4 — Prepare: Export and Pre-Stage](markdown/-[S2S]-04-step-4-prepare.md) — Exporting configurations and pre-staging for migration
5. [Step 5 — Execute: Configuration Import and Agent Cutover](markdown/-[S2S]-05-step-5-execute.md) — Importing configuration and redirecting agents to the target environment
6. [Step 6 — Integrate: Cloud, Dashboards, and Workflows](markdown/-[S2S]-06-step-6-integrate.md) — Migrating cloud integrations, dashboards, and workflows
7. [Step 7 — Expand: OpenPipeline, SLOs, and Alerting](markdown/-[S2S]-07-step-7-expand.md) — Configuring data pipelines, SLOs, and alerting rules
8. [Step 8 — Enable: Parallel Operation and Stakeholder Handover](markdown/-[S2S]-08-step-8-enable.md) — Running source and target in parallel and handing over to operations
9. [Step 9 — Optimize: Cutover Validation and Decommission](markdown/-[S2S]-09-step-9-optimize.md) — Validating the migration, optimizing the target, and decommissioning source
10. [Migration Scripts](markdown/-[S2S]-10-migration-scripts.md) — Reusable Bash and PowerShell scripts for Monaco export and SaaS Upgrade Assistant packaging
99. [Best Practice Summary](markdown/-[S2S]-99-best-practice-summary.md) — Comprehensive reference of all best practices from the S2S series

## Usage
1. Choose a format: import JSON from notebooks/, read pdfs/ for print, or view markdown/ for lightweight browsing.
2. Start with Migration Scenarios and Readiness Assessment for an overview, then follow the numbered sequence through each migration phase.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
