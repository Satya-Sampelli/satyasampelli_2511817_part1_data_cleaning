# Cleaning Log — Part 1: Business Data Cleaning, Validation & Excel Reporting

**Student:** Satya Sampelli | **Student ID:** 211817  
**Dataset:** raw_orders.xlsx | **Total Raw Records:** 932

---

## 1. Issues Found in Raw Data

| Issue Area | Detail | Count |
|---|---|---|
| Extra/leading/trailing spaces | Found in segment, region, category, sub_category, ship_mode, payment_status, order_status | Multiple columns |
| Case inconsistencies | e.g. 'CONSUMER', 'consumer', 'Consumer' all in same column | Multiple columns |
| Double spaces inside values | e.g. 'Small  Business', 'Standard  Class', 'Office  Supplies' | ~15 records |
| Missing region | NULL values in region column | 26 |
| Missing ship_mode | NULL values in ship_mode column | 22 |
| Missing discount | NULL values in discount column | 18 |
| Negative discount values | e.g. -0.14, -0.23 — logically invalid | 15 |
| Percentage-format discounts | e.g. "70%", "85%" instead of decimal | 8 |
| Mixed date formats | Dates appear in multiple formats: DD Mon YYYY, MM/DD/YYYY, YYYY-MM-DD, DD-MM-YYYY | All date cols |
| Ship date before order date | Logically invalid shipping records | 93 |
| Exact duplicate rows | Fully identical records | 20 |
| Duplicate order_id (conflicting) | Same order_id with different field values | 12 IDs |
| Sales calculation mismatch | calculated_sales ≠ original sales column | 59 |

---

## 2. Cleaning Actions Performed

### 2.1 Text Field Cleaning
- Applied TRIM equivalent to all text fields: `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`, `product_name`
- Collapsed multiple internal spaces (e.g. "Small  Business" → "Small Business")
- Applied canonical case mapping using a lookup dictionary:
  - segment: Consumer, Small Business, Corporate, Home Office
  - region: North, South, East, West
  - category: Furniture, Technology, Office Supplies
  - ship_mode: First Class, Second Class, Standard Class, Same Day
  - payment_status: Paid, Pending, Failed, Refunded
  - order_status: Completed, Cancelled, Returned
- Applied Title Case to: customer_name, state, city, sub_category, product_name

### 2.2 Date Cleaning
- All date columns (order_date, ship_date) contained mixed formats
- Used dateutil.parser to parse all valid dates into a consistent DD-MMM-YYYY format
- Created column `shipping_delay_days` = ship_date − order_date (in days)
- Created `order_month` = month extracted from order_date
- Created `order_year` = year extracted from order_date

### 2.3 Discount Cleaning
- Percentage-format values (e.g. "70%") converted to decimal (0.70)
- Negative discount values flagged as invalid; `cleaned_discount` set to NaN for these records
- Missing discount values treated as 0 (all other sales fields were valid)
- New column `cleaned_discount` added with cleaned values

### 2.4 Duplicate Handling
- Identified 20 exact duplicate rows → removed all duplicates, keeping first occurrence
- Identified 12 order_ids with conflicting records (same ID, different field values)
  → These were NOT deleted; they were flagged as 'Warning' in `data_quality_flag` column
  → Logic: conflicting records need business review before deletion — cannot assume which is correct

### 2.5 Missing Value Handling
- Missing `region` (26 records): Filled with 'Unknown', flagged
- Missing `ship_mode` (22 records): Filled with 'Unknown', flagged
- Missing `discount` (18 records): Treated as 0 (all other fields valid)

---

## 3. Business Rules Applied

| Rule | Action |
|---|---|
| Missing region | Fill as 'Unknown', flag in quality report |
| Missing ship_mode | Fill as 'Unknown', flag in quality report |
| Missing discount (all other fields valid) | Treat as 0 |
| Negative discount | Flag as Invalid in data_quality_flag |
| Discount in % format | Convert to decimal |
| Cancelled orders | Excluded from sales pivot summaries |
| Failed payments | Excluded from sales pivot summaries |
| Refunded orders | Separately summarized in pivot sheet "Issues by Region" |
| Ship date before order date | Flag as Invalid |
| Conflicting duplicate order_ids | Flagged as Warning, retained for review |

---

## 4. Calculated Columns Added

| Column | Formula Logic |
|---|---|
| `cleaned_discount` | Standardized discount: % format → decimal, negatives → NaN, missing → 0 |
| `calculated_sales` | quantity × unit_price × (1 − cleaned_discount) |
| `calculated_profit` | calculated_sales − cost |
| `profit_margin` | calculated_profit / calculated_sales |
| `shipping_delay_days` | ship_date − order_date (in days) |
| `order_month` | Month number extracted from order_date |
| `order_year` | Year extracted from order_date |
| `data_quality_flag` | 'Clean', 'Warning', or 'Invalid' based on issue rules below |

**data_quality_flag logic:**
- **Invalid**: bad/missing date, ship before order, negative discount
- **Warning**: conflicting duplicate order_id, cancelled/returned order, failed/refunded payment, sales mismatch
- **Clean**: no issues found

---

## 5. Records Removed / Flagged

| Action | Count |
|---|---|
| Exact duplicate rows removed | 20 |
| Records flagged as 'Invalid' | 106 |
| Records flagged as 'Warning' | 298 |
| Records flagged as 'Clean' | 508 |
| Final record count | 912 |

---

## 6. Assumptions Made

1. When discount is missing and all other sales fields (quantity, unit_price, sales, cost, profit) are present, discount is assumed to be 0.
2. For percentage-format discounts (e.g. "70%"), these are assumed to be discount percentages and converted by dividing by 100.
3. For conflicting duplicate order_ids, neither record is deleted without business confirmation — both are flagged as Warning.
4. Cancelled and Returned orders are excluded from sales summary pivots but retained in the dataset.
5. Date parsing uses MM/DD/YYYY as the primary format assumption for ambiguous dates (e.g. 06/08/2024 is treated as June 8, not August 6).
6. Calculated_sales is considered the "correct" sales figure; the original `sales` column is retained for comparison.

---

## 7. Limitations of This Cleaning Process

1. Ambiguous date formats (e.g. 06/08/2024) may be misinterpreted — business should confirm date format convention.
2. Conflicting duplicate order_ids cannot be auto-resolved without knowing the source system of truth.
3. Sales mismatches may result from rounding differences or promotional discounts not captured in the discount column.
4. Negative discount records may represent adjustments or surcharges — business interpretation needed before deletion.
5. No external reference was available to validate product names, customer IDs, or geographic data.
