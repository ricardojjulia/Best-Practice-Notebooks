# S2S-06: Cloud Integration Migration

> **Series:** S2S | **Notebook:** 6 of 12 | **Created:** March 2026 | **Last Updated:** 03/23/2026

## Overview

Cloud integrations (AWS, Azure, GCP) connect your Dynatrace tenant to cloud provider APIs for infrastructure metrics, log ingestion, and service discovery. These integrations must be reconfigured in the target tenant — credentials, IAM roles, and monitoring configurations are all tenant-specific.

---

## Table of Contents

1. [Cloud Integration Inventory](#cloud-inventory)
2. [AWS Integration Migration](#aws-migration)
3. [Azure Integration Migration](#azure-migration)
4. [GCP Integration Migration](#gcp-migration)
5. [Multi-Cloud Considerations](#multi-cloud)
6. [Cloud Transformation Scenarios](#cloud-transformations)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Cloud Provider Access** | Admin access to AWS/Azure/GCP accounts |
| **Target Tenant** | Provisioned with ActiveGate (for cloud integrations) |
| **IAM Roles** | Cloud provider IAM roles created for target tenant |

<a id="cloud-inventory"></a>
## 1. Cloud Integration Inventory

> **Monaco download limitation:** Cloud provider credentials (AWS, Azure, Kubernetes) **cannot be exported** via `monaco download`. Monaco can deploy credential configurations but not extract existing ones. You must recreate credentials manually in the target tenant. This is documented in the [Monaco API coverage](https://github.com/Dynatrace/dynatrace-configuration-as-code/blob/main/api_coverage.md).

Before migration, document all active cloud integrations in the source tenant:

| Integration | Source Config | Credentials | Metrics Collected |
|------------|-------------|-------------|-------------------|
| AWS CloudWatch | Regions, services, namespaces | IAM Role ARN or Access Key | EC2, RDS, Lambda, ELB, etc. |
| Azure Monitor | Subscriptions, resource groups | Service Principal or Managed Identity | VMs, App Service, AKS, etc. |
| GCP Cloud Monitoring | Projects, services | Service Account Key or Workload Identity | GCE, GKE, Cloud SQL, etc. |
| AWS Log Forwarder | S3 buckets, Lambda functions | IAM Role | CloudWatch Logs, S3 access logs |

<a id="aws-migration"></a>
## 2. AWS Integration Migration

### Step-by-Step

| Step | Action | Where |
|------|--------|-------|
| 1 | Create new IAM role in AWS with trust policy for target tenant | AWS IAM Console |
| 2 | Attach monitoring policies to the role | AWS IAM Console |
| 3 | Configure AWS integration in target Dynatrace tenant | Dynatrace UI → Settings → Cloud & Virtualization |
| 4 | Enter role ARN and external ID | Dynatrace UI |
| 5 | Select regions and services to monitor | Dynatrace UI |
| 6 | Verify metrics flowing | Dynatrace UI → Infrastructure |

### IAM Trust Policy for Target Tenant

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::<dynatrace-account-id>:root"
    },
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {
        "sts:ExternalId": "<target-tenant-external-id>"
      }
    }
  }]
}
```

> **Note:** The Dynatrace account ID and external ID are different for each tenant. Get these from the target tenant's AWS integration setup wizard.

### AWS Log Forwarding

If using the Dynatrace AWS Log Forwarder Lambda:

1. Deploy a new Log Forwarder stack pointing to the target tenant
2. Update S3 event notifications to trigger the new Lambda
3. Verify logs appear in target tenant

<a id="azure-migration"></a>
## 3. Azure Integration Migration

### Step-by-Step

| Step | Action | Where |
|------|--------|-------|
| 1 | Create new App Registration in Azure AD | Azure Portal → App Registrations |
| 2 | Grant `Monitoring Reader` role on subscriptions | Azure Portal → Subscriptions → IAM |
| 3 | Configure Azure integration in target tenant | Dynatrace UI → Settings → Cloud & Virtualization |
| 4 | Enter Tenant ID, Client ID, Client Secret | Dynatrace UI |
| 5 | Select subscriptions and services | Dynatrace UI |
| 6 | Verify metrics flowing | Dynatrace UI → Infrastructure |

### Azure Log Ingestion

| Method | Migration Action |
|--------|-----------------|
| **Azure Event Hub** | Create new Event Hub consumer group for target tenant; update Dynatrace ActiveGate config |
| **Azure Functions** | Deploy new function with target tenant endpoint |
| **Diagnostic Settings** | Update to forward to new Event Hub / ActiveGate |

<a id="gcp-migration"></a>
## 4. GCP Integration Migration

### Step-by-Step

| Step | Action | Where |
|------|--------|-------|
| 1 | Create new Service Account in GCP project | GCP Console → IAM → Service Accounts |
| 2 | Grant `Monitoring Viewer` and `Compute Viewer` roles | GCP Console → IAM |
| 3 | Generate JSON key for Service Account | GCP Console |
| 4 | Configure GCP integration in target tenant | Dynatrace UI → Settings → Cloud & Virtualization |
| 5 | Upload Service Account key | Dynatrace UI |
| 6 | Select projects and services | Dynatrace UI |
| 7 | Verify metrics flowing | Dynatrace UI → Infrastructure |

<a id="multi-cloud"></a>
## 5. Multi-Cloud Considerations

When migrating a tenant that monitors multiple cloud providers:

| Consideration | Action |
|---------------|--------|
| **Naming consistency** | Standardize tag naming across providers (`app`, `env`, `team`) |
| **Unified dashboards** | Recreate cross-cloud dashboards with new entity IDs |
| **Network zones** | Configure zones for each cloud's ActiveGate deployment |
| **Cost allocation** | Align Grail bucket and enrichment rules across providers |
| **Credential rotation** | Schedule rotation for all cloud provider credentials post-migration |

<a id="cloud-transformations"></a>
## 6. Cloud Transformation Scenarios

When the migration includes a cloud provider change, additional integration work is required:

### AWS → Azure

| Component | AWS | Azure Equivalent | Action |
|-----------|-----|-----------------|--------|
| Compute monitoring | EC2 metrics via CloudWatch | VM metrics via Azure Monitor | Configure Azure integration |
| Container platform | EKS | AKS | Deploy DynaKube to AKS, configure Workload Identity |
| Serverless | Lambda | Azure Functions | Install OneAgent extension or configure OpenTelemetry |
| Log forwarding | CloudWatch → Dynatrace Lambda | Event Hub → ActiveGate | Deploy new log pipeline |
| Load balancer | ALB/NLB metrics | Application Gateway / Load Balancer metrics | Update dashboards |
| Database | RDS metrics | Azure SQL / Cosmos DB metrics | Update SLOs |

### Azure → AWS

| Component | Azure | AWS Equivalent | Action |
|-----------|-------|---------------|--------|
| Compute monitoring | VM metrics via Azure Monitor | EC2 metrics via CloudWatch | Configure AWS integration |
| Container platform | AKS | EKS | Deploy DynaKube to EKS, configure IRSA |
| Serverless | Azure Functions | Lambda | Install OneAgent Lambda layer |
| Log forwarding | Event Hub → ActiveGate | CloudWatch → Dynatrace Lambda | Deploy log forwarder |

### GCP → AWS / Azure

| Component | GCP | AWS / Azure Equivalent | Action |
|-----------|-----|----------------------|--------|
| Compute | GCE metrics | EC2 / VM metrics | Configure cloud integration |
| Container platform | GKE | EKS / AKS | Deploy DynaKube, configure identity |
| Serverless | Cloud Functions / Cloud Run | Lambda / Azure Functions | Configure monitoring |
| Log forwarding | Pub/Sub → ActiveGate | CloudWatch Lambda / Event Hub | Deploy new pipeline |

---

## Next Steps

Continue to **S2S-07: Dashboard, Workflow, and Integration Migration** to migrate dashboards and automations.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
