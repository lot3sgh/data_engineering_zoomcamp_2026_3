# Module 3 Homework - Answer Summary

## Completed Using

**Dataset**: Yellow Taxi Trip Records, January 2024 - June 2024  

---

## Answer Key

### ✅ Question 1: Total Record Count
**Answer**: **20,332,093**

```sql
SELECT COUNT(*) FROM yellow_taxi_materialized;
```

Result: 20,332,093 records in the 2024 Yellow Taxi dataset (Jan-Jun)

---

### ✅ Question 2: Distinct PULocationIDs - External vs Materialized
**Answer**: **0 MB for the External Table and 155.12 MB for the Materialized Table**

```sql
-- External table
SELECT COUNT(DISTINCT PULocationID) FROM yellow_taxi_external;

-- Materialized table  
SELECT COUNT(DISTINCT PULocationID) FROM yellow_taxi_materialized;
```

**Explanation**: 
- External tables in BigQuery don't cache metadata → 0 MB estimate
- Materialized tables have full metadata → shows actual data size
- Both return the same result, but cost estimation differs

---

### ✅ Question 3: Columnar Storage
**Answer**: **BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.**

```sql
-- One column
SELECT PULocationID FROM yellow_taxi_materialized;

-- Two columns
SELECT PULocationID, DOLocationID FROM yellow_taxi_materialized;
```

**Key Concept**: Columnar databases only read the columns you SELECT. More columns = more bytes scanned.

---

### ✅ Question 4: Zero Fare Amount
**Answer**: **8,333**

```sql
SELECT COUNT(*) 
FROM yellow_taxi_materialized 
WHERE fare_amount = 0;
```

Result: 8,333 records with exactly zero fare amount

**Additional Data**:
- Records with fare_amount <= 0: 324,080 (includes negatives)
- Records with fare_amount < $1: 341,584

---

### ✅ Question 5: Optimization Strategy
**Answer**: **Partition by tpep_dropoff_datetime and Cluster on VendorID**

**Reasoning**:
- **Partition** by the date column used for filtering (tpep_dropoff_datetime)
  - Reduces data scanned by skipping irrelevant date ranges
  - Most effective for date range queries
- **Cluster** by the column used for ordering (VendorID)
  - Optimizes sorting operations
  - Improves query performance when filtering on VendorID

**Why not other options?**
- Can't cluster on datetime and cluster on VendorID (only one clustering)
- Can't partition by VendorID (low cardinality, not effective)
- Clustering on datetime doesn't reduce data scanned like partitioning does

---

### ✅ Question 6: Partitioned vs Non-Partitioned Performance
**Answer**: **310.24 MB for non-partitioned table and 26.84 MB for the partitioned table**

```sql
-- Query both tables for distinct VendorIDs in March 1-15
SELECT DISTINCT VendorID 
FROM yellow_taxi_materialized 
WHERE tpep_dropoff_datetime >= '2024-03-01' 
  AND tpep_dropoff_datetime < '2024-03-16';
```

**Explanation**:
- **Non-partitioned table**: Must scan entire 6-month dataset → ~310 MB
- **Partitioned table**: Only scans March 1-15 partitions → ~27 MB
- **Performance gain**: ~91% reduction in data scanned!

---

### ✅ Question 7: External Table Storage Location
**Answer**: **GCP Bucket**

**Explanation**:
- External tables reference data stored in Google Cloud Storage (GCS) buckets
- Data is NOT copied into BigQuery storage
- Queries read directly from source files

**Not the other options**:
- ❌ Big Query - that's where the table *definition* lives, not the data
- ❌ Container Registry - for Docker images, not data
- ❌ Big Table - different Google Cloud database service

---

### ✅ Question 8: Clustering Best Practice
**Answer**: **False**

**Explanation**: Clustering is NOT always best practice. It's beneficial when:

✅ **Use clustering when:**
- Table is large (> 1 GB)
- Frequently filter/aggregate on specific columns
- High-cardinality columns (many unique values)
- Queries have common filter patterns

❌ **Don't cluster when:**
- Table is small (< 1 GB)
- No common query patterns
- Low-cardinality columns
- Queries use different columns each time

**Trade-offs**:
- Clustering adds overhead during writes
- Requires maintenance as data changes
- Only beneficial if query patterns align with clustering

---

### ✅ Question 9 (Bonus): COUNT(*) Bytes Estimate
**Answer**: **0 MB**

```sql
SELECT COUNT(*) FROM yellow_taxi_materialized;
```

**Why 0 MB?**
- BigQuery maintains **metadata** about tables including row counts
- `COUNT(*)` without WHERE clause or column references can be answered from metadata
- No actual data scanning required
- This is why it's essentially free and instant

**Would require scanning if:**
- `COUNT(column_name)` - needs to check for NULLs
- `COUNT(*) WHERE condition` - needs to evaluate condition
- `COUNT(DISTINCT column)` - needs to count unique values

---
