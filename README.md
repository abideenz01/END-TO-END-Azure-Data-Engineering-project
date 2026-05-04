<div align="center">

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&size=26&pause=1000&color=0078D4&width=900&lines=Azure+Data+Engineering+Pipeline+🎵;ADF+%2B+Databricks+%2B+Delta+Lake;CDC+%2B+Medallion+Architecture+Implementation)](https://git.io/typing-svg)

</div>

<div align="center">

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure)
![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge)
![ADF](https://img.shields.io/badge/Azure_Data_Factory-orange?style=for-the-badge)
![Spark](https://img.shields.io/badge/Apache_Spark-E25A1C?style=for-the-badge&logo=apachespark)
![Delta Lake](https://img.shields.io/badge/Delta_Lake-003366?style=for-the-badge)

</div>

## 📌 Project Overview
### A production-grade, end-to-end Azure Data Engineering project that ingests streaming music data from an Azure SQL Database, processes it incrementally through Bronze → Silver → Gold layers using the Medallion Architecture, and serves it as a Star Schema dimensional model for analytics consumption.
### The pipeline features CDC-based incremental loading, Auto Loader streaming, Jinja2 dynamic SQL, Delta Live Tables, email alerting and CI/CD via GitHub and Databricks Asset Bundles.

# 🏗️ Architecture

https://github.com/user-attachments/assets/5e3a2888-f822-4c57-ad98-624df5b5af08





# 🛠️ Tech Stack
<img width="300" height="449" alt="T 1" src="https://github.com/user-attachments/assets/8a6cb791-9700-4deb-928c-cf097e6a15c7" />
<img width="290" height="145" alt="T 2" src="https://github.com/user-attachments/assets/454cded9-961c-46d9-a060-8dead18bcb4d" />



# 📂 Repository Structure
<img width="343" height="174" alt="structure 2" src="https://github.com/user-attachments/assets/fb8622f1-45a4-42ad-a558-cf6cc7ec0ffb" />


# databricks/
<img width="284" height="305" alt="structure" src="https://github.com/user-attachments/assets/6a392592-9db2-4474-a98c-2bb9e8e980dc" />


# 1️⃣ ADF Metadata Driven Pipeline — CDC ForEach Loop

Incremental pipeline running inside ForEach with last_cdc Lookup → azuresqlToLake Copy → if_incremental_data Condition. All activities succeeded in 4m 31s.
<img width="1903" height="865" alt="Azure Project Pipeline" src="https://github.com/user-attachments/assets/e51216d8-64c6-4a6e-ab18-2b99357d88b9" />

# 2️⃣ ADF Pipeline — Full Run (26 Activities Succeeded)
Complete pipeline run showing all 26 activities succeeded including Delete, Copy Data and If Condition across all tables dynamically.
<img width="949" height="431" alt="-Azure Project Pipeline" src="https://github.com/user-attachments/assets/be816597-298c-49f2-a705-e98f422f488a" />

# 3️⃣ ADF Pipeline — Web Activity Email Integration
Pipeline architecture showing ForEach loop connected to Web activity triggering email alerts on both pipeline success ✅ and failure ❌ paths. Connected to both for testing purpose.
<img width="929" height="361" alt="Final Running Pipeline" src="https://github.com/user-attachments/assets/694ff145-ff84-40dc-9050-31d4be87e219" />

# 4️⃣ Delta Live Tables — Gold Layer Pipeline
DLT pipeline showing staging tables (dimdate_stg, dimtrack_stg, dimuser_stg, factstream_stg) flowing into Gold dimension and fact tables. All 8 streaming tables completed with data quality expectations applied.
<img width="952" height="451" alt="Delta Live Tables Pipeline with incremental load" src="https://github.com/user-attachments/assets/010a14e9-b4fa-48c7-af7b-54700274ca57" />

# 7️⃣ Email Alert — Pipeline Success Notification
Automated success email confirming pipeline executed successfully with Pipeline Name and RunID details.
<img width="448" height="295" alt="Screenshot 2026-05-01 144759" src="https://github.com/user-attachments/assets/aba5ea57-279b-4274-bde1-bddf3e279a1b" />

# 🔄 Pipeline Details
1. ADF — Metadata Driven CDC Pipeline
ForEach Loop Flow
ForEach (Dynamic Table List)
    ├── last_cdc       → Lookup last CDC timestamp from JSON
    ├── azuresqlToLake → Copy WHERE updated_at > last_cdc
    ├── if_incremental → Update watermark if new rows exist
    └── Web Activity   → Email on success or failure
CDC Watermark (FactStream_cdc/cdc_json)
json{"cdc": "2025-10-07T19:49:56"}

# 2. Bronze → Silver — Auto Loader Streaming
pythondf = (
    spark.readStream
        .format("cloudFiles")
        .option("cloudFiles.format", "parquet")
        .option("cloudFiles.schemaLocation", schema_path)
        .option("schemaEvolutionMode", "addNewColumns")
        .load(bronze_path)
)

df_transformed = reusable().dropColumns(df, ['_rescued_data'])
df_transformed = df_transformed.withColumn("user_name", upper(col("user_name")))

df_transformed.writeStream \
    .format("delta") \
    .outputMode("append") \
    .option("checkpointLocation", checkpoint_path) \
    .trigger(availableNow=True) \
    .toTable(f"`spotify-cata`.`silver`.`{table_name}`") \
    .awaitTermination()

# 3. Jinja2 Dynamic Joins — Silver Layer
pythonparameters = [
    {
        "table": "spotify-cata.silver.factstream",
        "alias": "factstream",
        "cols": "factstream.stream_id, factstream.listen_duration"
    },
    {
        "table": "spotify-cata.silver.dimuser",
        "alias": "dimuser",
        "cols": "dimuser.user_id, dimuser.user_name",
        "condition": "factstream.user_id = dimuser.user_id"
    },
    {
        "table": "spotify-cata.silver.dimtrack",
        "alias": "dimtrack",
        "cols": "dimtrack.track_id, dimtrack.track_name",
        "condition": "factstream.track_id = dimtrack.track_id"
    }
]
Jinja2 template dynamically generates LEFT JOIN SQL across all Silver tables — no hardcoded logic.

## 4. Gold Layer — Delta Live Tables Star Schema
<img width="313" height="102" alt="data model" src="https://github.com/user-attachments/assets/aee8b763-36a1-478f-bd6a-1af6b4a9d824" />

                
# DLT Data Quality
python@dlt.expect("valid_user_id", "user_id IS NOT NULL")
@dlt.expect("valid_stream_id", "stream_id IS NOT NULL")
@dlt.expect_or_drop("valid_duration", "listen_duration > 0")

# 5. CI/CD — Databricks Asset Bundles
yamlbundle:
  name: spotify_dab

targets:
  dev:
    mode: development
    presets:
      source_linked_deployment: false
    workspace:
      host: ${var.workspace_host}

  prod:
    mode: production
    workspace:
      host: ${var.workspace_host}
bash# Deploy to dev
databricks bundle deploy --target dev

# Deploy to prod
databricks bundle deploy --target prod


# 🔑 Key Design Decisions
<img width="381" height="203" alt="key design decisions" src="https://github.com/user-attachments/assets/49a47bd3-f2e6-4846-93b1-fbf06fd1fea1" />



# 📧 Email Alerting
Sends instant email on success/failure with pipeline Name and pipeline ID crucial for identification of successful/failed pipelines o among multiple pipelines running simultaneously


# Setup local bundle config
cp databricks/bundles/databricks.dev.yml.example \
   databricks/bundles/databricks.dev.yml
### 🌟 About Me: 👋 Hi, I'm Zain UL Abideen | Data Engineer

### I am passionate about Building scalable data pipelines that turn raw data into actionable insights

### 🎓 Computer Engineering Graduate | 💡 Data Engineering Enthusiast | ☁️ Cloud-Native Advocate

### About Me: I'm a data engineer who loves transforming messy data into clean, reliable pipelines. My passion lies in architecting robust data solutions that scale—whether it's processing terabytes in Spark or optimizing complex SQL queries that make databases sing.

### 🔧 Tech Stack:

### • Python • SQL • Big Data: Apache Spark • Databricks • Cloud: Microsoft Azure

### Specialties: • ETL/ELT Pipelines • Data Warehousing • Data Design Architecture • System Design • Performance Optimization

### 🚀 What I'm About:

### 📊 Building end-to-end data solutions from ingestion to visualization ⚡ Optimizing Spark jobs to squeeze out every bit of performance 🏗️ Designing cloud-native architectures on Azure 🧩 Turning complex data problems into elegant engineering solutions

### 📫 Let's Connect! I'm always excited to collaborate on data engineering projects or discuss the latest in distributed computing.

### Email: abideenz095@gmail.com


