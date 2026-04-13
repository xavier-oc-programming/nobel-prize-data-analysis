# Nobel Prize Data Analysis

Analyses 120+ years of Nobel Prize data to uncover trends in gender, country, research institutions, and laureate age using Plotly, Matplotlib, and Seaborn.

This project explores the Nobel Prize dataset from 1901 onwards to answer a series of questions about the history and patterns of one of the world's most prestigious awards. It investigates the gender split across all categories, identifies which countries dominate the prize, tracks the growth in awards over time, and examines whether laureates are winning at an older age today than they were a century ago.

The dataset is a curated CSV file covering individual laureate records, including birth country, affiliated research institution, prize category, year of award, and prize share. After loading, the data is cleaned — handling NaN values and type conversions — and enriched with two computed columns: `share_pct` (the laureate's share as a decimal percentage) and `winning_age` (the laureate's age in the year of the award). Aggregations are performed with `groupby`, `cumsum`, and rolling averages to surface trends.

No external APIs or credentials are required. All data is bundled in `data/nobel_prize_data.csv`.

---

## Table of Contents

1. [Quick start](#1-quick-start)
2. [Analysis flow](#2-analysis-flow)
3. [Features](#3-features)
4. [Dataset schema](#4-dataset-schema)
5. [Architecture](#5-architecture)
6. [Notebook reference](#6-notebook-reference)
7. [Configuration reference](#7-configuration-reference)
8. [Course context](#8-course-context)
9. [Dependencies](#9-dependencies)

---

## 1. Quick start

```bash
git clone https://github.com/xavier-oc-programming/nobel-prize-data-analysis.git
cd nobel-prize-data-analysis
pip install -r requirements.txt
jupyter notebook
```

Open `practice/A_01_Full_Analysis.ipynb` to run the complete analysis. For lesson notes and method explanations, start with `theory/00__Overview.ipynb`.

---

## 2. Analysis flow

```
data/nobel_prize_data.csv
        │
        ▼
pd.read_csv('../data/nobel_prize_data.csv')  →  df_data
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

## 3. Features

**Gender & firsts**
- Donut chart: percentage of prizes won by men vs. women
- First 3 female laureates identified with year, category, and birth country

**Repeat winners**
- All individuals who won the Nobel Prize more than once, with categories and years

**Prize categories**
- Total prizes awarded per category (bar chart, colour scale)
- Male/female split per category (stacked bar chart)
- Year the Economics prize was first awarded

**Trends over time**
- Prizes awarded per year with 5-year rolling average (Matplotlib scatter + line)
- Average prize share per year with 5-year rolling average on a secondary axis

**Countries**
- Top 20 countries by total prizes (horizontal bar chart)
- Prizes per country broken down by category (stacked horizontal bar)
- World choropleth map coloured by prize count
- Cumulative prizes over time for the top 10 countries (line chart)

**Research institutions & cities**
- Top 20 organisations by laureate count (horizontal bar)
- Top 20 organisation cities (horizontal bar)
- Top 20 laureate birth cities (horizontal bar)
- Sunburst chart: country → city → organisation breakdown

**Laureate age**
- Descriptive statistics for winning age (youngest, oldest, average, 75th percentile)
- Distribution histogram (Seaborn histplot)
- Age over time with LOWESS trend line (Seaborn regplot)
- Age distribution per category (Seaborn boxplot)
- Per-category age trend lines (Seaborn lmplot, one chart per category)
- Combined trend lines for all categories (Seaborn lmplot with hue)

---

## 4. Dataset schema

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
├── theory/                            # Lesson notes — annotated methods, no raw exercises
│   ├── 00__Overview.ipynb             # Day goals and project overview
│   ├── 01__Loading_and_Cleaning.ipynb # Package setup, data exploration, type conversion
│   ├── 02__Plotly_Charts.ipynb        # Donut chart, bar charts, gender/category analysis
│   ├── 03__Matplotlib_Trends.ipynb    # Scatter + rolling average, dual y-axes
│   ├── 04__Choropleth_and_Countries.ipynb  # Top countries, choropleth map, cumulative lines
│   ├── 05__Sunburst_Charts.ipynb      # Research institutions, cities, sunburst drill-down
│   ├── 06__Age_Analysis.ipynb         # Winning age distribution, trends, category comparison
│   └── 07__Summary.ipynb              # Learning points and method recap
│
├── practice/                          # Student's actual analysis code
│   └── A_01_Full_Analysis.ipynb       # Complete end-to-end analysis notebook
│
├── data/                              # Seed dataset
│   └── nobel_prize_data.csv           # Nobel Prize laureates 1901–2023 (~270 KB)
│
├── docs/
│   └── COURSE_NOTES.md                # Exercise brief and key methods
│
├── requirements.txt                   # Pinned package versions
├── .gitignore
└── README.md
```

---

## 6. Notebook reference

### theory/

| Notebook | Key methods covered | Question answered |
|----------|--------------------|--------------------|
| 00__Overview.ipynb | — | What does this project build? |
| 01__Loading_and_Cleaning.ipynb | `pd.read_csv`, `.isna()`, `.duplicated()`, `pd.to_datetime`, `str.split` | How is the data structured and cleaned? |
| 02__Plotly_Charts.ipynb | `px.pie`, `px.bar`, `.value_counts`, `.groupby().agg` | Gender split; prizes per category; male/female by category |
| 03__Matplotlib_Trends.ipynb | `.rolling().mean()`, `ax.twinx()`, `np.arange` | Are more prizes awarded over time? Are they more shared? |
| 04__Choropleth_and_Countries.ipynb | `px.choropleth`, `px.bar`, `pd.merge`, `.cumsum()`, `px.line` | Which countries dominate? When did the US eclipse others? |
| 05__Sunburst_Charts.ipynb | `px.sunburst`, `.value_counts` | Which institutions and cities produce the most laureates? |
| 06__Age_Analysis.ipynb | `.dt.year`, `.describe()`, `sns.histplot`, `sns.regplot`, `sns.boxplot`, `sns.lmplot` | Age distribution; oldest/youngest; age trends by category |
| 07__Summary.ipynb | — | What were the key learning points? |

### practice/

| Notebook | Key methods covered | Question answered |
|----------|--------------------|--------------------|
| A_01_Full_Analysis.ipynb | All of the above | Complete Nobel Prize analysis — all questions |

---

## 7. Configuration reference

| Value | Location | Description |
|-------|----------|-------------|
| `'../data/nobel_prize_data.csv'` | `practice/A_01_Full_Analysis.ipynb` cell 9 | Relative path to dataset from practice/ |
| `pd.options.display.float_format = '{:,.2f}'.format` | practice notebook cell 7 | Display floats with 2 decimal places and comma separator |
| `rolling(window=5)` | practice notebook cells 53, 56 | 5-year window for moving averages |
| `np.arange(1900, 2021, step=5)` | practice notebook cell 54 | X-axis tick marks every 5 years |
| `figsize=(16, 8), dpi=200` | practice notebook cells 54, 57 | Standard figure size for Matplotlib charts |
| `color_continuous_scale='Aggrnyl'` | practice notebook cell 45 | Colour scale for prize-per-category bar chart |
| `color_continuous_scale=px.colors.sequential.matter` | practice notebook cell 67 | Colour scale for choropleth map |

---

## 8. Course context

100 Days of Code: The Complete Python Pro Bootcamp — Day 79 — Data visualisation with Plotly, Matplotlib, and Seaborn.

See [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md) for the full exercise brief and method summary.

---

## 9. Dependencies

| Module | Used in | Purpose |
|--------|---------|---------|
| pandas | practice/, theory/ | Data loading, cleaning, aggregation |
| numpy | practice/ | Tick mark arrays, type handling |
| matplotlib | practice/, theory/03 | Scatter plots, rolling average overlays, dual axes |
| seaborn | practice/, theory/06 | Histplot, regplot, boxplot, lmplot |
| plotly | practice/, theory/02–05 | Interactive bar, pie, choropleth, sunburst, line charts |
| notebook | all | Jupyter notebook runtime |
