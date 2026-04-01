# GrafanaCON 2026 — Advanced Dashboarding Lab

A hands-on lab for GrafanaCON 2026 where participants build a production-grade Grafana dashboard from scratch, using the **Grot Plushies** e-commerce platform as the scenario.

## What You'll Build

A single, multi-tab SRE dashboard that takes you from alert to root cause without leaving the page. The dashboard serves three personas — on-call SRE, service owner, and engineering manager — through four interconnected tabs:

| Tab | Focus |
|-----|-------|
| Fleet Overview | Top-level KPIs, per-service health, conditional panels |
| Service Deep Dive | Endpoint latency, error breakdown, slow span investigation |
| Business Metrics | SLO compliance, error budgets, risk ranking |
| Database | Postgres correlation for DB-backed services |

## Features Covered

Tabs & auto layout, template variables, panel repeats, ad-hoc filters, show/hide rules, section-level variables, field overrides, data links, saved queries, SQL Expressions, and the Dashboard datasource.

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
