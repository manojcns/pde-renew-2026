# Professional Data Engineer Renewal: Comprehensive Study Guide

This detailed guide expands on the core exam topics with technical deep-dives, code examples, and architectural patterns. It focuses on the "how" and "why" to help you solve scenario-based questions.

---

## 🏗️ Section 1: Designing Data Processing Systems

### 1.1 Security & Compliance: The "Hidden" Requirements
The exam often hides security requirements in business constraints.
*   **VPC Service Controls (VPC-SC)**: Creates a security perimeter around your resources (like BigQuery or Storage buckets) to prevent data exfiltration.
    *   *Real-World Use Case*: Prevent a developer from copying data from a corporate BigQuery dataset to their personal Gmail-associated project.
*   **Organizational Policies**: High-level rules.
    *   *Example*: `constraints/gcp.resourceLocations` ensures buckets are *only* created in `europe-west1` (Data Sovereignty).

### 1.2 Data Processing Frameworks: The "Big Three" Comparison
This is a high-probability topic. You must distinguish between Dataform, Dataflow, and Data Fusion based on *persona* and *workload*.

#### **A. Dataform (SQL-Based ELT)**
*   **Persona**: Data Analysts, Data Engineers who love SQL.
*   **Concept**: You write SQL files (`.sqlx`). Dataform compiles them into a dependency graph and executes them in BigQuery. It brings software engineering (Git, CI/CD, Testing) to SQL.
*   **Code Example (`definitions/clean_users.sqlx`)**:
    ```sql
    config {
      type: "table",
      schema: "analytics",
      description: "Cleaned user data",
      assertions: {
        uniqueKey: ["user_id"],
        nonNull: ["email"]
      }
    }

    SELECT
      user_id,
      LOWER(email) as email,
      DATE(created_at) as signup_date
    FROM ${ref("raw_users")}
    WHERE user_id IS NOT NULL
    ```
*   *Exam Keyword*: "Data quality assertions", "Version control for SQL", "Transformations inside BigQuery".

#### **B. Dataflow (Apache Beam ETL)**
*   **Persona**: Data Engineers, Software Developers.
*   **Concept**: true "Pipeline" processing. Data moves through a series of stages. It uses **Apache Beam** (Java/Python/Go).
*   **Key Capability**: Unbeatable for **Streaming** and complex, per-row processing that SQL cannot do (e.g., calling an external API for every row, complex windowing).
*   **Code Example (Python Beam)**:
    ```python
    import apache_beam as beam

    with beam.Pipeline() as p:
      (p
       | 'ReadEvents' >> beam.io.ReadFromPubSub(topic=input_topic)
       | 'ParseJSON' >> beam.Map(json.loads)
       | 'FilterSpam' >> beam.Filter(lambda x: x['score'] > 0.5)
       | 'WriteToBQ' >> beam.io.WriteToBigQuery(table_spec)
      )
    ```
*   **Windowing Types**:
    1.  **Fixed**: Every 5 minutes (e.g., 12:00-12:05).
    2.  **Sliding**: Every 5 mins, starting every 1 min (overlapping).
    3.  **Session**: Based on user activity (e.g., session closes after 30 mins of inactivity).

#### **C. Cloud Data Fusion (Visual ETL)**
*   **Persona**: Citizen Integrators, ETL Developers (Informatica/Talend background).
*   **Concept**: Drag-and-drop GUI. Under the hood, it spins up a **Dataproc (Spark)** cluster to run the job.
*   **Wrangler**: A feature inside Data Fusion that lets you visually "clean" data (split columns, parse CSV) and see the results immediately on a sample.
*   *Exam Keyword*: "Visual interface", "Pre-built connectors (Salesforce, SAP, Oracle)", "No-code".

---

## 🔄 Section 2: Ingesting and Operationalizing Pipelines

### 2.1 Orchestration: Composer vs. Workflows
This is the "Control Plane" of your data platform.

#### **A. Cloud Composer (Apache Airflow)**
*   **Type**: Heavyweight, Managed Service.
*   **Best For**: Complex data pipelines with many dependencies, backfills, and cross-cloud tasks.
*   **Architecture**: Runs on a dedicated GKE cluster (can be expensive/slow to start).
*   **Code Example (DAG)**:
    ```python
    from airflow import DAG
    from airflow.operators.bash import BashOperator

    with DAG('daily_report', schedule_interval='@daily') as dag:
        t1 = BashOperator(task_id='pull_data', bash_command='python pull_data.py')
        t2 = BashOperator(task_id='process_data', bash_command='java -jar process.jar')
        t1 >> t2  # t2 runs only after t1 succeeds
    ```

#### **B. Cloud Workflows**
*   **Type**: Serverless, Lightweight.
*   **Best For**: Chaining APIs, Microservices, Event-driven steps (e.g., File lands in bucket -> Trigger Cloud Run).
*   **Cost**: Pay per execution step. Zero cost when idle.
*   **Code Example (YAML)**:
    ```yaml
    main:
      steps:
        - call_function:
            call: http.get
            args:
              url: https://us-central1-my-project.cloudfunctions.net/process_data
            result: api_response
        - return_result:
            return: ${api_response.body}
    ```

---

## 💾 Section 3: Storing the Data (Deep Dive)

### 3.1 BigQuery: Optimization is Key
The exam expects you to know how to make BigQuery fast and cheap.

*   **Partitioning**: Divides a table into segments (Physical separation).
    *   *Best For*: Filtering by `DATE`, `TIMESTAMP`, or `INTEGER` range.
    *   *Example*: `PARTITION BY DATE(transaction_date)`. Queries scanning only "2024-01-01" save huge money.
*   **Clustering**: Sorts data *inside* a partition.
    *   *Best For*: Filtering/Aggregating by high-cardinality columns (e.g., `user_id`, `product_id`).
    *   *Use Both*: Partition by Date, Cluster by Customer ID.

### 3.2 Cloud Bigtable: The "Firehose" Database
*   **Schema Design**:
    *   **Tall (Narrow)**: Optimized for time-series. One row per event.
        *   RowKey: `device_id#timestamp`.
    *   **Fat (Wide)**: Many columns. Bad for time-series updates.
*   **Row Key Best Practices**:
    *   **Anti-Pattern**: Using purely logical timestamps (e.g., `20240101...`). This causes **Hotspotting** (all writes hit one node).
    *   **Solution**: "Salting" or "Field Promotion". Use `device_id#reversed_timestamp` to spread writes and allow getting "latest" records first.

### 3.3 Cloud Spanner: The "Impossible" Database
*   **Key Feature**: Global Horizontal Scaling + ACID Transactions.
*   **Interleaved Tables**:
    *   *Concept*: Physically storing child rows next to parent rows.
    *   *Example*: `Singers` table and `Albums` table. If you Interleave `Albums` into `Singers`, Spanner stores Album A right next to Singer A on the disk.
    *   *Benefit*: Super fast joins for Hierarchical data.

### 3.4 Storage Decision Matrix (Refined)

| Requirement | Solution | Why? |
| :--- | :--- | :--- |
| **Global** Transactional Consistency (Banking) | **Spanner** | Strict consistency across regions. |
| **Regional** Transactional (Legacy App) | **Cloud SQL** | Standard MySQL/Postgres support. |
| **High Performance** Postgres (Analytics + Tx) | **AlloyDB** | "Polaris" architecture separates compute/storage. 4x faster Trans, 100x faster Analytical. |
| **Ad-Tech / IoT** (Millions of writes/sec) | **Bigtable** | Wide-column store designed for write throughput. |
| **Mobile App** (User State/Profiles) | **Firestore** | Document store, easy SDKs, offline sync. |
| **Data Warehouse** (BI Reporting) | **BigQuery** | SQL, Serverless, Columnar storage. |

---

## 🧠 Section 4: Analytics & AI

### 4.1 BigQuery ML
No need to export data to Python/Spark.
*   **Supported Models**:
    *   `LINEAR_REG`: Forecasting sales.
    *   `KMEANS`: Customer segmentation (clustering).
    *   `LOGISTIC_REG`: Classification (Spam/Not Spam).
*   *Snippet*:
    ```sql
    CREATE OR REPLACE MODEL `mydataset.churn_model`
    OPTIONS(model_type='LOGISTIC_REG') AS
    SELECT
      customer_churned_label,
      usage_minutes,
      contract_type
    FROM `mydataset.customer_data`
    ```

### 4.2 Data Loss Prevention (DLP)
*   **Inspection**: Finding the sensitive data.
*   **De-identification**: Masking it.
    *   *Masking*: `1234-5678` -> `XXXX-XXXX`.
    *   *Tokenization*: Replaces sensitive data with a token locally, preserving the format but securing the original value.

---

## 🛠️ Section 5: Maintenance & Capacity

### BigQuery Editions & Slots
*   **Slots**: The currency of compute in BigQuery. 1 Slot ~ 1 Virtual CPU.
*   **Baseline vs Autoscaling**:
    *   *Baseline*: "I always need 500 slots for my daily reports." (You pay for them 24/7).
    *   *Autoscaling*: "Burst up to 2000 slots if a complex query comes in."
*   **Scenario**: A CEO's dashboard is slow because the Marketing team is running huge queries.
    *   *Solution*: Create a **Reservation** for the "Executive Dashboard" project (e.g., 100 baseline slots) to guarantee resources. Ideally, move Marketing to a separate reservation or limit their usage only to "idle" slots.
