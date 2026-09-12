# Customer Intelligence Framework

This is a customer analytics project I built using the Online Retail II dataset (UCI) — about two years of transaction data from a UK-based online gift retailer, Dec 2009 to Dec 2011.

Most beginner projects on a dataset like this stop at one thing — predict churn, or cluster customers, and call it done. I wanted to go a step further and actually connect a few different analyses together: who are the customers, how much are they worth, who's likely to leave, and does price even matter to them. So this project ties together four pieces — segmentation, lifetime value, churn, and price elasticity — into one final table that a business could actually act on.

## How it's organized

Each part of the analysis lives in its own notebook. They run in order, and each one saves its output as a CSV that the next notebook picks up — nothing is hardcoded between notebooks, they're just chained through files in `Data/Processed/`.

```
Raw data
   ↓
01_data_cleaning
   ↓
02_segmentation  (RFM + K-Means)
   ↓
   ├── 03_clv
   ├── 04_churn
   └── 05_price_sensitivity
   ↓
06_final_summary  → combined table + recommendations
```

## Folder structure

```
├── Data/
│   ├── Raw/            → put online_retail_II.csv here (not included in repo, see below)
│   └── Processed/      → outputs saved by each notebook, included in repo
├── Notebooks/
│   ├── 01_data_cleaning.ipynb
│   ├── 02_segmentation.ipynb
│   ├── 03_clv.ipynb
│   ├── 04_churn.ipynb
│   ├── 05_price_sensitivity.ipynb
│   └── 06_final_summary.ipynb
├── Project_Documentation.docx
├── Customer_Intelligence_Report.docx
└── README.md
```

## What each notebook does

**01 – Data Cleaning**
Drops rows with no Customer ID (can't use them), pulls cancelled orders into a separate "cancel count" feature instead of just deleting them, removes a handful of bad price rows, and builds a clean transaction table.

**02 – Segmentation**
Builds RFM features (Recency, Frequency, Monetary) per customer, scales them, and runs K-Means. Landed on 4 clusters using the elbow method + silhouette score. Named the clusters based on what they actually looked like: Champions, New/Occasional, At-Risk, Lost/Inactive.

**03 – CLV**
Uses the `lifetimes` library to fit a BG/NBD model (predicts future purchase count) and a Gamma-Gamma model (predicts spend per purchase), combined into a 12-month CLV prediction per customer.

**04 – Churn**
Defines churn based on the actual gap between orders in the data (not a guessed number) — 75 days. Trains a logistic regression to predict churn, leaving Recency out of the model since that's literally what defines churn. Compares churn rate across segments.

**05 – Price Sensitivity**
Runs a log-log regression (price vs. quantity) per segment to estimate elasticity. Tried this per-product first but it was too noisy, so switched to per-segment instead.

**06 – Final Summary**
Pulls CLV, churn, and elasticity together into one table by segment, and adds a recommended action for each one.

## What I found

| Segment | CLV | Churn | Elasticity | What to do |
|---|---|---|---|---|
| Champions | $5,393 | 5.6% | -1.87 | Don't discount — protect the relationship |
| At-Risk | $827 | 81.9% | -0.44 (not significant) | Retention priority — win-back, not discounts |
| New/Occasional | $1,031 | 3.1% | -2.37 | Most price-responsive — promos work here |
| Lost/Inactive | $93 | 95.0% | +1.95 (confounded) | Not worth chasing |

At-Risk is the segment that matters most here — decent historical spend, but 82% have already gone quiet. The positive elasticity for Lost/Inactive looked wrong at first (higher price → more buying doesn't make sense), and turned out to be a seasonality artifact, not a real effect — their remaining purchases were all bunched around one holiday period. Kept it in the report anyway since it's a good example of why you can't just trust a regression coefficient without checking what's behind it.

## Tools used

pandas, numpy, scikit-learn, statsmodels, `lifetimes`, matplotlib.

## Getting the data

The raw file isn't in this repo (it's ~90MB and freely downloadable). To run this yourself:

1. Get `online_retail_II.csv` from [UCI](https://archive.ics.uci.edu/dataset/502/online+retail+ii) or Kaggle
2. Drop it into `Data/Raw/`
3. Run the notebooks in order, 01 through 06

The processed outputs are already in the repo if you just want to look at results without re-running everything.

Full write-up with more detail on methodology is in `Project_Documentation.docx`.
