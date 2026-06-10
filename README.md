<div align="center">

# 🛒 Olist E-Commerce Analytics — Power BI Dashboard

[![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/dax/)
[![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/power-query/)
[![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)](https://github.com/HarshalVara86/E-commerce_Sales_Dashboard_Power_BI)

<br/>

> **End-to-end Power BI project on the Brazilian Olist Marketplace dataset (Sep 2016 – Oct 2018) — ETL in Power Query, DAX-built DimDate, star schema data model, and 3-page interactive dashboard.**

<br/>

### 📺 Project Overview Video

[![Watch Walkthrough](https://img.shields.io/badge/▶%20Watch%20Demo-Google%20Drive-4285F4?style=for-the-badge&logo=googledrive&logoColor=white)](https://drive.google.com/file/d/1rvyQFDeU9Y3usLty8RbA8mQXrAGEbi4o/view?usp=sharing)

</div>

---

## 📋 Table of Contents
- [Key Metrics](#-key-metrics)
- [Tech Stack](#️-tech-stack)
- [Dataset](#-dataset)
- [Data Model — Star Schema](#-data-model--star-schema)
- [ETL Pipeline — Power Query](#-etl-pipeline--power-query)
- [Dashboard Pages](#-dashboard-pages)
- [How to Run](#-how-to-run)

---

## 💡 Key Metrics

| Metric | Value |
|--------|-------|
| Total Orders | 99,441 |
| Total Revenue | $13.59M |
| Avg Customer Rating | 4.09 |
| Avg Order Value | $137.75 |
| Top Category by Revenue | Health & Beauty — $1.26M |
| Top Seller City | São Paulo — $2.70M |
| Dominant Payment Type | Credit Card — 78.34% |

---

## 🛠️ Tech Stack

| Tool | Used For |
|------|---------|
| Power BI Desktop | Dashboard design & data modeling |
| Power Query (M) | ETL — cleaning, merging, transforming raw CSVs |
| DAX | DimDate calendar table, calculated columns & measures |

---

## 📂 Dataset

**Source:** [Olist Brazilian E-Commerce Public Dataset — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)  
**Period:** September 2016 – October 2018

| File | Rows | Columns |
|------|------|---------|
| `olist_orders_dataset.csv` | 99,441 | order_id, customer_id, order_status, purchase_timestamp, approved_at, delivery dates |
| `olist_order_items_dataset.csv` | 112,650 | order_id, order_item_id, product_id, seller_id, shipping_limit_date, price, freight_value |
| `olist_customers_dataset.csv` | 99,441 | customer_id, customer_unique_id, zip_code_prefix, city, state |
| `olist_products_dataset.csv` | 32,951 | product_id, category_name, name_length, description_length, photos_qty, weight_g, dimensions |
| `olist_sellers_dataset.csv` | 3,095 | seller_id, zip_code_prefix, city, state |
| `olist_order_payments_dataset.csv` | 103,886 | order_id, payment_sequential, payment_type, installments, payment_value |
| `olist_order_reviews_dataset.csv` | 104,719 | review_id, order_id, review_score, comment_title, comment_message, creation_date |
| `olist_geolocation_dataset.csv` | 1,000,163 | zip_code_prefix, lat, lng, city, state |
| `product_category_name_translation.csv` | 70 | product_category_name (PT), product_category_name_english |

---

## 🗂 Data Model — Star Schema

![Data Model](screenshots/ModelView.png)

The model uses a **star schema** with 2 Fact tables and 5 Dimension tables, all relationships set in Power BI Model View.

| Table | Type | Source |
|-------|------|--------|
| `FactOrderItems` | Fact | olist_order_items_dataset |
| `FactPayments` | Fact | olist_order_payments_dataset |
| `FactReviews` | Fact | olist_order_reviews_dataset |
| `DimOrders` | Dimension | olist_orders_dataset |
| `DimCustomers` | Dimension | olist_customers_dataset |
| `DimProducts` | Dimension | olist_products_dataset |
| `DimSellers` | Dimension | olist_sellers_dataset |
| `DimDate` | Dimension | **Built via DAX** — not from CSV |

> `DimDate` was created entirely using DAX `CALENDAR()` with added columns: Year, Quarter, Month_Num, Month_Name, Weekday. This table drives all time-based filtering and slicers across the dashboard.

**Relationships configured:**

```
DimDate        ──(1:*)──  DimOrders      (Date → order_purchase_timestamp)
DimOrders      ──(1:*)──  FactOrderItems (order_id)
DimOrders      ──(1:1)──  FactPayments   (order_id)
DimOrders      ──(1:*)──  FactReviews    (order_id)
DimOrders      ──(*)──    DimCustomers   (customer_id)
DimProducts    ──(1:*)──  FactOrderItems (product_id)
DimSellers     ──(1:*)──  FactOrderItems (seller_id)
```

---

## ⚙️ ETL Pipeline — Power Query

All transformations were done inside **Power Query Editor** before loading to the model.

```
9 Raw CSV Files
      │
      ▼
┌──────────────────────────────────────────────────────┐
│               POWER QUERY EDITOR                      │
│                                                       │
│  1. Merge Queries                                     │
│     • olist_orders ⋈ olist_customers → DimOrders     │
│     • olist_order_items ⋈ olist_products             │
│       → FactOrderItems (with category info)          │
│                                                       │
│  2. Change Data Types                                 │
│     • Date/timestamp columns → DateTime              │
│     • price, freight_value → Decimal                 │
│     • zip_code_prefix → Text (keep leading zeros)    │
│                                                       │
│  3. Rename & Clean Columns                           │
│     • Standardized column names across all tables    │
│     • Removed null rows in key join columns           │
│                                                       │
│  4. Remove Duplicates                                │
│     • order_id in DimOrders (for 1-side of relation) │
│                                                       │
│  5. Load 7 tables to Model                           │
└──────────────────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────┐
│  DAX — DimDate Creation  │
│  CALENDAR() +            │
│  Year, Quarter,          │
│  Month_Num, Month_Name,  │
│  Weekday columns         │
└──────────────────────────┘
      │
      ▼
  Star Schema → 3-Page Dashboard
```

---

## 📊 Dashboard Pages

### Page 1 — Sales Overview

![Sales Overview](screenshots/Page1.png)

| Visual | Details |
|--------|---------|
| KPI Cards | Total Orders · Avg Customer Rating · Avg Order Value · Total Revenue |
| Line Chart | Monthly Order Volume 2016–2018 |
| Bar Chart (horizontal) | Top 10 Product Categories by Revenue |
| Slicers | Order Status · Year · Product Category |
| Text Cards | About the Dataset · Key Findings |

---

### Page 2 — Geographic Distribution — Customers & Sellers

![Geographic Analysis](screenshots/Page2.png)

| Visual | Details |
|--------|---------|
| Bing Map | Orders by Customer State (bubble map) |
| Bar Chart | Top 10 Seller Cities by Revenue |
| Bar Chart | Revenue by Seller State (all 27 states) |
| Slicer | Select Year |

---

### Page 3 — Payment Methods · Customer Satisfaction

![Payments & Reviews](screenshots/Page3.png)

| Visual | Details |
|--------|---------|
| Matrix Table | Payment Value by Type × Year (2016 · 2017 · 2018 · Total) |
| Donut Chart | Payment Type share — Credit Card 78.34%, Boleto 17.92%, Voucher 2.37%, Debit Card |
| Horizontal Bar Chart | Avg Review Score by Product Category (44 categories) |
| Slicer | Select Year |

---

## ▶️ How to Run

1. Clone the repo
   ```bash
   git clone https://github.com/HarshalVara86/E-commerce_Sales_Dashboard_Power_BI.git
   ```
2. Open `E-commerce_dataset.pbix` in [Power BI Desktop](https://powerbi.microsoft.com/desktop/)
3. If data source paths break, go to `Transform Data → Data Source Settings` and point to your local `/dataset/` folder
4. Click **Refresh** and explore the 3-page dashboard

---

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/harshal-vora)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/HarshalVara86)

*Built by **Harshal Vora** · Data Analytics Portfolio*

⭐ Star this repo if you found it useful!

</div>
