# Irish Housing Pipeline

Housing is probably the biggest talking point in Ireland right now. I wanted a dataset that anyone could relate to without needing a data background — and property prices fit that perfectly. Every Irish person has an opinion on house prices.

## What I built

An end-to-end data pipeline on Databricks that ingests 785,912 Irish property transactions from 2010 to 2026, cleans and enriches the data through a medallion architecture, and surfaces it through an interactive dashboard.

The best part of building this was seeing the final outcome — the dashboard telling a clear story from raw messy CSV data. Also using PySpark in Python properly for the first time rather than just knowing it in theory.

## The story in the data

Dublin hit its lowest point in 2012 at a median of €202,643 — down from €267,500 in 2010. From there it never looked back. By 2026 the median is €474,000. That's a 134% rise from the bottom in 14 years.

**Dublin median prices year by year:**

| Year | Median Price | Total Sales |
|------|-------------|-------------|
| 2010 | €267,500 | 6,943 |
| 2012 | €202,643 | 8,926 |
| 2016 | €297,357 | 15,815 |
| 2020 | €354,737 | 14,787 |
| 2024 | €440,000 | 19,308 |
| 2026 | €474,000 | 19,216 |

Nationally, nearly half of all sales since 2010 were under €200k — mostly pre-2015 during the recovery. That band has almost disappeared today.

**National sales by price band:**

| Price Band | Transactions |
|-----------|-------------|
| Under €200k | 309,153 |
| €200k–€400k | 319,309 |
| €400k–€600k | 99,947 |
| Over €600k | 57,503 |

## Dashboard

The dashboard has three interactive charts:

- **Dublin Property Price Trends** — Property price trends for Dublin
- **County trends over time** — select any combination of counties and see how their prices diverged since 2010
- **County comparison by year** — select multiple years and compare every county side by side

📄 [View Interactive Dashboard](images/Dublin Property Price Trends.pdf)
📄 [View Interactive Dashboard](images/Property Prices by County Over Time.pdf)
📄 [View Interactive Dashboard](images/County Price Comparison by Year.pdf)


## How it's built

I used a medallion architecture — three layers, each with a clear purpose.

**Bronze** — load the raw CSV as-is, just fix the column names so Delta Lake can store it. Nothing else changes. If something goes wrong downstream you always have the raw data to go back to.

**Silver** — clean it up. Parse dates properly, strip the € symbol from prices, cast to the right types, drop nulls. This is where the data becomes usable.

**Gold** — add business context. Price bands, decade groupings, Dublin flag, new build flag. The Gold table is what you'd hand to an analyst or connect to a BI tool. Aggregations happen in the reporting layer, not here.

```
Raw CSV (785,912 records)
        │
        ▼
  Bronze (ppr_bronze)       — raw, column names fixed
        │
        ▼
  Silver (ppr_silver)       — cleaned, typed, filtered
        │
        ▼
  Gold (ppr_gold)           — enriched, ready for querying
```

The pipeline runs automatically via Databricks Workflows — Bronze, Silver and Gold execute in sequence with dependencies, so if Silver fails Gold won't run.

## Tech stack

- Databricks — notebook environment and compute
- PySpark — processing 785k records distributed
- Delta Lake — ACID storage with schema enforcement
- Databricks Workflows — pipeline orchestration
- Databricks Dashboards — interactive visualisation
- Unity Catalog — table management
- GitHub — version control

## What I learned

The PPR file uses CP1252 encoding — an old Windows format that breaks most standard CSV readers. Had to handle that explicitly in the ingestion layer.

Median rather than average price throughout — a handful of high-end sales would skew the average significantly and misrepresent what a typical buyer actually paid.

Coming from Azure Data Factory the Databricks learning curve was straightforward — the concepts translate well. The main difference is how much more flexibility you get with PySpark compared to ADF's drag and drop approach.

## What's next

The obvious next step is pulling in more datasets related to the Irish property market — planning permissions, rental prices, mortgage approvals — and joining them with the PPR data to tell a richer story about supply, demand and affordability.

## How to run

1. Clone this repo into Databricks as a Git folder
2. Download PPR-ALL.zip and upload to `/Volumes/workspace/default/ppr/`
3. Run notebooks in order: 01 → 02 → 03, or trigger the `ppr_pipeline` Workflow
4. Gold table lands in `workspace.default.ppr_gold`

## Data source

Property Price Register — published by the Property Services Regulatory Authority (PSRA) under the Property Services (Regulation) Act 2011. Free to use with attribution.

👉 [Download PPR-ALL.zip](https://propertypriceregister.ie/website/npsra/ppr/npsra-ppr.nsf/Downloads/PPR-ALL.zip/$FILE/PPR-ALL.zip)