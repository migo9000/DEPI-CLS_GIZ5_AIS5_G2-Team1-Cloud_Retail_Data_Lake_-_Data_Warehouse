# Task Assignment & Roles

> **Status:** Draft v1 · **Owner:** M1 (Project Lead) · **Last updated:** 2026-10-10
> Replace M1–M6 with real names. Roles are balanced for a 6-person team; swap them to fit people's strengths.

---

## 1. Team Roster

| Label | Name | GitHub handle | Role | Backup (buddy) |
|---|---|---|---|---|
| M1 | _[name]_ | _@handle_ | Project Lead & Documentation | M6 |
| M2 | _[name]_ | _@handle_ | Cloud Infrastructure, Security & Pipeline Ops | M3 |
| M3 | _[name]_ | _@handle_ | Ingestion & Data Lake Engineer | M2 |
| M4 | _[name]_ | _@handle_ | Transformation Engineer (PySpark) | M5 |
| M5 | _[name]_ | _@handle_ | Warehouse & Dimensional Modeling Engineer | M4 |
| M6 | _[name]_ | _@handle_ | Data Quality & Analytics Engineer | M1 |

Each person has a **backup** who understands their work well enough to continue it if they're unavailable. Backups review the owner's pull requests.

---

## 2. Role Descriptions

### M1 — Project Lead & Documentation
- **Mission:** keep the project on schedule, scope, and documented.
- **Responsibilities:** run the meeting cadence, maintain the board / Gantt / risk register / KPI scorecard, approve scope changes, unblock teammates, own the final demo.
- **Owns:** Technical README, pipeline screenshots, KPI scorecard, documentation consolidation.

### M2 — Cloud Infrastructure, Security & Pipeline Ops
- **Mission:** a secure, cost-controlled Azure environment that everyone can use.
- **Responsibilities:** provision resource group, ADLS, Synapse workspace and pools; RBAC / ACLs and least privilege; budget alerts and auto-pause; secrets handling; pipeline orchestration and monitoring.
- **Owns:** access-control setup, pipeline triggers and monitoring, role-assignment documentation.

### M3 — Ingestion & Data Lake Engineer
- **Mission:** get data into the lake in an organised, repeatable way.
- **Responsibilities:** select or generate retail datasets and the data dictionary; ADLS folder structure and naming; raw uploads; Parquet conversion and partitioning strategy; architecture diagram.
- **Owns:** ADLS folder structure, architecture diagram (v1 and final), ingestion documentation.

### M4 — Transformation Engineer (PySpark)
- **Mission:** turn raw files into clean, typed, analysis-ready datasets.
- **Responsibilities:** PySpark notebooks for raw → processed → curated; explicit schemas; handling nulls, duplicates, bad dates; conformed dimensions with surrogate keys; optional Delta Lake extension.
- **Owns:** PySpark notebooks.

### M5 — Warehouse & Dimensional Modeling Engineer
- **Mission:** a correct, documented star schema that answers business questions.
- **Responsibilities:** grain statement, ERD, keys; external tables and views; warehouse DDL and load scripts for Fact Sales, Dim Product, Dim Customer, Dim Store, Dim Date.
- **Owns:** dimensional model, Synapse SQL scripts.

### M6 — Data Quality & Analytics Engineer
- **Mission:** make sure the data can be trusted and the platform proves its value.
- **Responsibilities:** serverless SQL exploration; profiling; automated DQ checks (row counts, nulls, duplicates, referential integrity); sample analytical queries; supports KPI measurement.
- **Owns:** data-quality validation, sample analytical queries.

---

## 3. RACI Matrix

**R** = does the work · **A** = accountable / signs off · **C** = consulted · **I** = informed

| Activity | M1 | M2 | M3 | M4 | M5 | M6 |
|---|---|---|---|---|---|---|
| Architecture design & diagram | A | C | R | C | C | I |
| Azure provisioning, RBAC, cost control | A | R | C | C | I | I |
| Source data & ingestion to raw zone | A | C | R | I | C | C |
| Serverless SQL exploration | A | I | C | I | C | R |
| PySpark cleaning & curated transformations | A | C | C | R | C | C |
| Parquet conversion & partitioning | A | I | R | C | I | I |
| Dimensional model design | A | I | C | C | R | C |
| External tables, views, warehouse load | A | C | C | C | R | I |
| Data-quality checks | A | I | C | C | C | R |
| Access control | A | R | I | I | C | I |
| Orchestration & monitoring | A | R | I | C | C | C |
| Sample analytical queries | A | I | I | I | C | R |
| KPI tracking | A / R | C | I | I | I | C |
| README, screenshots, demo | A / R | C | C | C | C | C |

---

## 4. Task Backlog

Estimates are in hours. Task IDs are used as GitHub issue titles and branch names (e.g. `feature/T12-clean-sales`).

### Phase 1 — Setup & Design (12 – 18 Oct)

| ID | Task | Owner | Support | Est. | Depends on | Status |
|---|---|---|---|---|---|---|
| T01 | Agree scope, roles, meeting cadence | M1 | All | 4 | — | ⬜ |
| T02 | Create resource group, ADLS Gen2 account, Synapse workspace | M2 | M3 | 6 | — | ⬜ |
| T03 | Configure RBAC for all members and budget alerts | M2 | M1 | 3 | T02 | ⬜ |
| T04 | Repo structure, branch protection, project board | M1 | M6 | 4 | — | ⬜ |
| T05 | Architecture diagram v1 and zone layout | M3 | M2, M5 | 6 | T01 | ⬜ |

### Phase 2 — Ingestion & Data Lake (19 Oct – 1 Nov)

| ID | Task | Owner | Support | Est. | Depends on | Status |
|---|---|---|---|---|---|---|
| T06 | Select / generate retail datasets (sales, products, customers, stores) in CSV / JSON / Parquet + data dictionary | M3 | M5 | 10 | T05 | ⬜ |
| T07 | Create `raw` / `processed` / `curated` structure and naming conventions | M3 | M2 | 4 | T02 | ⬜ |
| T08 | Upload raw data and document the steps | M3 | M6 | 8 | T06, T07 | ⬜ |
| T09 | Serverless SQL exploration scripts over raw files | M6 | M3 | 8 | T08 | ⬜ |
| T10 | Data-profiling notebook (nulls, duplicates, types, ranges) | M6 | M4 | 8 | T08 | ⬜ |

### Phase 3 — Transformation (26 Oct – 15 Nov)

| ID | Task | Owner | Support | Est. | Depends on | Status |
|---|---|---|---|---|---|---|
| T11 | Spark pool setup and ADLS connectivity | M2 | M4 | 4 | T02, T03 | ⬜ |
| T12 | PySpark raw → processed cleaning notebooks | M4 | M6 | 24 | T10, T11 | ⬜ |
| T13 | Convert datasets to Parquet; define partitioning strategy | M3 | M4 | 10 | T12 | ⬜ |
| T14 | Processed → curated transformations (conformed dimensions, surrogate keys) | M4 | M5 | 20 | T12 | ⬜ |

### Phase 4 — Modeling & Warehouse (9 – 29 Nov)

| ID | Task | Owner | Support | Est. | Depends on | Status |
|---|---|---|---|---|---|---|
| T15 | Dimensional model design (grain, ERD, keys) | M5 | M4 | 10 | T10 | ⬜ |
| T16 | Create Synapse external tables and views | M5 | M3 | 12 | T13, T15 | ⬜ |
| T17 | Warehouse DDL and load scripts for Fact Sales + 4 dimensions | M5 | M4 | 20 | T14, T15 | ⬜ |
| T18 | Sample analytical queries (≥ 8) | M6 | M5 | 10 | T17 | ⬜ |

### Phase 5 — Quality, Security, Monitoring (16 – 29 Nov)

| ID | Task | Owner | Support | Est. | Depends on | Status |
|---|---|---|---|---|---|---|
| T19 | Data-quality checks (row counts, nulls, uniqueness, referential integrity) | M6 | M4, M5 | 16 | T14, T17 | ⬜ |
| T20 | Access controls: RBAC / ACLs, least privilege, secrets handling | M2 | M1 | 8 | T03 | ⬜ |
| T21 | Pipeline orchestration (triggers) and monitoring | M2 | M6 | 14 | T14, T17 | ⬜ |
| T22 | KPI scorecard and weekly measurement routine | M1 | M6 | 6 | T21 | ⬜ |

### Phase 6 — Documentation & Delivery (23 Nov – 6 Dec)

| ID | Task | Owner | Support | Est. | Depends on | Status |
|---|---|---|---|---|---|---|
| T23 | Final architecture diagram | M3 | M2 | 4 | T21 | ⬜ |
| T24 | Technical README and docs consolidation | M1 | All | 12 | T19 – T21 | ⬜ |
| T25 | Pipeline screenshots, demo script, rehearsal | M1 | All | 8 | T21 | ⬜ |

### Optional stretch

| ID | Task | Owner | Support | Est. | Depends on | Status |
|---|---|---|---|---|---|---|
| T26 | Delta Lake conversion; partitioning / Z-ordering on selected datasets | M4 | M5 | 16 | M5 green | ⬜ |

**Total planned effort:** ~239 h (+16 h optional).

---

## 5. Working Agreements

### Git workflow
- `main` is protected: changes only through pull requests, **at least 1 approval** (the owner's backup where possible).
- Branch names: `feature/<task-id>-<short-name>`, e.g. `feature/T17-load-fact-sales`.
- Small PRs (one task or less), reviewed within 24 h.
- **Never commit secrets** (account keys, SAS tokens, connection strings). Use environment variables or Key Vault; keep a `.gitignore` for credentials and local data.

### Notebooks
- Name: `NN_<zone>_<dataset>.ipynb`, e.g. `02_processed_sales.ipynb`.
- Clear outputs before committing (or use `nbstripout`) to keep diffs readable.
- One owner per notebook; others propose changes through PRs.
- Every notebook states its inputs, outputs, and run order in the first cell.

### Suggested repo layout
```
.
├── README.md
├── docs/                 # these four documents + architecture diagram
├── notebooks/            # PySpark / profiling notebooks
├── sql/                  # serverless SQL, external tables, warehouse DDL, sample queries
├── data_quality/         # DQ checks and results
├── sample_data/          # small samples only (large data stays in ADLS)
└── screenshots/
```

### Definition of Done (every task)
- [ ] Code / SQL committed through a reviewed PR
- [ ] Runs from a clean state without manual fixes
- [ ] Inputs, outputs and assumptions documented (README section or notebook header)
- [ ] No secrets or large data files in Git
- [ ] Board updated and, if relevant, a screenshot saved

### Escalation
1. Blocked > 24 h: post in the team channel and tag the buddy and M1.
2. Unresolved after 48 h: M1 reassigns or adjusts scope, and logs it in the risk register.
3. Disagreement on design: timebox 20 min in the next sync, then M1 decides and records it in `docs/decisions.md`.
