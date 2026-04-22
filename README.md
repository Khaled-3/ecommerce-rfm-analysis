# E-Commerce RFM Analysis & Dashboard

A complete end-to-end data analysis project on a real-world e-commerce dataset — covering data cleaning, customer segmentation using the RFM framework, and an interactive Power BI dashboard.

---

## Dashboard Preview

### Page 1 — Executive Overview
![Executive Overview](screenshots/Retail_Dashboard_Page1.png)

### Page 2 — RFM Segments
![RFM Segments](screenshots/Retail_Dashboard_Page2.png)

### Page 3 — Product Analysis
![Product Analysis](screenshots/Retail_Dashboard_Page3.png)

---

## Project Structure

```
ecommerce-rfm-analysis/
│
├── data/
│   ├── online_retail.csv          # Raw dataset (not included — see below)
│   ├── retail_clean.csv           # Cleaned dataset (output of Phase 1)
│   ├── rfm_scored.csv             # RFM scored customers (output of Phase 2)
│   └── cancelled_invoices.csv     # Cancelled orders for returns analysis
│
├── notebooks/
│   ├── online_retail_EDA.ipynb    # Phase 1 — Data Cleaning & EDA
│   └── rfm_phase2_scoring.ipynb   # Phase 2 — RFM Scoring & Segmentation
│
├── dashboard/
│   └── Retail_Dashboard.pbix      # Phase 3 — Power BI Dashboard
│
├── screenshots/
│   ├── page1_overview.png
│   ├── page2_segments.png
│   └── page3_products.png
│
└── README.md
```

---

## Dataset

**Online Retail II** — UCI Machine Learning Repository  
Transactions from a UK-based online retailer between **December 2009 and December 2011**.

| Column | Description |
|--------|-------------|
| `Invoice` | Invoice number (prefix 'C' = cancellation) |
| `StockCode` | Product code |
| `Description` | Product name |
| `Quantity` | Units per transaction |
| `InvoiceDate` | Transaction date and time |
| `Price` | Unit price (£) |
| `Customer ID` | Unique customer identifier |
| `Country` | Customer's country |

> Download from Kaggle: [Online Retail II Dataset](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci)  
> Place `online_retail.csv` inside the `data/` folder before running the notebooks.

---

## Phase 1 — Data Cleaning & EDA

**Notebook:** `notebooks/online_retail_EDA.ipynb`

### Cleaning steps

| Step | Action | Rows affected |
|------|--------|--------------|
| Drop missing Customer ID | Anonymous transactions excluded | ~243K rows |
| Remove duplicates | Exact duplicate rows | ~5K rows |
| Isolate cancellations | Invoices starting with 'C' kept separately | ~7.9K invoices |
| Remove invalid values | Quantity ≤ 0 or Price ≤ 0 | ~1K rows |

**Clean dataset:** 397K rows · 5,878 unique customers

### Key findings

- **Total Revenue:** £17.37M over 2 years
- **Best month:** November 2011 — £2.32M (Q4 seasonal spike)
- **Top export market:** EIRE — £616K
- **Top product:** REGENCY CAKESTAND 3 TIER
- **Returns rate:** 17.6% — notably high, likely driven by B2B bulk buyers

### EDA Charts

| # | Chart | Insight |
|---|-------|---------|
| 1 | Monthly Revenue Trend | Clear Q4 seasonal spike every year |
| 2 | Top Countries (excl. UK) | EIRE, Netherlands, Germany lead exports |
| 3 | Top 10 Products | Cakestand dominates — hints at B2B buyers |
| 4 | Orders by Day of Week | Thursday is peak day; Sunday is near-zero |
| 5 | Revenue Distribution | Right-skewed — small group of high-value customers |

---

## Phase 2 — RFM Scoring & Segmentation

**Notebook:** `notebooks/rfm_phase2_scoring.ipynb`

### What is RFM?

| Dimension | Question | Measurement |
|-----------|----------|-------------|
| **Recency (R)** | How recently did they buy? | Days since last purchase |
| **Frequency (F)** | How often do they buy? | Number of unique orders |
| **Monetary (M)** | How much do they spend? | Total revenue |

Each dimension is scored **1–5** using percentile ranking, then combined into a single RFM score (3–15).

### Customer Segments

| Segment | R score | F score | Customers | Revenue Share |
|---------|---------|---------|-----------|--------------|
| **Champions** | 4–5 | 4–5 | 1,780 (30.3%) | ~60% |
| **Loyal** | 2–5 | 3–5 | 1,480 (25.1%) | ~25% |
| **Potential Loyal** | 3–5 | 1–3 | 980 (16.7%) | ~8% |
| **At Risk** | 2–3 | 2–5 | 830 (14.0%) | ~5% |
| **Lost** | 1–2 | 1–2 | 820 (13.9%) | ~2% |

### Scoring method

```python
def score_column(series, ascending=True):
    ranked = series.rank(method='first', ascending=ascending, pct=True)
    return np.ceil(ranked * 5).clip(1, 5).astype(int)

rfm['R_score'] = score_column(rfm['Recency'],   ascending=False)
rfm['F_score'] = score_column(rfm['Frequency'], ascending=True)
rfm['M_score'] = score_column(rfm['Monetary'],  ascending=True)
```

> `rank()` is used instead of `pd.qcut()` to avoid bin edge errors caused by skewed distributions (especially Frequency).

---

## Phase 3 — Power BI Dashboard

**File:** `dashboard/Retail_Dashboard.pbix`

### Pages

**Page 1 — Executive Overview**
- KPI Cards: Total Revenue · Total Orders · Total Customers · Avg Order Value
- Monthly Revenue Trend (line chart)
- Top Export Markets by Revenue (bar chart)
- Best Selling Products by Revenue (bar chart)
- Filter: Year slicer

**Page 2 — RFM Segments**
- Customer Distribution by Segment (donut chart)
- Revenue Contribution per Segment (bar chart)
- Recency vs Frequency by Segment (scatter plot — size = Monetary)
- KPI Cards: Champions % · Total Customers
- Filter: Segment slicer

**Page 3 — Product Analysis**
- Top 10 Products by Revenue (bar chart)
- Monthly Revenue Heatmap (matrix with conditional formatting)
- Orders by Day of Week (bar chart)
- Returns Rate card (17.6%)
- Filter: Country slicer

### DAX Measures

```dax
Total Revenue    = SUM(retail_clean[TotalPrice])
Total Orders     = DISTINCTCOUNT(retail_clean[Invoice])
Total Customers  = DISTINCTCOUNT(retail_clean[Customer ID])
Avg Order Value  = DIVIDE([Total Revenue], [Total Orders])

Champions % =
DIVIDE(
    CALCULATE(
        DISTINCTCOUNT(rfm_scored[Customer ID]),
        rfm_scored[Segment] = "Champions"
    ),
    DISTINCTCOUNT(rfm_scored[Customer ID])
)

Returns Rate =
DIVIDE(
    DISTINCTCOUNT(cancelled_invoices[Invoice]),
    DISTINCTCOUNT(cancelled_invoices[Invoice]) + DISTINCTCOUNT(retail_clean[Invoice]),
    0
)
```

---

## Tools & Libraries

| Tool | Purpose |
|------|---------|
| Python 3.11 | Data processing |
| Pandas | Data manipulation |
| NumPy | Numerical operations |
| Matplotlib | Static visualisations |
| Seaborn | Statistical plots |
| Power BI Desktop | Interactive dashboard |

---

## How to Run

```bash
# 1. Clone the repository
git clone https://github.com/Khaled-3/ecommerce-rfm-analysis.git
cd ecommerce-rfm-analysis

# 2. Install dependencies
pip install pandas numpy matplotlib seaborn openpyxl

# 3. Download the dataset and place it in data/
# https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci

# 4. Run Phase 1
jupyter notebook notebooks/online_retail_EDA.ipynb

# 5. Run Phase 2
jupyter notebook notebooks/rfm_phase2_scoring.ipynb

# 6. Open the dashboard
# Open dashboard/Retail_Dashboard.pbix in Power BI Desktop
```

---

## Author

**Khaled Waleed** — Junior Data Analyst  
[GitHub](https://github.com/Khaled-3) · [LinkedIn](https://linkedin.com/in/khaledwaleed0)
