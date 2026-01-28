# Best-Practice-Notebooks

Dynatrace best-practice notebooks with matching PDF and Markdown exports. These assets are not officially supported by Dynatrace.

> **Recommended:** Import the JSON files from NOTEBOOKS/ into a Dynatrace tenant for the best experience. These notebooks contain interactive DQL queries that execute against your environment's data.

## Layout

Each topic follows the same structure:
- NOTEBOOKS/ — Dynatrace notebook JSON files (import into Dynatrace)
- PDFs/ — Printable/exported versions of the notebooks
- markdown/ — Markdown exports of the same content
- README.md — Topic overview and usage guide

## Table of Contents

### [AUTOM - Dynatrace automation](AUTOM%20-%20Dynatrace%20automation/README.md)
Comprehensive guide to automating Dynatrace configuration and operations.
- [AUTOM-01: Automation Landscape](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-01-automation-landscape.md) — Overview of Dynatrace automation options
- [AUTOM-02: Settings API](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-02-settings-api.md) — Working with the Settings 2.0 API
- [AUTOM-03: Monaco](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-03-monaco.md) — Configuration as code with Monaco
- [AUTOM-04: Terraform](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-04-terraform.md) — Infrastructure as code with Terraform provider
- [AUTOM-05: Workflows](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-05-workflows.md) — Dynatrace Workflows automation
- [AUTOM-06: SDKs](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-06-sdks.md) — Using Dynatrace SDKs for automation
- [AUTOM-07: CI/CD Integration](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-07-cicd-integration.md) — Integrating Dynatrace into CI/CD pipelines
- [AUTOM-08: Migration Automation](AUTOM%20-%20Dynatrace%20automation/markdown/-AUTOM-08-migration-automation.md) — Automating configuration migrations

### [IAMADM - IAM administration](IAMADM%20-%20IAM%20administration/README.md)
Enterprise identity and access management administration for Dynatrace.
- [IAMADM-01: Governance Foundations](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-01-governance-foundations.md) — IAM governance principles and strategy
- [IAMADM-02: SSO & Authentication](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-02-sso-authentication.md) — Single sign-on and authentication setup
- [IAMADM-03: Group Architecture](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-03-group-architecture.md) — Designing group structures
- [IAMADM-04: Policy Authoring](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-04-policy-authoring.md) — Writing and managing policies
- [IAMADM-05: Boundary Design](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-05-boundary-design.md) — Defining access boundaries
- [IAMADM-06: User Lifecycle](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-06-user-lifecycle.md) — Managing user provisioning and deprovisioning
- [IAMADM-07: Audit & Compliance](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-07-audit-compliance.md) — Auditing access and ensuring compliance
- [IAMADM-08: Multi-Environment](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-08-multi-environment.md) — IAM across multiple environments
- [IAMADM-09: Troubleshooting](IAMADM%20-%20IAM%20administration/markdown/-IAMADM-09-troubleshooting.md) — Diagnosing and resolving IAM issues

### [K8S - Kubernetes monitoring](K8S%20-%20Kubernetes%20monitoring/README.md)
Best practices for monitoring Kubernetes with Dynatrace.
- [K8S-01: Fundamentals](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-01-fundamentals.md) — Core Kubernetes monitoring concepts
- [K8S-02: Dynakube Deployment](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-02-dynakube-deployment.md) — Deploying Dynatrace Operator and Dynakube
- [K8S-03: GitOps Dynakube](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-03-gitops-dynakube.md) — Managing Dynakube with GitOps
- [K8S-04: Cluster Monitoring](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-04-cluster-monitoring.md) — Monitoring Kubernetes clusters
- [K8S-05: Workload Monitoring](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-05-workload-monitoring.md) — Monitoring workloads and applications
- [K8S-06: Namespace Organization](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-06-namespace-organization.md) — Organizing and filtering by namespace
- [K8S-07: Events and Logs](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-07-events-and-logs.md) — Kubernetes events and log monitoring
- [K8S-08: DQL for K8s](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-08-dql-for-k8s.md) — Querying Kubernetes data with DQL
- [K8S-09: Troubleshooting](K8S%20-%20Kubernetes%20monitoring/markdown/-K8S-09-troubleshooting.md) — Diagnosing Kubernetes monitoring issues

### [M2S - Managed to SaaS migration](M2S%20-%20Managed%20to%20SaaS%20migration/README.md)
Guide for migrating from Dynatrace Managed to Dynatrace SaaS.
- [M2S-01: What's Different](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-01-whats-different.md) — Key differences between Managed and SaaS
- [M2S-02: Migration Framework](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-02-migration-framework.md) — Overall migration approach and framework
- [M2S-03: Planning & Assessment](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-03-planning-assessment.md) — Assessing readiness and planning migration
- [M2S-04: Architecture Design](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-04-architecture-design.md) — Designing your SaaS architecture
- [M2S-05: Configuration Migration](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-05-configuration-migration.md) — Migrating configurations and settings
- [M2S-06: OneAgent & ActiveGate](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-06-oneagent-activegate.md) — Migrating agents and gateways
- [M2S-07: Security & Privacy](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-07-security-privacy.md) — Security considerations for migration
- [M2S-08: Validation & Optimization](M2S%20-%20Managed%20to%20SaaS%20migration/markdown/-M2S-08-validation-optimization.md) — Validating and optimizing post-migration

### [MZ2POL - Management Zone to Policy migration](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/README.md)
Tools and guidance for migrating from Management Zones to Policy-based access control.
- [MZ2POL-00: SDK MZ Analysis Tool](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-00-sdk-mz-analysis-tool.md) — Analyze existing management zones
- [MZ2POL-01: Introduction: Why Migrate](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-01-introduction-why-migrate.md) — Benefits and overview
- [MZ2POL-02: Access Control Model](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-02-access-control-model.md) — Policies and access control concepts
- [MZ2POL-03: Assessment & Planning](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-03-assessment-planning.md) — Migration assessment and planning
- [MZ2POL-04: Policies and Boundaries](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-04-policies-boundaries.md) — Defining policies and boundaries
- [MZ2POL-05: Segments Implementation](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-05-segments-implementation.md) — Implementing segments
- [MZ2POL-06: Migration Execution](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-06-migration-execution.md) — Executing the migration
- [MZ2POL-07: Validation & Troubleshooting](MZ2POL%20-%20Management%20Zone%20to%20Policy%20migration/markdown/-MZ2POL-07-validation-troubleshooting.md) — Validating and resolving issues

### [ONBRD - Dynatrace onboarding](ONBRD%20-%20Dynatrace%20onboarding/README.md)
Step-by-step onboarding series for new Dynatrace users.
- [ONBRD-01: First Steps](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-01-first-steps.md) — Getting started with Dynatrace
- [ONBRD-02: IAM and Authentication](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-02-iam-and-authentication.md) — Identity and access management
- [ONBRD-03: Deploying ActiveGate](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-03-deploying-activegate.md) — Installing and configuring ActiveGate
- [ONBRD-04: Cloud/SaaS Integrations](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-04-cloud-saas-integrations.md) — Connecting cloud platforms
- [ONBRD-05: Deploying OneAgent](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-05-deploying-oneagent.md) — Deployment strategies
- [ONBRD-06: Organizing Your Environment](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-06-organizing-your-environment.md) — Structuring environments and entities
- [ONBRD-07: Understanding Your Data](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-07-understanding-your-data.md) — Data model and exploration
- [ONBRD-08: Your First Queries](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-08-your-first-queries.md) — Introduction to DQL
- [ONBRD-09: Setting Up Alerts](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-09-setting-up-alerts.md) — Alerting and notifications
- [ONBRD-10: Building Dashboards](ONBRD%20-%20Dynatrace%20onboarding/markdown/-ONBRD-10-building-dashboards.md) — Creating stakeholder dashboards

### [OPLOGS - OpenPipeline logs](OPLOGS%20-%20OpenPipeline%20logs/README.md)
End-to-end guidance for managing logs with Dynatrace OpenPipeline.
- [OPLOGS-01: Fundamentals](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-01-fundamentals.md) — Core OpenPipeline concepts
- [OPLOGS-02: Migration](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-02-migration.md) — Moving from other logging platforms
- [OPLOGS-03: Pipeline Processing](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-03-pipeline-processing.md) — Transformations and processing stages
- [OPLOGS-04: Buckets & Governance](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-04-buckets-governance.md) — Storage management and governance
- [OPLOGS-05: Querying & Parsing](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-05-querying-parsing.md) — Query techniques and parsing rules
- [OPLOGS-06: Topology](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-06-topology.md) — Log topology and relationships
- [OPLOGS-07: Analytics](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-07-analytics.md) — Advanced log analytics
- [OPLOGS-08: Security](OPLOGS%20-%20OpenPipeline%20logs/markdown/-OPLOGS-08-security.md) — Security, masking, and compliance

### [OPMIG - OpenPipeline migration](OPMIG%20-%20OpenPipeline%20migration/README.md)
Structured migration path to Dynatrace OpenPipeline.
- [OPMIG-01: Introduction: Why Migrate](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-01-introduction-why-migrate.md) — Benefits and goals
- [OPMIG-02: Architecture & Key Concepts](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-02-architecture-key-concepts.md) — OpenPipeline architecture overview
- [OPMIG-03: Migration Assessment & Planning](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-03-migration-assessment-planning.md) — Migration plan and readiness
- [OPMIG-04: Pipeline Configuration](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-04-pipeline-configuration.md) — Configuring OpenPipeline
- [OPMIG-05: Routing & Buckets](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-05-routing-buckets.md) — Routing rules and storage strategy
- [OPMIG-06: Processing & Parsing](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-06-processing-parsing.md) — Transformation and parsing rules
- [OPMIG-07: Metric & Event Extraction](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-07-metric-event-extraction.md) — Deriving metrics/events from logs
- [OPMIG-08: Security & Masking](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-08-security-masking.md) — Security controls and data masking
- [OPMIG-09: Troubleshooting & Validation](OPMIG%20-%20OpenPipeline%20migration/markdown/-OPMIG-09-troubleshooting-validation.md) — Validation and issue resolution

### [OTEL - OpenTelemetry integration](OTEL%20-%20OpenTelemetry%20integration/README.md)
Integrating OpenTelemetry with Dynatrace for observability.
- [OTEL-01: Fundamentals](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-01-fundamentals.md) — Core OpenTelemetry concepts
- [OTEL-02: Collector Architecture](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-02-collector-architecture.md) — Understanding the OTel Collector
- [OTEL-03: Collector Deployment](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-03-collector-deployment.md) — Deploying and configuring collectors
- [OTEL-04: Trace Instrumentation](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-04-trace-instrumentation.md) — Instrumenting applications for traces
- [OTEL-05: Metrics Instrumentation](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-05-metrics-instrumentation.md) — Instrumenting applications for metrics
- [OTEL-06: Logs Instrumentation](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-06-logs-instrumentation.md) — Instrumenting applications for logs
- [OTEL-07: Dynatrace Integration](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-07-dynatrace-integration.md) — Sending OTel data to Dynatrace
- [OTEL-08: Troubleshooting](OTEL%20-%20OpenTelemetry%20integration/markdown/-OTEL-08-troubleshooting.md) — Diagnosing OpenTelemetry issues

### [SPANS - Distributed tracing and spans](SPANS%20-%20Distributed%20tracing%20and%20spans/README.md)
Guidance for working with distributed traces and spans in Dynatrace.
- [SPANS-01: Fundamentals](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-01-fundamentals.md) — Core tracing concepts
- [SPANS-02: Querying](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-02-querying.md) — Querying spans and traces with DQL
- [SPANS-03: Troubleshooting](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-03-troubleshooting.md) — Using traces for issue investigation
- [SPANS-04: Topology](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-04-topology.md) — Service topology from traces
- [SPANS-05: Analytics](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-05-analytics.md) — Advanced trace analytics
- [SPANS-06: Security](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-06-security.md) — Span security and data protection
- [SPANS-07: Buckets & Pipeline](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-07-buckets-pipeline.md) — Storage and processing
- [SPANS-08: Cost Optimization](SPANS%20-%20Distributed%20tracing%20and%20spans/markdown/-SPANS-08-cost-optimization.md) — Optimizing trace ingestion costs

### [SYNTH - Synthetic monitoring](SYNTH%20-%20Synthetic%20monitoring/README.md)
Series covering Dynatrace Synthetic Monitoring.
- [SYNTH-01: Fundamentals](SYNTH%20-%20Synthetic%20monitoring/markdown/-SYNTH-01-fundamentals.md) — Core synthetic monitoring concepts
- [SYNTH-02: Browser Monitors](SYNTH%20-%20Synthetic%20monitoring/markdown/-SYNTH-02-browser-monitors.md) — Creating and managing browser monitors
- [SYNTH-03: HTTP Monitors](SYNTH%20-%20Synthetic%20monitoring/markdown/-SYNTH-03-http-monitors.md) — HTTP/API monitors
- [SYNTH-04: Private Locations](SYNTH%20-%20Synthetic%20monitoring/markdown/-SYNTH-04-private-locations.md) — Deploying private synthetic locations
- [SYNTH-05: Network Monitoring](SYNTH%20-%20Synthetic%20monitoring/markdown/-SYNTH-05-network-monitoring.md) — Network performance monitoring
- [SYNTH-06: Analytics](SYNTH%20-%20Synthetic%20monitoring/markdown/-SYNTH-06-analytics.md) — Analyzing synthetic monitoring data

### [WFLOW - Workflows and alert notifications](WFLOW%20-%20Workflows%20and%20alert%20notifications/README.md)
Automating workflows and configuring alert notifications in Dynatrace.
- [WFLOW-01: Fundamentals](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-01-fundamentals.md) — Core workflow concepts
- [WFLOW-02: Triggers](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-02-triggers.md) — Workflow trigger configuration
- [WFLOW-03: Notification Basics](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-03-notification-basics.md) — Setting up basic notifications
- [WFLOW-04: Notification Routing](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-04-notification-routing.md) — Advanced notification routing
- [WFLOW-05: Incident Management](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-05-incident-management.md) — Integrating with incident management
- [WFLOW-06: Custom Templates](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-06-custom-templates.md) — Creating custom notification templates
- [WFLOW-07: Remediation](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-07-remediation.md) — Automated remediation workflows
- [WFLOW-08: JavaScript & HTTP](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-08-javascript-http.md) — JavaScript actions and HTTP requests
- [WFLOW-09: Governance](WFLOW%20-%20Workflows%20and%20alert%20notifications/markdown/-WFLOW-09-governance.md) — Workflow governance and best practices

## How to Use

1. Open a topic README for context and prerequisites.
2. Choose your format:
	- Import JSON from NOTEBOOKS/ into Dynatrace Notebooks for interactive use.
	- Read PDFs/ for printable versions.
	- Use markdown/ for lightweight viewing or diffs.
3. Follow the numbered sequence to progress through each series.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
