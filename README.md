# GrafanaCON 2026 — Advanced Dashboarding Lab

A hands-on lab for GrafanaCON 2026 where participants build a many-in-one Grafana dashboard from scratch, using the **Grot Plushies** e-commerce platform as the scenario.


## Facilitator
**Yaëlle Chaudy** 
* [<img src="https://cdn-icons-png.flaticon.com/512/25/25231.png" alt="GitHub" width="20" height="20" style="vertical-align: middle;"> yaelleC](https://github.com/yaelleC) 
* [<img src="https://img.icons8.com/color/1200/linkedin.jpg" alt="LinkedIn" width="20" height="20" style="vertical-align: middle;"> yaellechaudy](https://www.linkedin.com/in/yaellechaudy/)


## What You'll Build

A single, many-in-one dashboard that gives SREs, engineering managers, PMs and engineers a shared view of the platform — without opening a second tab. The dashboard is organised around three pillars:

| Tab | Focus |
|-----|-------|
| Fleet Overview | Top-level KPIs, per-service health |
| Logs | Logs per container |
| Service Deep Dive | Endpoint latency, error breakdown, slow span investigation |
| Business Metrics | SLO compliance, active users, sessions, orders |
| Github Stats | Github data about bug reports and customer escalations |

## Features Covered

Tabs & auto layout, template variables, panel repeats, ad-hoc filters, show/hide rules, section-level variables, field overrides, data links, saved queries, SQL Expressions.

## Repo Structure

```
exercises.md                Step-by-step lab exercises (Tasks 1–10)
narrative.md                Scenario context, persona flows, and dashboard design
resources/                  CSV datasets (OSS issue history)
img/                        Screenshots referenced in exercises
step by step - Solutions/   Dashboard JSON snapshots after each step (step1–step8)
```

## Prerequisites

- A Grafana Cloud instance with the OnlineBoutique demo environment
- Datasources: `grafanacloud-prom` (Prometheus) and `grafanacloud-logs` (Loki)

## Getting Started

Open `exercises.md` and follow along from Task 1. Each task builds on the previous one and includes a checkpoint to verify your progress.

For the full scenario context, persona flows and dashboard design notes, see `narrative.md`.
