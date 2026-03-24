# S2S-11: Cutover Execution and Validation

> **Series:** S2S | **Notebook:** 11 of 12 | **Created:** March 2026 | **Last Updated:** 03/23/2026

## Overview

Cutover is the point of no return — all agents, integrations, and users switch to the target tenant. This notebook provides the go/no-go checklist, cutover execution steps, and DQL validation queries to confirm migration success.

---

## Table of Contents

1. [Go/No-Go Checklist](#go-no-go)
2. [Cutover Execution Steps](#cutover-steps)
3. [Post-Cutover Validation Queries](#validation-queries)
4. [Rollback Procedures](#rollback)
5. [Stakeholder Sign-Off](#sign-off)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **All Prior Notebooks** | S2S-01 through S2S-10 completed |
| **Parallel Period** | Target tenant has been running in parallel |
| **Change Approval** | Change management approved |

<a id="go-no-go"></a>
## 1. Go/No-Go Checklist

### Infrastructure

| Check | Status | Owner |
|-------|--------|-------|
| All OneAgents reporting to target | ☐ | Platform Team |
| All ActiveGates healthy in target | ☐ | Platform Team |
| K8s Operators reconfigured | ☐ | Platform Team |
| Network zones configured | ☐ | Network Team |
| Cloud integrations active | ☐ | Cloud Team |

### Configuration

| Check | Status | Owner |
|-------|--------|-------|
| Settings 2.0 count matches source | ☐ | Migration Lead |
| Dashboards imported and verified | ☐ | Dashboard Owners |
| SLOs created and evaluating | ☐ | SRE Team |
| Notification rules active | ☐ | SRE Team |
| Workflows deployed and tested | ☐ | Automation Team |
| Synthetic monitors executing | ☐ | QA Team |
| Extensions installed | ☐ | Platform Team |

### Security

| Check | Status | Owner |
|-------|--------|-------|
| IAM groups and policies deployed | ☐ | Security Team |
| SSO/SAML configured and tested | ☐ | Identity Team |
| OAuth clients created | ☐ | Platform Team |
| API tokens created for all consumers | ☐ | Development Teams |
| External user access configured | ☐ | Security Team |

### Data

| Check | Status | Owner |
|-------|--------|-------|
| Grail buckets created | ☐ | Platform Team |
| OpenPipeline rules deployed | ☐ | Platform Team |
| Enrichment rules active | ☐ | Platform Team |
| Retention policies verified | ☐ | Compliance Team |
| Davis AI baselines stabilizing | ☐ | SRE Team |

<a id="cutover-steps"></a>
## 2. Cutover Execution Steps

| Step | Action | Duration | Rollback |
|------|--------|----------|----------|
| 1 | Freeze configuration changes in source | — | Unfreeze |
| 2 | Final configuration sync (Monaco download → deploy) | 30 min | — |
| 3 | Redirect remaining OneAgents to target | 1-2 hours | `oneagentctl` back to source |
| 4 | Update DNS/bookmarks to target tenant URL | 15 min | Revert DNS |
| 5 | Disable alerting in source tenant | 5 min | Re-enable |
| 6 | Verify all agents reporting in target | 30 min | — |
| 7 | Run validation DQL queries | 30 min | — |
| 8 | Notify stakeholders: cutover complete | — | — |
| 9 | Set source tenant to read-only retention | 15 min | — |

<a id="validation-queries"></a>
## 3. Post-Cutover Validation Queries

Run these against the **target** tenant:

```dql
// Verify host count matches expected
fetch dt.entity.host
| summarize host_count = count()
```

```dql
// Verify services are detected
fetch dt.entity.service
| summarize service_count = count()
```

```dql
// Verify logs are flowing
fetch logs, from:-1h
| summarize log_count = count()
| fieldsAdd status = if(log_count > 0, then: "HEALTHY", else: "NO DATA")
```

```dql
// Verify spans are flowing
fetch spans, from:-1h
| summarize span_count = count()
| fieldsAdd status = if(span_count > 0, then: "HEALTHY", else: "NO DATA")
```

```dql
// Verify enrichment tags are populated
fetch logs, from:-1h
| filter isNotNull(dt.security_context)
| summarize enriched_count = count()
| fieldsAdd status = if(enriched_count > 0, then: "ENRICHMENT ACTIVE", else: "CHECK ENRICHMENT RULES")
```

<a id="rollback"></a>
## 4. Rollback Procedures

| Trigger | Action | Impact |
|---------|--------|--------|
| Mass agent failure | `oneagentctl` fleet-wide back to source | 15-30 min |
| Critical configuration missing | Deploy from exported config | Minutes |
| SSO failure | Use fallback local admin account | Immediate |
| Data not flowing | Check OpenPipeline rules and ActiveGate | Debug and fix |

> **Critical:** Keep the source tenant running for at least 30 days after cutover. Do not decommission until all stakeholders confirm the target is fully operational.

<a id="sign-off"></a>
## 5. Stakeholder Sign-Off

| Stakeholder | Validates | Sign-Off Date |
|-------------|-----------|---------------|
| SRE Team | Alerting, SLOs, on-call routing | ☐ |
| Development Teams | Dashboards, service visibility | ☐ |
| Security Team | IAM, SSO, audit logging | ☐ |
| Management | SLO reporting, executive dashboards | ☐ |
| Compliance | Retention, data residency | ☐ |

---

## Next Steps

Continue to **S2S-12: Post-Migration Optimization and Decommission** for the final cleanup.

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
