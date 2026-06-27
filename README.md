# Part 1: Business Data Cleaning, Validation & Excel Reporting

**Student:** Satya | **Student ID:** 211817  
**Course:** Business Analytics — Bitsom  
**Repository:** `satya_211817_part1_data_cleaning`

---

## Business Problem Summary

A retail company has exported order-level sales data from multiple internal systems. The raw dataset contains numerous data quality issues including inconsistent text formatting, mixed date formats, duplicate records, missing values, invalid discounts, and calculation mismatches. As a business analyst, the task is to:

- Clean and standardize the dataset
- Validate business rules
- Document all issues found
- Create summary reports for business review

---

## Dataset Description

| Detail | Value |
|---|---|
| File | raw_orders.xlsx |
| Raw Records | 932 |
| Columns | 21 |
| Date Range | 2024–2025 |
| Geography | India (multi-region) |

**Columns:** order_id, order_date, ship_date, customer_id, customer_name, segment, region, state, city, category, sub_category, product_name, ship_mode, quantity, unit_price, discount, sales, cost, profit, payment_status, order_status

---

## Tools Used

- Microsoft Excel — manual cleaning reference
- Python (pandas, openpyxl, dateutil) — automated cleaning and report generation
- GitHub — version control and submission

---

## Cleaning Steps Performed

1. **Text Standardization** — Trimmed whitespace, collapsed double spaces, applied canonical case mapping for 6 categorical columns (segment, region, category, ship_mode, payment_status, order_status)
2. **Date Cleaning** — Parsed 5+ mixed date formats into a consistent DD-MMM-YYYY format using dateutil; calculated shipping_delay_days
3. **Discount Cleaning** — Converted % format discounts to decimal; flagged negative discounts as invalid; filled missing discounts as 0
4. **Duplicate Removal** — Removed 20 exact duplicate rows; flagged 12 conflicting order_ids for review
5. **Missing Value Handling** — Filled missing region and ship_mode with 'Unknown'; filled missing discount as 0
6. **Calculated Columns** — Added 7 new columns (see below)
7. **Data Quality Flagging** — Classified every record as Clean / Warning / Invalid

---

## Business Rules Applied

| Rule | Action |
|---|---|
| Missing region / ship_mode | Fill as 'Unknown', flag |
| Missing discount (other fields valid) | Treat as 0 |
| Negative discount | Flag as Invalid |
| Percentage-format discount | Convert to decimal |
| Ship date before order date | Flag as Invalid |
| Cancelled / Returned orders | Excluded from sales summaries |
| Failed / Refunded payments | Excluded from sales summaries |
| Conflicting duplicate order_ids | Flagged as Warning, retained |

---

## Summary of Data Quality Issues Found

| Issue | Count |
|---|---|
| Missing region | 26 |
| Missing ship_mode | 22 |
| Missing discount | 18 |
| Negative discounts | 15 |
| Percentage-format discounts | 8 |
| Ship date before order date | 93 |
| Exact duplicate rows removed | 20 |
| Conflicting duplicate order_ids | 12 |
| Sales calculation mismatches | 59 |
| Records flagged Invalid | 106 |
| Records flagged Warning | 298 |
| Records flagged Clean | 508 |

---

## Calculated Columns Added

| Column | Logic |
|---|---|
| cleaned_discount | Standardized discount value (decimal format) |
| calculated_sales | quantity × unit_price × (1 − cleaned_discount) |
| calculated_profit | calculated_sales − cost |
| profit_margin | calculated_profit / calculated_sales |
| shipping_delay_days | ship_date − order_date (days) |
| order_month | Month extracted from order_date |
| order_year | Year extracted from order_date |
| data_quality_flag | Clean / Warning / Invalid classification |

---

## Summary of Pivot Reports

| Sheet | Contents |
|---|---|
| Sales by Region | Sales, profit, margin by region (sorted by sales descending) |
| Sales by Category | Sales and profit by category and sub-category |
| Orders by Ship Mode | Order count and average shipping delay by ship mode |
| Profit by Segment | Profit margin by customer segment |
| Issues by Region | Cancelled/returned/failed orders by region |
| Monthly Sales Trend | Monthly order count, sales, and profit (completed + paid only) |

---

## Key Business Insights

1. **Data quality issues are widespread** — 44% of records have at least a Warning or Invalid flag, suggesting source systems need better input validation.
2. **Missing region data** (26 records) prevents accurate regional performance analysis and should be resolved at the source system level.
3. **Conflicting duplicate order_ids** suggest possible system integration issues between the multiple internal systems that exported this data.
4. **Cancelled and returned orders** contribute to revenue loss and should be tracked separately to identify patterns by region or category.
5. **Sales calculation mismatches** (59 records) indicate possible rounding or discount application inconsistencies in the source system.

---

## Assumptions and Limitations

**Assumptions:**
- Ambiguous date format (MM/DD vs DD/MM) resolved using MM/DD/YYYY as default
- Missing discount with all other fields valid = 0 discount
- Conflicting duplicates retained pending business confirmation

**Limitations:**
- Cannot auto-resolve conflicting duplicate order_ids without source system reference
- Date format ambiguity may result in some misinterpretations
- No external reference available to validate geographic or product data

---

## Screenshots

| File | Description |
|---|---|
| screenshots/raw_data_preview.png | Raw dataset before cleaning |
| screenshots/cleaned_data_preview.png | Cleaned dataset with all calculated columns |
| screenshots/pivot_summary_1.png | Sales and profit by region pivot |
| screenshots/pivot_summary_2.png | Monthly sales trend pivot |


---

