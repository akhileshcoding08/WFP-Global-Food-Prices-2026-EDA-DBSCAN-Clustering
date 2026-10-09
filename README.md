# 🌾 WFP Global Food Prices 2026: EDA & DBSCAN Clustering

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![scikit-learn](https://img.shields.io/badge/scikit--learn-DBSCAN-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

Exploratory data analysis and density-based clustering (DBSCAN) of the **World Food Programme (WFP) Global Food Prices** dataset. The project cleans and explores 85,000+ price records from 60 countries, then groups **1,722 markets** by location and price level, while automatically flagging outlier markets as noise.

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Dataset](#-dataset)
- [Project Workflow](#-project-workflow)
- [Key Results](#-key-results)
- [Visual Highlights](#-visual-highlights)
- [Limitations](#-limitations)
- [Project Structure](#-project-structure)
- [Installation & Usage](#-installation--usage)
- [Future Work](#-future-work)
- [Author](#-author)

---

## 🔎 Overview

Food prices are an early-warning signal for food crises, yet they vary by orders of magnitude across commodities, currencies and countries, and not in a simple linear way. This project:

1. Assesses data quality and cleans the data defensibly.
2. Performs univariate, categorical, temporal, geographic and multivariate EDA.
3. Builds a market-level feature set `(latitude, longitude, mean log USD price)`.
4. Tunes **DBSCAN** using a k-distance graph and an `eps` sweep.
5. Evaluates and profiles the resulting clusters.

**Why DBSCAN?** It needs no preset number of clusters, finds arbitrarily shaped dense groups, and labels isolated markets as *noise* instead of forcing them into a cluster (unlike k-means).

---

## 📂 Dataset

| Item | Detail |
|---|---|
| File | `wfp_food_prices_global_2026.csv` |
| Source | [WFP Food Prices on the Humanitarian Data Exchange](https://data.humdata.org/dataset/wfp-food-prices) |
| Size | 85,289 rows × 17 columns |
| Coverage | 60 countries, 1,730 market names, 541 commodities, 8 categories, 51 currencies, 6 monthly snapshots (2026) |

> Please check the source's licence and citation terms before redistributing the raw data.

**Main columns:** `countryiso3`, `date`, `admin1`, `admin2`, `market`, `market_id`, `latitude`, `longitude`, `category`, `commodity`, `unit`, `priceflag`, `pricetype`, `currency`, `price`, `usdprice` (main analysis variable).

---

## 🛠️ Project Workflow

| # | Stage | What happens |
|---|---|---|
| 1 | Setup | Import pandas, NumPy, matplotlib, seaborn, scikit-learn |
| 2 | Load & inspect | Shape, dtypes, summary statistics, column reference |
| 3 | Missing values | 340 missing `usdprice`; 29 rows missing location (< 0.4%) |
| 4 | Cleaning | Parse dates, add `month`, drop unusable rows → **84,920 rows** |
| 5 | Outlier check | 21.63% above the IQR fence (22.94 USD); **kept**, handled with `log1p` |
| 6 | Categorical EDA | Categories, countries, commodities, currencies, markets |
| 7 | Price distribution | Heavy right skew → `log1p(usdprice)` |
| 8 | Temporal trends | Monthly average prices, overall and by category |
| 9 | Geographic EDA | Market maps coloured by category and price |
| 10 | Correlation EDA | Heatmap and pairplot |
| 11 | Feature engineering | One profile per market, standardised with `StandardScaler` |
| 12 | k-distance graph | Elbow estimate of `eps` ≈ 1.146 |
| 13 | DBSCAN tuning | 8-value `eps` sweep; best configuration `eps = 0.712` |
| 14 | Evaluation | Silhouette, Calinski-Harabasz, Davies-Bouldin |
| 15 | Visualisation | Geographic map + PCA 2-D projection |
| 16 | Profiling | Cluster sizes, countries and price levels |

---

## 📊 Key Results

**Final model:** `DBSCAN(eps=0.712, min_samples=6)` on 1,722 markets

| Metric | Value |
|---|---|
| Clusters found | 3 |
| Noise markets | 4 (0.23%) |
| Silhouette score | 0.2999 |
| Calinski-Harabasz index | 168.87 |
| Davies-Bouldin index | 0.5305 |

**Cluster profile**

| Cluster | Markets | Countries | Avg lat. | Avg long. | Avg USD price |
|---|---|---|---|---|---|
| 0 | 1,636 | 54 | 9.63 | 40.50 | 6.86 |
| 1 | 9 | 1 | -17.38 | -65.91 | 1.95 |
| 2 | 73 | 1 | 34.68 | 37.29 | 84.71 |
| Noise | 4 | n/a | n/a | n/a | n/a |

**Takeaways**
- The data is very clean (< 0.4% missing); prices are extremely right-skewed, so a log transform is essential.
- Reporting and market locations concentrate in food-insecure regions (Sub-Saharan Africa, South/South-East Asia, Middle East).
- Latitude, longitude and month show almost no linear correlation with price, which means any structure is local and non-linear, a good fit for density-based clustering.
- Cluster 0 is a broad belt of moderate-price markets; cluster 2 corresponds to Syrian markets with very high USD prices; cluster 1 is a small Bolivian group with the lowest prices.

---

## 🖼️ Visual Highlights

| | |
|---|---|
| ![Top countries](images/top_countries.png) | ![Log price distribution](images/log_price_distribution.png) |
| ![Geo price map](images/geo_price_map.png) | ![k-distance graph](images/k_distance_graph.png) |
| ![eps tuning](images/eps_tuning.png) | ![DBSCAN geographic clusters](images/dbscan_geo_clusters.png) |

---

## ⚠️ Limitations

- **Unbalanced clusters:** about 95% of markets fall in one cluster; the other two are single-country groups.
- **Mixed commodity basket:** each market's price feature averages all commodities it reports (103 different units), so it reflects the basket as well as the cost of living.
- **Currency effects:** USD prices depend on exchange-rate conversion, which strongly affects hyper-inflated currencies.
- **Short time span:** only six monthly snapshots in 2026.
- **Moderate separation:** Silhouette ≈ 0.30 means reasonable but not crisp clusters.
- **Distance metric:** Euclidean distance on scaled degrees; a haversine metric would be more accurate for geography.

---

## 📁 Project Structure

```
wfp-food-prices-dbscan/
├── README.md
├── requirements.txt
├── .gitignore
├── LICENSE
├── data/
│   └── wfp_food_prices_global_2026.csv
├── notebooks/
│   └── WFP_Food_Prices_EDA_DBSCAN.ipynb
├── images/
│   └── *.png
└── docs/
    └── Project_Documentation.docx
```

---

## 🚀 Installation & Usage

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/wfp-food-prices-dbscan.git
cd wfp-food-prices-dbscan

# 2. Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch the notebook
jupyter notebook notebooks/WFP_Food_Prices_EDA_DBSCAN.ipynb
```

Make sure `wfp_food_prices_global_2026.csv` is available at the path used in the notebook's load cell (update the path if you keep it in `data/`), then run all cells from top to bottom.

---

## 🔮 Future Work

- Cluster within a single commodity or category (or use prices relative to the national median) to remove basket effects.
- Use a haversine distance and try **HDBSCAN**, which does not need a single global `eps`.
- Add economic indicators such as inflation, currency volatility or conflict data.
- Track month-to-month cluster membership to flag emerging regional price shocks.
- Build an interactive map (Folium/Plotly) and a Streamlit dashboard.

---

## 👤 Author

**Akhilesh**
- GitHub: akhileshcoding08(https://github.com/akhileshcoding08)
- LinkedIn: https://www.linkedin.com/in/akhileshhadke

---


⭐ If you found this project useful, consider giving it a star!
