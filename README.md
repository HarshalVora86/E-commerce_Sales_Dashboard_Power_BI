<div align="center">

# 🛒 Olist E-Commerce Analytics Dashboard
### Power BI | Brazilian E-Commerce | 99,441 Orders | 2016–2018

[![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)](https://learn.microsoft.com/en-us/dax/)
[![Power Query](https://img.shields.io/badge/Power%20Query-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)](https://learn.microsoft.com/en-us/power-query/)
[![Dataset](https://img.shields.io/badge/Dataset-Olist%20Kaggle-20BEFF?style=for-the-badge&logo=kaggle&logoColor=white)](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

</div>

---

## 📽️ Demo

> 📹 Full walkthrough video → [Watch E-commerce_dashboard.mp4](https://drive.google.com/file/d/1rvyQFDeU9Y3usLty8RbA8mQXrAGEbi4o/view?usp=sharing)

---

## 📌 Project Overview

This end-to-end Power BI project analyzes **Olist** — Brazil's largest e-commerce marketplace — using **8 raw CSV datasets** sourced from Kaggle. The goal was to build a recruiter-ready, analyst-grade dashboard covering sales performance, geographic distribution, payment behaviour, and customer satisfaction.

**Key figures at a glance:**

| Metric | Value |
|--------|-------|
| Total Orders | 99,441 |
| Total Revenue | $13.59M |
| Avg Order Value | $137.75 |
| Avg Customer Rating | 4.09 / 5 |
| Date Range | Jan 2016 – Dec 2018 |

---

## 📊 Dashboard Pages

### Page 1 — Sales Overview
![Sales Overview](Page1.png)

> KPI cards, monthly order trend (2016–2018), order distribution map across Brazilian states, and top 10 product categories by revenue. Includes slicers for Order Status, Year, and Product Category.

---

### Page 2 — Geographic Distribution
![Geographic Analysis](Page2.png)

> Customer order concentration by state on an interactive map, top 10 seller cities by revenue, and revenue breakdown by seller state. SP (São Paulo) dominates both customer demand and seller supply.

---

### Page 3 — Payments & Customer Satisfaction
![Payments & Reviews](Page3.png)

> Payment type mix (credit card = 78.34%), payment value by type and year matrix, and average review score ranked across all product categories.

---

## 🗂️ Dataset

**Source:** [Olist Brazilian E-Commerce — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

| File | Rows | Description |
|------|------|-------------|
| `olist_orders_dataset.csv` | 99,441 | Order master — status, timestamps |
| `olist_order_items_dataset.csv` | 112,650 | Line items — price, freight, seller |
| `olist_order_payments_dataset.csv` | 103,886 | Payment type, value, installments |
| `olist_order_reviews_dataset.csv` | 104,719 | Review scores and comments |
| `olist_customers_dataset.csv` | 99,441 | Customer city, state, zip |
| `olist_products_dataset.csv` | 32,951 | Product category, dimensions, weight |
| `olist_sellers_dataset.csv` | 3,095 | Seller city and state |
| `product_category_name_translation.csv` | 71 | Portuguese → English category names |

---

## 🔄 ETL Pipeline (Power Query)

```mermaid
flowchart TD
    A[📁 8 Raw CSV Files] --> B[Load into Power Query]
    B --> C{Transformations}

    C --> C1[olist_products\nMerge with translation table\nPortuguese → English categories]
    C --> C2[olist_orders\nChange date columns to Date type\nHandle nulls in delivery columns]
    C --> C3[olist_order_items\nRename columns\nVerify price & freight types]
    C --> C4[olist_customers\nStandardise city/state text case]
    C --> C5[olist_sellers\nClean zip_code_prefix type]
    C --> C6[olist_payments\nFilter out 'not_defined' where needed]
    C --> C7[olist_reviews\nParse review_creation_date to Date]

    C1 --> D[Load to Data Model as Dimension & Fact Tables]
    C2 --> D
    C3 --> D
    C4 --> D
    C5 --> D
    C6 --> D
    C7 --> D

    D --> E[DimDate — Created via DAX CALENDAR()]
    E --> F[⭐ Star Schema Model]
    F --> G[3-Page Power BI Dashboard]
```

### Key Power Query Steps

| Step | Detail |
|------|--------|
| **Merge Queries** | Merged `olist_products` with `product_category_name_translation` on `product_category_name` → added `Product_Category_EN` column |
| **Data Type Fixes** | Cast all timestamp columns to `Date/DateTime`; zip code prefixes cast to `Text` to preserve leading zeros |
| **Null Handling** | `order_delivered_customer_date` and `order_delivered_carrier_date` contain nulls for non-delivered orders — retained as null (not replaced) to avoid skewing delivery time calculations |
| **Rename Columns** | Standardised column names to PascalCase / snake_case for DAX readability |

---

## 🧮 DAX — DimDate Table

`DimDate` was built entirely in DAX (not from a CSV), enabling proper time intelligence across all date fields.

```dax
DimDate = 
ADDCOLUMNS(
    CALENDAR(DATE(2016, 1, 1), DATE(2018, 12, 31)),
    "Year",       YEAR([Date]),
    "Month_Num",  MONTH([Date]),
    "Month_Name", FORMAT([Date], "MMMM"),
    "Quarter",    "Q" & QUARTER([Date]),
    "Weekday",    FORMAT([Date], "DDDD")
)
```

This table connects to `DimOrders[order_purchase_timestamp]` enabling Year/Quarter/Month slicers across all pages.

---

## 🗃️ Data Model (Star Schema)

![Model View](ModelView.png)

### Relationships

| From Table | Key | → To Table | Key | Cardinality |
|------------|-----|-----------|-----|-------------|
| `DimOrders` | `order_id` | `FactOrderItems` | `order_id` | 1 : * |
| `DimOrders` | `order_id` | `FactPayments` | `order_id` | 1 : * |
| `DimOrders` | `order_id` | `FactReviews` | `order_id` | 1 : * |
| `DimOrders` | `customer_id` | `DimCustomers` | `customer_id` | * : 1 |
| `FactOrderItems` | `product_id` | `DimProducts` | `product_id` | * : 1 |
| `FactOrderItems` | `seller_id` | `DimSellers` | `seller_id` | * : 1 |
| `DimDate` | `Date` | `DimOrders` | `order_purchase_timestamp` | 1 : * |

**Schema type:** Star Schema with 3 fact tables (`FactOrderItems`, `FactPayments`, `FactReviews`) and 4 dimension tables (`DimOrders`, `DimCustomers`, `DimProducts`, `DimSellers`, `DimDate`).

---

## 📐 Key DAX Measures

```dax
-- Total Revenue
Total Revenue = SUMX(FactOrderItems, FactOrderItems[price] + FactOrderItems[freight_value])

-- Total Orders
Total Orders = DISTINCTCOUNT(DimOrders[order_id])

-- Average Order Value
Avg Order Value = DIVIDE([Total Revenue], [Total Orders])

-- Avg Customer Rating
Avg Rating = AVERAGE(FactReviews[review_score])
```

---

## 🗺️ Key Insights

- **São Paulo dominates:** SP accounts for ~42% of all orders and ~$10.3M of seller revenue — significant geographic concentration risk.
- **Credit card is king:** 78.34% of total payment value flows through credit cards; boleto (cash voucher) is second at 17.92%.
- **Peak demand:** Order volume peaks in **November–December** (Black Friday / holiday season), with July showing a sharp mid-year dip.
- **Top category:** `health_beauty` leads revenue at $1.26M, outpacing `watches_gifts` ($1.21M) and `bed_bath_table` ($1.04M).
- **Niche categories score highest:** `cds_dvds_musicals` (4.67) and `fashion_roupa_infanto_juvenil` (4.50) have the highest avg review scores despite lower sales volume.

---

## 🛠️ Tools & Skills Used

| Tool | Usage |
|------|-------|
| **Power BI Desktop** | Report authoring, visuals, slicers |
| **Power Query (M)** | ETL — data cleaning, merges, type casting |
| **DAX** | Calculated table (DimDate), KPI measures |
| **Bing Maps visual** | Geographic order and revenue distribution |
| **Matrix visual** | Payment value by type × year cross-tab |

---

## 📁 Repository Structure

```
E-commerce_Sales_Dashboard_Power_BI/
│
├── E-commerce_dataset.pbix          # Power BI report file
│
├── data/                            # Raw source CSVs
│   ├── olist_orders_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_order_payments_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   ├── olist_customers_dataset.csv
│   ├── olist_products_dataset.csv
│   ├── olist_sellers_dataset.csv
│   └── product_category_name_translation.csv
│
├── screenshots/
│   ├── Page1.png                    # Sales Overview
│   ├── Page2.png                    # Geographic Analysis
│   ├── Page3.png                    # Payments & Reviews
│   └── ModelView.png                # Data model diagram
│
└── README.md
```

---

## 🚀 How to Run

1. Clone this repository
2. Place all CSV files from `/data` in the same folder
3. Open `E-commerce_dataset.pbix` in **Power BI Desktop** (free)
4. If prompted, update the data source path to your local `/data` folder
5. Click **Refresh** — all 8 tables load automatically

> **Requirement:** Power BI Desktop (free) — [Download here](https://powerbi.microsoft.com/desktop/)

---

## 👤 Author

**Harshal Vora**
B.E. Computer Engineering | Data Analytics Enthusiast

[![GitHub](https://img.shields.io/badge/GitHub-HarshalVara86-181717?style=flat&logo=github)](https://github.com/HarshalVara86)

---

<div align="center">

*If this project was useful, consider giving it a ⭐*

</div>
