# Phase III — Gender Pay Gap Analysis

## Overview

This notebook investigates the gender pay gap across the United States using a combination of U.S. Census microdata (ACS PUMS), Bureau of Labor Statistics (BLS) data, and the Census API. It cleans and combines individual-level records from 10 major states, enriches the data with demographic and occupational context, and delivers statistical analysis, machine learning models, and interactive visualizations to understand earnings disparities between men and women.

---

## Data Sources

| Source | Description |
|---|---|
| **ACS PUMS CSVs** (`psam_p##.csv`) | Per-person microdata for CA, FL, GA, IL, MI, NC, NY, OH, PA, TX |
| **Census API** (`acs/acs1/subject`) | State-level male and female median earnings |
| **BLS Website** (scraped) | National and state-level unemployment rates |
| **BLS Excel (`womens-earnings-tables-2023.xlsx`)** | Female earnings as a % of men's by age, race, and occupation (1979–2023) |

---

## Setup

This notebook runs in **Google Colab** and requires Google Drive access for all data files.

**Install dependencies:**
```bash
pip install selenium
pip install -U kaleido plotly
```

**Key libraries:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `plotly`, `scikit-learn`, `scipy`, `requests`, `BeautifulSoup`, `selenium`

Place all input files in the appropriate Google Drive paths referenced in the notebook before running.

---

## Notebook Structure

### 1. Data Loading & Cleaning (Cells 0–15)
Loads ACS PUMS microdata for 10 states and applies a standardized cleaning pipeline:
- **Deduplication** — removes duplicate `SERIALNO` entries
- **Column filtering** — retains only relevant fields: `STATE`, `SEX`, `AGEP`, `RAC1P`, `SCHL`, `OCCP`, `INDP`, `WKHP`, `WKWN`, `PINCP`, `ADJINC`, `ESR`, `PWGTP`
- **Missing value handling** — drops rows missing key continuous fields (`WKHP`, `WKWN`, `PINCP`); imputes remaining nulls with column medians
- **Outlier removal** — filters to ages 16–90, weekly hours ≤ 80, weeks worked ≤ 52, and non-negative income
- **Feature enrichment** — maps race codes to readable labels and categorizes occupation codes into broad groups
- **Export** — merges all states into a single `cleaned_USA_data.csv`

### 2. External Data Collection (Cells 18–25)
- **Web scraping (BLS)** — scrapes national and state-level unemployment rates using `requests` and `BeautifulSoup`
- **Census API** — pulls male/female median earnings by state
- **Excel parsing** — extracts three BLS tables:
  - Female-to-male earnings ratio by **age group** (Table 12)
  - Female-to-male earnings ratio by **race** (Table 13)
  - Female-to-male earnings ratio by **occupation** (Table 2)

### 3. Statistical Analysis & Insights (Cells 32–34)
- **Insight 1** — Ranks states by gender pay gap percentage (largest to smallest)
- **Unemployment classification** — labels states as "Well-Performing" or "Poor" relative to national unemployment
- **T-test** — tests whether states with poor economic performance have statistically different gender pay gaps than well-performing states

### 4. Machine Learning (Cells 36–38)

**Random Forest Regressor** — Predicts individual income (`PINCP`) from:
- Age (`AGEP`), Education (`SCHL`), Occupation group (`OCCP_GROUP`), Race (`RAC1P`), Hours worked (`WKHP`), Weeks worked (`WKWN`), Sex (`SEX`)
- Uses a preprocessing pipeline with `OrdinalEncoder` for categorical features
- Trained on a 50,000-row sample; evaluated with R² and MSE

**K-Means Clustering** — Groups occupations into 3 clusters based on:
- Men's median weekly earnings
- Women's median weekly earnings
- Women's earnings as a percentage of men's

### 5. Visualizations (Cells 40–50)
All visualizations are built with **Plotly** for interactivity:
- **Dot-and-line plot** — Male vs. female median earnings side-by-side by state
- **Line chart** — Female earnings as % of male earnings by age group (1979–2023)
- **Line chart** — Female earnings as % of male earnings by race (1979–2023)
- **Scatter plot** — Occupation-level gender gap clusters with parity reference line

---

## Key Variables

| Variable | Description |
|---|---|
| `PINCP` | Personal income (target for ML) |
| `AGEP` | Age |
| `SCHL` | Educational attainment |
| `OCCP` / `OCCP_GROUP` | Occupation code / grouped category |
| `RAC1P` | Race |
| `WKHP` | Hours worked per week |
| `WKWN` | Weeks worked per year |
| `SEX` | Sex (1 = Male, 2 = Female) |

---

## Notes

- Texas (`tx`) data is loaded but excluded from the final combined export — verify intentionality before rerunning.
- BLS web scraping relies on page structure at `bls.gov`; results may vary if the site changes.
- Plotly exports require `kaleido` and a compatible Chrome installation in the Colab environment.
