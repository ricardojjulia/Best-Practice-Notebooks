# WEBRUM-05: Error Analysis

> **Series:** WEBRUM | **Notebook:** 5 of 8 | **Created:** March 2026 | **Last Updated:** 03/12/2026

## Overview

JavaScript errors are among the most impactful issues affecting web application user experience. A single unhandled exception can break page functionality, prevent form submissions, or cause entire features to stop working. Dynatrace RUM captures JavaScript errors, XHR/fetch errors, and custom errors in real-time, enabling you to quantify error impact on real users.

This notebook covers error types, error grouping, impact analysis (how many sessions/users are affected), rage click detection, and error-to-session correlation using DQL.

---

## Table of Contents

1. [Error Types in RUM](#error-types)
2. [Error Volume and Trends](#error-volume)
3. [Error Grouping and Top Errors](#error-grouping)
4. [Error Impact Analysis](#error-impact)
5. [XHR and Fetch Errors](#xhr-errors)
6. [Rage Click Detection](#rage-clicks)
7. [Error-to-Session Correlation](#error-session)
8. [Summary and Next Steps](#summary)

---

## Prerequisites

| Requirement | Details |
|-------------|----------|
| **Dynatrace Environment** | SaaS with Grail enabled |
| **RUM Enabled** | Web applications with error capture active |
| **Permissions** | `storage:events:read` |
| **Previous Notebook** | WEBRUM-01: RUM Fundamentals |

<a id="error-types"></a>

## 1. Error Types in RUM

Dynatrace captures several categories of browser-side errors:

| Error Type | Description | Example |
|-----------|-------------|----------|
| **JavaScript error** | Unhandled exception or runtime error | `TypeError: Cannot read property 'x' of undefined` |
| **XHR error** | Failed XHR/fetch request (HTTP 4xx/5xx or network error) | `HTTP 500 on POST /api/checkout` |
| **Custom error** | Errors reported via the RUM API (`dtrum.reportError()`) | `Payment validation failed` |
| **CSP violation** | Content Security Policy blocking a resource | `Refused to load script from 'http://evil.com'` |
| **Resource error** | Failed resource loading (images, scripts, stylesheets) | `Failed to load /assets/app.js` |

### Error Data Model

| Field | Description |
|-------|-------------|
| `error.message` | Error message text |
| `error.type` | Error classification |
| `error.source` | Source file and line number |
| `action.name` | User action during which the error occurred |
| `session.id` | Session containing the error |
| `app.name` | Application name |

```dql
// Recent RUM errors — explore the data structure
fetch dt.rum.error, from:-1h
| fieldsKeep timestamp, error.message, error.type, error.source, action.name, app.name, session.id
| sort timestamp desc
| limit 20
```

<a id="error-volume"></a>

## 2. Error Volume and Trends

Start by understanding overall error volume and how it changes over time.

```dql
// Error count by type over last 24 hours
fetch dt.rum.error, from:-24h
| summarize error_count = count(), by:{error.type}
| sort error_count desc
```

```dql
// Error trend over 24 hours — hourly bucketed by error type
fetch dt.rum.error, from:-24h
| makeTimeseries error_count = count(), interval:1h, by:{error.type}
```

```dql
// Error volume by application — which apps have the most errors?
fetch dt.rum.error, from:-24h
| summarize error_count = count(),
    unique_errors = countDistinct(error.message),
    affected_sessions = countDistinct(session.id),
    by:{app.name}
| sort error_count desc
```

<a id="error-grouping"></a>

## 3. Error Grouping and Top Errors

Error messages often contain variable data (stack traces, URLs, IDs). Grouping by error message helps identify the most frequent issues.

```dql
// Top 15 errors by frequency — the most common error messages
fetch dt.rum.error, from:-24h
| summarize error_count = count(),
    affected_sessions = countDistinct(session.id),
    by:{error.message, error.type}
| sort error_count desc
| limit 15
```

```dql
// Errors by page — which pages generate the most errors?
fetch dt.rum.error, from:-24h
| filter isNotNull(action.name)
| summarize error_count = count(),
    unique_errors = countDistinct(error.message),
    by:{action.name}
| sort error_count desc
| limit 10
```

<a id="error-impact"></a>

## 4. Error Impact Analysis

Not all errors are equal. An error affecting 1 session is different from one affecting 10,000. Impact analysis quantifies the blast radius of each error.

```dql
// Error impact — errors ranked by number of affected sessions
fetch dt.rum.error, from:-24h
| summarize total_occurrences = count(),
    affected_sessions = countDistinct(session.id),
    by:{error.message}
| sort affected_sessions desc
| limit 10
```

```dql
// Error rate per application — percentage of sessions with errors
fetch dt.rum.user_session, from:-24h
| filter user.type == "REAL_USER"
| summarize total_sessions = count(),
    error_sessions = countIf(error.count > 0),
    by:{app.name}
| fieldsAdd error_rate_pct = round(toDouble(error_sessions) / toDouble(total_sessions) * 100.0, decimals: 2)
| sort error_rate_pct desc
```

> **Tip:** Focus on errors with high session impact rather than high occurrence count. A single user hitting a retry loop can generate thousands of occurrences from one session.

<a id="xhr-errors"></a>

## 5. XHR and Fetch Errors

XHR/fetch errors indicate backend API failures impacting the frontend. These are particularly critical in SPAs where the UI depends entirely on API responses.

```dql
// XHR errors by action — identify failing API calls
fetch dt.rum.error, from:-24h
| filter error.type == "XHR_ERROR" or error.type == "FETCH_ERROR"
| summarize error_count = count(),
    affected_sessions = countDistinct(session.id),
    by:{error.message, action.name}
| sort error_count desc
| limit 15
```

```dql
// XHR error trend — are backend errors increasing?
fetch dt.rum.error, from:-24h
| filter error.type == "XHR_ERROR" or error.type == "FETCH_ERROR"
| makeTimeseries error_count = count(), interval:1h
```

<a id="rage-clicks"></a>

## 6. Rage Click Detection

A **rage click** occurs when a user rapidly clicks the same element multiple times — a strong signal of frustration. This usually indicates:

- A button that appears clickable but is not responding
- A form submission that is silently failing
- A UI element that is loading too slowly
- A dead link or broken navigation element

Dynatrace captures rage clicks as user action properties. They can also be detected through rapid action sequences:

```dql
// Sessions with rage clicks — identify frustrated users
fetch dt.rum.user_action, from:-24h
| filter isNotNull(rage.click) and rage.click == true
| summarize rage_count = count(),
    by:{action.name, app.name}
| sort rage_count desc
| limit 10
```

```dql
// Rage click trend over 7 days — is frustration increasing?
fetch dt.rum.user_action, from:-7d
| filter isNotNull(rage.click) and rage.click == true
| makeTimeseries rage_count = count(), interval:1d
```

<a id="error-session"></a>

## 7. Error-to-Session Correlation

Understanding how errors correlate with session outcomes (bounce, low engagement, failed conversion) helps prioritize fixes.

```dql
// Compare sessions with and without errors — engagement impact
fetch dt.rum.user_session, from:-24h
| filter user.type == "REAL_USER"
| fieldsAdd has_errors = if(error.count > 0, "With Errors", else: "No Errors")
| summarize session_count = count(),
    avg_actions = avg(action.count),
    avg_duration_sec = avg(toDouble(duration) / 1000000000.0),
    bounce_rate = countIf(action.count == 1),
    by:{has_errors}
| fieldsAdd bounce_pct = round(toDouble(bounce_rate) / toDouble(session_count) * 100.0, decimals: 1)
```

```dql
// Errors by browser — are certain browsers more error-prone?
fetch dt.rum.user_session, from:-24h
| filter user.type == "REAL_USER"
| filter isNotNull(browser.family)
| summarize total = count(),
    with_errors = countIf(error.count > 0),
    by:{browser.family}
| fieldsAdd error_rate = round(toDouble(with_errors) / toDouble(total) * 100.0, decimals: 1)
| filter total > 10
| sort error_rate desc
| limit 10
```

<a id="summary"></a>

## 8. Summary and Next Steps

In this notebook, we covered:

- **Error types** — JavaScript errors, XHR errors, custom errors, resource errors
- **Error volume and trends** — Tracking error rates over time
- **Error grouping** — Identifying the most frequent error messages
- **Impact analysis** — Measuring how many sessions each error affects
- **XHR/fetch errors** — Backend API failures visible from the frontend
- **Rage clicks** — Detecting user frustration through rapid repeated clicks
- **Error-session correlation** — How errors impact engagement and bounce rates

### Next Steps

- **WEBRUM-06: Performance Analysis** — Page load waterfall and performance bottleneck identification
- **WEBRUM-07: Session Replay** — Visually investigate error-impacted sessions

### References

- [JavaScript Error Tracking](https://docs.dynatrace.com/docs/platform-modules/digital-experience/web-applications/analyze-and-use/javascript-errors)
- [Rage Click Detection](https://docs.dynatrace.com/docs/platform-modules/digital-experience/web-applications/analyze-and-use/rage-clicks)

---

<sub>*This notebook was AI-generated from community-submitted and publicly available sources. This notebook series is not officially supported by Dynatrace. Always verify information against official Dynatrace documentation.*</sub>
