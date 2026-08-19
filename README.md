# Data Pre-processing

Two independent pre-processing pipelines, each applied to a real-world dirty dataset, covering the steps required to prepare data for analysis / machine learning.

1. [Dirty Café Sales](#1-dirty-café-sales) (`pre_processing_data_1/`)
2. [Messy Customer Sales](#2-messy-customer-sales) (`pre_processing_data_2/`)

## 1. Dirty Café Sales

`dirty_cafe_sales.csv` — 10 000 café transactions with intentionally dirty data: mixed types (`ERROR`, `UNKNOWN` strings in numeric columns), missing values, and outliers.

| Column | Type | Notes |
|---|---|---|
| Transaction ID | string | row identifier |
| Item | string | product sold |
| Quantity | int | number of units |
| Price Per Unit | float | unit price (contains `ERROR`/`UNKNOWN`) |
| Total Spent | float | total value (contains `ERROR`/`UNKNOWN`) |
| Payment Method | string | 25.8% missing |
| Location | string | 32.7% missing |
| Transaction Date | date | |

## Pipeline

### 1. Overview
Inspect column types, missing value counts and percentages, and unique value counts before any transformation.

### 2. Rename columns
Standardise column names to snake_case.

### 3. Fix dtypes
- `price_per_unit`, `total_spent`, `quantity` → `pd.to_numeric(errors='coerce')` to handle `ERROR`/`UNKNOWN` strings as NaN
- `transaction_date` → `pd.to_datetime(errors='coerce')`

### 4. Handle missing values
- Columns with < ~10% missing (`item`, `quantity`, `price_per_unit`, `total_spent`, `transaction_date`) → drop rows
- Columns with > 10% missing (`payment_method` 25.8%, `location` 32.7%) → impute with **mode** (most frequent category)

### 5. Outlier detection
Three visualisations to identify outliers:
- Histogram of `total_spent`
- Boxplot of `total_spent` per item
- Boxplot of `quantity` per item

### 6. Remove outliers
IQR method applied to `total_spent`:
- Lower fence = Q1 − 1.5 × IQR
- Upper fence = Q3 + 1.5 × IQR

### 7. Normalise numeric columns
`MinMaxScaler` scales `quantity`, `price_per_unit`, and `total_spent` to [0, 1].  
Chosen over `StandardScaler` because the distributions are not assumed to be normal.

### 8. Feature engineering + encode
- Extract `year` and `month` from `transaction_date` as integer features (captures seasonal patterns without keeping raw datetime)
- Drop `transaction_id` (identifier, no predictive value) and `transaction_date` (replaced by year/month)
- One-hot encode `item`, `payment_method`, `location` with `pd.get_dummies`

### Tech stack

- Python 3.12
- pandas
- scikit-learn
- seaborn / matplotlib

### Run it

```bash
uv sync
```

Open `pre_processing_data_1/pre_processing_script.ipynb` and run all cells from top to bottom.

## 2. Messy Customer Sales

`messy_sales_data.csv` — 68 customer sales records with intentionally dirty data: inconsistent column names/casing, mixed date formats, currency-formatted prices (`$`, `£`, negative values), inconsistent category text (typos, casing, extra whitespace), duplicate rows, and missing values.

| Column | Type | Notes |
|---|---|---|
| customer_id | int | row identifier |
| fullname | string | customer name |
| email | string | customer email |
| signup_date | date | mixed formats (`dd/mm/yyyy`, `Month dd, yyyy`, `yyyy-mm-dd`, `dd-Mon-yyyy`) |
| country | string | inconsistent casing/abbreviations (`UK`, `U.K.`, `United Kingdom`) |
| age | float | contains invalid outliers (e.g. `999`, negative values) |
| order_date | date | same mixed formats as `signup_date` |
| product_category | string | typos/casing (`Clothin`, `cloths`, `electronicss`) |
| price | string → float | currency-formatted, some negative |
| quantity | float | |
| payment_method | string | inconsistent casing/spacing (`debit  card` vs `Debit Card`) |
| rating | float | contains out-of-range outliers |

### Pipeline

1. **Profile the raw data** — generate a `ydata-profiling` report (`report.html`) and inspect shape, dtypes, missing values, head/tail before any transformation.
2. **Standardize column names** — strip whitespace, lowercase, then map abbreviated/inconsistent names to clear snake_case (`customerid` → `customer_id`, `orderdate` → `order_date`, etc.).
3. **Handle duplicates** — detect exact duplicate rows and drop them.
4. **Handle missing values** — numeric columns imputed with the column median, categorical columns filled with `'unknown'`.
5. **Standardize messy text** — strip, lowercase, and collapse internal whitespace across all text columns.
6. **Standardize category values** — mapping dicts (`country_map`, `product_category_map`, `payment_method_map`) fix typos and inconsistent variants (e.g. `'clothin'`/`'cloths'` → `'clothing'`).
7. **Clean and convert price** — strip currency symbols (`$`, `£`) and commas, then convert to numeric with `pd.to_numeric(errors='coerce')`.
8. **Parse dates** — `pd.to_datetime(format='mixed', errors='coerce')` to handle the multiple date formats present in the same column.
9. **Feature engineering** — extract `order_year`, `order_month`, and `order_day_of_week` from `order_date`.
10. **Data integrity checks** — flag (rather than silently drop/alter) rows failing plausibility or consistency rules: `age` in range, `rating` in range, `price` non-negative, `order_date` on/after `signup_date`, `email` matches a basic email pattern.

### Tech stack

- Python 3.12
- pandas
- ydata-profiling

### Run it

```bash
uv sync
```

Open `pre_processing_data_2/sales_clean_up.ipynb` and run all cells from top to bottom.
