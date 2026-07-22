# Pharos Analytics Lab – Data Engineer Assignment

## Overview

This repository contains my solution for the **Pharos Analytics Lab Data Engineer Assignment**. The solution demonstrates an end-to-end Data Engineering pipeline using the **Medallion Architecture (Bronze → Silver → Gold)** implemented with **Azure Databricks**, **PySpark**, **Delta Lake**, and **Unity Catalog Managed Volumes**.

The pipeline ingests university chapter data from a public ArcGIS REST API, applies data quality validations, transforms the raw data into a curated dataset, and publishes a governed Gold data product suitable for downstream analytics and reporting.

---

# Solution Architecture

```text
                    Public ArcGIS REST API
                              │
                              ▼
                    Bronze Ingestion Layer
           (Raw JSON + Metadata + Run History)
                              │
                              ▼
                 Silver Transformation Layer
      (Flattening + Data Quality + Standardization)
                    │                      │
                    │                      ▼
                    │              Quarantine Layer
                    │         (Invalid Coordinate Records)
                    ▼
                Gold Publish Layer
      (Clean + Warning Records Only)
                              │
                              ▼
                Analytics / Reporting / BI
```

---

# Technology Stack

* Azure Databricks (Free Edition - Serverless Compute)
* Apache Spark (PySpark)
* Delta Lake
* Unity Catalog
* Unity Catalog Managed Volumes
* Python
* GitHub

---

# Environment & Platform Considerations

This solution was developed using **Azure Databricks Free Edition (Serverless Compute)**.

Azure Databricks Free Edition has several platform limitations compared to a standard Azure Databricks workspace. In particular:

* Custom Azure Data Lake Storage Gen2 (ADLS Gen2) authentication using legacy Hadoop configurations (`fs.azure.account.*`) is not supported.
* Creating and using Unity Catalog External Locations backed by customer-managed ADLS Gen2 storage is not available.
* SQL Warehouses are not available in the Free Edition.
* Storage is limited to Unity Catalog Managed Volumes.

Therefore, instead of using customer-managed ADLS Gen2 containers, this implementation stores Bronze, Silver, Gold, and Quarantine datasets in **Unity Catalog Managed Volumes**.

The data processing logic remains identical to a production implementation. In an enterprise Azure Databricks environment, the same notebooks can be migrated to ADLS Gen2-backed External Volumes or direct `abfss://` storage paths with only storage configuration changes.

---

# Repository Structure

```text
pharos_lab_assignment
│
├── README.md
├── LICENSE
├── .gitignore
│
├── notebooks
│   ├── 00_Create_Catalog.dbc
│   ├── 00_Create_Catalog.ipynb
│   ├── 01_Bronze_Ingestion.dbc
│   ├── 01_Bronze_Ingestion.ipynb
│   ├── 02_Silver_Transformation.dbc
│   ├── 02_Silver_Transformation.ipynb
│   ├── 03_Gold_Publish.dbc
│   ├── 03_Gold_Publish.ipynb
│   └── utils.py
│
├── docs
    ├── Data_Product_Contract.md
    ├── Solution_Approach.md
    └── Architecture.md
```

---

# Storage Architecture

The solution uses Unity Catalog Managed Volumes to implement the Medallion Architecture.

```text
Unity Catalog
└── assignment
    └── university
        ├── bronze
        │   └── university_chapters
        ├── silver
        │   └── university_chapters
        ├── gold
        │   └── university_chapters
        └── quarantine
            └── university_chapters
```

---

# Medallion Architecture

## Bronze Layer

### Purpose

* Ingest raw data from the public ArcGIS REST API.
* Preserve the original JSON payload.
* Maintain ingestion history.
* Generate a unique Run ID for every pipeline execution.

### Processing

* Reads data from the REST API.
* Stores raw JSON without transformation.
* Generates ingestion metadata.
* Creates a unique Run ID for traceability.

### Output

```text
/Volumes/assignment/university/bronze/university_chapters/<run_id>/
```

---

## Silver Layer

### Purpose

Transform raw JSON into a clean, validated, analytics-ready dataset.

### Processing

* Reads the latest Bronze ingestion.
* Dynamically flattens nested JSON structures.
* Converts data types.
* Standardizes column names.
* Formats coordinates.
* Removes duplicate records.
* Applies Data Quality rules.
* Creates Warning and Quarantine datasets.

---

### Data Quality Rules

#### DQ-Q1 – Invalid Coordinates

Records are quarantined if:

* Longitude is outside the range **-180 to 180**
* Latitude is outside the range **-90 to 90**
* Longitude is NULL
* Latitude is NULL

Action:

* Move record to the Quarantine layer.

---

#### DQ-W1 – Missing or Unknown City

Records with missing or unknown city values are still published.

Action:

* `dq_status = WARNING`
* `dq_warnings = ["MISSING_OR_UNKNOWN_CITY"]`

---

### Outputs

* Silver Dataset
* Quarantine Dataset

---

## Gold Layer

### Purpose

Publish a governed Data Product for analytics and reporting.

### Processing

* Reads the Silver dataset.
* Publishes only:

  * Clean records
  * Warning records
* Excludes quarantined records.
* Adds product metadata.
* Creates a versioned Data Product.

### Output

```text
/Volumes/assignment/university/gold/university_chapters/v1
```

---

# Data Product

| Property     | Value                          |
| ------------ | ------------------------------ |
| Product Name | University Chapters            |
| Version      | v1                             |
| Data Source  | ArcGIS REST API                |
| Refresh Type | On-demand                      |
| Storage      | Unity Catalog Managed Volume   |
| Format       | Delta                          |
| Consumers    | Power BI, Reporting, Analytics |

---

# Data Quality Summary

| Rule  | Description             | Action               |
| ----- | ----------------------- | -------------------- |
| DQ-Q1 | Invalid coordinates     | Quarantine           |
| DQ-W1 | Missing or Unknown City | Publish with WARNING |

---

# Assumptions

* The source API is publicly accessible.
* Bronze layer stores raw ingestion data for every execution.
* Silver layer processes the latest successful Bronze ingestion.
* Gold layer publishes only validated records.
* Coordinates are supplied in WGS84 (EPSG:4326).
* Unity Catalog Managed Volumes are used due to Azure Databricks Free Edition limitations.

---

# How to Execute

Execute the notebooks in the following order:

1. `01_Bronze_Ingestion`
2. `02_Silver_Transformation`
3. `03_Gold_Publish`

---

# Expected Outputs

### Bronze

* Raw JSON
* One folder per Run ID

### Silver

* Clean records
* Warning records

### Quarantine

* Invalid coordinate records

### Gold

* Analytics-ready curated Data Product

---

# Key Features

* End-to-End Medallion Architecture
* Dynamic JSON Flattening
* Data Quality Validation
* Quarantine Handling
* Delta Lake Storage Format
* Unity Catalog Managed Volumes
* Metadata-driven Processing
* Modular Notebook Design
* Production-style Logging
* Versioned Gold Data Product

---

# Production Considerations

In an enterprise Azure Databricks environment, this implementation can be enhanced by:

* Using Azure Data Lake Storage Gen2 as the primary storage layer.
* Configuring Unity Catalog External Volumes.
* Orchestrating notebooks using Databricks Workflows.
* Implementing Incremental Processing with Watermarks.
* Integrating CI/CD pipelines using Azure DevOps or GitHub Actions.
* Monitoring Data Quality using Soda or Great Expectations.
* Enabling Unity Catalog Governance and Lineage.

---

# Future Enhancements

* Incremental data ingestion
* Delta Live Tables
* Change Data Capture (CDC)
* Schema Evolution
* Automated Monitoring
* Pipeline Alerting
* Data Lineage
* Unit Testing
* CI/CD Automation

---

# Author

**Sairam Prasad Gurajapu**

GitHub Repository:

https://github.com/sairam-prasad-g/pharos_lab_assignment

---

# Acknowledgement

This project was developed as part of the **Pharos Analytics Lab Data Engineer Assignment** to demonstrate practical Data Engineering skills using Azure Databricks, PySpark, Delta Lake, Unity Catalog, and the Medallion Architecture.
