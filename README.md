# E-commerce Analytics Pipeline

<div align="center">
  <img src="https://github.com/user-attachments/assets/ec173501-09e6-4319-846e-04586fa843d9" alt="unigit_src_image" width="512" height="512" />
</div>

This project demonstrates the development of a scalable and automated data engineering pipeline using PySpark and Delta Lake on Databricks. The pipeline processes raw e-commerce event logs and transforms them into structured dimensional and fact tables suitable for analytical queries and business intelligence reporting.

---

## Project Structure

- **Notebook:** `etl_pipeline.ipynb` contains the end-to-end logic for staging, transformation, loading, and scheduling.
- **Delta Tables:**
  - `dim_user`
  - `dim_item`
  - `dim_event`
  - `dim_category`
  - `fact_events`
  - `etl_metadata` (tracks last load date for incremental loading)
- **Documentation:**
  - `job_scheduling.md` – Details about how ETL is scheduled and monitored

---

## Key Features

- Handles raw input in CSV format from Databricks Volumes
- Cleans and infers schema using PySpark
- Creates star schema (dimension and fact tables)
- Supports incremental data loading using `etl_metadata` table
- Daily job automation using Databricks Jobs
- Supports analytical queries like active users, most viewed items, daily trends, etc.

---

## Sample Queries

### Most Viewed Item

```
SELECT item.itemid AS item_id, COUNT(*) AS viewCount
FROM workspace.default.fact_events f
JOIN workspace.default.dim_event ev ON f.event_sk = ev.event_sk AND ev.event = 'view'
JOIN workspace.default.dim_item item ON item.item_sk = f.item_sk
GROUP BY item.itemid
ORDER BY viewCount DESC
LIMIT 1;
```

### Active Users Per Day

```
SELECT event_date, COUNT(DISTINCT user_sk) AS active_users
FROM workspace.default.fact_events
GROUP BY event_date
ORDER BY event_date;
```

---

## Technologies Used

- Apache Spark with PySpark
- Databricks (for notebook execution and job scheduling)
- Delta Lake (for ACID-compliant tables)
- SQL (for transformation and querying)
- Databricks Volumes (for raw data storage)

---

## Dataset

The dataset is based on a real-world e-commerce event log and includes:

- Event logs (`events.csv`)
- Item properties (`item_properties_part1.csv`, `item_properties_part2.csv`)
- Category hierarchy (`category_tree.csv`)

The raw files are stored in:

```
/Volumes/workspace/default/my_volume/
```

---

## Getting Started

1. Upload raw CSV files to your Databricks Volume.
2. Open the notebook `etl_pipeline.ipynb`.
3. Execute each cell sequentially to:
   - Ingest raw data into staging views
   - Build dimensional and fact tables
   - Perform initial and incremental data loads
4. Configure a Databricks Job to run this notebook on a daily schedule.

---

## Automation

Refer to `job_scheduling.md` for detailed explanation of job setup, triggers, logging, and failure handling.

---

## Project Goals

- Simulate a production-ready data engineering pipeline
- Understand scheduling, logging, and metadata tracking
- Provide a foundation for scalable analytics on e-commerce data

---

