# Data Product Contract

## 1. Overview

### Data Product Name

**University Chapters**

### Version

**v1.0**

### Product Owner

Sairam Prasad Gurajapu

### Source System

ArcGIS REST API – University Chapter Feature Service

### Purpose

The **University Chapters** data product provides a curated, analytics-ready dataset containing university chapter information retrieved from the ArcGIS REST API. The product is designed for reporting, visualization, geospatial analysis, and downstream analytical workloads.

The data product is built using the Medallion Architecture (Bronze → Silver → Gold) and published as a Delta Lake dataset.

---

# 2. Business Objective

The objective of this data product is to:

* Ingest university chapter data from a public REST API.
* Preserve raw source data for audit and traceability.
* Standardize and cleanse incoming records.
* Validate geographic coordinates.
* Produce a trusted dataset for analytics.
* Maintain data quality through validation and quarantine processes.

---

# 3. Data Flow

```text
Source API
     │
     ▼
Bronze Layer
(Raw JSON)
     │
     ▼
Silver Layer
(Cleansed & Validated)
     │
     ├────────────► Quarantine
     │
     ▼
Gold Layer
(Data Product)
```

---

# 4. Data Source

| Property       | Value     |
| -------------- | --------- |
| Source Type    | REST API  |
| Provider       | ArcGIS    |
| Format         | JSON      |
| Authentication | Public    |
| Refresh Type   | On-demand |

---

# 5. Storage Details

| Layer      | Storage                      |
| ---------- | ---------------------------- |
| Bronze     | Unity Catalog Managed Volume |
| Silver     | Unity Catalog Managed Volume |
| Gold       | Unity Catalog Managed Volume |
| Quarantine | Unity Catalog Managed Volume |

Storage Format

* Delta Lake

---

# 6. Processing Layers

## Bronze Layer

### Purpose

Stores raw API responses exactly as received from the source system.

### Characteristics

* Immutable
* Raw JSON
* Run ID generated for every execution
* Ingestion metadata captured
* No business transformations

---

## Silver Layer

### Purpose

Transforms raw JSON into a structured dataset.

### Processing

* Dynamic JSON flattening
* Column standardization
* Data type conversion
* Duplicate removal
* Data quality validation
* Warning generation
* Quarantine identification

---

## Gold Layer

### Purpose

Publishes a governed data product for downstream consumers.

### Characteristics

* Analytics-ready
* Standardized schema
* Data quality validated
* Delta Lake storage
* Versioned output

---

# 7. Dataset Schema

| Column                     | Data Type     | Description                          |
| -------------------------- | ------------- | ------------------------------------ |
| chapter_id                 | Integer       | Unique university chapter identifier |
| university_chapter         | String        | University chapter name              |
| city                       | String        | City                                 |
| state                      | String        | State                                |
| longitude                  | Double        | Longitude (WGS84)                    |
| latitude                   | Double        | Latitude (WGS84)                     |
| coordinates                | String        | Longitude and Latitude combined      |
| dq_status                  | String        | CLEAN, WARNING, or QUARANTINE        |
| dq_warnings                | Array<String> | Warning messages                     |
| run_id                     | String        | Pipeline execution identifier        |
| ingestion_timestamp        | Timestamp     | Bronze ingestion timestamp           |
| silver_processed_timestamp | Timestamp     | Silver processing timestamp          |
| gold_processed_timestamp   | Timestamp     | Gold publishing timestamp            |

---

# 8. Data Quality Rules

## DQ-Q1 – Invalid Coordinates

### Rule

A record is considered invalid if:

* Longitude < -180
* Longitude > 180
* Latitude < -90
* Latitude > 90
* Longitude is NULL
* Latitude is NULL

### Action

* Record is moved to the Quarantine layer.
* Record is excluded from the Gold layer.

---

## DQ-W1 – Missing or Unknown City

### Rule

City value is:

* NULL
* Empty
* "Unknown"

### Action

* Record is published.
* dq_status = WARNING
* dq_warnings populated with MISSING_OR_UNKNOWN_CITY

---

# 9. Data Quality Status

| Status     | Description                                       |
| ---------- | ------------------------------------------------- |
| CLEAN      | Passed all validation rules                       |
| WARNING    | Passed validation with non-critical issues        |
| QUARANTINE | Failed critical validation and excluded from Gold |

---

# 10. Refresh Strategy

| Property     | Value              |
| ------------ | ------------------ |
| Refresh Type | Manual / On-demand |
| Frequency    | Pipeline Execution |
| Load Type    | Full Refresh       |

Future enhancement:

* Incremental Processing
* Change Data Capture (CDC)

---

# 11. Consumers

This data product is intended for:

* Power BI Dashboards
* Business Reporting
* GIS Applications
* Data Scientists
* Analysts
* Downstream ETL Pipelines

---

# 12. Security

The source API is publicly accessible.

Within Azure Databricks:

* Unity Catalog provides governance.
* Managed Volumes provide secure storage.
* Data is stored in Delta Lake format.

---

# 13. Lineage

```text
ArcGIS REST API
        │
        ▼
Bronze
(Raw JSON)
        │
        ▼
Silver
(Cleansed Data)
        │
        ▼
Gold
Published Dataset
```

---

# 14. Data Retention

| Layer      | Retention                           |
| ---------- | ----------------------------------- |
| Bronze     | Retained for audit and reprocessing |
| Silver     | Latest processed dataset            |
| Gold       | Current published version           |
| Quarantine | Retained for investigation          |

---

# 15. Assumptions

* Source API is publicly accessible.
* API returns data in JSON format.
* Coordinates are supplied in WGS84 (EPSG:4326).
* Unity Catalog Managed Volumes are used due to Azure Databricks Free Edition limitations.
* The Silver layer processes the latest successful Bronze ingestion.

---

# 16. Versioning

| Version | Description            |
| ------- | ---------------------- |
| v1.0    | Initial implementation |

Future versions may include:

* Incremental ingestion
* Schema evolution
* Delta Live Tables
* Automated monitoring
* Data lineage enhancements

---

# 17. SLA

| Metric                | Target                            |
| --------------------- | --------------------------------- |
| Pipeline Availability | 99%                               |
| Data Completeness     | 100% of source records ingested   |
| Data Accuracy         | Validated using DQ rules          |
| Gold Publication      | Successful completion of pipeline |

---

# 18. Success Criteria

The data product is considered successful when:

* All source records are ingested into Bronze.
* JSON is successfully transformed in Silver.
* Invalid records are quarantined.
* Gold contains only CLEAN and WARNING records.
* Dataset is available for downstream analytics.

---

# 19. Contact

**Author:** Sairam Prasad Gurajapu

**Repository:** https://github.com/sairam-prasad-g/pharos_lab_assignment
