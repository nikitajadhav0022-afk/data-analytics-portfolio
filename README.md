
# Supply-Demand-PowerBI-Dashboard

### SQl Server Dashboard Link : https://app.powerbi.com/links/UlrpURFuhC?ctid=91577f61-0170-494b-a390-0f33a2bb8755&pbi_source=linkShare&bookmarkGuid=77504715-b383-48ed-8e4f-9613392dccb1

### MYSQL Dashboard Link : https://app.powerbi.com/links/TMghX1saJ1?ctid=91577f61-0170-494b-a390-0f33a2bb8755&pbi_source=linkShare

## ## Dashboard Walkthrough
[Watch the Dashboard Demo on YouTube](https://youtu.be/sUJrro1DbGo)

## Problem Statement

This dashboard helps businesses monitor and analyze product supply and demand trends across their inventory. It enables analysts to identify supply shortages, track profitability, and understand daily demand-availability patterns. Through KPI cards and interactive filters, stakeholders can quickly spot fulfillment gaps and financial impact — and take corrective action.

Since supply shortages directly result in losses, this dashboard helps teams proactively manage inventory by highlighting products and dates where availability falls short of demand.

---

## Project Scenarios Covered

**Scenario 1:** Transitioning a Power BI report from a **Test environment to a Production environment** using SQL Server as the data source — without recreating the report.

**Scenario 2:** Migrating the Power BI report's data source from **SQL Server to MySQL database** — again without recreating the report, by modifying Power Query Editor settings.

---

## Data Sources

- **Microsoft SQL Server** (Test & Production environments)
- **MySQL Database** (Post-migration)

### Tables Used

| Table | Description |
|---|---|
| DA Table | Order Date, Product ID, Availability (A), Demand (D) |
| Products Table | Product ID, Product Name, Unit Price |

### Supply-Demand Logic
- `A = D` → Demand fully fulfilled
- `A > D` → Demand fully fulfilled (surplus)
- `A < D` → Demand **not** fulfilled (shortage)

---

## Steps Followed

### Data Preparation — SQL Server (Test Environment)

- **Step 1:** Imported DA table and Products table into SQL Server (test env). Checked for nulls and blanks — none found.
- **Step 2:** Performed a **LEFT JOIN** on both tables and inserted the combined result into a new table using a subquery:

```sql
select * into New_table from 
(select a.[Order_Date_DD_MM_YYYY], a.Product_ID, a.[Availability], a.[Demand],
 b.[Product_Name], b.[Unit_Price] 
from [dbo].[DA_Table] a 
left join [dbo].[Products] b ON a.Product_ID = b.Product_ID) X
```

### Power BI Report Creation (Test Environment)

- **Step 3:** Connected Power BI to SQL Server using **Import mode** with a `SELECT * FROM New_table` query.
- **Step 4:** Transformed data in Power Query Editor and verified data types.
- **Step 5:** Created **6 DAX measures** for Page 1 KPIs and **3 DAX measures** for Page 2 KPIs.
- **Step 6:** Built card visuals for all KPIs and added **Date** and **Product** filters via the Filters Pane.

---

### KPIs — Page 1

| KPI | Description |
|---|---|
| Average Demand Per Day | Mean daily demand across all products |
| Average Availability Per Day | Mean daily availability across all products |
| Total Supply Shortage | Sum of shortfall where Availability < Demand |

### KPIs — Page 2

| KPI | Description |
|---|---|
| Total Profit | Revenue earned from fulfilled demand |
| Total Loss | Revenue lost due to unmet demand |
| Average Daily Loss | Mean daily loss across the date range |

---

## Report Snapshot (Power BI Desktop)

### Page 1 — Supply & Demand KPIs

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/238170c5-90b3-40d2-9c3c-c4276847d4a8"/>


*Page 1 showing Average Demand Per Day (48.65), Average Availability Per Day (24.70), and Total Supply Shortage (61K)*

### Page 2 — Financial KPIs

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/8d7f5aa4-c569-4472-a1a3-28b5579ce1d1" />

*Page 2 showing Total Profit (301K), Total Loss (8M), and Average Daily Loss (2.97K)*

---

### Data Preparation — SQL Server (Production Environment)

- **Step 7:** Imported production data — **1,043 records** (vs. 99 in test). Verified nulls/blanks — none found.
- **Step 8:** Identified a **data quality issue**: 22 distinct `Product_ID` values in the inventory dataset, but only 20 in the Products table.
  - Root cause: Product IDs `21` and `22` were duplicates of `7` and `11` respectively (confirmed by data engineering team).
  - Raised the issue with the data engineering team and managers via email.
  - Applied fixes directly in SQL:

```sql
update [dbo].[Prod+Env+Inventory+Dataset] 
set Product_ID = 7 where Product_ID = 21

update [dbo].[Prod+Env+Inventory+Dataset] 
set Product_ID = 11 where Product_ID = 22
```

- **Step 9:** Verified 20 distinct Product IDs post-update. Created `New_table` in production using the same LEFT JOIN subquery with **identical column/table names** as test env.

### Transitioning Report — Test to Production (SQL Server)

- **Step 10:** In Power BI Desktop → `Home` → `Transform Data` → `Data Source Settings` → **Changed database name** from test to production.
- **Step 11:** Re-ran the same `SELECT * FROM New_table` query. Report refreshed automatically — KPIs updated on both pages. ✅

---

## Snapshot of Dashboard (Power BI Service — SQL Server Workspace)

### Page 1

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/d281e21e-8aa6-4385-a752-156d6ca7c611" />

### Page 2

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/843d1f1e-8069-4198-b2d6-9cfd18cb1126" />

---

### Migrating Data Source — SQL Server to MySQL

- **Step 12:** Installed **MySQL .NET Connector** to enable Power BI–MySQL connectivity.
- **Step 13:** Imported both tables into MySQL using the **Table Data Import Wizard** in MySQL Workbench.
- **Step 14:** Re-applied product ID fixes in MySQL:

```sql
update prod.`prod+env+inventory+dataset`
set `Product ID` = 7 where `Product ID` = 21;

update prod.`prod+env+inventory+dataset`
set `Product ID` = 11 where `Product ID` = 22;
```

- **Step 15:** Created `New_table` in MySQL with **matching column names** (aliased to match SQL Server naming):

```sql
create table New_table as
select a.`Order Date (DD/MM/YYYY)` as `Order_Date_DD_MM_YYYY`, 
a.`Product ID` as `Product_ID`,
a.`Availability`, a.`Demand`,
b.`Product Name` as `Product_Name`,
b.`Unit Price ($)` as `Unit_Price` 
from prod.`prod+env+inventory+dataset` a 
left join prod.`products+(1)` b ON a.`Product ID` = b.`Product ID`
```

- **Step 16:** Published the SQL Server report to Power BI Service under a dedicated workspace before migration.

- **Step 17:** In Power BI Desktop, used **Advanced Editor** in Power Query to change the data source connection from SQL Server to MySQL:

### Advanced Editor — MySQL Source Connection

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/dac4c7e0-5bdc-4ecc-be44-0512ca12c7ca" />

*Before (SQL Server):*
```
Source = Sql.Database("DHRUVI", "prod", [Query="select * from New_table"])
```
*After (MySQL):*
```
Source = MySQL.Database("localhost", "PROD", [ReturnSingleDatabase=true, Query="SELECT * FROM New_table;"])
```

- **Step 18:** Validated KPI values matched between SQL Server and MySQL reports. ✅ Published MySQL-connected report to a separate Power BI Service workspace.

### Data Source Settings — Changed to MySQL

<img width="1905" height="995" alt="Image" src="https://github.com/user-attachments/assets/f5a5f515-19eb-4b23-8e34-2bd912593aea" />

---

## Snapshot of Dashboard (Power BI Service — MySQL Workspace)

### Page 1

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/14eea498-fd42-4a65-92bb-d732c84de2f3" />

### Page 2

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/e83aae6f-14e0-4552-8615-744c91707462" />

---

## DAX Measures

All 9 DAX measures were created and stored in a separate **Measures Table** for clean report organization.

### Page 1 Measures (6)

**Average Availability Per Day**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/f5921922-4978-485a-9dec-2e4d49231a72" />

```dax
Average Availability Per Day = DIVIDE([Total Availability],[Total number of Days])
```

**Average Demand Per Day**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/ffd0676a-5582-48ed-a6e1-5dfa985e2e8b" />

```dax
Average Demand Per Day = DIVIDE([Total demand],[Total number of Days])
```

**Total Supply Shortage**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/66d68f4c-dcfc-475f-834b-3f035975ebf3" />

```dax
Total Supply Shortage = [Total demand]-[Total Availability]
```

**Total Availability**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/6b686eaf-156a-4f95-a26a-fc2af833b11f" />

```dax
Total Availability = SUM('Demand/Availability Data'[Availability])
```

**Total demand**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/08555b85-270a-4227-b7bb-f6b1ebe3719c" />

```dax
Total demand = SUM('Demand/Availability Data'[Demand])
```

**Total number of Days**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/083ae75c-c012-409c-a1d4-a48948043f61" />

```dax
Total number of Days = DISTINCTCOUNT('Demand/Availability Data'[Order_Date_DD_MM_YYYY].[Date])
```

### Page 2 Measures (3)

**Total Profit**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/d99b3e78-e761-497a-9187-5f77f867736d" />

```dax
Total Profit = SUMX(FILTER('Demand/Availability Data','Demand/Availability Data'[Profit/Loss]>0),
'Demand/Availability Data'[Profit/Loss]*'Demand/Availability Data'[Unit_Price])
```

**Total Loss**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/9b925ec3-b399-46ef-b18c-88a494aab6ad" />

```dax
Total Loss = SUMX(FILTER('Demand/Availability Data','Demand/Availability Data'[Profit/Loss]<0),
'Demand/Availability Data'[Profit/Loss]*'Demand/Availability Data'[Unit_Price]) *-1
```

**Average Daily Loss**

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/71f42776-9d4c-42d1-9e8e-70ff7abbc80e" />

```dax
Average Daily Loss = DIVIDE([Total Loss],[Total number of Days])
```

---

## Key Learnings & Best Practices

- Always **clean and fix data at the source** (SQL/MySQL) before bringing it into Power BI.
- Keep **column names and table names identical** across environments — DAX measures will break if they differ.
- When shifting from one DB type to another (e.g., SQL Server → MySQL), use **Advanced Editor in PQE** to update the `Source` connector rather than rebuilding the report.
- Always **raise data quality issues** (e.g., fact table values not present in dimension tables) with the data engineering team and keep managers in the loop.
- Test and production environments may differ in **data volume and data quality** — always perform data checks in both.

---

## Tools & Technologies

- Microsoft SQL Server
- MySQL Workbench
- Power BI Desktop & Power BI Service
- DAX (Data Analysis Expressions)
- Power Query Editor (M Language)
- Microsoft Excel (initial data staging)
