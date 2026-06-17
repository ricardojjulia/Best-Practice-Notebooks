# ALERT-99: Best-Practice Summary and Setup Checklist

> **Series:** ALERT — Alerting Strategy and Design | **Notebook:** 99 of 05 | **Created:** June 2026 | **Last Updated:** 06/16/2026

## Overview

The series on one page: the principles, a complete end-to-end setup checklist, and the cross-series map of where each piece is built. Use this as the standing reference once you have read ALERT-01 through ALERT-04.

---

## Table of Contents

1. [Principles](#principles)
2. [Complete Setup Checklist](#checklist)
3. [Cross-Series Map](#map)

---

## Prerequisites

| Requirement | Details |
|-------------|---------|
| **Prior reading** | ALERT-01 through ALERT-04 |
| **Companion series** | AIOPS, SLO, WFLOW, OPIPE |

<a id="principles"></a>
## 1. Principles

1. **OOTB Davis first.** Tune before you build; descend the funnel only when the layer above cannot express the condition.
2. **One enriched problem is the hub.** Every mechanism converges on a Davis problem; everything downstream operates on it.
3. **Enrich upstream or you cannot route downstream.** Team/zone/severity belong on the problem before a workflow ever sees it.
4. **Burn-rate beats threshold-on-SLI.** Alert on how fast the budget drains, with multiwindow confirmation.
5. **Prefer simple workflows.** Pay for multi-step only when you need conditional/multi-team logic.
6. **Match destination to urgency.** Fast-burn → page; slow-burn → ticket.
7. **Validate every query before it becomes an alert.** A detector on a silent query is worse than no alert.

<a id="checklist"></a>
## 2. Complete Setup Checklist

**Detection**
- [ ] Reviewed OOTB Davis anomaly detection settings and tuned sensitivity
- [ ] Custom anomaly detectors only where Davis does not cover the condition (auto-adaptive/seasonal, not static-on-traffic)
- [ ] Log/span signals extracted to OpenPipeline metrics before alerting on them
- [ ] SLOs defined for critical user journeys with burn-rate alerts

**Enrichment**
- [ ] Detector event templates carry team / zone / service properties
- [ ] Entity tags and Smartscape ownership populated for OOTB problems
- [ ] `event.severity` used to drive priority

**Routing**
- [ ] Problem-trigger workflows filter on enriched metadata
- [ ] Simple workflows used wherever conditional logic is not needed
- [ ] Fast-burn → page channel; slow-burn → ticket channel
- [ ] One notification per problem (no duplicate alerts on raw metrics)

**ITSM / destinations**
- [ ] ServiceNow incidents carry assignment group, priority, and a de-dup correlation key
- [ ] Legacy alerting-profile integrations identified; migration to workflows considered

**Governance**
- [ ] Production detectors/SLOs promoted to config-as-code
- [ ] Alerts that fire without action reviewed and tuned

<a id="map"></a>
## 3. Cross-Series Map

| Piece | Built in |
|-------|----------|
| Anomaly detector mechanics (analyzer, tuning, event template) | AIOPS-02 |
| Metric events / detector schemas as code | AIOPS-02 §6, AUTOM-05/06 |
| OpenPipeline metric extraction | OPIPE, FAQ-09 |
| SLIs, error budgets, burn-rate alerting | SLO-02 / 03 / 04 |
| SLOs as code | SLO-05 |
| Routing, conditions, escalation | WFLOW-03 / 04 |
| HTTP / webhook actions | WFLOW-08 |
| ServiceNow incident integration | ALERT-04 |
| Cost / DPS framing | FINOPS, FAQ-09 |

> <sub>**Sources:** [Alerting and notifications (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/notifications-and-alerting), [Workflows (DT docs)](https://docs.dynatrace.com/docs/analyze-explore-automate/workflows). **Derived:** the checklist and funnel synthesise the ALERT, AIOPS, SLO, and WFLOW series into one operational sequence.</sub>

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
