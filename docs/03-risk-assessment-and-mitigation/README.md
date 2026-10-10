# Risk Assessment & Mitigation Plan

> **Status:** Draft v1 · **Owner:** M1 (Project Lead) · **Last updated:** 2026-10-10
> Review weekly in the Monday planning meeting. Update scores, status, and add new risks as they appear.

---

## 1. Method

**Score = Likelihood × Impact**, each rated 1 – 5.

| Rating | Likelihood | Impact |
|---|---|---|
| 1 | Rare | Negligible: no schedule effect |
| 2 | Unlikely | Minor: < 2 days of delay, workaround is easy |
| 3 | Possible | Moderate: a milestone slips a few days |
| 4 | Likely | Major: a milestone is missed or a deliverable is degraded |
| 5 | Almost certain | Severe: project goal at risk |

| Score | Level | Response |
|---|---|---|
| 15 – 25 | 🔴 High | Mitigation starts now; reviewed every week |
| 8 – 14 | 🟠 Medium | Mitigation planned and owned; reviewed every 2 weeks |
| 1 – 7 | 🟢 Low | Monitor |

---

## 2. Risk Register

Sorted by score. **L** = likelihood, **I** = impact.

| ID | Risk | Category | L | I | Score | Level | Mitigation (prevent / reduce) | Contingency (if it happens) | Owner |
|---|---|---|---|---|---|---|---|---|---|
| R01 | Azure costs exceed available credits (Spark or warehouse compute left running) | Cost | 4 | 4 | 16 | 🔴 | Budget alerts at 50 / 80 / 100%; Spark auto-pause ≈ 15 min; pause the warehouse pool when idle; smallest sizes; end-of-day shutdown checklist | Stop non-essential compute; use serverless SQL + CETAS as the serving layer; delete unused resources | M2 |
| R02 | Subscription limits, quota or region restrictions block a service (Spark pool, dedicated SQL pool) | Technical | 3 | 5 | 15 | 🔴 | Provision every resource in week 1 as a smoke test; verify quotas; choose a region with availability | Switch the warehouse to Azure SQL Database or serverless SQL external tables; run Spark locally against exported files | M2 |
| R08 | Credentials (account keys, SAS tokens, connection strings) committed to Git, especially if the repo is public | Security | 3 | 5 | 15 | 🔴 | `.gitignore`; Azure AD / managed identity instead of keys; short-lived SAS; secret scanning (e.g. gitleaks) as a pre-commit hook; PR checklist | Rotate the exposed secret immediately; remove it from Git history; review access logs | M2 |
| R04 | Skill gaps in Synapse, PySpark, or dimensional modeling | People | 4 | 3 | 12 | 🟠 | Week-1 learning tasks (Microsoft Learn modules); buddy system; pair on first notebook / first table | Reassign tasks to stronger members; reduce scope of non-critical features | M1 |
| R05 | A team member becomes unavailable (exams, illness, other commitments) | People | 3 | 4 | 12 | 🟠 | Backup named for every role; small PRs; documented work; shared board | Backup takes over; drop or defer optional scope | M1 |
| R06 | Scope creep (extra datasets, Delta Lake, dashboards) | Scope | 4 | 3 | 12 | 🟠 | MVP defined in the project plan; change control through `scope-change` issues; Delta only after M5 | Freeze scope; move new items to a "later" list | M1 |
| R09 | Source data quality problems (nulls, duplicates, schema drift, bad dates) break the pipeline | Data | 4 | 3 | 12 | 🟠 | Profile data in phase 2; explicit schemas in Spark; quarantine bad records instead of failing the whole run; DQ checks at each zone | Fix at the transformation layer; regenerate data if it was synthetic | M6 |
| R10 | Permission / connectivity failures between Synapse, Spark, and ADLS (missing role assignments) | Technical | 4 | 3 | 12 | 🟠 | Assign storage roles early (T02, T03, T11); connectivity smoke test; keep a role-assignment table | Escalate to M2 same day; use a temporary fix documented in an issue and revert it | M2 |
| R11 | Dimensional modeling mistakes (wrong grain, orphan keys, duplicate dimension rows) | Technical | 3 | 4 | 12 | 🟠 | Written grain statement; team review of the ERD before loading; surrogate keys; referential-integrity DQ tests | Rebuild affected tables from curated data (the pipeline is re-runnable) | M5 |
| R14 | Documentation is incomplete or work isn't reproducible near the deadline | Delivery | 3 | 4 | 12 | 🟠 | Documentation is part of Definition of Done; fresh-clone test of the README in week 7; docs freeze 2 days before submission | Dedicate the last week to documentation; cut optional work | M1 |
| R15 | Demo fails (paused pools, expired access, live query errors) | Delivery | 3 | 4 | 12 | 🟠 | Dry run 48 h before; resume compute ahead of time; screenshots saved after every milestone | Present from the recorded demo and screenshots | M1 |
| R03 | No suitable retail dataset, or licensing issues | Data | 3 | 3 | 9 | 🟠 | Decide dataset (public or synthetic) by end of week 1; agree on a data dictionary | Generate synthetic data with Faker following the agreed schema | M3 |
| R16 | Unclear ownership or poor communication | People | 3 | 3 | 9 | 🟠 | RACI matrix; weekly cadence; board with one owner per task | M1 re-assigns and clarifies in the next sync | M1 |
| R07 | Notebook merge conflicts and noisy diffs | Process | 4 | 2 | 8 | 🟠 | Branch per task; one owner per notebook; clear outputs before commit; small PRs | Resolve in a pairing session; re-run notebooks from clean | M1 |
| R12 | Slow queries or Spark jobs (small-files problem, cold starts) | Performance | 3 | 2 | 6 | 🟢 | Parquet; partition by date; avoid many tiny files; select only needed columns | Repartition / coalesce; materialise heavy joins | M4 |
| R13 | Delta Lake or Z-ordering not supported by the available Spark runtime | Technical | 3 | 2 | 6 | 🟢 | Check the Delta version of the Spark pool early; keep it optional | Use partitioning and file compaction instead; drop Z-ordering | M4 |

---

## 3. Top Risks to Watch

1. **R01 Cost.** The most likely cause of a stalled project. Check spend every Friday and run the shutdown checklist.
2. **R02 Platform limits.** Test every Azure service in week 1. Finding a quota problem in week 6 would be expensive.
3. **R08 Secrets in Git.** Easy to cause, hard to undo. Enforce the PR checklist and a secret scanner from day one.

---

## 4. Risk Heat Map

|  | Impact 1 | Impact 2 | Impact 3 | Impact 4 | Impact 5 |
|---|---|---|---|---|---|
| **Likelihood 5** | | | | | |
| **Likelihood 4** | | R07 | R04, R06, R09, R10 | R01 | |
| **Likelihood 3** | | R12, R13 | R03, R16 | R05, R11, R14, R15 | R02, R08 |
| **Likelihood 2** | | | | | |
| **Likelihood 1** | | | | | |

---

## 5. Review Process

- **Weekly (Monday):** 10 minutes. Re-score, update owner notes, close mitigated risks, add new ones.
- **Triggers for immediate review:** a missed milestone, spend above 80% of budget, a member unavailable for > 3 days, or a failed pipeline run lasting > 1 day.
- **Where to log:** this file for the register; GitHub issues labelled `risk` for actions that need tracking.

### Risk change log

| Date | Risk | Change | By |
|---|---|---|---|
| 2026-10-10 | All | Initial register created | M1 |
