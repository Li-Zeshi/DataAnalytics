# DataAnalytics

A **Delta Lakehouse** data pipeline built with **Apache Spark** and **Delta Lake**, following the **Medallion Architecture** (Bronze → Silver → Gold) on **Azure Databricks**.

## Architecture Overview

```
                       01-Ingestion/                           02-Transform/
                     ┌─────────────────┐                   ┌──────────────────────────┐
 HTTP Pricing Data ─►│ HTTP Source      │                   │ Bronze → Delta Table     │
 Streaming Data   ──►│ Streaming Source │─── Bronze ──────►│ Silver Layer Modeling    │─── Gold ────► Reports
 Relational DB    ──►│ DB Source        │   Delta Lake     │   SCD Type 1 & Type 2   │   Delta Lake
 GeoLocation API  ──►│ API Source       │                   │ Fact + Dimension Tables │
 Weather API      ──►│ API Source       │                   │ Geocoding Transform     │
 Streaming Proc.  ──►│ Streaming Proc.  │                   └──────────────────────────┘
                     └─────────────────┘                              ▲
                         │                                            │
                         │  Streaming pricing data                    │  Batch
                         ▼  (JSON, 5s trigger, ADLS Gen2)            │  transformations
                    ┌───────────────────────────────┐                 │
                    │  Bronze Layer (Delta Lake)     │─────────────────┘
                    │  - Raw daily pricing data      │
                    │  - Streaming pricing data      │
                    └───────────────────────────────┘
```

## Pipeline Stages

### 01-Ingestion — Data Ingestion

Ingests raw data from multiple sources into the Bronze layer:

| # | Notebook | Source Type | Description |
|---|---|---|---|
| 1 | `01-Deltalakehouse-pre-setup` | — | Environment setup (Spark, Delta Lake configuration) |
| 2 | `01-Ingest-Daily-Pricing-HTTP-Source-Data` | **HTTP REST API** | Batch ingestion of daily agricultural pricing |
| 3 | `01-Ingest-Daily-Pricing-Streaming-Source-Data` | **Streaming** | Stream ingestion of daily pricing data |
| 4 | `02-Deltalakehouse-bronze-layer-tables-setup` | — | Bronze layer schema & table creation |
| 5 | `03-Ingest-Pricing-Reference-DB-Source-Data` | **JDBC / Relational DB** | Reference pricing data from databases |
| 6 | `04-Ingest-GeoLocation-API-Source-Data` | **External API** | Geographic location data |
| 7 | `05-Ingest-WeatherData-API-Source-Data-Solution` | **External API** | Weather data |
| 8 | `06-Processing-Streaming-Source-Data` | **Azure Data Lake (ABFSS)** | Consumes & processes streaming pricing data within Bronze layer using Spark Structured Streaming (5s trigger, JSON format, exactly-once semantics via checkpoint) |

### 02-Transform — Data Transformation

Transforms data through the Medallion layers:

**Bronze → Silver:**
- **01-Transform-Daily-Pricing-CSV-to-Delta-Table** — Converts raw CSV pricing to Delta format
- **03-Deltalakehouse-silver-layer-table-setup** — Silver layer schema & table creation
- **03-Transform-Reporting-Date-Dimension-Table** — Date dimension table
- **03-Transform-Reporting-Dimension-Tables** — General dimension tables
- **03-Transform-Reporting-Dimension-Tables-SCD-TYPE1** — Slowly Changing Dimension Type 1 (overwrite)
- **03-Transform-Reporting-Dimension-Tables-SCD-TYPE2** — Slowly Changing Dimension Type 2 (history tracking)

**Silver → Gold:**
- **04-Deltalakehouse-gold-layer-reporting-tables-setup** — Gold layer reporting table creation
- **03-Transform-Reporting-Fact-Table-Solution** — Fact table construction (star schema)
- **04-Transform-DataLake-Geocoding-Solution** — Geocoding enrichment in the data lake

## Data Domain

The pipeline processes **agricultural commodity pricing data**, including fields such as:
- `PRODUCT_NAME`, `MARKET_NAME`, `VARIETY`, `ORIGIN`, `STATE_NAME`
- `ARRIVAL_IN_TONNES`, `MINIMUM_PRICE`, `MAXIMUM_PRICE`, `MODAL_PRICE`

## Tech Stack

| Component | Technology |
|---|---|
| **Platform** | Databricks |
| **Processing** | Apache Spark (PySpark) |
| **Storage** | Delta Lake (Azure Data Lake Storage Gen2) |
| **Streaming** | Spark Structured Streaming |
| **Data Modeling** | Star Schema, SCD Type 1 & Type 2 |
| **Format** | Jupyter Notebooks (.ipynb) |

## Key Features

- **Medallion Architecture** — Bronze (raw) → Silver (cleaned) → Gold (aggregated/reporting)
- **Multi-source ingestion** — HTTP, streaming, databases, external APIs
- **Streaming pipeline** — Real-time processing with exactly-once guarantees via checkpoint
- **Slowly Changing Dimensions** — Both SCD Type 1 (overwrite) and Type 2 (versioned history)
- **Delta Lake** — ACID transactions, schema evolution, time travel
