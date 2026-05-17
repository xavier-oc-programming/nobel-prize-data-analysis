![Python](https://img.shields.io/badge/Python-3.9+-blue)
![pandas](https://img.shields.io/badge/pandas-2.0+-blue)
![NumPy](https://img.shields.io/badge/NumPy-1.24+-blue)
![Matplotlib](https://img.shields.io/badge/Matplotlib-3.7+-orange)
![Seaborn](https://img.shields.io/badge/Seaborn-0.12+-orange)
![Plotly](https://img.shields.io/badge/Plotly-5.14+-purple)
![Jupyter](https://img.shields.io/badge/Jupyter-7.0+-orange)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-black)

# Nobel Prize Data Analysis

The United States accounts for 281 Nobel Prizes — more than twice the United Kingdom's 105 — and that dominance only solidified after 1945. This analysis traces 120+ years of Nobel Prize data to map where prizes are won, who wins them, and how the patterns have shifted across gender, geography, age, and category.

The dataset covers 962 individual prize shares from 1901 to 2023. It is cleaned and enriched at runtime: birth dates are parsed to datetime, prize fractions (e.g. `1/2`) are converted to percentages, and a `winning_age` column is computed for each laureate. Aggregations using `groupby`, `cumsum`, and 5-year rolling averages then surface the structural trends underneath the headline numbers.

Women hold approximately 6.2% of all prizes — 58 out of the 934 individual laureates with sex recorded. The gap is widest in Physics, where only 4 of 216 prizes went to women, and narrowest in Peace and Literature. The average laureate is 60 years old at the time of the award; the youngest ever was Malala Yousafzai at 17, the oldest John Goodenough at 97.

---

## Table of Contents

1. [Quick Start](#1-quick-start)
2. [Analysis Flow](#2-analysis-flow)
3. [Key Findings](#3-key-findings)
4. [Dataset Schema](#4-dataset-schema)
5. [Architecture](#5-architecture)
6. [Visualisations](#6-visualisations)
7. [Operations Reference](#7-operations-reference)
8. [Background](#8-background)
9. [Dependencies](#9-dependencies)
10. [Portfolio Integration](#10-portfolio-integration)

---

## 1. Quick Start

```bash
git clone https://github.com/xavier-oc-programming/nobel-prize-data-analysis.git
cd nobel-prize-data-analysis
pip install -r requirements.txt
jupyter notebook
```

Open `notebooks/analysis/A_01_Full_Analysis.ipynb` to run the complete analysis. Concept notes and method explanations are in `notebooks/concepts/`.

---

## 2. Analysis Flow

```
data/nobel_prize_data.csv
        │
        ▼
pd.read_csv('../../data/nobel_prize_data.csv')  →  df_data
        │
        │  ── CLEAN ──────────────────────────────────────────────────────
        ├── .isna().sum()               →  identify missing values per column
        ├── pd.to_datetime(birth_date)  →  datetime dtype
        ├── prize_share.str.split('/')  →  share_pct (float, 0–100)
        ├── year − birth_date.dt.year   →  winning_age (int)
        │
        │  ── AGGREGATE ──────────────────────────────────────────────────
        ├── .value_counts()             →  prizes per category / org / city
        ├── .groupby().agg(count)       →  prizes by country, category
        ├── .groupby().cumsum()         →  cumulative prizes per country over time
        ├── .rolling(window=5).mean()   →  5-year moving average
        ├── pd.merge()                  →  join category breakdown → top-20 countries
        │
        │  ── VISUALISE ──────────────────────────────────────────────────
        ├── px.pie / px.bar             →  gender split, category counts
        ├── px.choropleth               →  world map of prizes
        ├── px.sunburst                 →  country → city → organisation drill-down
        ├── px.line                     →  cumulative prizes over time
        ├── matplotlib scatter + line   →  prizes per year + rolling average
        └── sns.histplot / regplot / lmplot  →  age distributions and trends
```

---

## 3. Key Findings

- **US dominance**: The United States has 281 prizes — 29% of all individual prizes. It overtook Germany and the UK as the cumulative leader during the post-WWII period and has extended that lead every decade since.
- **Gender gap**: 58 of 934 individually tracked prizes went to women (6.2%). Peace and Literature show the highest female share; Physics has the lowest at 4 out of 216.
- **Female firsts**: Marie Curie (Physics, 1903) was the first woman to win; all three early female laureates were born in countries that no longer exist under those names — Russian Empire, Austrian Empire, Sweden.
- **Repeat winners**: Six winners have received the prize more than once — including Marie Curie (Physics 1903, Chemistry 1911) and Linus Pauling (Chemistry 1954, Peace 1962).
- **Prize inflation**: The number of annual prizes has risen over time, driven largely by more prizes being shared. The average laureate share has fallen as the number of co-recipients increased.
- **World Wars**: Both World War I and World War II produced clear dips in the number of prizes awarded, visible in the rolling average chart.
- **Research geography**: Harvard University and the University of Chicago are the two leading affiliated institutions. New York is the top laureate birth city in the dataset.
- **Laureate age**: The average winning age is approximately 60. The LOWESS trend line shows laureates were around 55–57 at the time of the award in the early 20th century and now trend closer to 65–70 — suggesting prizes increasingly recognise work done decades earlier.
- **Category ages**: Physics and Chemistry have the highest average winning ages; Peace has the widest spread and the longest "whiskers" in the boxplot.
- **Economics**: The Economics prize was not part of Alfred Nobel's original 1895 will. It was first awarded in 1969 to Jan Tinbergen and Ragnar Frisch and has the fewest total prizes (86) of any category.

---

## 4. Dataset Schema

### `data/nobel_prize_data.csv`

| Column | Type | Description |
|--------|------|-------------|
| year | int | Year the prize was awarded |
| category | str | Nobel Prize category (Chemistry, Economics, Literature, Medicine, Peace, Physics) |
| prize | str | Full prize name |
| motivation | str | Official motivation text; NaN for some entries |
| prize_share | str | Laureate's fraction of the prize (e.g. `1/2`) |
| laureate_type | str | `Individual` or `Organisation` |
| full_name | str | Full name of the laureate |
| birth_date | str → datetime | Date of birth; NaN for organisations and a few individuals |
| birth_city | str | City of birth; NaN for organisations |
| birth_country | str | Country of birth as it was at the time |
| birth_country_current | str | Country of birth using current borders |
| sex | str | `Male`, `Female`, or NaN for organisations |
| organization_name | str | Affiliated institution; NaN for Peace/Literature |
| organization_city | str | City of affiliated institution |
| organization_country | str | Country of affiliated institution |
| ISO | str | ISO 3166-1 alpha-3 country code for birth_country_current |

**Computed columns added at runtime**

| Column | Type | Description |
|--------|------|-------------|
| share_pct | float | Prize share as a percentage (0–100) |
| winning_age | int | Laureate's age in the year of the award |

---

## 5. Architecture

```
nobel-prize-data-analysis/
│
├── notebooks/
│   ├── analysis/
│   │   └── A_01_Full_Analysis.ipynb       # Complete end-to-end analysis notebook
│   └── concepts/
│       ├── 00__Overview.ipynb             # Project overview and goals
│       ├── 01__Loading_and_Cleaning.ipynb # Data exploration, type conversion
│       ├── 02__Plotly_Charts.ipynb        # Donut chart, bar charts, gender/category analysis
│       ├── 03__Matplotlib_Trends.ipynb    # Scatter + rolling average, dual y-axes
│       ├── 04__Choropleth_and_Countries.ipynb  # Top countries, choropleth map, cumulative lines
│       ├── 05__Sunburst_Charts.ipynb      # Research institutions, cities, sunburst drill-down
│       ├── 06__Age_Analysis.ipynb         # Winning age distribution, trends, category comparison
│       └── 07__Summary.ipynb              # Key method recap
│
├── data/
│   └── nobel_prize_data.csv               # Nobel Prize laureates 1901–2023 (~270 KB)
│
├── plots/                                 # Plotly charts as .html; Matplotlib/Seaborn as .png at 150 dpi
│
├── notebook_web_render/
│   └── index.html                         # Rendered notebook (outputs only, no code)
│
├── docs/
│   └── COURSE_NOTES.md
│
├── .github/
│   └── workflows/
│       └── publish_notebook.yml           # Auto-renders notebook on push
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## 6. Visualisations

Plotly charts are saved as interactive `.html`; Matplotlib/Seaborn charts as `.png` at 150 dpi.

| File | Type | Description |
|------|------|-------------|
| `gender_split_donut.html` | Plotly | Donut chart: male vs. female prize share (%) |
| `prizes_per_category.html` | Plotly | Bar chart: total prizes by category |
| `gender_by_category.html` | Plotly | Stacked bar: male/female split per category |
| `top20_countries.html` | Plotly | Horizontal bar: top 20 countries by total prizes |
| `world_choropleth.html` | Plotly | Choropleth map: prizes by birth country |
| `categories_by_country.html` | Plotly | Stacked horizontal bar: category breakdown by country |
| `cumulative_prizes_by_country.html` | Plotly | Line chart: cumulative prizes over time, top 10 countries |
| `top20_organisations.html` | Plotly | Horizontal bar: top 20 research institutions |
| `top20_org_cities.html` | Plotly | Horizontal bar: top 20 organisation cities |
| `top20_birth_cities.html` | Plotly | Horizontal bar: top 20 laureate birth cities |
| `sunburst_country_city_org.html` | Plotly | Sunburst: country → city → organisation drill-down |
| `prizes_per_year.png` | Matplotlib | Scatter + 5-year rolling average: prizes per year |
| `prizes_per_year_with_share.png` | Matplotlib | Dual-axis: prizes per year and average prize share |
| `winning_age_distribution.png` | Seaborn | Histogram: age distribution at time of award |
| `winning_age_over_time.png` | Seaborn | Regression plot with LOWESS: age trend 1901–2023 |
| `winning_age_by_category_boxplot.png` | Seaborn | Boxplot: age distribution per category |
| `winning_age_by_category_lmplot.png` | Seaborn | Grid of LOWESS lines: one panel per category |
| `winning_age_all_categories_lmplot.png` | Seaborn | Combined LOWESS lines: all six categories overlaid |

---

## 7. Operations Reference

| Value | Location | Description |
|-------|----------|-------------|
| `'../../data/nobel_prize_data.csv'` | `notebooks/analysis/A_01_Full_Analysis.ipynb` | Relative path to dataset |
| `pd.options.display.float_format = '{:,.2f}'.format` | analysis notebook | Display floats with 2 decimal places |
| `rolling(window=5)` | analysis notebook | 5-year window for moving averages |
| `np.arange(1900, 2021, step=5)` | analysis notebook | X-axis tick marks every 5 years |
| `figsize=(16, 8), dpi=200` | analysis notebook | Standard figure size for Matplotlib charts |
| `color_continuous_scale='Aggrnyl'` | analysis notebook | Colour scale for prize-per-category bar chart |
| `color_continuous_scale=px.colors.sequential.matter` | analysis notebook | Colour scale for choropleth map |

---

## 8. Background

100 Days of Code: The Complete Python Pro Bootcamp — Day 79 — Data visualisation with Plotly, Matplotlib, and Seaborn.

See [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md) for the full exercise brief and method summary.

---

## 9. Dependencies

| Module | Used in | Purpose |
|--------|---------|---------|
| pandas | notebooks/ | Data loading, cleaning, aggregation |
| numpy | analysis notebook | Tick mark arrays, type handling |
| matplotlib | analysis notebook, concepts/03 | Scatter plots, rolling average overlays, dual axes |
| seaborn | analysis notebook, concepts/06 | Histplot, regplot, boxplot, lmplot |
| plotly | analysis notebook, concepts/02–05 | Interactive bar, pie, choropleth, sunburst, line charts |
| kaleido | analysis notebook | Static image export for Plotly charts |
| notebook | all | Jupyter notebook runtime |

---

## 10. Portfolio Integration

Rendered notebook (outputs and charts only, no code):
https://xavier-oc-programming.github.io/nobel-prize-data-analysis/notebook_web_render/

Regenerated automatically via GitHub Actions on every commit to `notebooks/analysis/A_01_Full_Analysis.ipynb`.

To regenerate manually:

```bash
jupyter nbconvert --to html --no-input \
  --output index.html \
  notebooks/analysis/A_01_Full_Analysis.ipynb
mv index.html notebook_web_render/index.html
```
