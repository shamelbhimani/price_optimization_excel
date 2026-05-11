# Multimodal Price Optimization

An end-to-end **price optimization** project built entirely in Excel. The workbooks layer competitive benchmarking, an elasticity-based pricing recommendation, a customer-churn risk overlay, and a weighted Monte Carlo simulation to produce defensible price recommendations on a SKU-by-SKU basis.

The result is a transparent, formula-driven pipeline that an analyst, category manager, or finance partner can audit cell-by-cell — no Python, no black-box ML.

---

## Project Outcome

Across 24 candidate SKUs (out of 50 in the catalog), the final model recommends:

| Metric | Result |
|---|---|
| Total gross profit impact | **+$5,575** on $44,901 of current GP |
| Average gross profit lift | **+12.33%** |
| GP margin shift | 53.73% → **57.28%** |
| Average price change | **+8.88%** (only 59.9% of the V1 model's naive suggestion) |
| Action mix | 20 Raise · 4 Hold · 0 Reduce |

Selection criteria for the final candidate set: Gross Profit Lift > 5% **and** LTV Protection Ratio > 40%.

---

## Repository Contents

```
price_optimization_excel/
├── data/
│   ├── 1_product_catalog_pricing.csv     # 50 SKUs: cost, retail, sale price, margin, stock, ratings
│   ├── 2_sales_transactions.csv          # ~1,000 transactions across channels and regions
│   ├── 3_customer_segments.csv           # ~4,000 customers: segment, LTV, NPS, discount sensitivity
│   └── 4_competitor_pricing.csv          # Competitor prices observed across 4 rival retailers
│
├── multimodal_analysis.xlsx              # Main workbook — full 4-step pipeline + raw data tabs
├── monte_carlo_price_optimization.xlsx   # Standalone Monte Carlo engine (2,200-row simulation grid)
└── model_output.xlsx                     # Clean executive summary + pricing detail deck
```

---

## Methodology

The pipeline runs in four sequential steps, each implemented as its own sheet inside `multimodal_analysis.xlsx`.

### Step 1 — Competitive Analysis
For every SKU, the current price is compared to the basket of competitor prices using a configurable **tolerance band** (default ±10%). Each SKU is flagged as **Raise**, **Hold**, or **Reduce** based on where it sits relative to the competitive median, with a tie-break rule that defaults to `Raise`.

### Step 2 — V1 Pricing Model
The Step 1 recommendation is translated into a target price using a constant-elasticity demand model with separate elasticities for upward and downward moves:
- Raise elasticity: **−0.4** (demand is relatively inelastic on the way up)
- Reduce elasticity: **−0.8** (more responsive on the way down)
- Margin floor: **20%** (no recommendation may push a SKU below this)

This produces a pure profit-optimal price ignoring customer behavior.

### Step 3 — Churn Overlay
The V1 price is then stress-tested against the customer base. Each SKU is scored for churn risk by joining sales transactions to customer segments and weighting by:
- Segment base score (At-Risk Churner: 70, Dormant: 65, etc.)
- Discount sensitivity of the buying cohort
- LTV exposure on the SKU

The output is an **LTV-at-Risk** estimate that dampens the V1 price move when too much customer lifetime value would be jeopardized.

### Step 4 — Monte Carlo Simulation
A 2,200-row simulation grid sweeps a price grid (2% steps) for each SKU and scores every candidate price under five strategic scenarios:

| Scenario | GR Weight | GP Weight | LTV Weight |
|---|---|---|---|
| Aggressive Conservative | 0.10 | 0.10 | 0.80 |
| Conservative | 0.15 | 0.20 | 0.65 |
| Balanced | 0.25 | 0.35 | 0.40 |
| Growth | 0.35 | 0.35 | 0.30 |
| Aggressive Growth | 0.20 | 0.55 | 0.25 |

The objective function rewards gross revenue lift, gross profit lift, and LTV protection per the weights above, with a quadratic margin penalty and an LTV recovery multiplier to keep recommendations grounded.

---

## Tuning Knobs

All assumptions are exposed at the top of each step's sheet — no formula edits required. Headline parameters:

| Parameter | Default | Lives in |
|---|---|---|
| Tolerance band | 8–10% | Steps 1 & 2 |
| Raise / Reduce elasticity | −0.4 / −0.8 | Steps 2 & 4 |
| Margin floor | 20% | Steps 2 & 4 |
| Min GR lift | 5% | Step 4 |
| Min GP margin lift | 2% | Step 4 |
| Quadratic margin penalty | 50 | Step 4 |
| LTV recovery penalty multiplier | 2× | Step 4 |
| Grid step size | 2% | Step 4 |
| Reward multiplier / Penalty dampener | 5 / 4 | Step 4 |

---

## How to Use

1. Open `multimodal_analysis.xlsx` to walk through the full methodology — raw data tabs are included for traceability.
2. Open `monte_carlo_price_optimization.xlsx` to re-run the simulation with different elasticity assumptions or scenario weights.
3. Open `model_output.xlsx` for the executive summary and per-SKU recommendations formatted for stakeholders.

The `Path` / `FilePath` sheet inside each workbook holds the file-location reference used by cross-workbook formulas. If you move the files, update that cell.

---

## Data Sources

The CSVs are synthetic and self-contained — no external dependencies. They cover the four data domains a real price-optimization exercise typically needs: product catalog with cost and margin, transactional sales history, customer segmentation with LTV, and competitor pricing observations.

---

## Author

**Shamel Bhimani** · [github.com/shamelbhimani](https://github.com/shamelbhimani)
