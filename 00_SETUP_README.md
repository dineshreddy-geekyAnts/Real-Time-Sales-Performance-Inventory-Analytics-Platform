# 🚀 Real-Time Sales Performance & Inventory Analytics Platform

## 📋 Project Overview

This project implements a **Medallion Architecture (Bronze → Silver → Gold)** data pipeline using **Databricks Serverless Compute** to transform raw AdventureWorks operational data into business-ready analytics.

### Business Problem Solved
- **Unified Analytics**: Consolidate Sales, Production, Purchasing, Person, and HR data
- **Real-Time Insights**: Enable near real-time reporting with automated pipeline orchestration
- **Data Quality**: Implement cleansing, validation, and standardization in Silver layer
- **Performance**: Pre-aggregated Gold tables for sub-second dashboard queries
- **Cost Efficiency**: Serverless compute scales automatically and charges only for active time

---

## 🏗️ Architecture: Medallion Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│                 SOURCE: adventureworks_live_conn_catalog        │
│  Sales │ Production │ Purchasing │ Person │ HumanResources     │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  🥉 BRONZE LAYER (workspace.adventureworks_bronze)              │
│  • Raw data ingestion with audit columns                        │
│  • Full snapshot loads                                          │
│  • 70+ tables with lineage tracking                             │
│  • Minimal transformation                                       │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  🥈 SILVER LAYER (workspace.adventureworks_silver)              │
│  • Data quality & deduplication                                 │
│  • Business rules applied                                       │
│  • SCD Type 2 for slowly changing dimensions                    │
│  • Conformed dimensions: Customer, Product, Employee            │
│  • Fact tables: Sales Orders, Inventory                         │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│  🥇 GOLD LAYER (workspace.adventureworks_gold)                  │
│  • Business-level aggregates                                    │
│  • Pre-calculated metrics & KPIs                                │
│  • Optimized for BI dashboards                                  │
│  • Tables: Sales Summary, Inventory Status, Customer 360        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📁 File Structure

```
Real-Time-Sales-Performance-Inventory-Analytics-Platform/
├── 00_SETUP_README.md                    ← You are here!
├── 01_Setup_Schemas                      ← Create Bronze/Silver/Gold schemas
│
├── 02_Bronze_Sales_Ingestion             ← Ingest Sales tables (19 tables)
├── 02_Bronze_Production_Ingestion        ← Ingest Production tables (25 tables)
├── 02_Bronze_Purchasing_Ingestion        ← Ingest Purchasing tables (5 tables)
├── 02_Bronze_Person_Ingestion            ← Ingest Person tables (13 tables)
├── 02_Bronze_HumanResources_Ingestion    ← Ingest HR tables (6 tables)
│
├── 03_Silver_Dimensions                  ← Create dimension tables (SCD Type 2)
├── 03_Silver_Facts                       ← Create fact tables
│
├── 04_Gold_Sales_Analytics               ← Sales KPIs & metrics
├── 04_Gold_Inventory_Analytics           ← Inventory metrics
├── 04_Gold_Customer_360                  ← Customer analytics
│
└── 05_Orchestration_Workflow             ← Master pipeline orchestration
```

---

## ✅ Prerequisites

### Required Access
- ✅ Access to `adventureworks_live_conn_catalog` (source data)
- ✅ CREATE SCHEMA permission on `workspace` catalog
- ✅ Databricks Serverless SQL Warehouse or Serverless Cluster

### Source Data Summary
| Schema | Tables | Total Rows | Key Tables |
|--------|--------|------------|------------|
| Sales | 19 | ~250K | Customer, SalesOrderHeader, SalesOrderDetail |
| Production | 25 | ~350K | Product, WorkOrder, TransactionHistory |
| Purchasing | 5 | ~13K | Vendor, PurchaseOrderHeader |
| Person | 13 | ~140K | Person, Address, EmailAddress |
| HumanResources | 6 | ~934 | Employee, Department |

**Total**: 68 tables, ~750K rows

---

## 🚀 Quick Start Guide

### Step 1: Create Schemas (2 minutes)
```bash
Run: 01_Setup_Schemas
```
**What it does**: Creates three schemas in the workspace catalog:
- `workspace.adventureworks_bronze`
- `workspace.adventureworks_silver`
- `workspace.adventureworks_gold`

**Expected Output**: 3 schemas created with comments

---

### Step 2: Ingest Bronze Layer (5-10 minutes)
Run all Bronze ingestion notebooks **in parallel** (they don't depend on each other):

```bash
Run in parallel:
├── 02_Bronze_Sales_Ingestion          (~3 min)
├── 02_Bronze_Production_Ingestion     (~4 min)
├── 02_Bronze_Purchasing_Ingestion     (~1 min)
├── 02_Bronze_Person_Ingestion         (~2 min)
└── 02_Bronze_HumanResources_Ingestion (~1 min)
```

**What it does**: 
- Copies all source tables to Bronze layer
- Adds audit columns: `bronze_load_timestamp`, `bronze_source_system`, `bronze_source_table`
- Uses full snapshot (OVERWRITE mode)

**Expected Output**: ~70 Bronze tables with 750K+ total rows

---

### Step 3: Build Silver Layer (3-5 minutes)
Run Silver notebooks **in sequence**:

```bash
Step 3a: Run 03_Silver_Dimensions  (creates dim_customer, dim_product, dim_employee)
Step 3b: Run 03_Silver_Facts       (creates fact_sales_order, fact_inventory)
```

**What it does**:
- **Dimensions**: Deduplication, SCD Type 2 tracking, business key standardization
- **Facts**: Join dimension keys, data quality checks, metric calculations

**Expected Output**: 
- 3 dimension tables (with surrogate keys and valid_from/valid_to dates)
- 2 fact tables (denormalized for query performance)

---

### Step 4: Create Gold Layer (2-3 minutes)
Run Gold notebooks **in parallel**:

```bash
Run in parallel:
├── 04_Gold_Sales_Analytics
├── 04_Gold_Inventory_Analytics
└── 04_Gold_Customer_360
```

**What it does**: Creates pre-aggregated business metrics:
- **Sales Analytics**: Daily sales summary by territory/product/salesperson
- **Inventory Analytics**: Current stock levels, turnover rates, reorder alerts
- **Customer 360**: Lifetime value, purchase frequency, product affinity

**Expected Output**: 3 Gold tables with aggregated metrics, updated daily

---

### Step 5: Automate with Orchestration (Optional)
```bash
Run: 05_Orchestration_Workflow
```

**What it does**: 
- Chains all notebooks together with dependencies
- Adds error handling and retry logic
- Can be scheduled as a Databricks Job (daily/hourly)

**Expected Output**: Full pipeline execution Bronze → Silver → Gold in ~15 minutes

---

## 📊 Expected Results

### Bronze Layer
```sql
-- Verify Bronze tables
SHOW TABLES IN workspace.adventureworks_bronze;
-- Expected: ~70 tables named bronze_{schema}_{table}

-- Check row counts
SELECT COUNT(*) FROM workspace.adventureworks_bronze.bronze_sales_salesorderdetail;
-- Expected: 121,317 rows
```

### Silver Layer
```sql
-- Check dimension tables
SELECT COUNT(*) FROM workspace.adventureworks_silver.dim_customer;
-- Expected: 19,820 customers with surrogate keys

-- Check fact tables
SELECT COUNT(*) FROM workspace.adventureworks_silver.fact_sales_order;
-- Expected: 121,317 order line items with dimension keys
```

### Gold Layer
```sql
-- Sales summary by territory
SELECT * FROM workspace.adventureworks_gold.sales_daily_summary
ORDER BY order_date DESC, total_revenue DESC
LIMIT 10;
-- Expected: Daily aggregates with revenue, order count, avg order value

-- Top customers
SELECT * FROM workspace.adventureworks_gold.customer_360
ORDER BY lifetime_value DESC
LIMIT 10;
-- Expected: Customer metrics with LTV, purchase frequency, recency
```

---

## 🎯 Key Features

### Data Quality
- ✅ Null value handling
- ✅ Duplicate detection & removal
- ✅ Data type validation
- ✅ Referential integrity checks
- ✅ Audit trail with load timestamps

### Performance Optimization
- ✅ Pre-aggregated Gold tables
- ✅ Partitioning on date columns
- ✅ Z-ordering on frequently filtered columns
- ✅ Serverless auto-scaling

### Slowly Changing Dimensions (SCD Type 2)
- ✅ Tracks historical changes in Customer, Product, Employee
- ✅ `valid_from` and `valid_to` date tracking
- ✅ `is_current` flag for latest records
- ✅ Surrogate keys for fact table joins

---

## 🔧 Configuration

### Modify Source/Target Catalogs
Edit the configuration in each notebook:
```python
# Change these values in each Bronze notebook
source_catalog = "adventureworks_live_conn_catalog"  # Source
target_schema = "workspace.adventureworks_bronze"    # Target
```

### Change Refresh Frequency
- **Bronze**: Daily full refresh (can change to incremental with Change Data Feed)
- **Silver**: Daily refresh with SCD Type 2 merge
- **Gold**: Hourly refresh for near real-time dashboards

---

## 📈 Performance Metrics

### Execution Time (Serverless Compute)
| Layer | Execution Time | Tables Created | Rows Processed |
|-------|---------------|----------------|----------------|
| Bronze | 5-10 min | 70 | 750K |
| Silver | 3-5 min | 5 | 500K |
| Gold | 2-3 min | 3 | Pre-aggregated |
| **Total** | **~15 min** | **78** | **750K+** |

### Cost Estimate (Serverless SQL Warehouse - Pro)
- **Daily Run**: ~$0.50 - $1.00 per day
- **Hourly Run**: ~$5 - $10 per day
- **Monthly**: ~$15 - $30 (daily) or ~$150 - $300 (hourly)

---

## 🐛 Troubleshooting

### Issue: Schema creation fails
```
Error: Permission denied to create schema
```
**Solution**: Request CREATE SCHEMA permission on `workspace` catalog from admin

### Issue: Source tables not found
```
Error: Table adventureworks_live_conn_catalog.Sales.Customer not found
```
**Solution**: Verify connection to source catalog and table names

### Issue: Bronze ingestion timeout
```
Error: Command execution timed out
```
**Solution**: Increase warehouse size or split large tables into separate notebooks

### Issue: SCD Type 2 merge fails
```
Error: Duplicate records found
```
**Solution**: Check for duplicate business keys in source data, add deduplication logic

---

## 📚 Additional Resources

### Databricks Documentation
* [Medallion Architecture](https://docs.databricks.com/lakehouse/medallion.html)
* [Serverless SQL Warehouses](https://docs.databricks.com/sql/admin/serverless.html)
* [Delta Lake SCD Type 2](https://docs.databricks.com/delta/tutorial.html)

### Code Patterns
* Bronze: Full snapshot with audit columns
* Silver: MERGE with SCD Type 2 logic
* Gold: CREATE OR REPLACE TABLE with aggregations

---

## 🎉 Next Steps

1. ✅ Run all notebooks in sequence (or use orchestration)
2. ✅ Create Databricks SQL dashboards on Gold tables
3. ✅ Schedule daily refresh via Databricks Jobs
4. ✅ Set up data quality alerts on key metrics
5. ✅ Implement incremental loads for Bronze layer (CDC)

---

## 📞 Support

For issues or questions:
* Check notebook comments for inline documentation
* Review execution logs for error details
* Verify source data quality and completeness

---

**Created**: 2024
**Architecture**: Medallion (Bronze → Silver → Gold)
**Compute**: Databricks Serverless
**Data Source**: AdventureWorks Live Connection Catalog
**Target**: Workspace Unity Catalog

---

## 🔄 Execution Summary

```bash
# Complete pipeline execution order:

# STEP 1: Setup (Required)
01_Setup_Schemas

# STEP 2: Bronze Layer (Parallel execution OK)
02_Bronze_Sales_Ingestion
02_Bronze_Production_Ingestion
02_Bronze_Purchasing_Ingestion
02_Bronze_Person_Ingestion
02_Bronze_HumanResources_Ingestion

# STEP 3: Silver Layer (Sequential - Dimensions first)
03_Silver_Dimensions
03_Silver_Facts

# STEP 4: Gold Layer (Parallel execution OK)
04_Gold_Sales_Analytics
04_Gold_Inventory_Analytics
04_Gold_Customer_360

# STEP 5: Orchestration (Optional - automates all above)
05_Orchestration_Workflow
```

**Total Time**: ~15-20 minutes for complete pipeline
**Result**: Production-ready analytics platform with 78 tables across 3 layers

---

✨ **Ready to start? Run `01_Setup_Schemas` now!** ✨
