# Pharos Analytics Lab – Data Engineer Assignment

## Overview

This repository contains my solution for the **Pharos Analytics Lab Data Engineer Assignment**, demonstrating the implementation of an end-to-end Medallion Architecture (Bronze → Silver → Gold) using **Azure Databricks**, **PySpark**, and **Unity Catalog Volumes**.

The solution ingests university chapter data from a public ArcGIS REST API, applies data quality validations, transforms the data into a curated dataset, and publishes a governed Gold data product following modern Data Engineering best practices.

---

## Solution Architecture

```
                 Public ArcGIS REST API
                          │
                          ▼
                 Bronze Ingestion Layer
          (Raw JSON + Run ID + Metadata)
                          │
                          ▼
             Silver Transformation Layer
     (Flattening + Data Quality + Cleansing)
                          │
            ┌─────────────┴─────────────┐
            ▼                           ▼
     Silver Layer               Quarantine Layer
 (Clean + Warning Records)   (Invalid Coordinates)
            │
            ▼
              Gold Data Product (v1)
     (Clean + Warning Records Only)
```

---

## Technology Stack

* Azure Databricks
* Apache Spark (PySpark)
* Delta Lake
* Unity Catalog
* Unity Catalog Volumes
* Azure Data Lake Storage (conceptual architecture)
* GitHub
* Python

---

## Repository Structure

```
pharos_lab_assignment
│
├── README.md
├── requirements.txt
├── LICENSE
├── .gitignore
│
├── notebooks
│   ├── 01_Bronze_Ingestion.py
│   ├── 02_Silver_Transformation.py
│   ├── 03_Gold_Publish.py
│   └── utils.py
│
├── docs
│   ├── Data_Product_Contract.md
│   ├── Solution_Approach.md
│   └── Architecture.md
│
├── screenshots
│
└── sample_output
```

---

# Medallion Architecture

## Bronze Layer

Purpose:

* Ingest raw data from the source API
* Preserve original JSON payload
* Maintain ingestion history
* Store one folder per pipeline execution using a unique `run_id`

### Processing

* Connects to ArcGIS REST API
* Downloads raw JSON
* Generates unique Run ID
* Stores raw JSON records
* Adds ingestion metadata

### Output

```
/Volumes/assignment/university/bronze/university_chapters/<run_id>/
```

---

## Silver Layer

Purpose:

Transform raw JSON into a clean, structured dataset.

### Processing

* Reads latest Bronze ingestion
* Dynamically flattens nested JSON
* Renames business columns
* Converts data types
* Formats coordinates
* Removes duplicates
* Applies data quality rules
* Creates warning and quarantine datasets

### Data Quality Rules

#### DQ-Q1

Invalid coordinates:

* Longitude outside -180 to 180
* Latitude outside -90 to 90
* Null coordinates

Action:

* Move record to Quarantine layer

---

#### DQ-W1

Missing or unknown city

Action:

* Publish record
* Set

```
dq_status = WARNING
```

---

### Outputs

```
Silver
```

and

```
Quarantine
```

---

## Gold Layer

Purpose

Publish a governed data product for downstream analytics.

### Processing

* Reads Silver layer

* Publishes only:

  * Clean records
  * Warning records

* Excludes quarantined records

* Adds product metadata

* Creates versioned data product

### Output

```
/Volumes/assignment/university/gold/university_chapters/v1
```

---

# Data Product

Product Name

```
University Chapters
```

Version

```
v1
```

Contains

* Clean records
* Warning records

Excludes

* Quarantined records

---

# Key Features

* End-to-End Medallion Architecture
* Dynamic JSON Flattening
* Data Quality Framework
* Quarantine Handling
* Delta Lake Storage
* Unity Catalog Volumes
* Metadata-driven Processing
* Modular Notebook Design
* Production-ready Logging
* Versioned Gold Data Product

---

# Data Quality Summary

| Rule  | Description                   | Action               |
| ----- | ----------------------------- | -------------------- |
| DQ-Q1 | Invalid longitude or latitude | Quarantine           |
| DQ-W1 | Missing or Unknown City       | Publish with WARNING |

---

# Assumptions

* Source API is publicly accessible.
* Bronze layer preserves raw data for each execution.
* Silver layer always processes the latest Bronze ingestion.
* Gold layer publishes only curated records.
* Coordinates are provided in WGS84 (EPSG:4326).
* Unity Catalog Volumes are used for storage.

---

# How to Run

## Step 1

Execute

```
01_Bronze_Ingestion
```

---

## Step 2

Execute

```
02_Silver_Transformation
```

---

## Step 3

Execute

```
03_Gold_Publish
```

---

# Expected Outputs

Bronze

* Raw JSON
* One folder per Run ID

Silver

* Clean records
* Warning records

Quarantine

* Invalid coordinate records

Gold

* Analytics-ready data product

---

# Improvements Beyond Assignment

The implementation includes several enhancements beyond the minimum assignment requirements:

* Dynamic recursive JSON flattening without hardcoded field names
* Modular notebook design
* Metadata-driven processing
* Reusable utility functions
* Delta Lake implementation
* Processing metrics and logging
* Versioned Gold data product
* Production-ready code structure
* Comprehensive documentation

---

# Future Enhancements

* Incremental ingestion using watermarks
* Delta Live Tables
* Unity Catalog Governance
* Databricks Workflows
* Automated CI/CD using GitHub Actions
* Data Quality monitoring with Soda
* Schema evolution support
* Automated alerting and monitoring

---

# Author

**Sairam Prasad Gurajapu**

Microsoft Certified Azure Data Engineer Associate

Microsoft Certified Power BI Data Analyst Associate

GitHub Repository

https://github.com/sairam-prasad-g/pharos_lab_assignment

---

# Acknowledgement

This project was developed as part of the **Pharos Analytics Lab Data Engineer Assignment** to demonstrate practical data engineering skills using Azure Databricks, PySpark, Delta Lake, and Medallion Architecture.
