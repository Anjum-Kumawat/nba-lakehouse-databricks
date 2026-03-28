# 🏀 NBA Stats Databricks Lakehouse

A production-grade data engineering project implementing the **Medallion Architecture** (Bronze → Silver → Gold) on **Databricks**, powered by NBA statistics from 2000–2026, with an interactive **Power BI dashboard**.

---

## 📌 Project Overview

This project builds a fully automated data lakehouse pipeline that ingests raw NBA player and team statistics, cleans and enriches them through multiple transformation layers, and surfaces analytics-ready data in a Power BI dashboard — all built on Databricks Community Edition with Unity Catalog governance.

**Key questions answered by this project:**
- Who are the most efficient scorers in the modern NBA era (2000–2026)?
- Which franchises have consistently dominated since 2000?
- How does scoring volume correlate with player efficiency (PER)?

---

## 🏗️ Architecture
```
┌─────────────────────────────────────────────────────┐
│                   Data Sources                      │
│    NBA Stats CSVs (basketball-reference via Kaggle) │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│         Databricks Unity Catalog Volume             │
│   /Volumes/nba_lakehouse_catalog/raw/raw_files/     │
└─────────────────────┬───────────────────────────────┘
                      │
          ┌───────────▼───────────┐
          │                       │
          ▼                       ▼
┌──────────────────┐   ┌──────────────────┐
│  🥉 Bronze Layer │   │  Medallion       │
│  Raw Delta tables│   │  Architecture    │
│  No transforms   │   │  Raw → Clean     │
└────────┬─────────┘   │  → Analytics     │
         │             └──────────────────┘
         ▼
┌──────────────────┐
│  🥈 Silver Layer │
│  Cleaned + typed │
│  NBA only, 2000+ │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  🥇 Gold Layer   │
│  Joined+enriched │
│  Custom metrics  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  📊 Power BI     │
│  3-page dashboard│
│  DirectQuery     │
└──────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| Compute | Databricks Community Edition |
| Storage | Databricks Unity Catalog (managed volumes) |
| Data Format | Delta Lake (ACID transactions, time travel) |
| Governance | Unity Catalog (3-level namespace) |
| Transformation | PySpark (Python) |
| Visualisation | Power BI Desktop (DirectQuery) |
| Version Control | Git + GitHub |

---

## 📂 Project Structure
```
nba-lakehouse-databricks/
│
├── notebooks/
│   ├── 01_Raw_To_Bronze.ipynb      # CSV → Delta tables (raw ingestion)
│   ├── 02_Bronze_To_Silver.ipynb   # Cleaning, type casting, filtering
│   └── 03_Silver_To_Gold.ipynb     # Joins, aggregations, custom metrics
│
├── dashboard/
│   └── NBA_Lakehouse_Dashboard.pbix  # Power BI dashboard (3 pages)
│
└── README.md
```

---

## 📊 Dataset

**Source:** [NBA Players Stats — basketball-reference.com](https://www.kaggle.com/datasets/sumitrodatta/nba-aba-baa-stats) via Kaggle

| File | Description | Rows |
|------|-------------|------|
| Player Per Game.csv | Per-game stats per player-season | 33,278 |
| Advanced.csv | Advanced metrics (PER, WS, BPM, VORP) | 33,278 |
| Team Stats Per Game.csv | Team-level per-game stats | 1,907 |
| Team Summaries.csv | Team season summaries with ratings | 1,907 |
| Player Play By Play.csv | Play-by-play derived metrics | 18,193 |
| Player Career Info.csv | Player biographical data | 5,396 |

---

## 🔄 Pipeline Walkthrough

### 🥉 Bronze — Raw Ingestion
- Source CSVs stored in a **Databricks Unity Catalog Volume**
- Each file read with `inferSchema=True` and written as a **Delta table**
- **Zero transformations** — raw data preserved for full lineage
- Tables registered in `nba_lakehouse_catalog.bronze`

### 🥈 Silver — Cleaning & Validation
Transformations applied:
- **League filter:** `lg == 'NBA'` (removes historical ABA/BAA data)
- **Season filter:** `season >= 2000` (modern era only)
- **NA handling:** String `'NA'` values replaced with `null` before type casting
- **Type casting:** Numeric string columns cast to `double` or `integer`
- **Feature engineering:** `win_pct = w / (w + l)` added to team_summaries
- **Null dropping:** Rows missing critical identity fields removed

Tables registered in `nba_lakehouse_catalog.silver`

### 🥇 Gold — Analytics Layer
Three analytics-ready tables built for Power BI:

**`player_season_summary`** — joins player_per_game + advanced stats on 4-key join

**`team_performance_agg`** — team KPIs including win%, offensive/defensive ratings, pace, playoff flag

**`clutch_scoring_analysis`** — filters to ≥20 MPG + ≥20 games players with two custom metrics:
- `scoring_efficiency = (pts_per_game × fg_percent) + (ast_per_game × 0.5)`
- `all_around_score = pts + ast + reb + stl + blk`

Tables registered in `nba_lakehouse_catalog.gold`

---

## 📈 Power BI Dashboard

3-page interactive dashboard connected to Databricks via **DirectQuery**:

| Page | Title | Key Visuals |
|------|-------|-------------|
| 1 | Player Performance Analysis | Top 10 scorers bar chart, stats table, highest PPG card |
| 2 | Team Performance Analysis | Winningest teams bar chart, O-rtg vs D-rtg scatter |
| 3 | Clutch Scoring & Efficiency | Top 10 efficiency bar chart, PPG vs PER scatter |

---

## 🔑 Key Findings

| Insight | Value |
|---------|-------|
| Most efficient scorer (all time) | Nikola Jokić — 2025 (score: 22.15) |
| Winningest franchise since 2000 | San Antonio Spurs (~60% win rate) |
| Highest single-season PPG | 36.10 — Kobe Bryant (2005-06) |
| Best two-way player | Giannis Antetokounmpo (top 3 efficiency + all-around) |

---

## ⚙️ Unity Catalog Structure
```
nba_lakehouse_catalog/
├── raw/
│   └── raw_files/          ← Unity Catalog Volume (CSV files)
├── bronze/
│   ├── player_per_game     ← 33,278 rows
│   ├── advanced            ← 33,278 rows
│   ├── team_stats_per_game ← 1,907 rows
│   ├── team_summaries      ← 1,907 rows
│   ├── player_play_by_play ← 18,193 rows
│   └── player_career_info  ← 5,396 rows
├── silver/
│   ├── player_per_game     ← 16,565 rows (cleaned)
│   ├── advanced            ← 16,565 rows
│   ├── team_summaries      ← 805 rows
│   ├── team_stats_per_game ← 832 rows
│   ├── player_play_by_play ← 16,565 rows
│   └── player_career_info  ← 5,396 rows
└── gold/
    ├── player_season_summary    ← 16,565 rows
    ├── team_performance_agg     ← 805 rows
    └── clutch_scoring_analysis  ← 7,190 rows
```

---

## 🚀 How to Run

### Prerequisites
- Databricks account (Community Edition is free)
- Power BI Desktop (free download)

### Steps

1. **Clone the repo**
```bash
git clone https://github.com/Anjum-Kumawat/nba-lakehouse-databricks.git
```

2. **Set up Databricks Unity Catalog**
   - Create catalog `nba_lakehouse_catalog`
   - Create schemas: `raw`, `bronze`, `silver`, `gold`
   - Create volume `raw_files` under `raw` schema
   - Upload the 6 CSV files from the dataset

3. **Run notebooks in order**
```
01_Raw_To_Bronze.ipynb  →  02_Bronze_To_Silver.ipynb  →  03_Silver_To_Gold.ipynb
```

4. **Open Power BI Dashboard**
   - Open `NBA_Lakehouse_Dashboard.pbix`
   - Update Databricks connection with your workspace hostname and HTTP path
   - Authenticate with your personal access token

---

## 👤 Author

**Anjum Kumawat**
- M.Sc. Data Engineering & Cloud Computing — Aivancity Paris
- Ex IIIT Bangalore
- GitHub: [github.com/Anjum-Kumawat](https://github.com/Anjum-Kumawat)

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).