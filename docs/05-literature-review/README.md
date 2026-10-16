# Literature Review — Cloud Retail Data Lake & Data Warehouse

> **Status:** v1 · **Owner:** _[MAGED GAMAL AHMED]_ · **Last updated:** 2026-10-16 · **Sources accessed:** 2026-10-16

---

## 1. Purpose, Scope & Method

**Purpose.** Place our project (an Azure data lake → processing → warehouse platform for retail data) in the context of existing solutions, learn from their strengths and weaknesses, and show how our design differs.

**Our project in one paragraph.** Raw CSV / JSON / Parquet retail files are stored in Azure Data Lake Storage Gen2 (ADLS) organised into raw / processed / curated zones, explored with Synapse serverless SQL, transformed with Synapse Spark (PySpark), modelled as a star schema (Fact Sales + Dim Product, Customer, Store, Date), and loaded into a warehouse, with data-quality checks, access control, and pipeline monitoring. Delta Lake with partitioning / Z-ordering is an optional extension.

**How sources were found.**

| Step | What was done |
|---|---|
| Search venues | Microsoft Learn (Architecture Center, Cloud Adoption Framework, Synapse and Fabric docs), GitHub, VLDB / CIDR proceedings, arXiv and Springer, publisher pages |
| Example queries | "Azure Synapse end-to-end analytics architecture", "data lake zones and containers", "Delta Lake VLDB 2020", "Lakehouse CIDR 2021", "Azure data engineering project retail ADLS Synapse star schema github", "Synapse runtime Spark 3.5 Delta", "Fabric vs Synapse" |
| Inclusion criteria | Relevant to Azure lakes / Synapse / dimensional modelling / lakehouse design; publicly accessible; identifiable author or organisation; sufficiently documented to assess |
| Exclusion | Paywalled content we could not read, undated or anonymous posts, tutorials with no architecture detail |

**Source tiers** (used to weigh evidence)

| Tier | Meaning | Examples |
|---|---|---|
| **1** | Official vendor documentation, peer-reviewed papers, standard textbooks | [M1]–[M9], [A1]–[A3], [B1] |
| **2** | Open-source project repositories (self-reported, not peer-reviewed) | [P1]–[P3] |
| **3** | Analyst or consultancy commentary | [C1], [C2] |

---

## 2. Background Concepts

**Data lakes and zones.** A data lake stores large volumes of data in any format; its main risks are weak architecture and weak metadata, which can turn it into an unusable "data swamp" [A3]. Microsoft's cloud-scale analytics guidance recommends organising the lake into layers: a raw layer that is kept immutable, an enriched (standardised) layer, and a curated layer that may hold star schemas or data marts. It also notes that a small deployment can consolidate the layers into one storage account [M3].

**Medallion naming.** The same idea is commonly named bronze (raw), silver (validated / enriched), and gold (curated). Microsoft's Fabric documentation describes it this way and recommends Delta tables for silver and gold [M8]. We map our zones as **raw = bronze, processed = silver, curated = gold**.

**Lakehouse and Delta Lake.** The lakehouse argument is that open formats such as Parquet on low-cost storage can support warehouse-style workloads and avoid the staleness, cost, and lock-in of two-tier lake-plus-warehouse systems [A2]. Delta Lake adds ACID transactions, time travel, and faster metadata operations to files in cloud object storage through a transaction log stored as Parquet [A1]. This is the basis of our optional extension.

**Dimensional modelling.** Kimball and Ross's star-schema approach structures analytical data as fact tables surrounded by descriptive dimensions, with retail sales as one of its standard case studies [B1]. It is the model behind our Fact Sales and four dimensions.

**Azure Synapse.** Microsoft's reference architecture uses ADLS as the home of data across all stages, Synapse pipelines to land data in the raw layer, serverless SQL and Spark pools for processing, and a SQL pool or the lake for serving [M1].

---

## 3. Similar Solutions Reviewed

### 3.1 Microsoft reference architectures — [M1], [M2], [M4]

**What they are.** Official Azure Architecture Center scenarios, plus a deployment accelerator on GitHub.

- **[M1] Analytics end-to-end with Azure Synapse** shows ingest → store → process → serve → visualise. ADLS is the central store, pipelines copy relational and semi-structured sources into a raw layer, and both serverless SQL and Spark pools are available for processing. It gives guidance on lake layers, folder structure, and file formats.
- **[M2] Data warehouse scenario** (Architecture Center) takes a different shape: source exports land in a staging area in ADLS, Data Factory loads them incrementally into Synapse staging tables, then transforms them into a common structure. The lake is used mainly as a staging area, and the pipeline authenticates to Synapse with a service principal or managed identity.
- **[M4] Azure Synapse end-to-end accelerator** deploys a reference set of resources: a Synapse workspace, a small three-node Spark pool, a DW100 dedicated SQL pool, a Data Explorer pool, a standard LRS storage account, and a Purview account. Several of these are flagged as cost-incurring.

| Strengths | Weaknesses / limitations |
|---|---|
| Authoritative, maintained, and consistent with Microsoft's best practices | Generic enterprise scenarios, not a worked retail dimensional model |
| [M1] covers the whole lifecycle and both SQL engines | [M2] treats the lake as staging rather than as an analytical layer |
| [M4] shows realistic sizing and which resources cost money | [M4] provisions more services (Data Explorer, Purview) than a small student project needs, so cost control matters |
| Managed-identity authentication is shown ([M2]) | Pages describe architecture, not measurable success criteria |

**Relevance.** We follow the [M1] flow and the zone guidance in [M3], but we build the retail model and the quality / KPI layer that the reference material leaves open. We reviewed [M4] from its README only.

### 3.2 Sales Analytics Dashboard – Azure — [P1]

**What it is.** A public GitHub project using Synapse, ADLS Gen2, Azure Data Factory, T-SQL / PySpark, and Power BI. It follows the bronze → silver → gold pattern. The gold layer is described as a **snowflake schema with one fact table and six dimension tables**, created with Synapse SQL scripts. Entities are e-commerce style (orders, sellers, reviews, geolocation).

| Strengths | Weaknesses / limitations |
|---|---|
| Clear medallion layering with a plain description of each layer | Snowflake design with six dimensions means more joins for analysts |
| Complete path from lake to a Power BI dashboard | The README does not list a **date dimension** |
| Repository includes data, source code, a report file, and the dashboard file | The README does not describe data-quality checks, access control, monitoring, or measurable results |
| Uses Synapse SQL scripts for modelling, similar to our approach | No description or topics on the repository, and no sign of external use or review |

**Relevance.** Confirms that our stack is a common pattern. It also shows two things to avoid: a missing date dimension and undocumented quality and security.

### 3.3 DataFusion — [P2]

**What it is.** An end-to-end pipeline using the AdventureWorksLT sample database on a local SQL Server, Azure Data Factory (with a self-hosted integration runtime), ADLS Gen2, **Azure Databricks** for Spark transformations, **Delta** tables in the silver and gold layers, Synapse **serverless SQL** over gold with views and stored procedures, and Power BI with DirectQuery.

| Strengths | Weaknesses / limitations |
|---|---|
| Most complete treatment of **security and governance**: Azure AD role-based access, least privilege, Key Vault for secrets, encryption | Depends on several separately billed services (Databricks, Data Factory, Synapse, Power BI), which raises cost and complexity for a small team |
| Uses Delta for silver and gold, giving versioning and ACID behaviour | Source is a small sample database, yet the README describes handling "large datasets" without evidence |
| Includes validation rules in transformations and Azure Monitor for pipeline tracking | Quality, performance, and testing claims are stated without numbers (no row counts, runtimes, or test results) |
| Serverless SQL over the lake keeps the serving layer lightweight | The README says data is in "fact and dimension tables" but does not specify the model |

**Relevance.** The closest match to our lake-first approach, and the best model for the security section of our README. We replace Databricks and Data Factory with Synapse Spark and Synapse pipelines to keep everything in one workspace, and we add measurable evidence (KPIs).

### 3.4 SuperStore Sales Data Engineering — [P3]

**What it is.** A pipeline that extracts sales data from PostgreSQL, cleans it and builds a **star schema** in Python, uploads the result to **Azure Blob Storage**, loads it into a **Synapse dedicated SQL pool**, and builds Power BI dashboards. Apache Airflow, run in a local WSL environment, orchestrates the process. The dataset has 9,993 transactions from 2019 to 2022.

| Strengths | Weaknesses / limitations |
|---|---|
| Simple, easy to follow, with an ERD and screenshots of the pipeline run and role assignment | Blob Storage is used only as a hand-off, with **no lake zones and no Spark processing** |
| Star schema with a dedicated warehouse | Small, already-clean dataset (the README says no missing values), so little data-quality work is shown |
| Orchestration is automated and visible | Orchestration runs from a local environment rather than a managed cloud service |
| Clear technology list | No monitoring, quality checks, or measurable targets described |

**Relevance.** Shows a minimal warehouse-first pipeline. It highlights what a zoned lake and Spark processing add (handling raw, multi-format, messy data), which is the core of our proposal.

### 3.5 The lakehouse and Fabric direction — [A1], [A2], [M8], [M9], [M5], [C1], [C2]

**What it is.** Not a competing project but the direction in which platforms are moving. Lakehouse research argues for open formats and a single platform for analytics [A2], Delta Lake supplies the transactional layer [A1], and Microsoft documents a medallion pattern in Fabric in which silver and gold use Delta tables [M8]. One documented option there keeps bronze and silver in lakehouses and makes gold a **data warehouse**, which is the same shape as our design [M8].

**Platform status relevant to us.**
- Microsoft documents migration paths from Synapse dedicated SQL pools and Synapse Spark to Fabric [M9].
- The Synapse Spark 3.5 runtime (Spark 3.5, Delta Lake 3.2) enters long-term support on 1 Nov 2027, running to 31 Oct 2028, and Microsoft encourages moving to Fabric Runtime 2.0 [M5].
- The Synapse Spark 3.4 runtime reached end of support on 31 Mar 2026 [M6], so new pools should use 3.5.
- Independent analysts reported in 2025 that Synapse remained supported for now but that customers should not expect to stay on it indefinitely [C1], and that moving workloads to Fabric requires manual changes to notebooks, SQL scripts, and pipelines [C2].

| Strengths | Weaknesses / limitations |
|---|---|
| Open formats and ACID tables reduce lock-in and staleness [A1][A2] | The lakehouse papers come from the Databricks authors and argue a position |
| Fabric simplifies the number of services to manage [C2] | Synapse (the platform in our proposal) is on a longer-term path toward Fabric [M5][C1] |
| Official migration guidance exists [M9] | Migration is not automatic for everything [C2] |

**Relevance.** Our proposal specifies Synapse, so we stay on it. We keep the logic portable (PySpark, SQL, Parquet / Delta) and note Fabric as future work.

---

## 4. Comparison With Our Project


| Criterion | **Our project** | Microsoft reference [M1][M4] | Sales Analytics Dashboard [P1] | DataFusion [P2] | SuperStore [P3] |
|---|---|---|---|---|---|
| Source data | Files: CSV / JSON / Parquet | Relational and semi-structured via pipelines | Multiple sources (unspecified) | SQL Server sample DB | PostgreSQL (CSV-derived) |
| Lake storage | ADLS Gen2 | ADLS Gen2 | ADLS Gen2 | ADLS Gen2 | Blob Storage (hand-off only) |
| Zones | raw / processed / curated | Layered raw zone and others | Bronze / silver / gold | Bronze / silver / gold | None |
| Processing | Synapse Spark (PySpark) | Spark pools and pipelines | T-SQL / PySpark in Synapse | Databricks (Spark) | Python ETL + Airflow |
| Serverless SQL on lake | **Yes** | Yes | Not documented | **Yes** | No |
| Warehouse / serving | Synapse warehouse, star schema | SQL pool or lake | Gold layer via Synapse SQL | Serverless over Delta gold | Dedicated SQL pool |
| Model | Star: Fact Sales + 4 dims **incl. Dim Date** | Not applicable | Snowflake: 1 fact + 6 dims, no date dim listed | Fact and dims (unspecified) | Star (ERD shown) |
| File format | Parquet (+ optional Delta) | Guidance on formats | Not documented | Delta (silver / gold) | Not documented |
| Orchestration | Synapse pipelines | Synapse pipelines | Data Factory | Data Factory | Airflow (local WSL) |
| Data quality | Automated checks, tracked as KPIs | Guidance only | "Cleaned, validated" silver layer | Validation rules in notebooks | Cleaning and dedup in ETL |
| Security | RBAC / ACLs, least privilege, no secrets in Git | Not assessed | Not documented | **RBAC, Key Vault, encryption** | Role assignment shown |
| Monitoring | Synapse Monitor + KPI scorecard | Not assessed | Not documented | Azure Monitor | DAG run screenshots |
| BI layer | Sample SQL queries | Power BI | Power BI | Power BI | Power BI |
| Measurable targets (KPIs) | **Yes** | Not applicable | Not documented | Not documented | Not documented |
| Project management artefacts (plan, RACI, risks) | **Yes** | Not applicable | Not documented | Not documented | Not documented |

---

## 5. Synthesis

**What works across the projects**
- Layered storage (bronze / silver / gold) and a star-like model are the common, accepted pattern [M3][M8][P1][P2].
- Keeping everything inside one Azure ecosystem simplifies access control and monitoring [M1][P2].
- Open formats (Parquet / Delta) are the dominant storage choice for processed data [A1][A2][M8][P2].

**Recurring weaknesses**
1. **Quality, security, and monitoring are thin or undocumented**, apart from [P2].
2. **Cost and complexity** grow when many paid services are stacked [P2][M4].
3. **Modelling gaps**: a missing date dimension and an over-normalised snowflake in [P1]; an unspecified model in [P2].
4. **Weak lake usage**: [P3] and [M2] treat storage as a pass-through rather than an analytical layer.
5. **Platform drift**: Spark runtimes and the Synapse-to-Fabric direction change quickly, and some third-party material is already out of date [M5][M6].

---

## 6. How Our Project Differs

1. **Lake-first, multi-format ingestion.** We start from CSV / JSON / Parquet files, not a single relational database as in [P2] and [P3].
2. **Explicit zone design** with an immutable raw zone, following Microsoft's guidance [M3].
3. **Both lake and warehouse access.** Serverless SQL over the lake *and* a warehouse serving layer; [P2] has only the former and [P3] only the latter.
4. **A complete star schema with a date dimension**, designed from a written grain statement, rather than the snowflake in [P1] [B1].
5. **Measured quality.** Automated data-quality checks and a KPI scorecard (see `04-kpis.md`), where the reviewed projects give only statements.
6. **Engineering discipline.** Documented plan, RACI, risk register, and cost controls (see `01`–`03`).
7. **Optional Delta Lake / Z-ordering extension.** Not unique ([P2] uses Delta), but included as a stretch goal [A1].

**Honest limits of the difference.** The architecture is conventional, so our contribution is mostly in rigour, evidence, and documentation, not novelty. We also deliberately leave out a Power BI dashboard, which all three projects include. The team can add a small dashboard if time allows.

---

## 7. Insights That Will Be Applied to Our Design

| # | Insight | Source | Decision for our project |
|---|---|---|---|
| 1 | Synapse Spark 3.4 is out of support; 3.5 is current (Delta Lake 3.2) | [M5][M6][M7] | Create the Spark pool on **runtime 3.5**; confirm in our pool that OPTIMIZE / Z-ORDER commands run before promising them |
| 2 | Raw data should be immutable; layers can share one account at small scale | [M3] | Keep `raw/` write-once; one storage account with a container per zone |
| 3 | The curated layer may hold star schemas | [M3] | `curated/` stores model-ready Parquet for Fact Sales and the four dimensions |
| 4 | Bronze / silver / gold is the common naming | [M8] | Document the mapping to raw / processed / curated in the README |
| 5 | A documented pattern keeps lake layers and makes gold a warehouse | [M8] | Cite this as the justification for our lake + warehouse design |
| 6 | Open formats; Delta adds ACID and time travel | [A1][A2] | Parquet by default; Delta only in the stretch task (T26) |
| 7 | Weak metadata turns lakes into swamps | [A3] | Data dictionary (T06), naming conventions (T07), and a README section per dataset; Purview is optional because it adds cost [M4] |
| 8 | Dimensional modelling guidance and the retail example | [B1] | Write the grain statement first; include **Dim Date**; keep a star, not a snowflake |
| 9 | Least privilege, managed identity, Key Vault | [P2][M2] | RBAC / ACLs (T20); no secrets in Git (R08) |
| 10 | Compute services cost money | [M4] | Auto-pause Spark; pause the warehouse when idle (R01) |
| 11 | Pipelines can run inside the Synapse workspace | [M1] | Use Synapse pipelines instead of an external orchestrator ([P3]) or a separate Data Factory ([P1][P2]) |
| 12 | Synapse → Fabric is the platform trajectory | [M5][M9][C1] | Keep logic portable (PySpark, SQL, Parquet / Delta); list Fabric migration under future work |
| 13 | Reviewed projects make claims without numbers | [P2] | Record evidence in the KPI scorecard (row counts, runtimes, pass rates, spend) |

---

## 8. References

*Format: numbered, IEEE-style. All URLs accessed 2026-10-16. Microsoft Learn URLs are shown without a language code, so they open in your own locale.*

### Official documentation (Tier 1)

- **[M1]** Microsoft, "Analytics end-to-end with Azure Synapse," *Azure Architecture Center*, Microsoft Learn. https://learn.microsoft.com/azure/architecture/example-scenario/dataplate2e/data-platform-end-to-end
- **[M2]** Microsoft, Azure Architecture Center example scenario "data warehouse" (page path `/example-scenario/data/data-warehouse`), Microsoft Learn. https://learn.microsoft.com/en-us/azure/architecture/example-scenario/data/data-warehouse
- **[M3]** Microsoft, "Data lake zones and containers," *Cloud Adoption Framework: cloud-scale analytics best practices*, Microsoft Learn. https://learn.microsoft.com/azure/cloud-adoption-framework/scenarios/cloud-scale-analytics/best-practices/data-lake-zones
- **[M4]** Microsoft Azure, "Azure Analytics End to End with Azure Synapse – Deployment Accelerator," GitHub repository. https://github.com/Azure/azure-synapse-analytics-end2end
- **[M5]** Microsoft, "Azure Synapse Runtime for Apache Spark 3.5," Microsoft Learn. https://learn.microsoft.com/azure/synapse-analytics/spark/apache-spark-35-runtime
- **[M6]** Microsoft, "Azure Synapse Runtime for Apache Spark 3.4," Microsoft Learn (end-of-support notice). https://learn.microsoft.com/azure/synapse-analytics/spark/apache-spark-34-runtime
- **[M7]** Microsoft, "Synapse runtime for Apache Spark lifecycle and supportability," Microsoft Learn. https://learn.microsoft.com/azure/synapse-analytics/spark/runtime-for-apache-spark-lifecycle-and-supportability
- **[M8]** Microsoft, "Understand medallion lakehouse architecture for Microsoft Fabric with OneLake," Microsoft Learn. https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture
- **[M9]** Microsoft, Microsoft Fabric migration overview ("Migrate from legacy, on-premises, or PaaS platforms"), Microsoft Learn. https://learn.microsoft.com/fabric/fundamentals/migration

### Peer-reviewed papers and books (Tier 1)

- **[A1]** M. Armbrust *et al.*, "Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores," *Proc. VLDB Endowment*, vol. 13, no. 12, pp. 3411–3424, 2020. https://vldb.org/pvldb/vol13/p3411-armbrust.pdf
- **[A2]** M. Armbrust, A. Ghodsi, R. Xin, and M. Zaharia, "Lakehouse: A New Generation of Open Platforms that Unify Data Warehousing and Advanced Analytics," in *Proc. 11th Conf. on Innovative Data Systems Research (CIDR '21)*, Jan. 11–15, 2021. https://www.cidrdb.org/cidr2021/papers/cidr2021_paper17.pdf
- **[A3]** P. Sawadogo and J. Darmont, "On data lake architectures and metadata management," *Journal of Intelligent Information Systems*, vol. 56, no. 1, pp. 97–120, 2021. DOI: https://doi.org/10.1007/s10844-020-00608-7 (preprint: https://arxiv.org/abs/2107.11152)
- **[B1]** R. Kimball and M. Ross, *The Data Warehouse Toolkit: The Definitive Guide to Dimensional Modeling*, 3rd ed. Indianapolis, IN, USA: John Wiley & Sons, 2013. ISBN 978-1-118-53080-1.

### Open-source project repositories (Tier 2)

- **[P1]** Dayyak, "Sales-Analytics-Dashboard-Azure," GitHub repository. https://github.com/Dayyak/Sales-Analytics-Dashboard-Azure *(a search result listed this repository as "Ecom-Data-Platform"; GitHub redirects to the current name)*
- **[P2]** mehtavandit, "DataFusion," GitHub repository. https://github.com/mehtavandit/DataFusion
- **[P3]** ShawonSimon, "SuperStore-Sales-Data-Engineering," GitHub repository. https://github.com/ShawonSimon/SuperStore-Sales-Data-Engineering

### Analyst and consultancy commentary (Tier 3)

- **[C1]** Directions on Microsoft, "Synapse Analytics Roadmap," Oct. 16, 2025. https://www.directionsonmicrosoft.com/roadmaps/ref/synapse-analytics-roadmap/ *(partly behind a membership wall; we read only the visible summary)*
- **[C2]** B. Smart, "Azure Synapse Analytics vs Microsoft Fabric: side-by-side comparison," *endjin*. https://endjin.com/blog/azure-synapse-analytics-versus-microsoft-fabric-a-side-by-side-comparison *(publication date not confirmed)*

