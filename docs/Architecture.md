# Architecture

# 1. Overview

This document describes the technical architecture implemented for the **Pharos Analytics Lab Data Engineer Assignment**.

The solution follows the **Medallion Architecture** using **Azure Databricks**, **PySpark**, **Delta Lake**, and **Unity Catalog Managed Volumes** to ingest, transform, validate, and publish university chapter data obtained from a public ArcGIS REST API.

The architecture is designed to provide:

* Scalability
* Maintainability
* Traceability
* Data Quality
* Layered Processing
* Enterprise Readiness

---

# 2. High-Level Architecture

```text
                        +-------------------------+
                        |   ArcGIS REST API       |
                        |  (University Chapters)  |
                        +------------+------------+
                                     |
                                     |
                                     v
                    +-------------------------------+
                    | 01_Bronze_Ingestion Notebook  |
                    +-------------------------------+
                                     |
                                     |
                                     v
                  +------------------------------------+
                  | Bronze Layer (Raw Delta Storage)   |
                  | Unity Catalog Managed Volume       |
                  +----------------+-------------------+
                                   |
                                   |
                                   v
             +-------------------------------------------+
             | 02_Silver_Transformation Notebook         |
             +-------------------------------------------+
                    |                           |
                    |                           |
                    v                           v
      +-------------------------+      +-------------------------+
      | Silver Layer            |      | Quarantine Layer        |
      | Clean & Validated Data  |      | Invalid Records         |
      +------------+------------+      +-------------------------+
                   |
                   |
                   v
          +--------------------------------+
          | 03_Gold_Publish Notebook       |
          +--------------------------------+
                   |
                   |
                   v
        +--------------------------------------+
        | Gold Layer (Published Data Product)  |
        +----------------+---------------------+
                         |
                         |
                         v
         +--------------------------------------+
         | Power BI / Analytics / Reporting     |
         +--------------------------------------+
```

---

# 3. Medallion Architecture

The implementation follows the Medallion Architecture, where data progresses through multiple quality stages.

## Bronze Layer

### Objective

Preserve source data exactly as received.

### Characteristics

* Raw JSON
* Immutable
* Delta format
* One folder created for each pipeline execution
* Includes ingestion metadata

Example:

```text
/Volumes/assignment/university/bronze/university_chapters/<run_id>/
```

---

## Silver Layer

### Objective

Transform raw JSON into a structured, validated dataset.

### Activities

* Read latest Bronze execution
* Dynamic JSON flattening
* Data type conversion
* Standardized column names
* Duplicate removal
* Data Quality validation
* Metadata enrichment

Output

```text
/Volumes/assignment/university/silver/university_chapters
```

---

## Quarantine Layer

### Objective

Store records that fail critical validation rules.

Examples

* Invalid longitude
* Invalid latitude
* Missing coordinates

Output

```text
/Volumes/assignment/university/quarantine/university_chapters
```

---

## Gold Layer

### Objective

Publish a trusted, analytics-ready dataset.

Characteristics

* Clean records only
* Warning records included
* Invalid records excluded
* Delta format
* Versioned data product

Output

```text
/Volumes/assignment/university/gold/university_chapters/v1
```

---

# 4. Pipeline Flow

```text
ArcGIS REST API
        │
        ▼
01_Bronze_Ingestion
        │
        ▼
Bronze Delta Storage
        │
        ▼
02_Silver_Transformation
        │
        ├────────► Quarantine
        │
        ▼
Silver Delta Storage
        │
        ▼
03_Gold_Publish
        │
        ▼
Gold Delta Storage
        │
        ▼
Power BI / Analytics
```

Each notebook is independent and reads data from the previous layer rather than relying on in-memory variables.

---

# 5. Storage Architecture

```text
Unity Catalog
└── assignment
    └── university
        ├── bronze
        │   └── university_chapters
        │       └── <run_id>
        │
        ├── silver
        │   └── university_chapters
        │
        ├── quarantine
        │   └── university_chapters
        │
        └── gold
            └── university_chapters
                └── v1
```

---

# 6. Data Flow

## Step 1 – Source

The ArcGIS REST API returns nested JSON data containing:

* Attributes
* Geometry

---

## Step 2 – Bronze

The Bronze notebook:

* Calls the API
* Downloads JSON
* Generates a Run ID
* Adds ingestion metadata
* Stores raw records in Delta format

No business transformation occurs at this stage.

---

## Step 3 – Silver

The Silver notebook:

* Reads the latest Bronze dataset
* Dynamically flattens JSON
* Standardizes schema
* Validates coordinates
* Generates warnings
* Creates quarantine records
* Writes the Silver dataset

---

## Step 4 – Gold

The Gold notebook:

* Reads the Silver dataset
* Excludes quarantined records
* Publishes CLEAN and WARNING records
* Creates Version 1 of the data product

---

# 7. Data Quality Architecture

Two categories of validation are implemented.

## Critical Validation

Examples

* Longitude outside -180 to 180
* Latitude outside -90 to 90
* Missing coordinates

Action

* Move record to Quarantine

---

## Warning Validation

Examples

* Missing City
* Unknown City

Action

* Publish record
* Mark as WARNING

---

# 8. Metadata Flow

Metadata is maintained across all layers.

```text
Bronze
-------
Run ID
Pipeline Name
Source System
Ingestion Timestamp

          │

          ▼

Silver
-------
Run ID
Silver Processing Timestamp
DQ Status
DQ Warnings

          │

          ▼

Gold
-----
Run ID
Gold Processing Timestamp
Version
```

This enables end-to-end traceability.

---

# 9. Technology Architecture

| Component       | Technology                    |
| --------------- | ----------------------------- |
| Compute         | Azure Databricks              |
| Language        | PySpark                       |
| Storage Format  | Delta Lake                    |
| Governance      | Unity Catalog                 |
| Storage         | Unity Catalog Managed Volumes |
| Source          | ArcGIS REST API               |
| Version Control | GitHub                        |

---

# 10. Error Handling

The pipeline includes handling for:

* API connectivity failures
* Empty API responses
* Invalid JSON
* Data quality failures
* Storage write failures

Meaningful log messages are generated to assist troubleshooting.

---

# 11. Enterprise Architecture

In a production Azure environment, the architecture can be enhanced as follows:

```text
ArcGIS REST API
        │
        ▼
Azure Data Factory / Databricks Workflows
        │
        ▼
Azure Databricks
        │
        ▼
Azure Data Lake Storage Gen2
        │
        ▼
Delta Lake
        │
        ▼
Unity Catalog
        │
        ▼
Power BI
```

Additional enterprise capabilities include:

* Azure Data Lake Storage Gen2
* Unity Catalog External Volumes
* Databricks Workflows
* Incremental Processing
* Change Data Capture (CDC)
* Delta Live Tables
* Azure DevOps or GitHub Actions for CI/CD
* Data Quality monitoring with Soda or Great Expectations
* Unity Catalog Lineage
* Automated alerting and monitoring

---

# 12. Design Principles

The solution is based on the following engineering principles:

* Separation of ingestion, transformation, and publishing.
* Immutable Bronze layer for auditability.
* Independent notebook execution.
* Metadata-driven processing.
* Reusable and modular code.
* Delta Lake for reliable storage.
* Data quality enforcement.
* Scalable architecture suitable for enterprise deployment.

---

# 13. Conclusion

The implemented architecture demonstrates a modern cloud-native data engineering solution using the Medallion Architecture.

By separating raw ingestion, transformation, validation, and publication into dedicated layers, the solution provides a scalable, maintainable, and traceable data platform. While developed using Azure Databricks Free Edition with Unity Catalog Managed Volumes, the architecture can be migrated to a full enterprise Azure environment with minimal changes to the core processing logic.
