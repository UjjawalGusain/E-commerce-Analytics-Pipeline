# Job Scheduling and Pipeline Automation

This document outlines how the ETL pipeline in this project is scheduled and automated using Databricks Jobs.

---

## Scheduling Details

- Platform: Databricks
- Type: Time-based job
- Frequency: Daily
- Trigger: Scheduled via Databricks Jobs UI 
- Cluster: Configured with autoscaling and auto-termination
- Task: Executes a notebook that contains the ETL logic end-to-end

---

## Job Workflow Overview

### 1. Data Extraction

- Reads the raw CSV files from the Databricks Volumes:
  - `events.csv`
  - `item_properties_part1.csv`
  - `item_properties_part2.csv`
  - `category_tree.csv`
- Applies:
  - Schema inference
  - Null handling
  - Type casting (e.g., casting `timestamp` to BIGINT, `itemid` and `visitorid` to INT)

### 2. Transformation Logic

- Converts millisecond `timestamp` to a readable `event_date`
- Joins the raw `events_staging` data with dimensional tables:
  - `dim_event`
  - `dim_user`
  - `dim_item`

### 3. Incremental Loading

- Checks the last successfully loaded date from the metadata table:

  ```
  SELECT last_loaded_date FROM workspace.default.etl_metadata 
  WHERE table_name = 'fact_events'
  ```

- Filters the staging table:

  ```
  SELECT * FROM events_staging
  WHERE event_date > DATE('{last_loaded_date}')
  ```

- Transforms and loads only new data into `fact_events` table

### 4. Data Loading

- Uses `.write.format("delta")` and `.mode("append")` with partitioning:

  ```
  new_data.write.format("delta")
    .mode("append")
    .partitionBy("event_date")
    .saveAsTable("workspace.default.fact_events")
  ```

### 5. Metadata Update

- After successful write, updates the ETL metadata:

  ```
  DELETE FROM workspace.default.etl_metadata WHERE table_name = 'fact_events';

  INSERT INTO workspace.default.etl_metadata 
  VALUES ('fact_events', DATE('{max_event_date}'));
  ```

---

## Error Handling and Logging

- All steps are wrapped in a try-except block
- Logs every critical stage using `print()` statements
- Exits using `dbutils.notebook.exit()` to signal status to the Databricks Job:

  ```
  try:
      ...
      dbutils.notebook.exit("SUCCESS")
  except Exception as e:
      dbutils.notebook.exit(f"FAILURE: {str(e)}")
  ```

---


## Additional Notes

- Schema mismatches are prevented by maintaining alignment between transformed data and target table schema
- Partitioning by `event_date` supports efficient querying and storage
- Job failure notifications can be added by integrating with Databricks alerting or external systems
- Schema auto-merge is disabled due to ACLs; use `ALTER TABLE` to modify table schema if required
