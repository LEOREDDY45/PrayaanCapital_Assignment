# Prayaan Capital Data Engineering Assignment


A production-ready data pipeline engineered to process financial data, manage data quality anomalies, model complex risk metrics, and serve downstream analytics.

---

##  Technology Stack & Architecture


* **Cloud Platform**: Microsoft Azure
* **Compute & Orchestration**: Azure Databricks (PySpark / Spark SQL)
* **Version Control**: Git Integrated Notebooks
* **Storage Format**: Delta Lake (ACID transactions, time-travel ready)
* **BI Layer**: Power BI

---

##  Data Modeling (Star Schema)

To avoid performance bottlenecks from massive flat-table aggregations, the pipeline structures the **Silver-to-Gold** boundary using a clean Star Schema:

### 1. Fact Table (`fact_loans`)
Contains active loan specifications, risk metrics, and customer tracking points.
* `loan_id` (String) - Primary Key
* `borrower_id` (String)
* `branch_id` (String)
* `badge` (String)
* `urs_score` (Integer)
* `principal` (Integer)
* `outstanding_amount` (Double)

### 2. Repayments Table (`dim_repayments`)
Granular transactional log capturing historical and scheduled loan events.
* `repayment_id` (String) - Primary Key
* `loan_id` (String) - Foreign Key
* `installment_no` (Integer)
* `due_date` (Date)
* `paid_date` (Date)
* `amount_due` (Double)
* `amount_paid` (Double)
* `days_past_due` (Integer)
* `repayment_status` (String)

---

##  Pipeline Layers & Business Logic

### 🥉 Bronze: Ingestion
* Raw transactional data lands securely inside object storage.
* Maps raw files to operational schemas.

### 🥈 Silver: Data Quality & Structural Revamp
* **Anomaly Correction**: Filters negative values inside `amount_due` and removes logical anomalies like impossible future transaction dates.
* **De-duplication**: Drops duplicate transactions using explicit Spark partition window definitions.
* **Structural Optimization**: Transitions the pipeline away from massive single-table analytical queries into structured, aggregated dimension/fact sets to streamline memory use.

### 🥇 Gold: Business Intelligence & Aggregations

#### 1. Daily Portfolio Snapshot
Monitors the daily pulse of active capital. Uses targeted Common Table Expressions (CTEs) to isolate:
* Total active loan counts and absolute outstanding debt.
* Average internal credit rating (`urs_score`).
* Overdue rates, specifically flagging high-risk portfolio leakage where Days Past Due (DPD) exceeds 30 days.

#### 2. Risk Segmentation Matrix
Tracks performance groupings across time variables using the `badge` property as an anchor.
* Evaluates active volume concentrations per risk bracket.
* Derives real-time default proxy thresholds based on 60+ DPD trends.

#### 3. Early Warning Indicators (Branch Level)
Combines fact tables with transactional dimensions using multi-level windowing functions to detect localized portfolio stress.
* Calculates **Repayment Slippage Rates** dynamically across branches.
* Executes a **7-Day Rolling Average** comparison window to flag sharp variance swings in average DPD.
* Assigns risk profiles (`Low` / `Medium` / `High`) to prioritize local collection workflows.

---

## Engineering Challenges & Adaptations

* **Handling Relational Inconsistency**: Initial raw source data contained severe relational design gaps breaking logical dependencies. To prove out the business logic and pipeline scalability, a robust mock financial dataset was prompt-engineered via LLMs to emulate realistic transaction patterns without broken relations.
* **Refactoring Flat Joins to Star Schema**: Early iterations relied on expensive, massive multi-table CTE joins. Recognizing the data-modeling pitfall, the processing design was completely refactored in the Silver layer into optimized Fact and Dimension tables, drastically dropping query execution overhead.
* **Deciphering Rolling Financial Metrics**: Formulating rolling early-warning risk flags required utilizing advanced SQL windowing functions to generate moving rolling averages, balancing computation complexity within Spark.
