diff --git a/README.md b/README.md
new file mode 100644
index 0000000000000000000000000000000000000000..25c7226739a949550c9dc56ea427840a7500e1d7
--- /dev/null
+++ b/README.md
@@ -0,0 +1,54 @@
+# Milan Olympics Medal Data Agent (Databricks)
+
+This project prepares a **daily medal snapshot** for the Milan Olympics, validates it, and stages it for Databricks Delta ingestion.
+
+It is designed as an **agentic AI-style workflow**:
+1. Plan the run (date/source/table).
+2. Fetch latest medal data.
+3. Validate medal records.
+4. Write curated daily snapshot locally.
+5. (In Databricks runtime) upsert into Delta table.
+
+## Repository layout
+
+- `src/medals_agent.py` – end-to-end agentic pipeline.
+- `config/pipeline_config.json` – configurable source + Databricks target.
+- `data/latest_medals_source.csv` – source medal feed (replace with official Milan feed).
+- `data/current_medal_snapshot.csv` – latest curated local snapshot.
+- `tests/test_validation.py` – unit tests.
+
+## Current data prepared
+
+The repository includes a prepared medal source dataset at `data/latest_medals_source.csv`, and the pipeline generates the current curated snapshot in `data/current_medal_snapshot.csv`.
+
+> Note: Milan Olympics medals are not final yet. Replace `source.medal_csv_url` with the official live Milan medal endpoint when available.
+
+## Run
+
+```bash
+python src/medals_agent.py --config config/pipeline_config.json --write-local
+```
+
+For backfill runs:
+
+```bash
+python src/medals_agent.py --config config/pipeline_config.json --write-local --as-of-date 2026-02-10
+```
+
+## Databricks migration pattern
+
+Use this project as a bronze/silver ingestion step:
+
+1. Schedule the script daily (Databricks Job, Airflow, or cron).
+2. Persist CSV snapshots to cloud storage (DBFS/S3/ADLS).
+3. In Databricks, load snapshot and `MERGE` into `main.olympics.milan_medal_snapshots` keyed by `(snapshot_date, country)`.
+4. Keep running daily through Olympics end date.
+
+## Agentic daily automation idea
+
+Implement a daily loop with 3 tools/roles:
+- **Data Fetch Agent**: reads official medal endpoint.
+- **Validation Agent**: checks schema + medal arithmetic + anomalies.
+- **Load Agent**: performs Databricks merge and posts run summary.
+
+This repository already implements fetch/validate/snapshot steps; wire the `DatabricksWriter.upsert` method in your Databricks runtime with `databricks-sql-connector` or Spark SQL merge.
