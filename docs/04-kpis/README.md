# Key Performance Indicators (KPIs)

> **Status:** Draft v1 · **Owner:** M1 (Project Lead), measured with M6 · **Last updated:** 2026-10-10
> Targets are proposed starting points for a small-scale project. Items marked *(baseline)* are confirmed after the first full pipeline run (end of phase 3), then locked.

---

## 1. How the Standard KPIs Map to This Project

| Common KPI | What it means for a data platform project |
|---|---|
| Response time | Query response time on serverless SQL and the warehouse; Spark job runtime |
| System uptime | Pipeline success rate and availability of the serving layer during agreed working / demo windows (pools are paused outside them on purpose, so 24/7 uptime isn't meaningful here) |
| User adoption rate | How many business questions the platform answers, how many people can run the sample queries and reproduce the project, and stakeholder feedback |

---

## 2. KPI Catalogue

### 2.1 Delivery

| ID | KPI | Definition | Target | Source | Frequency | Owner |
|---|---|---|---|---|---|---|
| DEL-1 | Milestone on-time rate | Milestones met by planned date ÷ milestones due | ≥ 5 of 6 on time | Project plan | At each milestone | M1 |
| DEL-2 | Deliverable completion | Deliverables marked done ÷ 9 required | 100% by M6 | Deliverables table | Weekly | M1 |
| DEL-3 | Weekly task completion | Tasks completed ÷ tasks planned for the week | ≥ 85% | GitHub Projects board | Weekly | M1 |
| DEL-4 | Overdue tasks | Open tasks past their due date | ≤ 3 at any time | GitHub Projects board | Weekly | M1 |
| DEL-5 | PR review turnaround | Time from PR opened to first review | < 24 h | GitHub | Weekly | M1 |

### 2.2 Pipeline Reliability (≈ uptime)

| ID | KPI | Definition | Target | Source | Frequency | Owner |
|---|---|---|---|---|---|---|
| REL-1 | Pipeline run success rate | Successful scheduled / triggered runs ÷ total runs, last 2 weeks | ≥ 95% | Synapse Studio → Monitor → Pipeline runs | Weekly | M2 |
| REL-2 | Mean time to recover | Average time from failed run to next successful run | < 4 working hours | Monitor hub + issue log | Per incident | M2 |
| REL-3 | Serving-layer availability | Time the warehouse / serverless endpoint is reachable during agreed working windows ÷ window time | ≥ 99% | Azure Monitor / manual check log | Weekly | M2 |
| REL-4 | End-to-end pipeline runtime | Raw → curated → warehouse on the sample data | ≤ 30 min *(baseline)* | Monitor hub | Per release | M2 |

### 2.3 Performance (≈ response time)

| ID | KPI | Definition | Target | Source | Frequency | Owner |
|---|---|---|---|---|---|---|
| PERF-1 | Serverless SQL query time | 95th-percentile duration of the sample queries on curated Parquet | ≤ 10 s | Monitor → SQL requests | Weekly from M3 | M5 |
| PERF-2 | Warehouse query time | 95th-percentile duration of the sample analytical queries | ≤ 5 s | Query duration log | Weekly from M4 | M5 |
| PERF-3 | Spark job runtime | Runtime per notebook; improvement after optimisation | Baseline, then −20% on the 2 slowest jobs | Monitor → Apache Spark applications | Per release | M4 |
| PERF-4 | Storage reduction | Parquet size vs. original CSV for the same dataset | ≥ 50% smaller | Storage explorer | Once per dataset | M3 |

### 2.4 Data Quality

| ID | KPI | Definition | Target | Source | Frequency | Owner |
|---|---|---|---|---|---|---|
| DQ-1 | DQ check pass rate | Passed checks ÷ total checks per run | 100% of critical checks; ≥ 98% overall | `data_quality/` results | Every run | M6 |
| DQ-2 | Row-count reconciliation | Tables where raw → curated → warehouse counts match or differences are explained | 100% | DQ report | Every run | M6 |
| DQ-3 | Orphan fact rows | Fact Sales rows with no matching dimension key | 0 | SQL check | Every run | M5 |
| DQ-4 | Key null / duplicate rate | Nulls or duplicates in primary / business keys | 0% | DQ report | Every run | M6 |
| DQ-5 | Data freshness | Time between a raw load and updated warehouse tables | ≤ 1 h *(baseline)* | Pipeline timestamps | Per release | M2 |

### 2.5 Usability & Adoption (≈ user adoption)

| ID | KPI | Definition | Target | Source | Frequency | Owner |
|---|---|---|---|---|---|---|
| USE-1 | Business questions answered | Distinct analytical queries that run successfully on the warehouse (sales by product, store, customer, date, ...) | ≥ 8 | `sql/` folder | Weekly from M4 | M6 |
| USE-2 | Reproducibility | Team members / reviewers who follow the README on a fresh setup and reproduce the results | ≥ 2 reviewers outside the owner | Review log | Once, week 7 | M1 |
| USE-3 | Documentation coverage | Deliverables with a documented README section or notebook header ÷ total | 100% | Checklist | Weekly | M1 |
| USE-4 | Stakeholder feedback | Average demo score (clarity, usefulness, completeness) | ≥ 4 / 5 | Feedback form | After demo | M1 |

### 2.6 Security & Cost

| ID | KPI | Definition | Target | Source | Frequency | Owner |
|---|---|---|---|---|---|---|
| SEC-1 | Secrets in repository | Credentials found in Git history | 0 | Secret scanner | Every PR | M2 |
| SEC-2 | Access-control coverage | Containers, pools, and databases with least-privilege roles assigned ÷ total | 100% | Role-assignment table | At M5 | M2 |
| COST-1 | Azure spend vs. budget | Cumulative spend ÷ planned budget | ≤ 100%; alert at 80% | Cost Management | Weekly | M2 |
| COST-2 | Idle compute | Spark / warehouse pools running outside working hours | 0 occurrences | Monitor + spot check | Weekly | M2 |

---

## 3. Status Thresholds

| Status | Meaning | Action |
|---|---|---|
| 🟩 Green | Target met | None |
| 🟨 Amber | Within 15% of target, or a single miss | Owner explains in the next sync |
| 🟥 Red | Worse than 15% from target, or any **critical** KPI missed | Raise as a risk; M1 decides on corrective action |

**Critical KPIs:** DEL-2, REL-1, DQ-1 (critical checks), DQ-3, SEC-1, COST-1.

---

## 4. Weekly Scorecard (copy per week)

**Week:** __ · **Date:** __ · **Updated by:** __

| ID | KPI | Target | Actual | Status | Note |
|---|---|---|---|---|---|
| DEL-2 | Deliverable completion | 100% by M6 | | | |
| DEL-3 | Weekly task completion | ≥ 85% | | | |
| REL-1 | Pipeline success rate | ≥ 95% | | | |
| PERF-1 | Serverless SQL p95 | ≤ 10 s | | | |
| PERF-2 | Warehouse p95 | ≤ 5 s | | | |
| DQ-1 | DQ pass rate | 100% critical | | | |
| DQ-3 | Orphan fact rows | 0 | | | |
| USE-1 | Business questions answered | ≥ 8 | | | |
| SEC-1 | Secrets in repo | 0 | | | |
| COST-1 | Spend vs. budget | ≤ 100% | | | |

---

## 5. Measurement Plan

- **When:** M1 and M6 fill the scorecard on Friday after the demo and retro.
- **How:** use Synapse Studio's Monitor hub (pipeline runs, SQL requests, Spark applications), Azure Cost Management, the GitHub Projects board, and DQ output saved to `data_quality/`.
- **Baselines:** record REL-4, DQ-5, and PERF-3 after the first end-to-end run; lock targets at M3.
- **Trend, not just status:** keep each week's scorecard in `docs/kpi-history/` so the final report can show how the numbers improved.

---

## 6. Definition of Project Success

The project is successful at M6 if:

1. All 9 required deliverables are complete (DEL-2).
2. The pipeline runs end-to-end reliably (REL-1) and produces trusted data (DQ-1, DQ-2, DQ-3).
3. At least 8 business questions are answered from the warehouse (USE-1) within the performance targets (PERF-1, PERF-2).
4. No secrets leaked and access is least-privilege (SEC-1, SEC-2).
5. Spend stayed within budget (COST-1).
6. A reviewer outside the owner can reproduce the work from the README (USE-2).
