# GrafanaCON 2026 — Advanced Dashboarding Lab

A hands-on lab for GrafanaCON 2026 where participants build a production-grade Grafana dashboard from scratch, using the **Grot Plushies** e-commerce platform as the scenario.


## Facilitator
**Yaëlle Chaudy** 
* [<img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" alt="GitHub" width="20" height="20" style="vertical-align: middle;"> yaelleC](https://github.com/yaelleC) 
* [<img src="https://img.icons8.com/color/1200/linkedin.jpg" alt="LinkedIn" width="20" height="20" style="vertical-align: middle;"> yaellechaudy](https://www.linkedin.com/in/yaellechaudy/)


## What You'll Build

A single, multi-tab SRE dashboard that takes you from alert to root cause without leaving the page. The dashboard serves three personas — on-call SRE, service owner, and engineering manager — through four interconnected tabs:

| Tab | Focus |
|-----|-------|
| Fleet Overview | Top-level KPIs, per-service health |
| Logs | Logs per container |
| Service Deep Dive | Endpoint latency, error breakdown, slow span investigation |
| Business Metrics | SLO compliance, active users, sessions, orders |
| Bugs & Escalations | Github data about bug reports and customer escalations |

## Features Covered

Tabs & auto layout, template variables, panel repeats, ad-hoc filters, show/hide rules, section-level variables, field overrides, data links, saved queries, SQL Expressions.

## Repo Structure

```
exercises.md        Step-by-step lab exercises (Tasks 1–10)
narrative.md        Scenario context, persona flows, and dashboard design
metrics-schema.md   Metrics reference (Prometheus, Loki, Postgres, SLO)
resources/          CSV datasets (Grafana open issues, OSS issue history)
```

## Prerequisites

- A Grafana Cloud instance with the OnlineBoutique demo environment
- Datasources: `grafanacloud-prom` (Prometheus) and `grafanacloud-logs` (Loki)

## Getting Started

Open `exercises.md` and follow along from Task 1. Each task builds on the previous one and includes a checkpoint to verify your progress.
