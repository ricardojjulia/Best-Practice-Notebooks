# Custom Metrics and Attributes — Creation, Management, and Best Practices

## Overview
Custom metrics and attributes extend Dynatrace's observability beyond pre-configured instrumentation. This guide covers ingestion patterns, naming conventions, cardinality management, and integration strategies.

## Custom Metrics
### Sources & Ingestion Patterns
- **Metric API** — RESTful ingestion for application-emitted metrics
- **OpenMetrics/Prometheus** — Pull-based scraping via extension plugins
- **OneAgent API** — Agent-side custom metric capture
- **log2metrics** — Automated extraction from structured logs

### Naming & Organization
- **Metric naming** — Domain-qualified format (e.g., `app.payment.latency_ms`)
- **Dimensionality** — Limit cardinality via key dimensions (e.g., `payment_method`, not user ID)
- **Versioning** — Manage metric evolution without breaking queries or dashboards

### Cardinality & Scale
- **Unbounded dimensions** — Avoid high-cardinality attributes (user IDs, request IDs, exact IP addresses)
- **Rollup aggregation** — Pre-aggregate before ingestion when possible
- **Retention tiers** — Custom metrics follow standard DPU billing and retention

## Custom Attributes
### Application & Request Attributes
- **OneAgent capture** — Session properties, user actions, business context
- **Span attributes** — OpenTelemetry OTEL span enrichment
- **Service-level tagging** — Meta-enrichment for routing and grouping

### Best Practices
- **Schema consistency** — Coordinate naming across teams to avoid duplication
- **Masking sensitive data** — PII scrubbing policies and regex patterns
- **Indexing strategy** — Choose indexed vs. non-indexed attributes based on query patterns

## Integration Workflows
1. **Define scope** — What data, where collected, what dimension?
2. **Ingest & validate** — Use Logs viewer or Metrics Explorer to confirm receipt
3. **Query & alert** — Build dashboards and problem detection rules
4. **Iterate** — Refine based on observability gaps and cost feedback

## Common Pitfalls
- High-cardinality attributes (unbounded user/request IDs)
- Inconsistent naming across teams
- Over-instrumentation (collect everything, manage nothing)
- Missing dimension context for root-cause isolation

---

*For detailed API examples, see the [Dynatrace API documentation](https://docs.dynatrace.com/). For OneAgent custom metric capture, see the Dynatrace plugin SDKs.*
