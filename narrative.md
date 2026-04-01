# GrafanaCON Lab — Narrative & Dashboard Design

---

## Narrative: "Grot Plushies SRE"

You're the SRE on-call for Grot Plushies, a microservices e-commerce platform. An alert just fired. You open the dashboard to investigate... Over the course of this lab, you will build the very dashboard you'd want in that moment.

The platform runs five services:

- **Frontend** : the user-facing storefront
- **CartService** : manages shopping carts (backed by Redis + Postgres)
- **CheckoutService** : orchestrates order placement (backed by Postgres)
- **PaymentService** : processes payments
- **ProductCatalog** : serves product listings

Your job: build a single dashboard that gives you - and your team - everything needed to go from alert to root cause and impact on business metrics without opening a second tab.

---

## Personas


| Persona    | Role                | Needs                                                            |
| ---------- | ------------------- | ---------------------------------------------------------------- |
| **Alex**   | SRE On-call         | Fleet health, error spikes, latency, SLO burn rate               |
| **Priya**  | Service Owner       | Per-service deep dive, endpoint breakdown, logs                  |
| **Jordan** | Engineering Manager | SLO compliance, error budgets, business impact (orders, revenue) |


---

## Persona Navigation Flows


| Persona    | Path                                       | Trigger                                        |
| ---------- | ------------------------------------------ | ---------------------------------------------- |
| **Alex**   | Tab 1 → Tab 4                              | Alert fires → fleet anomaly → DB investigation |
| **Priya**  | Tab 1 (click service card) → Tab 2 → Tab 4 | Investigating a specific service end-to-end    |
| **Jordan** | Tab 3 direct                               | Reviewing SLO compliance and business impact   |


---

## Datasources


| Datasource          | Type       | Used for                                           |
| ------------------- | ---------- | -------------------------------------------------- |
| `grafanacloud-prom` | Prometheus | All service metrics, SLO metrics, Postgres metrics |
| `grafanacloud-logs` | Loki       | Service logs, feature flag events, pod events      |


---

## Dashboard Variables


| Variable       | Type   | Query                                                                                           | Scope                      |
| -------------- | ------ | ----------------------------------------------------------------------------------------------- | -------------------------- |
| `cluster`      | Query  | `label_values(traces_spanmetrics_calls_total, k8s_cluster_name)`                                | Dashboard                  |
| `namespace`    | Query  | `label_values(traces_spanmetrics_calls_total{k8s_cluster_name="$cluster"}, k8s_namespace_name)` | Dashboard                  |
| `service_name` | Query  | `label_values(traces_spanmetrics_calls_total{k8s_namespace_name="$namespace"}, service_name)`   | Dashboard                  |
| `span_name`    | Query  | `label_values(traces_spanmetrics_calls_total{service_name="$service_name"}, span_name)`         | Tab 2 only (section-level) |
| `time_window`  | Custom | `1h : 1h, 6h : 6h, 1d : 1d`                                                                     | Tab 3 only (section-level) |


---

## Tabs & Content

### Tab 1 — Fleet Overview

*Exercises: tabs, layouts, variables, repeats, ad-hoc filters, show/hide rules*

**Stat row — top KPIs:**

- Total RPS across all services
- Overall error rate
- Active SLO breaches (from `grafana_slo_`*)

**Repeated panel group — one group per service** (`repeat by service_name`):

- Request rate sparkline (stat panel)
- Error rate (stat panel, color threshold: green < 1%, red > 1%)
- p95 latency (stat panel)
- Data link on each panel group → Tab 2 (Service Deep Dive), pre-filtered to that service

**Conditional panels (show/hide rules):**

- Postgres saturation panel → visible only when `service_name` is `cartservice` or `checkoutservice`
- Feature flag activity panel (Loki) → visible only when a single service is selected


**Annotations:**

- Deployment releases (Prometheus, `release` tag)
- Feature flag changes (Loki, `{job="$namespace/flagapi"}`)

---

### Tab 2 — Service Deep Dive

*Exercises: section-level variables, field overrides, data links, saved queries*

**Section-level variables:** `span_name`

**Panels:**

- Latency timeseries by span/endpoint — p50, p95, p99
  - Field overrides: p99 dashed red, p95 orange, p50 green
- Error breakdown by status code (bar chart)
  - Field overrides: 5xx red, 4xx orange, 2xx green
- Request rate by span (timeseries)
- Top 10 slowest spans (table)
  - Data link on each row → Loki logs filtered to `service_name` + `span_name` + current time range
  - Secondary data link (visible only when `service_name` is `cartservice` or `checkoutservice`) → Tab 4 (Database), scoped to that service
- Saved query: **p95 latency** (`histogram_quantile(0.95, ...)`) — reused across this tab and Tab 1

---

### Tab 3 — SLOs & Business Impact

*Exercises: section-level variables, SQL Expressions, dashboard datasource, transformations*

**Section-level variable:** `time_window` (1h / 6h / 1d)

**Stat row — top KPIs (Jordan's first look):**

- Orders in time window (dashboard datasource — reuses Tab 1 query, no second fetch)
- Revenue in time window
- Sessions in time window

**Panels:**

- SLO compliance per service (stat panels, color-mapped: green ≥ 99.9%, red < 99.9%)
- 28-day rolling SLI timeseries per service
- Error budget burn rate — 1h and 6h windows (timeseries)
- 28-day remaining error budget (gauge per service)
- **SQL Expression panel** — service risk ranking:
  ```sql
  SELECT service_name,
         burn_rate,
         error_rate,
         burn_rate * error_rate AS risk_score
  FROM A
  JOIN B ON A.service_name = B.service_name
  ORDER BY risk_score DESC
  ```

---

### Tab 4 — Database

*Exercises: show/hide rules, dashboard datasource, field overrides*

**Entry point:** reached via data link from the slow spans table in Tab 2, or directly when investigating DB-backed services. Not intended as a default landing tab.

**Conditional visibility:** entire tab content scoped to DB-backed services (`cartservice`, `checkoutservice`)

**Panels:**

- QPS (queries per second) — commit + rollback rate
- Cache hit ratio (gauge, threshold: red < 90%, green > 99%)
- Active connections (timeseries, threshold overlay at max connections)
- Deadlocks & conflicts (timeseries, field overrides: deadlocks red, conflicts orange)
- Buffer allocation (timeseries)
- **Scoped context panel** — text panel at the top showing the currently selected `service_name`, so participants who land here via data link know what they're looking at
- **Dashboard datasource panel** — service RPS from Tab 1 overlaid with DB QPS to correlate load with database pressure

---

## Feature → Exercise Mapping


| #   | Feature                 | What participants build                                                                     |
| --- | ----------------------- | ------------------------------------------------------------------------------------------- |
| 1   | Tabs + Grid Layout      | 4-tab shell, stat row on Tab 1                                                              |
| 2   | Dashboard variables     | `cluster`, `namespace`, `service_name`                                                      |
| 3   | Panel repeats           | Repeat service stat group by `service_name`                                                 |
| 4   | Ad-hoc filters          | Add filter bar, test with `service_name`                                                    |
| 5   | Show/hide rules         | Postgres panel visible only for DB-backed services                                          |
| 6   | Section-level variables | Add `span_name` to Tab 2, `time_window` to Tab 3                                            |
| 7   | Field overrides         | Color latency lines by percentile, status code bars                                         |
| 8   | Data links              | Wire slow spans table → Loki logs; service card → Tab 2; slow spans → Tab 4 for DB services |
| 9   | Saved queries           | Extract p95 latency query into shared library                                               |
| 10  | Transformations         | Join span metrics + SLO data                                                                |
| 11  | SQL Expressions         | Risk score ranking table on Tab 3                                                           |
| 12  | Dashboard datasource    | Overlay service RPS on DB QPS in Tab 4                                                      |


