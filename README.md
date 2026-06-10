# Customer Support Operations Analytics

End-to-end analytics project built on **Azure Databricks** (SQL) and visualized in **Power BI**, tracking SLA performance, resolution times, and transfer rates across regions and departments.

---

## Project Overview

This project simulates a real-world business intelligence pipeline where raw support ticket data is ingested into a Databricks data lake, transformed through a medallion architecture, and surfaced via an interactive Power BI dashboard.

**Core stack:** SQL · Azure Databricks · Power BI · DAX

---

## Architecture

```
raw CSVs (6 tables)
      ↓
raw schema (Databricks)        ← support_tickets, escalations, crm_tickets, helpdesk_tickets
      ↓
silver schema (Databricks)     ← agents (enriched)
      ↓
ref schema (Databricks)        ← region_config (SLA targets)
      ↓
gold schema (Databricks)       ← tickets_clean (transformed, analysis-ready)
      ↓
Power BI Dashboard             ← 5 visuals with cross-filtering and date slicer
```

---

## SQL Queries

| File | Description |
|------|-------------|
| `createdatabase_gold.sql` | Creates gold.tickets_clean via extraction and transformation |
| `extraction_transformation.sql` | Cleans raw data: type casting, string normalization, derived columns |
| `aggregation.sql` | SLA %, avg resolution time, transfer rate by region and department |
| `query_optimization.sql` | ZORDER, partition pruning, CTE-based rolling averages |
| `transfer_rate.sql` | Escalation rate across distributed sources with deduplication |
| `customer_support_analysis` | Full notebook combining all queries end to end |

---

## Key SQL Techniques Demonstrated

**Extraction & Transformation**
- Type casting, string normalization (`UPPER`, `TRIM`)
- Derived columns: `minutes_to_first_response`, `hours_to_resolution`, `is_open`
- Null filtering and data quality enforcement

**Aggregation**
- SLA compliance rate by region and month
- Rolling 3-month average using window functions (`ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`)
- Transfer/escalation rate with `DISTINCT` counting

**Query Optimization**
- Partition pruning on date columns
- `OPTIMIZE ... ZORDER BY` for co-locating frequently filtered columns
- CTE reuse to avoid redundant table scans

**Multi-Source Joins**
- Joining across 4 distributed tables (`tickets_clean`, `agents`, `region_config`, `escalations`)
- Case-insensitive joins with `UPPER(TRIM(...))` to handle inconsistent source formatting
- Deduplication using `ROW_NUMBER()` across `UNION ALL` of CRM and helpdesk sources

---

## Power BI Dashboard

**Visuals:**
- SLA Met % by Region (bar chart)
- Monthly/Quarterly SLA Trend by Region (line chart)
- Transfer Rate % by Department (bar chart)
- Avg Resolution Time by Region in hours (bar chart)
- Date Range Slicer (cross-filters all visuals)

**DAX Measures:**
```
SLA Met % =
DIVIDE(
    COUNTROWS(FILTER(tickets_clean, tickets_clean[minutes_to_first_response] <= 60)),
    COUNTROWS(tickets_clean)
) * 100

Transfer Rate % =
DIVIDE(
    DISTINCTCOUNT(escalations[ticket_id]),
    DISTINCTCOUNT(tickets_clean[ticket_id])
) * 100
```

---

## Data Model

- `gold.tickets_clean` → fact table (1,000 rows)
- `silver.agents` → dimension (30 agents)
- `ref.region_config` → dimension (5 regions with SLA targets)
- `raw.escalations` → fact (150 escalated tickets)

Relationships follow a star schema: `tickets_clean` at center, all other tables linked via `ticket_id`, `agent_id`, or `region`.

---

## Key Insights from the Data

- **NORTHEAST** leads SLA compliance (~37%); **SOUTHWEST** is the lowest performer (~25%) — flagged as a priority for operations review
- **RETURNS** department has the highest transfer rate — indicates potential routing or training gap
- **SOUTHWEST** also shows the highest average resolution time, corroborating the SLA findings

---

## How to Run

1. Download the CSVs from `/data` folder
2. Upload to Databricks Community Edition under `workspace` catalog
3. Run SQL files in order: `createdatabase_gold` → `extraction_transformation` → remaining queries
4. Open `Customer_Support_Dashboard.pbix` in Power BI Desktop
5. Update the Databricks connection with your own server hostname and HTTP path

---

## Author

**Rohan Kumar**  
