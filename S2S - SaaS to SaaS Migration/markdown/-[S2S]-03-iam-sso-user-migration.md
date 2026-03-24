# S2S-03: IAM, SSO, and User Migration

> **Series:** S2S | **Notebook:** 3 of 12 | **Created:** March 2026 | **Last Updated:** 03/23/2026

## Overview

Identity and access management is often the most politically complex part of a SaaS-to-SaaS migration. Groups, policies, and SSO configurations must be rebuilt in the target tenant — and a consolidation is the perfect opportunity to rationalize overgrown IAM structures.

---

## Table of Contents

1. [IAM Migration Strategy](#iam-strategy)
2. [SSO and SAML Reconfiguration](#sso-reconfiguration)
3. [Group and Policy Migration](#group-policy-migration)
4. [User Migration](#user-migration)
5. [Service Account Migration](#service-account-migration)
6. [Cloud Provider Identity Mapping](#cloud-identity-mapping)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Source Tenant** | Admin access, IAM admin role |
| **Target Tenant** | Admin access, IAM admin role |
| **OAuth Client** | `account-idm-read`, `account-idm-write`, `iam-policies-management` scopes |
| **IdP Access** | Admin access to corporate identity provider (Okta, Azure AD, etc.) |

<a id="iam-strategy"></a>
## 1. IAM Migration Strategy

### Decision: Lift-and-Shift vs Redesign

| Approach | When to Use | Pros | Cons |
|----------|-------------|------|------|
| **Lift-and-Shift** | Simple relocation, no org changes | Fast, minimal disruption | Carries forward any existing IAM debt |
| **Redesign** | Consolidation, M&A, compliance changes | Clean architecture, right-sized access | More planning, requires stakeholder alignment |
| **Hybrid** | Migrate critical access first, redesign later | Quick initial migration | Requires follow-up project |

> **Recommendation:** Tenant consolidation is the best opportunity to redesign IAM. You are already asking users to change — use that window to fix group sprawl and overprivileged policies.

### IAM Inventory from Source Tenant

Export the current IAM structure:

```bash
# Export IAM groups
terraform import dynatrace_iam_group.group1 "<group-uuid>"

# Export IAM policies
terraform import dynatrace_iam_policy.policy1 "<policy-uuid>"

# Export policy bindings
terraform import dynatrace_iam_policy_bindings_v2.binding1 "<binding-uuid>"
```

<a id="sso-reconfiguration"></a>
## 2. SSO and SAML Reconfiguration

### SAML Configuration Checklist

| Step | Action | Notes |
|------|--------|-------|
| 1 | Create new SAML application in IdP for target tenant | New Entity ID, new ACS URL |
| 2 | Configure SAML assertion attributes | Must match target tenant expectations |
| 3 | Enable JIT provisioning (if used) | Users auto-created on first login |
| 4 | Configure SCIM endpoint (if used) | New SCIM URL and token for target |
| 5 | Test with a pilot user | Verify login, group assignment, permissions |
| 6 | Update IdP group assignments | Map IdP groups to new Dynatrace groups |

### IdP-Specific Guidance

| IdP | Key Steps | Gotcha |
|-----|-----------|--------|
| **Okta** | Create new SAML app, configure SCIM provisioning | SCIM token is per-application |
| **Azure AD (Entra ID)** | New Enterprise App, configure provisioning | Group claim limited to 150 groups per assertion |
| **OneLogin** | New SAML connector, configure SCIM | Verify attribute mapping format |
| **PingFederate** | New SP connection, provisioning adapter | Certificate format (PEM required) |
| **Google Workspace** | New custom SAML app | Limited SCIM support, may need adapter |

### Cross-Domain SSO for Cloud Transformations

When migrating between cloud providers, identity federation changes:

| Transformation | Source IdP | Target IdP | Action |
|---------------|-----------|-----------|--------|
| **AWS → Azure** | AWS IAM / Okta | Azure AD (Entra ID) | Federate Azure AD with Dynatrace; migrate Okta groups if keeping Okta |
| **Azure → AWS** | Azure AD | AWS IAM / Okta | Create new SAML app in Okta or configure AWS SSO federation |
| **Any → GCP** | Varies | Google Workspace / Cloud Identity | Create custom SAML app in Google Workspace |
| **Multi-Cloud** | Varies | Keep existing IdP | Update SAML app ACS URL to point to target tenant |

<a id="group-policy-migration"></a>
## 3. Group and Policy Migration

### Terraform Export from Source

```hcl
# Export current groups
resource "dynatrace_iam_group" "sre_team" {
  name        = "sre-team"
  description = "SRE team with full environment access"
}

# Export current policies
resource "dynatrace_iam_policy" "sre_settings" {
  name            = "SRE Team - Settings Access"
  environment     = var.dynatrace_environment_id
  statement_query = <<-EOT
    ALLOW settings:objects:read, settings:objects:write
      WHERE settings:schemaId startsWith "builtin:alerting";
    ALLOW settings:objects:read, settings:objects:write
      WHERE settings:schemaId startsWith "builtin:anomaly-detection";
  EOT
}
```

### Apply to Target

```bash
# Point Terraform at target tenant
export TF_VAR_target_tenant_url="https://<target>.live.dynatrace.com"
export TF_VAR_target_api_token="dt0c01.xxxx"
export TF_VAR_target_client_id="dt0s02.xxxx"
export TF_VAR_target_client_secret="xxxx"
export TF_VAR_target_account_id="xxxx-xxxx-xxxx"

terraform plan
terraform apply
```

<a id="user-migration"></a>
## 4. User Migration

| Method | Steps | Best For |
|--------|-------|----------|
| **SCIM Auto-Provision** | Configure SCIM on target tenant; users sync automatically | Organizations with SCIM already configured |
| **JIT Provisioning** | Enable JIT on target; users created on first SSO login | Organizations using SSO without SCIM |
| **Manual Invitation** | Invite users via Account Management UI or API | Small teams, external users |
| **API Bulk Invite** | `POST /iam/v1/accounts/{id}/users` for each user | Large teams without SCIM |

<a id="service-account-migration"></a>
## 5. Service Account Migration

Service accounts and OAuth clients **cannot be migrated** — secrets are not exportable. Create new ones in the target tenant:

| Item | Action | Critical Steps |
|------|--------|---------------|
| OAuth Clients | Create new in target | Copy scopes from source; update CI/CD secrets |
| API Tokens | Create new in target | Match scopes; update all consumers |
| Service Users | Create new in target | Reassign as workflow actors |

> **Warning:** API tokens and OAuth client secrets from the source tenant will stop working when agents cut over to the target. Ensure all consumers are updated before cutover.

<a id="cloud-identity-mapping"></a>
## 6. Cloud Provider Identity Mapping

When migrating between cloud providers, map identity constructs:

| Concept | AWS | Azure | GCP |
|---------|-----|-------|-----|
| **Identity Provider** | AWS IAM Identity Center / Okta | Azure AD (Entra ID) | Google Cloud Identity |
| **Service Principal** | IAM Role | Service Principal / Managed Identity | Service Account |
| **Group** | IAM Group / Okta Group | Azure AD Group | Google Group |
| **MFA** | AWS MFA / Okta MFA | Azure AD Conditional Access | Google 2-Step Verification |
| **SCIM** | Okta SCIM / AWS SSO SCIM | Azure AD Provisioning | Google SCIM (limited) |

---

## Next Steps

Continue to **S2S-04: OneAgent and ActiveGate Cutover** to plan the agent migration.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
