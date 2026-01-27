# What's Different in SaaS

> **Series:** M2S | **Notebook:** 1 of 8 | **Created:** January 2026

---

## Table of Contents

1. [Introduction](#introduction)
2. [Grail Architecture Overview](#grail)
3. [Key Differences from Managed](#differences)
4. [Assessing Your Managed Environment](#assessment)
5. [Next Steps](#next-steps)

---

## Prerequisites

Before starting this series, ensure you have:

| Requirement | Description |
|-------------|-------------|
| SaaS tenant provisioned | Your new Dynatrace SaaS environment is ready |
| Managed environment access | Administrator access to your existing Managed deployment |
| API Tokens | Tokens with `entities.read`, `settings.read` scopes on both environments |

---

## Learning Objectives

By the end of this notebook, you will:

- Understand the Grail architecture that powers your new SaaS tenant
- Know the key differences between Managed and SaaS
- Have a complete assessment of your Managed environment
- Be ready to plan your migration strategy

---

<a id="introduction"></a>
## 1. Introduction

Congratulations on your new Dynatrace SaaS tenant! This notebook series guides you through migrating your monitoring from Managed to SaaS.

### What This Series Covers

| Notebook | Focus |
|----------|-------|
| **M2S-01** (this notebook) | What's different in SaaS |
| **M2S-02** | Migration framework overview |
| **M2S-03** | Planning and assessment |
| **M2S-04** | Architecture and design |
| **M2S-05** | Configuration migration |
| **M2S-06** | OneAgent and ActiveGate migration |
| **M2S-07** | Security and privacy |
| **M2S-08** | Validation and optimization |

### Important: Your SaaS Tenant Uses Grail

Your new SaaS tenant is powered by **Grail**, Dynatrace's modern data lakehouse architecture. This means:

- Some features work differently than in Managed
- New capabilities are available that weren't in Managed
- Some legacy features are not available

This notebook helps you understand these differences so you can plan accordingly.

---

<a id="grail"></a>
## 2. Grail Architecture Overview

### What is Grail?

Grail is Dynatrace's unified data lakehouse that powers all data storage and analytics in SaaS. It replaces the traditional time-series database used in Managed.

### Grail vs Legacy Architecture

| Aspect | Legacy/Managed | Grail (SaaS) |
|--------|----------------|--------------|
| Data storage | Time-series database | Unified data lakehouse |
| Query language | USQL (limited) | DQL (full-featured) |
| Dashboards | Classic dashboards | Modern dashboards + Notebooks |
| Log analytics | Log Monitoring v1 | Log Management powered by Grail |
| Business events | Limited | Full business analytics |
| Retention | Fixed tiers | Flexible, cost-optimized |

### Grail-Exclusive Features

These capabilities are only available on your new SaaS tenant:

| Feature | Description |
|---------|-------------|
| **Grail** | Petabyte-scale data lakehouse |
| **DQL** | Context-aware queries across all data types |
| **Notebooks** | Interactive analysis with collaboration |
| **Automations** | Advanced workflow engine |
| **OpenPipeline** | Custom data processing and routing |
| **Business Analytics** | Full business context and KPIs |

---

<!-- MARKDOWN_TABLE_ALTERNATIVE
| What Changes | Managed | SaaS |
|--------------|---------|------|
| Queries | USQL | DQL (required) |
| Dashboards | Classic | Modern + Classic |
| Logs | Log Monitoring v1 | Log Management |
| Alerting | Classic alerts | Workflows |
-->

![SaaS Architecture](images/m2s-saas-benefits.svg)

---

<a id="differences"></a>
## 3. Key Differences from Managed

### What Changes

| Area | Managed | SaaS (Grail) |
|------|---------|--------------|
| **Queries** | USQL | DQL (required for new features) |
| **Dashboards** | Classic only | Modern dashboards (recommended) |
| **Log Management** | Log Monitoring v1 | Log Management via Grail |
| **Metrics** | Classic metrics | Grail-powered metrics |
| **Alerting** | Classic alerts | Workflows and Davis Analyzers |

### What Stays the Same

Your existing investments are preserved:

- OneAgent deployment patterns
- Monitoring configurations (with migration)
- DQL query skills (DQL works on both)
- Classic dashboards (still supported, modern recommended)
- Core alerting concepts

### What You'll Need to Learn

| Topic | Why |
|-------|-----|
| **DQL** | Required for querying Grail data |
| **Modern dashboards** | New visualization approach |
| **Workflows** | Replaced classic alerting profiles |
| **OpenPipeline** | New log processing model |
| **Notebooks** | Interactive analysis tool |

---

### Important Limitations to Know

| Limitation | Impact | Planning Note |
|------------|--------|---------------|
| **Historic data** | Cannot be migrated | Plan for baseline period in SaaS |
| **Host limit** | 25,000 per tenant | Split tenants if exceeding |
| **SAML signing** | IdP must sign full message | Verify IdP configuration |
| **OneAgent versions** | 9-12 month support window | Check oldest versions |

### SAML/SSO Requirements

> **🚨 Critical:** Your Identity Provider (IdP) must sign the **entire SAML message**, not just the assertion. Azure AD meets this requirement by default.

| IdP Consideration | Requirement |
|-------------------|-------------|
| SAML message signing | Full message, not just assertion |
| Group limit (Azure Entra) | 150 groups per user in SAML claim |
| Group filtering | Filter to Dynatrace-related groups only |

---

<a id="assessment"></a>
## 4. Assessing Your Managed Environment

Before planning your migration, document your current state. Run these queries in your Managed environment.

### Environment Size Assessment

```dql
// Count total monitored hosts
fetch dt.entity.host
| summarize totalHosts = count()
```

```dql
// Count hosts by operating system
fetch dt.entity.host
| summarize count = count(), by:{osType}
| sort count desc
```

```dql
// Count monitored services
fetch dt.entity.service
| summarize totalServices = count()
```

```dql
// Count applications (web and mobile)
fetch dt.entity.application
| summarize totalApplications = count()
```

### OneAgent Version Assessment

Understanding your OneAgent versions helps plan the migration:

```dql
// Check OneAgent versions across hosts
fetch dt.entity.host
| summarize hostCount = count(), by:{oneAgentVersion}
| sort hostCount desc
```

### ActiveGate Assessment

Identify your ActiveGate deployment:

```dql
// ActiveGate inventory - use the Entities API v2 or Settings API
// Note: ActiveGates are not directly queryable via DQL fetch
// Use: GET /api/v2/entities?entitySelector=type("ENVIRONMENT_ACTIVE_GATE")
// Or check the ActiveGates page in the Dynatrace UI
```

### Migration Complexity Indicators

| Factor | Low Complexity | High Complexity |
|--------|---------------|----------------|
| Host count | < 500 | > 5,000 |
| Service count | < 200 | > 2,000 |
| Custom integrations | Few or none | Many custom webhooks |
| Network restrictions | Open outbound | Strict firewall rules |
| Compliance requirements | Standard | PCI, HIPAA, SOC2 |
| Team availability | Dedicated team | Part-time resources |

### Pre-Migration Checklist

| Requirement | Status | Notes |
|-------------|--------|-------|
| Network can reach SaaS endpoints | [ ] | Port 443 outbound |
| SaaS tenant provisioned | [ ] | Tenant URL confirmed |
| API tokens created | [ ] | Both environments |
| Documentation of current config | [ ] | Settings inventory |
| Test environment available | [ ] | For validation |
| Migration window identified | [ ] | Low-traffic period |

---

<a id="next-steps"></a>
## 5. Next Steps

### The SaaS Upgrade Assistant

Dynatrace provides the **SaaS Upgrade Assistant** app to help automate your migration:

| Feature | Description |
|---------|-------------|
| **Migration Planning** | Automated discovery and assessment |
| **Configuration Export** | Bulk export of settings and dashboards |
| **Progress Tracking** | Visual migration status dashboard |
| **Validation Checks** | Pre and post-migration validation |

> **Tip:** Contact your Dynatrace account team to access the SaaS Upgrade Assistant app.

### Migration Tooling Options

| Tool | Best For |
|------|----------|
| **SaaS Upgrade Assistant** | Guided migration with UI |
| **Monaco** | Configuration-as-code approach |
| **Terraform** | Infrastructure-as-code environments |
| **Settings API** | Custom migration scripts |

### Immediate Actions

1. **Run the assessment queries** - Document your Managed environment size
2. **Verify network connectivity** - Can you reach your SaaS tenant?
3. **Create API tokens** - On both Managed and SaaS
4. **Review the checklist** - Identify any gaps

### Continue the Series

| Next Notebook | Focus |
|---------------|-------|
| **M2S-02: Migration Framework** | Understanding the 3-phase migration approach |

### Additional Resources

- [Dynatrace SaaS Documentation](https://docs.dynatrace.com/)
- [Grail Data Lakehouse](https://docs.dynatrace.com/docs/platform/grail)
- [DQL Reference](https://docs.dynatrace.com/docs/platform/grail/dynatrace-query-language)

---

## Summary

In this notebook, you learned:

- How Grail architecture differs from Managed
- Key changes to expect in your SaaS tenant
- Important limitations and requirements
- How to assess your current Managed environment
- Available migration tools

> **Key Takeaway:** Your SaaS tenant uses Grail, which means new capabilities but also some changes in how you work. Understanding these differences upfront helps you plan a successful migration.

---

*Continue to **M2S-02: Migration Framework Overview** to learn about the structured approach to migration.*

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
