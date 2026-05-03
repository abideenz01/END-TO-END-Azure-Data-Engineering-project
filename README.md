## 📌 Project Overview
### A production-grade, end-to-end Azure Data Engineering project that ingests streaming music data from an Azure SQL Database, processes it incrementally through Bronze → Silver → Gold layers using the Medallion Architecture, and serves it as a Star Schema dimensional model for analytics consumption.
### The pipeline features CDC-based incremental loading, Auto Loader streaming, Jinja2 dynamic SQL, Delta Live Tables, email alerting and CI/CD via GitHub and Databricks Asset Bundles.

## 🏗️ Architecture
┌──────────────────────────────────────────────────────────────────┐
│                        SOURCE LAYER                               │
│               Azure SQL Database                                  │
│     FactStream | DimUser | DimTrack | DimDate  ! DimArtist              │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                     INGESTION LAYER                               │
│          Azure Data Factory — Metadata Driven Pipeline            │
│                                                                   │
│  ┌─────────────┐   ┌──────────────┐   ┌────────────────────┐   │
│  │  last_cdc   │──►│azuresqlToLake│──►│ if_incremental_    │   │
│  │  (Lookup)   │   │  (Copy Data) │   │ data (Condition)   │   │
│  └─────────────┘   └──────────────┘   └────────────────────┘   │
│                                                 │                 │
│                              ┌──────────────────┤   (for testing alert is activated on success and failure of pipeline)            │
│                              ▼                  ▼                 │
│                         ✅ Success         ✅ Failure            │
│                         Email Alert         Email Alert           │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                      BRONZE LAYER                                 │
│              Azure Data Lake Storage Gen2                         │
│         Raw Parquet Files — Incremental Only                      │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                      SILVER LAYER                                 │
│               Databricks + Auto Loader + Delta Lake               │
│  • cloudFiles Streaming  • Schema Evolution                      │
│  • Transformations       • Jinja2 Dynamic Joins                  │
│  • Checkpoint Management • Unity Catalog                         │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                       GOLD LAYER                                  │
│            Databricks Delta Live Tables (DLT)                     │
│  Staging → DimDate | DimTrack | DimUser | FactStream             │
│  • Data Quality Expectations • Incremental Streaming Tables      │
└─────────────────────────┬────────────────────────────────────────┘
                          │
                          ▼
┌──────────────────────────────────────────────────────────────────┐
│                    DEPLOYMENT LAYER                               │
│             Databricks Asset Bundles (DAB)                        │
│         Dev Environment | Prod Environment | GitHub CI/CD        │
└──────────────────────────────────────────────────────────────────┘


## 🛠️ Tech Stack
## Category: Technology 
## Cloud Platform:     Microsoft AzureData 
## Ingestion:          Azure Data Factory (Metadata Driven) 
## Data Storage:       Azure Data Lake Storage Gen2 
## Data Processing:    Azure Databricks (Serverless Compute)
## File Format:        Delta Lake, Parquet 
## Streaming:          Auto Loader (cloudFiles)
## Gold Orchestration: Delta Live Tables (DLT) 
## Dynamic SQL:        Jinja2 Templating 
## Governance:         Unity Catalog 
## CI/CD:              GitHub + Databricks Asset Bundles 
## Alerting:           Web Activity + Email Notifications
## Source Database:    Azure SQL Database 
## Languages:          Python, PySpark, SQL 

## 📂 Repository Structure
END TO END AZURE DATA ENGINEERING PROJECT
│
├── adf/                                         
# Azure Data Factory
│   ├── pipeline/
│   │   └── incremental_Loop.json
│   ├── dataset/
│   ├── linkedService/
│   └── trigger/
│
## databricks/
│   ├── notebooks/
│   │   ├── bronze_to_silver/
│   │   │   ├── nb_silver_factstream.py
│   │   │   ├── nb_silver_dimuser.py
│   │   │   ├── nb_silver_dimtrack.py
│   │   │   └── nb_silver_dimdate.py
│   │   ├── silver_to_gold/
│   │   │   ├── dlt_gold_factstream.py
│   │   │   ├── dlt_gold_dimuser.py
│   │   │   ├── dlt_gold_dimtrack.py
│   │   │   └── dlt_gold_dimdate.py
│   │   └── utils/
│   │       └── reusable.py
│   └── bundles/
│       ├── databricks.yml
│       └── databricks.dev.yml.example
│
├── sql/
│   └── watermark_table.sql
│
├── docs/
│   └── screenshots/
│
├── .gitignore
└── README.md

## 1️⃣ ADF Metadata Driven Pipeline — CDC ForEach Loop

Incremental pipeline running inside ForEach with last_cdc Lookup → azuresqlToLake Copy → if_incremental_data Condition. All activities succeeded in 4m 31s.
<img width="1903" height="865" alt="Azure Project Pipeline" src="https://github.com/user-attachments/assets/e51216d8-64c6-4a6e-ab18-2b99357d88b9" />

## 2️⃣ ADF Pipeline — Full Run (26 Activities Succeeded)
Complete pipeline run showing all 26 activities succeeded including Delete, Copy Data and If Condition across all tables dynamically.

