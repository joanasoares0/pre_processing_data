# Data Pre-processing — Dirty Café Sales

Pre-processing pipeline applied to a real-world dirty dataset of café transactions, covering all steps required to prepare data for machine learning.

## Dataset

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

## Tech stack

- Python 3.12
- pandas
- scikit-learn
- seaborn / matplotlib

## Setup

```bash
uv sync
```

Open `pre_processing_data/pre_processing_script.ipynb` and run all cells from top to bottom.
