# Tableau Dashboard — Build Guide

This folder contains the analysis-ready dataset and step-by-step instructions for building the studio's genre/international/studio strategy dashboard in **Tableau Desktop or Tableau Public**.

> **Note on this deliverable:** Tableau Desktop is a licensed, GUI-based application that cannot be run from this environment. Instead of a `.twbx` workbook, this folder provides (1) a fully cleaned, analysis-ready CSV formatted for a direct Tableau connection, (2) exact step-by-step build instructions for each chart below, and (3) `dashboard_preview.html` — a fully interactive, browser-based preview of the finished dashboard (built with Plotly) so the layout and findings can be explored immediately without Tableau installed. Opening the CSV in Tableau and following the steps below reproduces the same three charts natively as Tableau worksheets.

## Files

| File | Purpose |
|---|---|
| `tableau_movie_dataset.csv` | 1,042 rows, one per film — connect Tableau directly to this file |
| `dashboard_preview.html` | Interactive preview of the full dashboard — open in any browser |

## Dataset fields

| Field | Type | Description |
|---|---|---|
| `Movie` | String | Film title |
| `Year` | Number | Release year |
| `Studio` | String | Distributing studio (Box Office Mojo abbreviation, e.g. `BV`, `Uni.`, `WB (NL)`) |
| `Production_Budget` | Number | Budget in USD |
| `Domestic_Gross` | Number | US box office in USD |
| `Foreign_Gross` | Number | Non-US box office in USD (missing for ~10% of rows) |
| `Worldwide_Gross` | Number | Global box office in USD |
| `ROI` | Number | (Worldwide gross − budget) / budget |
| `Profit_USD_Millions` | Number | Profit in $M |
| `International_Share` | Number | Foreign_Gross / (Domestic_Gross + Foreign_Gross) |
| `Genres` | String | Comma-separated list, e.g. `Action,Adventure,Sci-Fi` |
| `Runtime_Minutes` | Number | Film runtime |
| `IMDb_Rating` | Number | Average IMDb user rating (0–10) |
| `IMDb_Num_Votes` | Number | Number of IMDb ratings submitted |

## Step-by-step: rebuilding the dashboard

### 0. Connect and prepare
1. Open Tableau → **Connect → Text File** → select `tableau_movie_dataset.csv`.
2. Split `Genres` into rows so a film with 3 genres contributes 3 rows: right-click the `Genres` column header in the Data Source pane → **Split** (use a Custom Split on `,` with "Split into rows" if the default doesn't create one row per value).
3. Create a calculated field `Is Top Genre` (fill in your own top-5 genres if they differ from Mystery/Horror/Sci-Fi/Animation/Thriller once you've built Worksheet 1):
   ```
   IF [Genres] = "Mystery" OR [Genres] = "Horror" OR [Genres] = "Sci-Fi"
      OR [Genres] = "Animation" OR [Genres] = "Thriller" THEN 1 ELSE 0 END
   ```

### 1. Worksheet — "Genre vs. Financial Return"
- **Rows:** `Genres` (split), sorted descending by ROI
- **Columns:** `ROI` → right-click → **Measure = Median**
- **Marks type:** Bar
- **Filter:** `{FIXED [Genres]: COUNTD([Movie])} >= 15` (matches the 15-film minimum used in the notebook)
- **Color:** Highlight the top 3 genres (Mystery, Horror, Sci-Fi) in dark navy (`#1E3D59`), all others in gray (`#A6A6A6`)
- **Labels:** Median ROI, custom number format `0.00"x"`
- **Tooltip:** Also show `MEDIAN([Profit_USD_Millions])` so both financial-return metrics from the business question are visible together

### 2. Worksheet — "Genre vs. International Audience"
- **Rows:** `Genres` (split), sorted descending by `International_Share`
- **Columns:** `AVG([International_Share])` → set **Measure = Median**, format as percentage
- **Marks type:** Bar
- **Filter:** Same 15-film minimum as above; also filter out rows where `Foreign_Gross` is null (Tableau does this automatically when the field is used, since a null numerator produces a null share)
- **Reference line:** Add a reference line at the overall median `International_Share` (right-click axis → Add Reference Line → Median), labeled "All-genre median"
- **Color:** Genres above the reference line in teal (`#3FA796`), below in gray (`#A6A6A6`)

### 3. Worksheet — "Studio Performance vs. Genre Alignment"
- **Columns:** `AVG([Is Top Genre])` aggregated per studio (as a percentage) — or build a calculated field `Pct Top Genre` using a `{FIXED [Studio]: AVG([Is Top Genre])}` LOD expression
- **Rows:** `MEDIAN([ROI])` per studio
- **Marks type:** Circle (scatter plot)
- **Size:** `COUNTD([Movie])` per studio
- **Filter:** `{FIXED [Studio]: COUNTD([Movie])} >= 8`
- **Reference line:** Vertical reference line at the market-average `Pct Top Genre` value
- **Labels:** Label the top 5 studios by median ROI with their studio code
- **Color:** Top-5 studios in navy, others in gray

### 4. Dashboard assembly
1. **New Dashboard** → set size to **1300 × 900 (px), Automatic**.
2. Drag all three worksheets onto the canvas (e.g. 2 charts on top, 1 wide chart + a text summary box below).
3. Add a **Text** object at the top titled *"Movie Studio Strategy: Genre, International Reach & Studio Benchmarking"* with a one-line subtitle stating the data source and film count (n = 1,042).
4. Add filter actions: right-click the Genre ROI worksheet → **Use as Filter**, so clicking a genre bar filters the studio scatter plot to studios with films in that genre.
5. Publish to Tableau Public (or export as `.twbx`) and link it from the main project `README.md`.

## Design notes
- Color palette matches the notebook/presentation for consistency: navy `#1E3D59` (financial return), teal `#3FA796` (international audience), gold `#FFC13B` (accent), gray `#A6A6A6` (non-highlighted).
- Keep axis titles, tooltips, and a footer note on data source visible on every worksheet per the project's visualization checklist (labeled axes, no clutter, comparisons aided by color).
