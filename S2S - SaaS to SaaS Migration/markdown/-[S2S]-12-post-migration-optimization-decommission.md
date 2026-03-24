# S2S-12: Post-Migration Optimization and Decommission

> **Series:** S2S | **Notebook:** 12 of 12 | **Created:** March 2026 | **Last Updated:** 03/23/2026

## Overview

The migration is complete, but the work isn't over. This final notebook covers post-migration optimization, source tenant decommission, documentation, and lessons learned capture.

---

## Table of Contents

1. [Post-Migration Optimization](#optimization)
2. [Source Tenant Decommission](#decommission)
3. [Documentation and Handover](#documentation)
4. [Lessons Learned Framework](#lessons-learned)
5. [Series Summary](#series-summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Target Tenant** | Fully operational, all agents reporting |
| **Stakeholder Sign-Off** | From S2S-11 |
| **Source Tenant** | Still active for reference |

<a id="optimization"></a>
## 1. Post-Migration Optimization

### First 30 Days

| Task | Timeline | Owner |
|------|----------|-------|
| Monitor Davis AI false positives | Daily | SRE |
| Tune anomaly detection baselines | Week 1-2 | SRE |
| Expand SLO evaluation windows | Week 2-4 | SRE |
| Optimize dashboard performance | Week 2-4 | Dashboard owners |
| Review and tune OpenPipeline rules | Week 2-4 | Platform team |
| Rotate all credentials | Week 4 | Security team |

### Performance Optimization

| Area | Action | Expected Improvement |
|------|--------|---------------------|
| Dashboard queries | Replace entity ID filters with tags | Faster queries, portable |
| SLO definitions | Use tag-based filters exclusively | No entity ID maintenance |
| Log routing | Optimize OpenPipeline rules for new bucket structure | Reduced cost, better retention |
| Synthetic monitors | Consolidate redundant monitors | Reduced DPS consumption |

<a id="decommission"></a>
## 2. Source Tenant Decommission

### Decommission Timeline

| Week | Action | Status |
|------|--------|--------|
| 0 | Cutover complete | ☐ |
| 1-4 | Source in read-only mode for reference | ☐ |
| 4 | Export final audit logs from source | ☐ |
| 4 | Revoke all API tokens in source | ☐ |
| 4 | Remove SAML/SSO configuration from IdP | ☐ |
| 5 | Decommission source ActiveGates | ☐ |
| 6 | Remove source OneAgent configurations | ☐ |
| 8 | Request Dynatrace to deprovision source tenant | ☐ |

### Pre-Decommission Checklist

```
□ All stakeholders signed off on target tenant
□ No agents reporting to source (verify in source UI)
□ All API tokens revoked in source
□ All OAuth clients deactivated in source
□ SSO/SAML application removed from IdP
□ Audit logs exported for compliance retention
□ Source tenant bookmark/URL redirects configured
□ Documentation updated to reference target tenant only
```

<a id="documentation"></a>
## 3. Documentation and Handover

### Update These Documents

| Document | Update |
|----------|--------|
| Runbooks | Replace source tenant URLs with target |
| On-call documentation | Update dashboard links, SSO URLs |
| CI/CD pipelines | Update API endpoints and tokens |
| Architecture diagrams | Reflect new tenant and integrations |
| Compliance documentation | Update data residency records |
| Vendor contacts | Update Dynatrace support case references |

<a id="lessons-learned"></a>
## 4. Lessons Learned Framework

### Capture Template

| Category | Question |
|----------|----------|
| **Planning** | What did we underestimate? |
| **Tooling** | What tool gaps did we encounter? |
| **Communication** | Did stakeholders feel informed? |
| **Timeline** | Was the parallel period long enough? |
| **Cost** | Did parallel operation exceed budget? |
| **Technical** | What entity ID / integration issues surprised us? |
| **Security** | Were there access gaps during cutover? |

### Common Lessons from S2S Migrations

| Lesson | Detail |
|--------|--------|
| Entity IDs are everywhere | Audit ALL dashboards and SLOs before migration, not after |
| Davis AI needs time | Plan 2-4 weeks of parallel for baseline stability |
| Credentials are the long tail | Every team has API tokens you don't know about |
| Communication is 50% of the work | Over-communicate timeline changes |
| Test SSO early | SAML configuration issues are the #1 day-of-cutover blocker |

<a id="series-summary"></a>
## 5. Series Summary

### S2S Migration Path

| Notebook | Title | Key Deliverable |
|----------|-------|-----------------|
| S2S-01 | Migration Scenarios and Readiness | Go/no-go readiness assessment |
| S2S-02 | Discovery and Configuration Export | Complete configuration export |
| S2S-03 | IAM, SSO, and User Migration | Identity infrastructure in target |
| S2S-04 | OneAgent and ActiveGate Cutover | Agent migration plan and execution |
| S2S-05 | Settings and Configuration Import | Configuration deployed to target |
| S2S-06 | Cloud Integration Migration | Cloud provider connections active |
| S2S-07 | Dashboard, Workflow, and Integration Migration | User-facing config migrated |
| S2S-08 | OpenPipeline and Grail Bucket Migration | Data routing operational |
| S2S-09 | Data Continuity and Parallel Operation | Parallel operation plan |
| S2S-10 | SLO and Alerting Migration | Monitoring operational in target |
| S2S-11 | Cutover Execution and Validation | Migration complete and validated |
| S2S-12 | Post-Migration Optimization and Decommission | Source decommissioned |

### Key Principles

1. **Export early, validate often** — don't wait for cutover to discover missing config
2. **Tags over entity IDs** — make all configuration portable before migration
3. **Parallel is not optional** — historical data doesn't migrate
4. **IAM first** — identity is the foundation; everything else depends on it
5. **Communicate relentlessly** — migration is as much about people as technology

---

## References

- [Terraform Best Practices for Migration](https://docs.dynatrace.com/docs/deliver/configuration-as-code/terraform/best-practices/terraform-best-practices)
- [Monaco Documentation](https://docs.dynatrace.com/docs/deliver/configuration-as-code/monaco)
- [Dynatrace Terraform Provider](https://registry.terraform.io/providers/dynatrace-oss/dynatrace/latest/docs)
- [Settings API](https://docs.dynatrace.com/docs/dynatrace-api/environment-api/settings)
- [Account Management API](https://docs.dynatrace.com/docs/manage/access-control/account-management-api)
- [OneAgent Configuration](https://docs.dynatrace.com/docs/setup-and-configuration/dynatrace-oneagent/oneagent-configuration-via-command-line-interface)
- [AWS Integration](https://docs.dynatrace.com/docs/setup-and-configuration/setup-on-cloud-platforms/amazon-web-services)
- [Azure Integration](https://docs.dynatrace.com/docs/setup-and-configuration/setup-on-cloud-platforms/microsoft-azure)
- [GCP Integration](https://docs.dynatrace.com/docs/setup-and-configuration/setup-on-cloud-platforms/google-cloud-platform)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
