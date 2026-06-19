# Part 1: Business Data Cleaning, Validation & Excel Reporting

**Student:** Aaditya Abhay Bogar  
**Student ID:** bitsom_ba_2511804  
**Repository:** `aadityaabhaybogar_bitsom_ba_2511804_part1_data_cleaning`

---

## Problem Summary

A retail company exports order-level sales data from multiple internal systems. The raw dataset contains data quality issues including inconsistent text formatting, multiple date formats, duplicate records, missing values, invalid discounts, calculation mismatches, and order status inconsistencies. The objective is to clean and validate the dataset, apply business rules, and produce summary reports for business review.

---

## Dataset Description

| Attribute | Value |
|---|---|
| File | `data/raw_orders.xlsx` |
| Raw Records | 932 |
| Columns | 21 |
| Date Range | 2024–2025 |
| Key Fields | order_id, order_date, ship_date, customer details, product details, sales, cost, profit, discount, payment_status, order_status |

---

## Tools Used

- **Python 3.10** — primary cleaning and processing language
- **pandas** — data manipulation and analysis
- **openpyxl** — Excel file creation with formatting
- **dateutil** — robust multi-format date parsing
- **re (regex)** — text normalization

---

## Cleaning Steps Performed

### 1. Text Standardization
All text fields (`customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`) were cleaned by:
- Stripping leading/trailing whitespace
- Collapsing multiple internal spaces
- Applying Title Case uniformly
- Correcting known double-space variants (e.g., `Small  Business` → `Small Business`)

### 2. Date Cleaning
- Detected and handled 4 different date formats in the raw data: `DD Mon YYYY`, `MM/DD/YYYY`, `DD-MM-YYYY`, and `YYYY-MM-DD`
- Standardized all dates to `YYYY-MM-DD` format
- Computed `shipping_delay_days` (ship_date − order_date in days)
- Flagged 22 records where ship_date < order_date as `invalid`

### 3. Duplicate Handling
- Removed 20 exact duplicate rows (kept first occurrence)
- Identified 12 additional records with duplicate `order_id` but conflicting data — flagged as `warning`, not deleted

### 4. Discount Cleaning
- Converted percentage string values (`70%` → `0.70`, `85%` → `0.85`)
- Filled 18 missing discount values with `0` (all other sales fields valid)
- Flagged 15 negative discount values as `invalid`

### 5. Missing Value Imputation
- `region` (26 missing): Filled as `Unknown`
- `ship_mode` (22 missing): Filled as `Unknown`

### 6. Calculated Columns
Added 8 new columns: `cleaned_discount`, `calculated_sales`, `calculated_profit`, `profit_margin`, `shipping_delay_days`, `order_month`, `order_year`, `data_quality_flag`, plus 3 internal boolean flag columns.

---

## Business Rules Applied

| Rule | Action |
|---|---|
| Missing region/ship_mode | Filled as `Unknown`, flagged |
| Missing discount | Treated as `0` where all sales fields are valid |
| Negative discount | Flagged as `invalid` |
| Cancelled orders | Excluded from completed sales summaries |
| Failed payments | Excluded from completed sales summaries |
| Refunded/Returned orders | Summarized separately |
| Ship date before order date | Flagged as `invalid` |

---

## Summary of Data Quality Issues

| Issue | Count |
|---|---|
| Exact duplicate rows removed | 20 |
| Duplicate order_ids flagged | 12 |
| Missing region (filled Unknown) | 26 |
| Missing ship_mode (filled Unknown) | 22 |
| Missing discount (treated as 0) | 18 |
| Negative discounts flagged | 15 |
| Percentage-string discounts converted | 2 |
| Ship-before-order records | 22 |
| Sales calculation mismatches | 80 |
| Profit calculation mismatches | 70 |
| **Clean records** | **800 (87.7%)** |
| **Warning records** | **76 (8.3%)** |
| **Invalid records** | **36 (3.9%)** |
| **Total cleaned records** | **912** |

---

## Summary of Pivot Reports

| Sheet | Contents |
|---|---|
| 1_Region | Total sales, cost, profit, and avg profit margin by region — sorted by sales descending |
| 2_Category | Sales & profit breakdown by category and sub-category |
| 3_Ship_Mode | Order counts, status breakdown, and avg shipping delay by ship mode |
| 4_Segment_Margin | Profit margin statistics by customer segment — sorted by avg margin descending |
| 5_Issues_by_Region | Cancelled, returned, and failed payment orders by region |
| 6_Monthly_Trend | Monthly completed order count, total sales, and total profit |

---

## Key Business Insights

1. **~8.8% of records need review** — 76 warnings (mostly sales mismatches) and 36 invalid records (negative discounts, bad ship dates).
2. **Calculation mismatches are widespread** — 80 records have sales figures that don't match the formula `qty × price × (1−discount)`, suggesting potential data entry issues or undocumented pricing adjustments.
3. **Shipping quality issues** — 22 orders have ship dates before order dates, indicating a systemic data capture problem in the shipping system.
4. **Negative discounts (15 records)** may represent surcharges or pricing errors — require business confirmation before correction.
5. **Unknown region/ship_mode** — 26 and 22 records respectively had no location/shipping data, possibly from a legacy or incomplete system integration.

---

## Assumptions and Limitations

- Missing discount → 0 (assumed no discount applied, not missing data)
- `70%` and `85%` treated as formatting errors (should be `0.70`, `0.85`)
- Dates with `MM/DD` ambiguity defaulted to `MM/DD/YYYY` format
- Conflicting duplicate order_ids retained and flagged — not deleted without business confirmation
- `profit_margin` is `NaN` for rows with `calculated_sales = 0`
- Sales/profit mismatches retained — cannot determine authoritative source without system access

---

## Screenshots

| File | Description |
|---|---|
| `screenshots/raw_data_preview.png` | Raw dataset before any cleaning |
| `screenshots/cleaned_data_preview.png` | Cleaned dataset with calculated columns and color flags |
| `screenshots/pivot_summary_1.png` | Sales & Profit by Region pivot |
| `screenshots/pivot_summary_2.png` | Monthly Sales Trend pivot |

---

## Repository Structure

```
part1_data_cleaning/
├── data/
│   ├── raw_orders.xlsx          # Original unmodified dataset
│   └── cleaned_orders.xlsx      # Cleaned, validated, with calculated columns
├── outputs/
│   ├── data_quality_report.xlsx # 7-sheet quality report
│   ├── pivot_summary.xlsx       # 6 pivot summaries + overview sheet
│   └── cleaning_log.md          # Detailed cleaning decisions and documentation
├── screenshots/
│   ├── raw_data_preview.png
│   ├── cleaned_data_preview.png
│   ├── pivot_summary_1.png
│   └── pivot_summary_2.png
└── README.md
```
