# Data Cleaning Log — raw_orders.xlsx
**Analyst:** Aaditya Abhay Bogar (bitsom_ba_2511804)  
**Date:** 2026-06-19  
**Raw Records:** 932 | **Cleaned Records:** 912

---

## 1. Issues Found

### 1.1 Text / Formatting Issues
| Field | Issues Found |
|---|---|
| segment | Extra spaces, leading/trailing whitespace, mixed case (e.g. `SMALL BUSINESS`, `  Consumer `, `corporate`), double spaces (`Small  Business`) |
| region | Extra spaces, mixed case (`EAST`, `east`, `  North `) |
| category | Extra spaces, mixed case (`FURNITURE`, `office supplies`), double spaces (`Office  Supplies`) |
| sub_category | Extra spaces, mixed case (`COPIERS`, `binders`, `  Accessories `) |
| ship_mode | Extra spaces, mixed case (`FIRST CLASS`, `standard class`), double spaces (`Standard  Class`) |
| payment_status | Extra spaces, mixed case (`PENDING`, `failed`, `Paid `) |
| order_status | Extra spaces, mixed case (`CANCELLED`, `completed`, `  Completed `) |
| customer_name | Leading/trailing spaces |

### 1.2 Missing Values
| Field | Count Missing | % of Records |
|---|---|---|
| region | 26 | 2.79% |
| ship_mode | 22 | 2.36% |
| discount | 18 | 1.93% |

### 1.3 Duplicate Records
- **Exact duplicate rows (all 21 columns identical):** 20 rows
- **Duplicate order_id with conflicting data (after exact dup removal):** 12 records

### 1.4 Discount Issues
| Issue | Count |
|---|---|
| Negative discount values | 15 |
| Percentage string format (70%, 85%) | 2 |
| Missing/null discount | 18 |
| Discount > 1 (over 100%) | 0 |

### 1.5 Date Issues
- **Mixed formats in raw data:** 4 different formats found:
  - `DD Mon YYYY` (e.g., `21 Jul 2024`)
  - `MM/DD/YYYY` (e.g., `08/31/2024`)
  - `DD-MM-YYYY` (e.g., `28-11-2024`)
  - `YYYY-MM-DD` (e.g., `2024-05-24`)
- **Missing dates after parsing:** 0 (all dates parsed successfully)
- **Ship date before order date (invalid shipping records):** 22 records

### 1.6 Sales / Profit Calculation Mismatches
- **Sales mismatch** (recorded ≠ qty × unit_price × (1−discount)): **80 records**
- **Profit mismatch** (recorded ≠ calculated_sales − cost): **70 records**

---

## 2. Cleaning Actions Performed

### 2.1 Text Cleaning
- Applied `.strip()` to remove leading/trailing whitespace on all text columns
- Collapsed multiple internal spaces using `re.sub(r'\s+', ' ', s)`
- Converted all text fields to Title Case for consistency
- Removed unwanted special characters (kept basic punctuation: `,`, `.`, `-`)
- Manually corrected known double-space variants: `Small  Business` → `Small Business`, `Office  Supplies` → `Office Supplies`, `Standard  Class` → `Standard Class`
- Fields cleaned: `customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`

### 2.2 Date Standardization
- Wrote a custom multi-format date parser that tries 6 known formats in order
- Fell back to `dateutil.parser` with `dayfirst=True` for any remaining ambiguous formats
- All dates standardized to `YYYY-MM-DD` format in `cleaned_orders.xlsx`
- Computed `shipping_delay_days` = `ship_date − order_date` (in calendar days)

### 2.3 Duplicate Handling
- Identified 20 exact duplicate rows (identical across all 21 columns)
- Kept the first occurrence, removed the 20 duplicates
- After exact dup removal: identified 12 records with duplicate `order_id` but differing field values
- These 12 records were **retained but flagged** as `warning` in `data_quality_flag` — not silently deleted
- Duplicate handling summary documented in `outputs/data_quality_report.xlsx` Sheet 2

### 2.4 Discount Cleaning
- Converted percentage strings: `'70%'` → `0.70`, `'85%'` → `0.85`
- Retained converted values as valid discounts in `cleaned_discount`
- Negative discounts flagged as `invalid` — NOT corrected (business decision required)
- Missing discounts treated as `0.0` (all other sales fields were valid for these records)
- Created column `cleaned_discount` for all downstream calculations

### 2.5 Missing Value Imputation
- `region` (26 missing): Filled with `'Unknown'` and flagged in data quality report
- `ship_mode` (22 missing): Filled with `'Unknown'` and flagged in data quality report
- `discount` (18 missing): Treated as `0` where all other sales fields were valid

---

## 3. Business Rules Applied

| Rule | Action |
|---|---|
| Missing region | Filled as `Unknown`; flagged in quality report |
| Missing ship_mode | Filled as `Unknown`; flagged in quality report |
| Missing discount | Treated as `0` (all other sales fields valid for these rows) |
| Negative discount | Flagged as `invalid` in `data_quality_flag` |
| Discount > 1 | Would be flagged as `invalid` (none found in this dataset) |
| Cancelled orders | Excluded from completed sales summary in pivot reports |
| Failed payments | Excluded from completed sales summary in pivot reports |
| Refunded/Returned orders | Summarized separately in pivot Sheet 5 |
| Ship date before order date | Flagged as `invalid`; `shipping_delay_days` is negative for these records |
| Sales/profit calculation mismatch | Both original and recalculated columns retained; flagged as `warning` |

---

## 4. Calculated Columns Added

| Column | Formula / Logic |
|---|---|
| `cleaned_discount` | Normalized discount: pct strings converted, missing = 0 |
| `calculated_sales` | `quantity × unit_price × (1 − cleaned_discount)` |
| `calculated_profit` | `calculated_sales − cost` |
| `profit_margin` | `calculated_profit / calculated_sales` (rounded to 4 dp) |
| `shipping_delay_days` | `ship_date_parsed − order_date_parsed` (calendar days) |
| `order_month` | Month number extracted from `order_date` |
| `order_year` | Year extracted from `order_date` |
| `data_quality_flag` | `clean` / `warning` / `invalid` based on rule violations |
| `discount_is_negative` | Boolean — TRUE if cleaned_discount < 0 |
| `discount_above_max` | Boolean — TRUE if cleaned_discount > 1 |
| `ship_before_order` | Boolean — TRUE if ship_date < order_date |

---

## 5. Records Removed

| Reason | Count |
|---|---|
| Exact duplicate rows | 20 |
| **Total removed** | **20** |

No other records were deleted. All other problematic records were **flagged** in `data_quality_flag` and retained.

---

## 6. Records Flagged

| Flag | Count | Primary Reasons |
|---|---|---|
| `invalid` | 36 | Negative discount, ship date before order date |
| `warning` | 76 | Sales/profit mismatch, duplicate order_id |
| `clean` | 800 | No issues detected |

---

## 7. Assumptions Made

1. **Discount = 0 for missing**: Assumed missing discount means no discount was applied, not that the data is corrupted, because all other fields (quantity, unit_price, sales, cost, profit) were present and internally consistent for these rows.
2. **Percentage strings are valid**: `70%` and `85%` were assumed to be data entry formatting errors (should have been `0.70`, `0.85`), not actually invalid discounts.
3. **Date format ambiguity**: For ambiguous dates (e.g., `06/08/2024` which could be June 8 or August 6), the format was determined by pattern matching against the majority format in the surrounding rows. `MM/DD/YYYY` was prioritized when the pattern was ambiguous.
4. **Duplicate order_ids with conflicts**: Not deleted because conflicting records may represent legitimate updates or system sync issues — flagged for business review instead.
5. **Cancelled and Failed orders**: These statuses were treated as mutually exclusive exclusion criteria for the sales summary. A Cancelled order with a Paid status was still excluded from the sales summary (order status takes precedence).
6. **Negative discounts**: Treated as data entry errors (possibly meant as surcharges), but not corrected without business confirmation — flagged as `invalid`.

---

## 8. Limitations

- Cannot determine the "correct" values for conflicting duplicate order_ids without access to source systems.
- `MM/DD/YYYY` vs `DD/MM/YYYY` ambiguity for dates where day ≤ 12 could not be resolved with certainty; parser defaulted to `MM/DD/YYYY`.
- Sales/profit mismatches cannot be fully resolved — both original and recalculated values retained; business should confirm which is authoritative.
- Negative discounts could be legitimate surcharges in some business contexts — flagged for review rather than removed.
- `profit_margin` is undefined (NaN) for rows where `calculated_sales = 0`.
