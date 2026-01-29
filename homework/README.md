# Module 3 Homework - Data Warehousing with DuckDB

This directory contains the solution to Module 3 homework using **DuckDB** instead of BigQuery.

## Files

- `module3_homework.ipynb` - Original Jupyter notebook with homework questions and solutions
- `module3_homework_executed.ipynb` - Executed notebook with outputs and results
- `module3_homework_executed.html` - HTML version of executed notebook for easy viewing
- `data/` - Directory containing Yellow Taxi trip data (January-June 2024)
- `homework.duckdb` - DuckDB database file created during execution

## Dataset

Yellow Taxi Trip Records for January 2024 - June 2024 (6 months)
- Source: NYC Taxi & Limousine Commission
- Format: Parquet files
- Total records: **20,332,093**

## How to Run

1. Make sure you have the dependencies installed:
   ```bash
   uv sync
   ```

2. Start Jupyter Notebook:
   ```bash
   uv run jupyter notebook
   ```

3. Open `module3_homework.ipynb` and run all cells

Or execute the notebook from command line:
```bash
uv run jupyter nbconvert --to notebook --execute homework/module3_homework.ipynb --output module3_homework_executed.ipynb
```

## Homework Answers

### Question 1: Total Record Count
**Answer**: **20,332,093**

Count of records for the 2024 Yellow Taxi Data (January-June).

### Question 2: Distinct PULocationIDs - Data Size Estimation
**Answer**: **0 MB for the External Table and 155.12 MB for the Materialized Table**

External tables don't cache metadata in BigQuery, showing 0 MB estimate. Materialized tables have full metadata.

### Question 3: Columnar Storage
**Answer**: **BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.**

Columnar databases only read the columns you request. Two columns = twice the data scanned.

### Question 4: Zero Fare Amount
**Answer**: **128,210**

Number of records with `fare_amount = 0`.

### Question 5: Optimization Strategy
**Answer**: **Partition by tpep_dropoff_datetime and Cluster on VendorID**

- Partition by the date column used for filtering (reduces data scanned)
- Cluster by the column used for ordering (optimizes sorting)

### Question 6: Partitioned vs Non-Partitioned Performance
**Answer**: **310.24 MB for non-partitioned table and 26.84 MB for the partitioned table**

Partitioning by date allows BigQuery to skip entire partitions outside the date range (March 1-15), dramatically reducing bytes scanned.

### Question 7: External Table Storage Location
**Answer**: **GCP Bucket**

External tables reference data stored in Google Cloud Storage buckets. The data is not copied into BigQuery.

### Question 8: Clustering Best Practice
**Answer**: **False**

Clustering is not always beneficial. It's most useful for:
- Large tables (> 1 GB)
- Frequent filtering/aggregation on specific columns
- High-cardinality columns

For small tables or without common filter patterns, clustering adds overhead without benefit.

### Question 9 (Bonus): COUNT(*) Bytes Estimate
**Answer**: **0 MB**

BigQuery maintains metadata about tables including row counts. A simple `COUNT(*)` without WHERE clauses can be answered from metadata without scanning data.

## Key Concepts Demonstrated

### 1. External vs Materialized Tables
- **External**: Data stays in source (GCS/Parquet files), read on-demand
- **Materialized**: Data loaded into database storage for faster access

### 2. Columnar Storage
Both BigQuery and DuckDB use columnar storage:
- Only requested columns are scanned
- Efficient for analytical queries on wide tables
- Query cost proportional to columns selected

### 3. Partitioning
Dividing large tables into smaller segments based on a column (usually date):
- Reduces data scanned by skipping irrelevant partitions
- Significantly lowers query costs
- Best for time-series data with date range filters

### 4. Clustering
Organizing data within partitions based on column values:
- Optimizes queries that filter/sort on those columns
- Works well with high-cardinality columns
- Complements partitioning

### 5. Query Optimization
- Use `SELECT column_list` instead of `SELECT *`
- Filter early with WHERE clauses
- Partition on commonly filtered date columns
- Cluster on frequently filtered/sorted columns
- Check estimated bytes before running expensive queries

## DuckDB vs BigQuery

While this homework was designed for BigQuery, DuckDB provides similar analytical capabilities:

| Feature | BigQuery | DuckDB |
|---------|----------|---------|
| Storage | Cloud (GCS) | Local/Embedded |
| External Tables | ✅ GCS | ✅ Parquet files |
| Columnar Storage | ✅ | ✅ |
| Partitioning | ✅ Native | ⚠️ Via file organization |
| Clustering | ✅ Native | ⚠️ Via sorting |
| Cost Estimates | ✅ Before execution | ❌ N/A (local) |
| Performance | Cloud-scale | Excellent for local data |

DuckDB excels at:
- Local analytics on Parquet/CSV files
- Embedded analytical workloads
- Data processing without cloud dependencies
- Development and testing

## Additional Analysis

The notebook includes extra queries showing:
- Dataset statistics (trip counts, fare averages, etc.)
- Monthly trip distribution
- Pickup/dropoff location diversity
- Data quality checks

## Technologies Used

- **DuckDB** 1.4.4+ - In-process analytical database
- **Jupyter** - Interactive notebook environment
- **Pandas** - Data manipulation and display
- **Parquet** - Columnar storage format
