# Course Notes — Day 79: Analysing the Nobel Prize with Plotly, Matplotlib, and Seaborn

## Course
100 Days of Code: The Complete Python Pro Bootcamp — Day 79

## Topics Covered
- Data cleaning with pandas: handling NaN values, type conversion, computed columns
- Plotly Express: donut/pie charts, vertical and horizontal bar charts, choropleth maps, sunburst charts, line charts
- Matplotlib: scatter plots, rolling average overlays, dual y-axes
- Seaborn: histplot, regplot (lowess), boxplot, lmplot

## Exercise Brief
Analyse a dataset of Nobel Prize winners from 1901 onwards. The analysis answers:

1. **Gender split** — What percentage of all prizes went to women?  
   Who were the first 3 female laureates?
2. **Repeat winners** — Did any person win the Nobel Prize more than once?
3. **Prize categories** — How many prizes were awarded per category?  
   When was the Economics prize first given?
4. **Trends over time** — Are more prizes awarded now than in the early 1900s?  
   Are prizes more frequently shared?
5. **Countries** — Which countries produced the most laureates?  
   When did the United States eclipse all others?
6. **Research institutions & cities** — Which organisations and cities are most associated with laureates?
7. **Laureate age** — How old are laureates when they receive the prize?  
   Is the winning age increasing over time? Does it differ by category?

## Key Methods Practised
| Method | Purpose |
|--------|---------|
| `pd.to_datetime()` | Convert birth_date to datetime |
| `str.split('/', expand=True)` | Parse prize_share fraction |
| `.groupby().agg()` | Count prizes by category, country, organisation |
| `.rolling(window=5).mean()` | 5-year moving average |
| `.groupby().cumsum()` | Cumulative prize count per country |
| `pd.merge()` | Join category breakdown with top-20 country list |
| `.duplicated()` | Identify repeat winners |
| `.nlargest()` / `.nsmallest()` | Oldest and youngest laureates |
| `px.choropleth()` | World map coloured by prize count |
| `px.sunburst()` | Three-level breakdown: country → city → org |
| `sns.regplot(lowess=True)` | Trend line for age over time |
| `sns.lmplot(row='category')` | Separate trend chart per category |

## Dataset
- **File**: `data/nobel_prize_data.csv`
- **Source**: Kaggle / Nobel Prize Foundation public dataset
- **Rows**: ~1,000 laureate records (1901–2023)
- **Caveats**: Birth dates for three laureates (Michael Houghton, Venkatraman Ramakrishnan, Nadia Murad) are unknown; substituted with mid-year estimate (July 2).
