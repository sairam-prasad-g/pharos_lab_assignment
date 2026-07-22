# Solution Approach

## 1. Introduction

This document describes the design decisions, architecture, implementation approach, and technical considerations for the **Pharos Analytics Lab Data Engineer Assignment**.

The objective of the solution is to build an end-to-end data engineering pipeline that ingests university chapter data from a public ArcGIS REST API, applies data quality validation, and publishes an analytics-ready dataset using the Medallion Architecture.

The implementation is developed using **Azure Databricks**, **PySpark**, **Delta Lake**, and **Unity Catalog Managed Volumes**.

---

# 2. Solution Objectives

The solution was designed with the following objectives:

* Build an end-to-end ETL pipeline.
* Preserve raw source data for auditing.
* Apply data quality validation rules.
* Separate raw, curated, and published data.
* Produce an analytics-ready Gold dataset.
* Maintain modularity and scalability.
* Follow industry-standard Medallion Architecture.

---

# 3. High-Level Architecture

```text
                    ArcGIS REST API
                           │
                           ▼
                  Bronze Ingestion Layer
                  (Raw JSON Storage)
                           │
                           ▼
               Silver Transformation Layer
         (Flattening, Cleansing & Validation)
                     │               │
                     │               ▼
                     │        Quarantine Layer
                     │    (Invalid Records)
                     ▼
                 Gold Publish Layer
          (Curated Data Product - Delta)
                           │
                           ▼
             Analytics / BI / Reporting Tools
```

---

# 4. Architecture Decisions

## Medallion Architecture

The Medallion Architecture was selected because it provides a clear separation between ingestion, transformation, and publishing.

### Bronze Layer

Purpose:

* Preserve raw source data.
* Maintain ingestion history.
* Support reprocessing if business rules change.
* Avoid modifying original source records.

### Silver Layer

Purpose:

* Transform nested JSON into relational columns.
* Apply data quality validations.
* Standardize the dataset.
* Separate invalid records into a Quarantine layer.

### Gold Layer

Purpose:

* Publish trusted, analytics-ready data.
* Exclude invalid records.
* Provide a consistent schema for downstream consumers.

---

# 5. Technology Selection

| Technology       | Reason                               |
| ---------------- | ------------------------------------ |
| Azure Databricks | Distributed data processing platform |
| PySpark          | Scalable transformation engine       |
| Delta Lake       | Reliable ACID-compliant storage      |
| Unity Catalog    | Centralized governance and storage   |
| Python           | API integration and orchestration    |

---

# 6. Data Ingestion Strategy

The source system is a public ArcGIS REST API returning JSON data.

The Bronze notebook performs the following activities:

* Calls the REST API.
* Downloads all records.
* Generates a unique Run ID.
* Preserves the original JSON payload.
* Adds ingestion metadata.
* Stores data in Delta format.

No transformations are performed during ingestion to preserve the integrity of the source data.

---

# 7. Data Transformation Strategy

The Silver notebook transforms raw JSON into a structured dataset.

Key processing steps include:

* Reading the latest Bronze ingestion.
* Dynamically flattening nested JSON structures.
* Converting data types.
* Renaming columns to business-friendly names.
* Removing duplicate records.
* Applying data quality validation rules.
* Creating a Quarantine dataset for invalid records.

Dynamic JSON flattening was implemented to reduce dependency on the current API schema and improve maintainability if additional nested fields are introduced.

---

# 8. Data Quality Strategy

Two categories of validation were implemented.

## Critical Validation

Invalid geographic coordinates are considered critical.

Examples:

* Longitude outside -180 to 180
* Latitude outside -90 to 90
* Missing coordinates

Action:

* Record is moved to the Quarantine layer.
* Record is excluded from the Gold dataset.

---

## Warning Validation

Missing or unknown city values are treated as warnings.

Examples:

* NULL city
* Empty city
* "Unknown"

Action:

* Record remains in the dataset.
* Warning metadata is added.
* Record is published in Gold.

This approach prevents unnecessary data loss while still informing downstream consumers of potential quality issues.

---

# 9. Metadata Strategy

Metadata is captured throughout the pipeline to support traceability and auditing.

Captured metadata includes:

* Run ID
* Pipeline Name
* Source System
* Ingestion Timestamp
* Silver Processing Timestamp
* Gold Publishing Timestamp

This metadata allows each published record to be traced back to its original ingestion.

---

# 10. Storage Strategy

The solution uses Unity Catalog Managed Volumes.

Storage hierarchy:

```text
assignment
└── university
    ├── bronze
    ├── silver
    ├── gold
    └── quarantine
```

Each processing layer stores Delta tables independently.

This enables:

* Independent notebook execution
* Layer isolation
* Easy recovery
* Future orchestration

---

# 11. Notebook Independence

Each notebook has been designed to operate independently.

The pipeline follows this pattern:

```text
Bronze Notebook
        │
        ▼
Bronze Delta Storage
        │
        ▼
Silver Notebook
        │
        ▼
Silver Delta Storage
        │
        ▼
Gold Notebook
```

No notebook depends on variables created by another notebook.

Instead, each notebook reads the output generated by the previous layer.

This design aligns with enterprise ETL development practices.

---

# 12. Error Handling

The pipeline includes basic exception handling for:

* API connectivity failures
* Empty API responses
* Missing required JSON elements
* Invalid data
* Write failures

Meaningful log messages are generated to simplify troubleshooting.

---

# 13. Scalability Considerations

Although the assignment dataset is relatively small, the solution has been designed with scalability in mind.

Potential enterprise enhancements include:

* Incremental ingestion using watermarks.
* Change Data Capture (CDC).
* Schema evolution.
* Partitioned Delta tables.
* Delta Live Tables.
* Workflow orchestration.
* Automated retry logic.
* Monitoring and alerting.
* Data lineage.

---

# 14. Platform Considerations

This solution was developed using **Azure Databricks Free Edition (Serverless Compute)**.

The Free Edition does not support:

* Customer-managed ADLS Gen2 authentication using legacy Hadoop configurations (`fs.azure.account.*`).
* Unity Catalog External Volumes backed by customer-managed ADLS Gen2.
* SQL Warehouses.

Therefore, Unity Catalog Managed Volumes were used to implement the Bronze, Silver, Gold, and Quarantine layers.

The transformation logic remains identical to an enterprise deployment. Migrating to ADLS Gen2 in a production environment would primarily require storage configuration changes rather than modifications to the transformation logic.

---

# 15. Future Improvements

The following enhancements would further strengthen the solution in an enterprise environment:

* Incremental processing with watermarks.
* Change Data Capture (CDC).
* Databricks Workflows for orchestration.
* Delta Live Tables.
* Unity Catalog Lineage.
* Automated data quality monitoring using Soda or Great Expectations.
* CI/CD using Azure DevOps or GitHub Actions.
* Parameter-driven configuration files.
* Automated notifications and pipeline monitoring.

---

# 16. Conclusion

The implemented solution demonstrates an end-to-end data engineering pipeline that follows industry-standard architecture and engineering practices.

The solution separates ingestion, transformation, validation, and publishing into independent processing layers, ensuring maintainability, scalability, and traceability.

By combining Azure Databricks, PySpark, Delta Lake, and Unity Catalog Managed Volumes, the pipeline produces a governed, analytics-ready data product while remaining easily adaptable to enterprise Azure environments with minimal storage configuration changes.
