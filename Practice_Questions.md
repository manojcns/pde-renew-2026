# Professional Data Engineer Renewal: Practice Exam Questions

These 20 scenario-based questions are designed to test your understanding of the concepts covered in the study guide. Try to answer them without looking at the solutions first.

### Section 1: designing Data Processing Systems

**Q1.** Your global gaming company needs a database for player leaderboards and real-time game state. The database must be strongly consistent globally and support horizontal scaling to handle millions of concurrent players across US, Europe, and Asia. Which database should you choose?
A. Cloud Bigtable
B. Cloud Spanner
C. Cloud SQL for PostgreSQL
D. Firestore

**Q2.** You are designing a compliance requirement for a healthcare application. The application stores patient forms as PDF documents in Cloud Storage. You need to ensure that no patient social security numbers (SSNs) are accidentally stored in these files without redaction. You want to automate this checking process. What should you use?
A. Cloud IAM
B. Cloud Data Loss Prevention (DLP) API
C. VPC Service Controls
D. Dataplex

**Q3.** Your data analytics team consists primarily of SQL experts. They need to build a data pipeline that takes raw JSON logs already loaded into a BigQuery staging table, cleans them, aggregates them by user, and creates a final "Daily Active Users" table. They want to use version control and unit testing. Which tool is most appropriate?
A. Cloud Dataflow
B. Cloud Data Fusion
C. Dataform
D. Cloud Composer

**Q4.** A marketing team wants to combine customer data from Salesforce (CRM) and an on-premise Oracle database. They do not have coding experience but need to build an ETL pipeline to land this data in BigQuery. They prefer a visual interface. Which tool should you recommend?
A. Dataproc
B. Cloud Data Fusion
C. Cloud Dataflow
D. Cloud Functions

### Section 2: Ingesting and Processing

**Q5.** You are building a streaming pipeline to ingest sensor data from 100,000 IoT devices. The data arrives at 50,000 events per second. You need to calculate the average temperature per device every 1 minute and write the result to BigQuery. Which combination of services is best?
A. Pub/Sub -> Cloud Functions -> BigQuery
B. Pub/Sub -> Cloud Dataflow -> BigQuery
C. Cloud Storage -> Cloud Composer -> BigQuery
D. Pub/Sub -> Dataform -> BigQuery

**Q6.** You need to orchestrate a lightweight workflow. When a user uploads a profile picture to a specific Cloud Storage bucket, you need to trigger a Cloud Run service to resize the image, and then update a Firestore document with the new image URL. You want a serverless, low-cost solution with no cluster maintenance. Which orchestration tool should you use?
A. Cloud Composer
B. Cloud Workflows
C. Cloud Tasks
D. Cloud Scheduler

**Q7.** You have a complex ETL workflow that runs every night. It involves spinning up a Dataproc cluster, running a PySpark job, checking data quality with a custom Python script, and then loading data into BigQuery. The workflow has complex retry logic and dependencies. Which tool is best suited for orchestrating this?
A. Cloud Workflows
B. Cloud Scheduler
C. Cloud Composer
D. Cron on a Compute Engine VM

### Section 3: Storing the Data

**Q8.** You are designing a schema for Cloud Bigtable to store time-series data from weather sensors. You want to query data for a specific sensor over a specific time range. You also want to avoid hotspotting during writes. Which Row Key design is most effective?
A. `[timestamp]#[sensor_id]`
B. `[sensor_id]#[timestamp]`
C. `[sensor_id]#[reversed_timestamp]`
D. `[hash(sensor_id)]#[timestamp]`

**Q9.** You have a large fact table in BigQuery called `Transaction_Logs` (50 TB). Your analysts frequently run queries to analyze transactions for a specific `TransactionDate` and filtered by `Store_ID`. These queries are currently too slow and expensive. How should you optimize the table schema?
A. Partition by `Store_ID` and Cluster by `TransactionDate`.
B. Partition by `TransactionDate` and Cluster by `Store_ID`.
C. Create a separate table for each `Store_ID`.
D. Use `TransactionDate` as a shard key.

**Q10.** Your organization is migrating a legacy PostgreSQL database to Google Cloud. The database supports a mix of heavy transactional workloads (OLTP) during the day and complex analytical reporting queries (OLAP) at night. You want a managed service that can handle both efficiently without needing a separate data warehouse for the reporting. What should you use?
A. Cloud SQL for PostgreSQL
B. AlloyDB for PostgreSQL
C. Cloud Spanner
D. BigQuery

**Q11.** You are developing a mobile application that needs to sync user data (like to-do lists) across devices in real-time. The app must support offline mode where users can update their lists without internet, and sync when they reconnect. Which database is designed for this?
A. Cloud SQL
B. Bigtable
C. Firestore
D. Memorystore

**Q12.** You want to query massive datasets (Petabytes) of CSV and Parquet files stored in Google Cloud Storage and AWS S3 using standard SQL and BigQuery security controls, without actually moving the data into BigQuery storage. Which feature should you use?
A. External Tables
B. BigQuery Omni
C. BigLake
D. Data Transfer Service

### Section 4: Analytics and AI

**Q13.** You want to build a machine learning model to predict customer churn based on transaction history stored in BigQuery. Your data science team is small and prefers using SQL over Python/TensorFlow. What is the fastest way to build and deploy this model?
A. Export data to Vertex AI and use AutoML.
B. Use BigQuery ML to train a logistic regression model using SQL.
C. Spin up a Dataproc user to run Spark MLlib.
D. Use Dataflow to train a model.

**Q14.** You have a data warehouse in BigQuery. You want to share a dataset of "Public Holiday Sales" with a partner organization securely. You do not want to copy the data, and you want to be able to revoke access if the partnership ends. What should you use?
A. Create a Service Account for the partner and give them access.
B. Export the data to a GCS bucket and give the partner read access.
C. Use Analytics Hub to create a listing and share it with the partner.
D. Email the partner a CSV export.

### Section 5: Maintenance and Capacity

**Q15.** The Marketing team is running massive, inefficient queries in your organization's BigQuery project, causing "Slot Exhaustion" errors for the critical Finance team's scheduled reports. How can you ensure the Finance team always has enough capacity?
A. Enable creating a new BigQuery project for Finance.
B. Purchase a Slot Reservation and assign it specifically to the Finance project.
C. Tell the Marketing team to run queries only at night.
D. Switch the Finance team to On-Demand pricing.

**Q16.** You need to restrict your data engineering team so they can only create Cloud Storage buckets in the `us-east1` region to comply with new data residency laws. How should you enforce this at the organization level?
A. Use IAM Roles to remove "Storage Admin" permissions.
B. Configure a VPC Service Perimeter.
C. Use an Organization Policy constraint (`constraints/gcp.resourceLocations`).
D. Use Cloud Monitoring to alert when a bucket is created elsewhere.

**Q17.** You have a Bigtable cluster that uses HDD storage to save on costs. You notice that the read latency is very high (over 500ms). You need to lower the latency to under 50ms for a new real-time dashboard. What should you do?
A. Add more nodes to the cluster.
B. Change the storage type from HDD to SSD.
C. Change the schema to a wide-column format.
D. Add a Memorystore cache in front of Bigtable.

**Q18.** Your Cloud Spanner instance performs well for simple lookups, but joins between the `Customer` table and the `Orders` table are slower than expected. You query `Orders` for a specific `Customer` very frequently. How can you optimize this?
A. Create a secondary index on `Orders`.
B. Use Interleaved Tables to store `Orders` as a child of `Customers`.
C. Increase the number of nodes in the Spanner instance.
D. Denormalize the data into a single table.

**Q19.** You need to prevent data exfiltration. You want to ensure that if a virtual machine in your project is compromised, an attacker cannot copy data from your private Cloud Storage bucket to an external public bucket. What security control prevents this?
A. Identity-Aware Proxy (IAP)
B. VPC Service Controls (VPC-SC)
C. Cloud Armor
D. Private Google Access

**Q20.** You have a Dataflow pipeline that processes data from Pub/Sub. The pipeline is falling behind, and the "System Lag" metric is increasing. You notice the CPU utilization on the workers is consistently at 90%. What feature should you enable/configure to handle this spike automatically?
A. Dataflow Prime
B. Dataflow Vertical Autoscaling
C. Dataflow Horizontal Autoscaling
D. Switch to a larger machine type manually.
