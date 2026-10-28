# Telecom Data Engineering Project — Medallion Architecture with CDC & SCD Type 2

The use of smartphones and other connected devices is increasing rapidly. Because of this, telecom companies are receiving huge amounts of data.

Telecom companies need to process, store, and analyze this data to get useful information. Big Data Analytics helps companies increase profits, improve network performance, provide better customer service, and improve security.

Big Data Analytics helps telecom companies in many ways. For example, it can predict when network usage will be high, so companies can reduce network congestion. It can also identify customers who may not pay their bills on time or who may switch to another telecom operator. This helps companies reduce customer loss, also called churn.

Instead of deciding a problem first and then searching for related data, companies should also explore the available data to find useful patterns and connections.

By analyzing data properly, telecom companies can improve their operations, make better decisions, and increase profits.

## Architecture

This project uses Medallion Architecture with 4 layers:

- **Bronze** — Raw data, as received from source, no changes
- **Silver** — Cleaned data (null checks, duplicate removal, bad records sent to quarantine)
- **Gold** — Business-ready data, created by joining Silver tables (e.g. subscriber + address + city + country)
-**Platinum** — Aggregated data for reporting. Created a revenue report showing total subscribers and total revenue per country, visualized as a bar chart in Databricks.

Tech Stack:-
- Python
- PySpark
- SQL
- Databricks
- Delta Lake
- azure


## Key Features

**CDC (Change Data Capture)**
When new or updated records come in, we don't reload the whole table — only the changed records are identified and merged. I implemented this two ways: manually using left_anti join + union, and using Delta Lake's MERGE INTO command.

**SCD Type 2 (Slowly Changing Dimension)**
When a subscriber's data changes (e.g. phone number), we don't overwrite the old record — we keep history. The old record is marked `is_current = false` with an `end_date`, and a new record is inserted as the current one with `is_current = true`. This way, both old and new data are preserved.

**Orchestration**
Used Databricks Jobs to chain Bronze → Silver → Gold notebooks with dependencies, so the entire pipeline runs automatically in the correct order. 

challenges faced: -
While merging the complaint table, the match condition was only on complaint ID. Since one complaint ID had two rows (old history + current), MERGE updated both instead of just the current one. I fixed it by adding is_current = true to the match condition.
I learned that append is not idempotent — running the same write operation again inserts duplicate rows, since it always adds data without checking what’s already there. MERGE, on the other hand, checks for matches first, so it’s safer to re-run.
I learned that before writing or reading any data, I need to check what format it’s actually saved in (Parquet vs Delta) and the correct path — assuming without checking causes a lot of confusing errors. I also learned that a Spark session restart clears all variables, so imports and dataframes need to be reloaded before continuing.
