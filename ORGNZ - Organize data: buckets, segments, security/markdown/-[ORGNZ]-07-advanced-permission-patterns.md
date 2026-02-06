# ORGNZ-07: Advanced Permission Patterns

> **Series:** ORGNZ | **Notebook:** 7 of 9 | **Created:** January 2026 | **Last Updated:** 01/28/2026

## Overview

This notebook covers advanced permission patterns including record-level permissions, field-based access, and combining multiple access control mechanisms for enterprise-scale data governance.

## Prerequisites

| Requirement | Details |
|---

---

## Table of Contents

1. [Record-Level Permissions](#record-level-permissions)
2. [Record-Level Policy Examples](#record-level-policy-examples)
3. [Field-Level Access](#field-level-access)
4. [Combined Permission Patterns](#combined-permission-patterns)
5. [Enterprise Architecture Patterns](#enterprise-architecture-patterns)
6. [Testing Permissions](#testing-permissions)
7. [Best Practices](#best-practices)

---

-------------|----------|
| **Dynatrace Account** | Account-level administrative access |
| **Permissions** | IAM policy management permissions |
| **Knowledge** | Completed ORGNZ-04 through ORGNZ-06 |

## Learning Objectives

By the end of this notebook, you will:
- Implement record-level permissions using various attributes
- Understand field-level access restrictions
- Combine bucket, record, and field-level policies
- Design enterprise-scale permission architectures

<a id="record-level-permissions"></a>
## Record-Level Permissions
Record-level permissions filter data at query time based on record attributes:

![Record-Level Permissions Flow](images/record-level-permissions.svg)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Step | Action |
|------|--------|
| 1 | User Query: fetch logs |
| 2 | IAM Policy Evaluation |
| 3 | Record Filter (namespace, host group, security context) |
| 4 | Only authorized records returned |
For environments where SVG doesn't render
-->

### Supported Record-Level Conditions

| Condition | Description | Example |
|-----------|-------------|----------|
| `storage:k8s.namespace.name` | Kubernetes namespace | `= 'production'` |
| `storage:k8s.cluster.name` | Kubernetes cluster | `= 'main-cluster'` |
| `storage:host.name` | Host name | `= 'web-server-01'` |
| `storage:dt.host_group.id` | Host group | `STARTSWITH 'prod-'` |
| `storage:service.name` | Service name | `= 'checkout'` |
| `storage:aws.account.id` | AWS account | `= '123456789012'` |
| `storage:gcp.project.id` | GCP project | `= 'my-project'` |
| `storage:dt.security_context` | Custom context | `MATCH ('team-*')` |

<a id="record-level-policy-examples"></a>
## Record-Level Policy Examples
### Kubernetes Namespace Isolation

```json
{
  "name": "production-namespace-access",
  "description": "Access only to production namespace data",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:k8s.namespace.name = 'production';"
}
```

### Host Group Based Access

```json
{
  "name": "web-tier-access",
  "description": "Access to web tier hosts only",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read, storage:metrics:read WHERE storage:dt.host_group.id STARTSWITH 'web-';"
}
```

### Combined Namespace and Host Group

```json
{
  "name": "prod-web-tier-access",
  "description": "Production web tier only",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:k8s.namespace.name = 'production' AND storage:dt.host_group.id STARTSWITH 'web-';"
}
```

### Cloud Account Isolation

```json
{
  "name": "aws-team-account-access",
  "description": "Access to specific AWS account",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read, storage:metrics:read, storage:spans:read WHERE storage:aws.account.id = '123456789012';"
}
```

<a id="field-level-access"></a>
## Field-Level Access
Control access to specific fields within records:

| Use Case | Implementation |
|----------|----------------|
| Hide sensitive fields | Deny access to specific fields |
| Mask PII | Field-level policies |
| Compliance requirements | Restrict access to audit fields |

### Field-Level Policy Example

```json
{
  "name": "restricted-field-access",
  "description": "Hide sensitive fields from general users",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read; DENY storage:logs:read:user.email, storage:logs:read:user.ip_address;"
}
```

<a id="combined-permission-patterns"></a>
## Combined Permission Patterns
### Pattern 1: Layered Access Control

Combine bucket and record-level permissions:

```
Layer 1: Bucket access
  ALLOW storage:buckets:read WHERE bucket-name IN ('team_logs', 'shared_logs')

Layer 2: Record filtering
  ALLOW storage:logs:read WHERE k8s.namespace.name = 'team-namespace'

Layer 3: Field masking (optional)
  DENY storage:logs:read:sensitive_field
```

### Pattern 2: Multi-Team Shared Bucket

Multiple teams share a bucket with security context isolation:

```json
// Team A policy
{
  "name": "team-a-shared-bucket",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name = 'shared_logs'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('team-a*');"
}

// Team B policy
{
  "name": "team-b-shared-bucket",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name = 'shared_logs'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('team-b*');"
}
```

### Pattern 3: Environment-Based Tiering

```json
// Production access (restricted)
{
  "name": "production-access",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'prod_'; ALLOW storage:logs:read WHERE storage:dt.security_context = 'env:production';"
}

// Non-production access (broader)
{
  "name": "non-production-access",
  "statementQuery": "ALLOW storage:buckets:read WHERE storage:bucket-name STARTSWITH 'default_'; ALLOW storage:logs:read WHERE storage:dt.security_context MATCH ('env:dev*', 'env:staging*', 'env:qa*');"
}
```

<a id="enterprise-architecture-patterns"></a>
## Enterprise Architecture Patterns
### Tiered Access Model

![Tiered Access Model](images/tiered-access-model.svg)

<!-- MARKDOWN_TABLE_ALTERNATIVE
| Tier | Role | Access |
|------|------|--------|
| 1 | Platform Admins | All buckets, all data types, all records |
| 2 | Team Leads | Default + team buckets, team context |
| 3 | Developers | Default buckets, own namespace only |
| 4 | Read-Only Viewers | Default buckets, public context only |
For environments where SVG doesn't render
-->

### Geographic/Regulatory Model

| Region | Buckets | Context | Policy |
|--------|---------|---------|--------|
| EU | eu_* | region:eu | bucket-name STARTSWITH 'eu_' AND security_context MATCH ('region:eu*') |
| US | us_* | region:us | bucket-name STARTSWITH 'us_' AND security_context MATCH ('region:us*') |

<a id="testing-permissions"></a>
## Testing Permissions

```dql
// Test namespace-based access
fetch logs, from:-1h
| filter isNotNull(k8s.namespace.name)
| summarize count = count(), by:{k8s.namespace.name}
| sort count desc
```

```dql
// Test host group access
fetch logs, from:-1h
| filter isNotNull(dt.host_group.id)
| summarize count = count(), by:{dt.host_group.id}
| sort count desc
```

```dql
// Verify accessible data distribution
fetch logs, from:-1h
| summarize 
    total = count(),
    withSecurityContext = countIf(isNotNull(dt.security_context)),
    withNamespace = countIf(isNotNull(k8s.namespace.name)),
    withHostGroup = countIf(isNotNull(dt.host_group.id))
```

<a id="best-practices"></a>
## Best Practices
| Practice | Rationale |
|----------|----------|
| Start with least privilege | Grant minimum required access |
| Use groups, not individuals | Easier management, consistent access |
| Document all policies | Audit trail and governance |
| Test policies before deployment | Prevent access issues |
| Regular access reviews | Remove unnecessary permissions |
| Use MATCH for array fields | Required for correct evaluation |
| Combine bucket + record level | Defense in depth |

## Next Steps

Continue with the ORGNZ series:
- **ORGNZ-08**: Grail Segments

## References

- [Configure advanced permissions with security context](https://docs.dynatrace.com/docs/platform/grail/organize-data/advanced-permission-setup)
- [IAM policy reference](https://docs.dynatrace.com/docs/manage/identity-access-management/permission-management/manage-user-permissions-policies/advanced/iam-policystatements)
- [Enhance data management with Grail](https://www.dynatrace.com/news/blog/enhance-data-management-with-grail-ultimate-guide-to-custom-buckets-and-security-policies/)

---

<sub>*This notebook was AI-generated from Dynatrace documentation and enterprise best practices. It is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
