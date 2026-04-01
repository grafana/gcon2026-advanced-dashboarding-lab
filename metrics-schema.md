# GrafanaCON Lab — Metrics & Data Reference

All data comes from the existing OnlineBoutique demo environment.
No synthetic data needed — everything below is already present in the instance.

---

## Services

| Service | Label value | DB-backed |
|---|---|---|
| Frontend | `frontend` | No |
| Cart Service | `cartservice` | Yes (Postgres + Redis) |
| Checkout Service | `checkoutservice` | Yes (Postgres) |
| Payment Service | `paymentservice` | No |
| Product Catalog | `productcatalogservice` | No |

---

## Prometheus Metrics

### OpenTelemetry Span Metrics (all services)

```
traces_spanmetrics_calls_total
  desc:   Total span/request count
  labels: service_name, span_name, status_code, k8s_cluster_name, k8s_namespace_name
  use:    Request rate, error rate (filter status_code=~"STATUS_CODE_ERROR")

traces_spanmetrics_latency_bucket
  desc:   Latency histogram
  labels: service_name, span_name, le, k8s_cluster_name, k8s_namespace_name
  use:    p50/p95/p99 latency via histogram_quantile()
```

**Key PromQL patterns:**

```promql
# Request rate per service
sum by (service_name) (
  rate(traces_spanmetrics_calls_total{k8s_namespace_name="$namespace"}[2m])
)

# Error rate per service
sum by (service_name) (
  rate(traces_spanmetrics_calls_total{
    k8s_namespace_name="$namespace",
    status_code="STATUS_CODE_ERROR"
  }[2m])
)
/ sum by (service_name) (
  rate(traces_spanmetrics_calls_total{k8s_namespace_name="$namespace"}[2m])
)

# p95 latency per service (saved query candidate)
histogram_quantile(0.95,
  sum by (service_name, le) (
    rate(traces_spanmetrics_latency_bucket{k8s_namespace_name="$namespace"}[2m])
  )
)

# p95 latency by span/endpoint (Tab 2)
histogram_quantile(0.95,
  sum by (span_name, le) (
    rate(traces_spanmetrics_latency_bucket{
      service_name="$service_name",
      k8s_namespace_name="$namespace"
    }[2m])
  )
)
```

---

### SLO Metrics (Grafana SLO app)

```
grafana_slo_success_rate_5m
  desc:   Recorded 5-minute success rate per SLO
  labels: grafana_slo_uuid, service (cartservice, frontend, etc.)
  use:    SLI panels, burn rate calculations

grafana_slo_requests_total_5m
  desc:   Recorded 5-minute total request count per SLO
  labels: grafana_slo_uuid
  use:    Denominator for SLI/burn rate
```

**Key PromQL patterns:**

```promql
# Current SLI (success rate)
grafana_slo_success_rate_5m{service="$service_name"}

# Error budget burn rate (1h window)
(1 - grafana_slo_success_rate_5m{service="$service_name"}) / (1 - 0.999)

# 28-day rolling SLI
avg_over_time(grafana_slo_success_rate_5m{service="$service_name"}[28d])

# Remaining error budget (%)
100 * (
  1 - (
    (1 - avg_over_time(grafana_slo_success_rate_5m{service="$service_name"}[28d]))
    / (1 - 0.999)
  )
)
```

---

### PostgreSQL Metrics

```
pg_stat_database_xact_commit        — committed transactions (rate → QPS)
pg_stat_database_xact_rollback      — rolled back transactions
pg_stat_database_blks_hit           — buffer cache hits
pg_stat_database_blks_read          — disk reads (cache miss)
pg_stat_database_numbackends        — active connections
pg_stat_database_deadlocks          — deadlock count
pg_stat_database_conflicts          — conflict count
pg_stat_database_tup_fetched        — rows fetched
pg_stat_bgwriter_buffers_alloc_total — buffer allocations

Labels: job, cluster, instance, datname
```

**Key PromQL patterns:**

```promql
# QPS
sum(rate(pg_stat_database_xact_commit{job=~"$job"}[$__rate_interval]))
+ sum(rate(pg_stat_database_xact_rollback{job=~"$job"}[$__rate_interval]))

# Cache hit ratio
sum(rate(pg_stat_database_blks_hit{job=~"$job"}[$__rate_interval]))
/ (
  sum(rate(pg_stat_database_blks_hit{job=~"$job"}[$__rate_interval]))
  + sum(rate(pg_stat_database_blks_read{job=~"$job"}[$__rate_interval]))
) * 100

# Active connections
sum(pg_stat_database_numbackends{job=~"$job", datname=~"$db"})
```

---

### Business Metrics

```
# Revenue (available in System Overview dashboard)
# Orders count
# Sessions count
# Ads served
```
> Exact metric names to be confirmed from the instance — visible in the System Overview dashboard queries.

---

## Loki Log Streams

```
# Kubernetes pod events
{job="integrations/kubernetes/eventhandler", namespace="$namespace"}

# Feature flag changes
{job="$namespace/flagapi"} | json | message="Updated flag"
  → fields: checkoutWriteToSFDC, productCatalogReadFromPostgres

# Service logs (for data link target from Tab 2)
{namespace="$namespace", service_name="$service_name"}
```

---

## SQL Expression — Risk Ranking (Tab 3)

Query A (instant): p95 latency per service
Query B (instant): burn rate per service

```sql
SELECT
  A.service_name,
  A.p95_latency,
  B.burn_rate,
  A.p95_latency * B.burn_rate AS risk_score
FROM A
JOIN B ON A.service_name = B.service_name
ORDER BY risk_score DESC
```

---

## Dashboard Datasource — Tab 4 Correlation

Panel in Tab 1: `service_rps` — request rate for `cartservice` / `checkoutservice`
Panel in Tab 4: references that panel via Dashboard datasource, overlaid with `pg_stat_database_xact_commit` rate to show DB load correlation.

---

## Annotations

| Annotation | Source | Query |
|---|---|---|
| Deployments / Releases | Prometheus | `{__name__=~".*", release!=""}` tag filter |
| Feature flag changes | Loki | `{job="$namespace/flagapi"} \| json \| message="Updated flag"` |
| Pod events | Loki | `{job="integrations/kubernetes/eventhandler", namespace="$namespace"}` |
