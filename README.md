# 🎮 Video Game Sales & Engagement Analysis

A full end-to-end data analytics project that combines **video game sales data** with **player engagement metrics** to uncover insights about market trends, regional preferences, and the relationship between commercial success and player interest.

---

## 📌 Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [Dataset Description](#dataset-description)
- [Project Architecture](#project-architecture)
- [Setup & Installation](#setup--installation)
- [Data Pipeline](#data-pipeline)
- [Dashboard Pages](#dashboard-pages)
- [Key Insights](#key-insights)
- [Folder Structure](#folder-structure)
- [Future Improvements](#future-improvements)

## Project Overview

This project analyzes over **16,000+ video game records** spanning from 1980 to 2024, merging two datasets to explore:

- Which games, genres, and platforms sell the most?
- How do player ratings, wishlists, and backlogs relate to commercial success?
- What are regional differences in gaming preferences?
- How has the gaming market evolved over decades?

The output is a **9-page interactive Power BI dashboard** answering 30+ analytical questions.


## Tech Stack

| Layer | Tools Used |
|---|---|
| Data Cleaning & EDA | Python, Pandas, Jupyter Notebook |
| Database | PostgreSQL (pgAdmin, port 2423) |
| DB Connection | psycopg2, SQLAlchemy |
| Visualization | Power BI Desktop |
| Data Format | CSV → PostgreSQL → Power BI |

---

## Dataset Description

### 1. `vgsales.csv` — Video Game Sales
| Column | Description |
|---|---|
| `Name` | Game title |
| `Platform` | Console/platform (PS2, Wii, X360, etc.) |
| `Year` | Release year |
| `Genre` | Game genre (Action, Sports, RPG, etc.) |
| `Publisher` | Publishing company |
| `NA_Sales` | North America sales (in millions) |
| `EU_Sales` | Europe sales (in millions) |
| `JP_Sales` | Japan sales (in millions) |
| `Other_Sales` | Rest of world sales (in millions) |
| `Global_Sales` | Total global sales (in millions) |

### 2. `games.csv` — Player Engagement
| Column | Description |
|---|---|
| `title` | Game title (used for joining) |
| `rating` | Average user rating (0–5) |
| `plays` | Number of users who have played |
| `wishlist` | Number of users who wishlisted |
| `backlogs` | Number of users with the game in backlog |
| `genres` | Genre tags (multi-value) |
| `team` | Developer/studio name |
| `year` | Release year |

> **Note:** Both datasets are merged on normalized game titles using an inner join in Pandas before being loaded into PostgreSQL.

---

## Project Architecture

```
Raw CSVs
   │
   ▼
Python / Jupyter Notebook
  ├── Data Cleaning (null handling, K/M suffix parsing, genre normalization)
  ├── Exploratory Data Analysis (EDA)
  └── Merged DataFrame (inner join on normalized title)
   │
   ▼
PostgreSQL Database
  ├── Database: videogame_analysis
  ├── Port: 2423
  └── Tables: vgsales, games, merged_data
   │
   ▼
Power BI Dashboard
  └── 9 Pages | 30+ Visuals | Interactive Slicers
```

---

## Setup & Installation

### Prerequisites

- Python 3.8+
- PostgreSQL 13+ with pgAdmin
- Power BI Desktop (Windows)

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/videogame-sales-analysis.git
cd videogame-sales-analysis
```

### 2. Install Python Dependencies

```bash
pip install -r requirements.txt
```

**`requirements.txt`:**
```
pandas
numpy
matplotlib
seaborn
sqlalchemy
psycopg2-binary
jupyter
```

### 3. Set Up PostgreSQL Database

```sql
-- Run in pgAdmin or psql
CREATE DATABASE videogame_analysis;
```

### 4. Configure DB Connection

Update the connection string in the notebook:

```python
from sqlalchemy import create_engine

engine = create_engine(
    "postgresql+psycopg2://postgres:<your_password>@localhost:2423/videogame_analysis"
)
```

### 5. Run the Jupyter Notebook

```bash
jupyter notebook
```

Open `data_pipeline.ipynb` and run all cells. This will:
- Clean and merge the datasets
- Load the data into PostgreSQL

### 6. Connect Power BI to PostgreSQL

1. Open Power BI Desktop
2. Go to **Get Data → PostgreSQL Database**
3. Server: `localhost:2423`
4. Database: `videogame_analysis`
5. Load the required tables

---

## Data Pipeline

### Step 1 — Data Cleaning (`vgsales.csv`)
- Dropped rows with missing `Year` and `Publisher`
- Converted `Year` from float to int
- Filled missing sales columns with `0`

### Step 2 — Data Cleaning (`games.csv`)
- Parsed `plays`, `wishlist`, `backlogs` columns — converted `K`/`M` suffixes to numeric values
- Normalized `title` field: lowercased, stripped punctuation for joining
- Exploded multi-value `genres` column where needed

### Step 3 — Merging
```python
# Normalize titles for matching
vgsales['title_clean'] = vgsales['Name'].str.lower().str.strip()
games['title_clean'] = games['title'].str.lower().str.strip()

merged_df = pd.merge(vgsales, games, on='title_clean', how='inner')
```

### Step 4 — Load to PostgreSQL
```python
merged_df.to_sql('merged_data', engine, if_exists='replace', index=False)
```

---

## Dashboard Pages

| Page | Title | Key Visuals |
|---|---|---|
| 1 | **Overview** | KPI cards, Top 10 rated & wishlisted games, Games by Genre treemap |
| 2 | **Engagement Deep Dive** | Top developers by rating, Avg plays per genre, Backlog vs Wishlist comparison |
| 3 | **Release & Rating Trends** | Games released per year (1980–2024), Rating distribution histogram |
| 4 | **Sales Overview** | KPI cards, Top 10 best-selling games, Best-selling platforms, Regional donut chart |
| 5 | **Platform Sales Analysis** | Publisher sales bar, Platform trend lines, Regional sales table, Top games by platform |
| 6 | **Regional & Publisher Analysis** | Yearly sales by region, Regional genre preferences, Avg sales per publisher |
| 7 | **Sales vs Engagement Correlation** | Genre vs Global Sales, Rating vs Sales scatter, High-rated platforms |
| 8 | **Hidden Engagement Insights** | High Engagement–Low Sales genres, Wishlist vs Sales scatter, Bubble chart |
| 9 | **Genre × Platform Matrix** | Cross-tab matrix (Genre × Platform), Regional Sales by Genre table |

---

## Key Insights

- 🌎 **North America** dominates global sales at ~49%, followed by EU at ~27%
- 🏆 **Wii Sports** is the best-selling game of all time in this dataset (83M units)
- 🎮 **Wii** is the top-selling platform; **PS3, X360, PC** are the highest-rated
- 🏢 **Nintendo** leads as the #1 publisher by both total and average sales
- ⚔️ **Action** genre leads in global sales across all regions
- 🔍 **Outer Wilds & Disco Elysium** top the wishlist charts — high engagement despite modest sales
- 📈 Global sales **peaked around 2008–2009**, then declined sharply post-2015 despite more releases
- ⭐ Most games cluster in the **3.5–4.5 rating** range — very few outliers

---

## Folder Structure

```
videogame-sales-analysis/
│
├── data/
│   ├── vgsales.csv               # Raw sales dataset
│   └── games.csv                 # Raw engagement dataset
│
├── notebooks/
│   └── data_pipeline.ipynb       # Cleaning, EDA, and DB loading
│
├── powerbi/
│   └── video_game_dashboard.pbix # Power BI dashboard file
│
├── exports/
│   └── video_game_dashboard.pdf  # Exported dashboard PDF
│
├── requirements.txt
└── README.md
```

---

## Future Improvements

- [ ] Add a **Streamlit web app** for interactive exploration without Power BI
- [ ] Expand dataset with **Steam/Metacritic** data via API scraping
- [ ] Build a **recommendation engine** based on genre + rating similarity
- [ ] Add **time-series forecasting** for global sales trends
- [ ] Automate the ETL pipeline using **Apache Airflow** or **Prefect**

---

## 👤 Author

**Vikrant Sonawane**
- 📍 Kalyan, Maharashtra, India
- 📧 vikrantsonawane24@gmail.com
- 🔗 https://www.linkedin.com/in/vikrantsonawane24/

---

> *This project was built as part of a structured data analytics learning journey covering Python, SQL, and Business Intelligence tools.*
