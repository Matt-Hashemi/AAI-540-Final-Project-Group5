# SDWA Week 3 — Ingestion, Validation, and PWS Master Table Construction

This notebook implements the full Week‑3 pipeline for the SDWA project: ingestion of
all EPA SDWA datasets into S3 as Parquet, registration of Athena tables, schema
validation, joinability checks, feature aggregation, and construction of the unified
PWS‑level master table. It concludes by splitting the master dataset into
train/val/test/prod partitions and registering each split in Athena.

---

## 1. Environment & Configuration

The notebook initializes the SageMaker Studio environment and defines all shared
configuration:

- AWS region and boto3 session  
- S3 bucket and prefixes for raw CSVs and Parquet outputs  
- Athena/Glue database (`sdwa_db`)  
- Mapping of logical SDWA table names → raw CSV filenames  

These settings ensure all downstream ingestion and Athena operations are consistent
and reproducible.

---

## 2. Ingestion Pipeline (Normal + Streaming)

The ingestion workflow supports two modes:

### **Normal ingestion**
Used for small/medium CSVs.  
Steps:
1. Read entire CSV from S3  
2. Force all columns to `STRING`  
3. Write Parquet dataset to S3  

### **Streaming ingestion**
Used for large CSVs (e.g., 3–4 GB violations file).  
Steps:
1. Stream CSV in chunks  
2. Convert each chunk to `STRING`  
3. Append Parquet chunks to S3  

All ingestion paths guarantee:
- Schema stability  
- Deterministic output  
- Athena‑friendly Parquet datasets  

---

## 3. Athena Table Registration (STRING‑only schema)

Each Parquet dataset is registered as an Athena table using a helper function that:

1. Reads a sample Parquet file  
2. Extracts column names  
3. Maps **every column to STRING**  
4. Creates/overwrites the Athena table  

This ensures all SDWA tables share a consistent schema, preventing downstream type
mismatches.

---

## 4. Schema Sanity Checks

For each SDWA table, the notebook:
- Reads a small Parquet chunk  
- Prints column names and dtypes  
- Confirms all columns are `object/string`  

This validates ingestion correctness and Glue/Athena compatibility.

---

## 5. Joinability Checks

The notebook performs joinability checks between:

- `pws` ↔ `violations`  
- `pws` ↔ `lcr`  
- `pws` ↔ `service_areas`  

Each check reports:
- Total rows in the left table  
- Number of rows with matching keys in the right table  

This ensures relational integrity before building the master table.

---

## 6. **Construction of the PWS Master Table**

The master table is built using Athena SQL with four CTEs:

### **1. `viol_agg`**
Aggregates violations per PWSID:
- `violation_count`
- `enforcement_actions`

### **2. `lcr_agg`**
Aggregates LCR sampling:
- `lcr_samples`
- `avg_lcr_result` (numeric conversion applied)

### **3. `service_agg`**
Summaries of service area metadata:
- `service_area_count`
- `primary_service_area_type`

### **4. `geo_agg`**
Geographic attributes:
- `state`
- `county`
- `city`
- `area_type`

### **Final Join**
The master table is created by LEFT JOINing all aggregates onto the `pws` table:

pws
LEFT JOIN viol_agg
LEFT JOIN lcr_agg
LEFT JOIN service_agg
LEFT JOIN geo_agg


This produces a unified, feature‑rich PWS‑level dataset suitable for modeling.

---

## 7. Cleanup & Type Normalization

The notebook:
- Drops duplicates  
- Converts numeric‑like columns to numeric (`to_numeric(errors="coerce")`)  
- Fills missing values only in string columns  

This ensures modeling‑ready consistency.

---

## 8. Train/Val/Test/Prod Split (40/10/10/40)

The master table is shuffled and split into:

- **40%** train  
- **10%** validation  
- **10%** test  
- **40%** production holdout  

Each split is written to S3 as Parquet.

---

## 9. Athena Registration of Splits

Each split is registered as its own Athena table:

- `pws_master_train`
- `pws_master_val`
- `pws_master_test`
- `pws_master_prod`

All tables use STRING‑only schemas for consistency.

---

## 10. Row Count Verification

The notebook issues `COUNT(*)` queries to confirm that Athena table row counts
match the in‑memory DataFrame splits.

---

## Summary

By the end of Week 3, the notebook delivers:

- Fully ingested SDWA datasets in Parquet format  
- Athena tables for all raw and split datasets  
- Schema validation and joinability checks  
- A unified, feature‑rich **PWS master table**  
- Train/val/test/prod splits ready for Week‑4 modeling  

This README documents the full Week‑3 pipeline and its outputs.
