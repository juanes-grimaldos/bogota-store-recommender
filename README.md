
# 📍 Store Type Recommender — Bogotá

> ML-powered location intelligence system that recommends what type of business to open at any location in Bogotá, based on existing competition, neighborhood profile, population density, and socioeconomic data.

![Python](https://img.shields.io/badge/Python-3.10+-blue?style=flat-square)
![OSMnx](https://img.shields.io/badge/OSMnx-geospatial-1D9E75?style=flat-square)
![Scikit-learn](https://img.shields.io/badge/scikit--learn-clustering-F7931E?style=flat-square)
![Streamlit](https://img.shields.io/badge/Streamlit-interactive_map-FF4B4B?style=flat-square)
![FastAPI](https://img.shields.io/badge/FastAPI-REST_API-009688?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-containerized-2496ED?style=flat-square)

---

## What it does

Drop a pin anywhere on a map of Bogotá. The system analyzes a 500m radius around that point and recommends which type of business has the highest opportunity score — based on what's missing compared to similar neighborhoods across the city.

**No proprietary data. No paid APIs. Entirely built on public sources.**

---

## Demo

> *(Add GIF of Streamlit app here after building)*

**Example output for a location in Chapinero:**

```
📍 Location: Chapinero, Bogotá (4.6486° N, 74.0641° W)
🏘️  Zone profile: Cluster 2 — mid-high density, stratum 4, mixed commercial

Top recommendations:
  1. 🥗 Healthy food / juice bar        Score: 0.87  (high demand, low supply)
  2. 💻 Tech repair shop                Score: 0.74  (present in 78% of similar zones)
  3. 📦 Convenience store               Score: 0.61  (underrepresented for population density)

Saturated categories to avoid:
  ✗ Barbershops     (3.2x above zone average)
  ✗ Pharmacies      (already well-covered)
```

---

## How it works

### Core idea
Zones with similar profiles (stratum, density, demographics) tend to have the same optimal business mix. The model learns that mix from existing successful zones and applies it to new locations. **Underserved demand = business categories present in similar zones but missing at the target location.**

### Pipeline

```
User drops pin on map
        │
        ▼
Extract features within 500m radius (OSMnx)
  - Count businesses by category
  - Calculate category density ratios
        │
        ▼
Load zone features (DANE UPZ data)
  - Population density
  - Socioeconomic stratum
  - NBI index
        │
        ▼
Assign zone to cluster (K-Means, pre-trained)
  "This location looks like Cluster 3"
        │
        ▼
Compare local business mix vs cluster average
  "Cluster 3 zones have 2.1 coffee shops per 1000 people"
  "This location has 0.3 — opportunity score: 0.81"
        │
        ▼
Rank categories by opportunity score
        │
        ▼
Return top recommendations + map visualization
```

---

## Tech Stack

| Layer | Tools |
|-------|-------|
| Geospatial data | OSMnx, GeoPandas, Shapely |
| Clustering | scikit-learn (K-Means, DBSCAN) |
| Feature engineering | pandas, numpy |
| Experiment tracking | MLflow |
| API | FastAPI + Pydantic |
| App | Streamlit + Folium |
| Containerization | Docker, docker-compose |
| CI/CD | GitHub Actions |

---

## Data Sources

All sources are free and publicly available.

| Source | Data | How it's used |
|--------|------|---------------|
| OpenStreetMap (via OSMnx) | All tagged businesses in Bogotá | Primary source of business category data |
| DANE — Censo 2018 | Population, stratum, NBI by UPZ | Socioeconomic zone features |
| Secretaría Distrital de Planeación | UPZ shapefiles (117 zones) | Geographic boundaries for zone assignment |
| datos.gov.co | Internet coverage, infrastructure | Supplementary zone features |
| Cámara de Comercio de Bogotá *(optional)* | Registered businesses by CIIU code | Validation of business density estimates |

---

## Quickstart

### Run with Docker (recommended)

```bash
git clone https://github.com/YOUR_USERNAME/bogota-store-recommender.git
cd bogota-store-recommender
docker-compose up
```

- Streamlit app: `http://localhost:8501`
- FastAPI docs: `http://localhost:8000/docs`

### Run locally

```bash
pip install -r requirements.txt

# Download Bogotá business data from OSMnx (runs once, ~5 min)
python src/data_loader.py

# Train clustering model
python src/train.py

# Launch app
streamlit run app/streamlit_app.py
```

---

## API Usage

### POST /recommend

Send coordinates and get store type recommendations.

```bash
curl -X POST "http://localhost:8000/recommend" \
  -H "Content-Type: application/json" \
  -d '{
    "lat": 4.6486,
    "lon": -74.0641,
    "radius_meters": 500
  }'
```

Response:

```json
{
  "location": {"lat": 4.6486, "lon": -74.0641},
  "zone_cluster": 2,
  "zone_profile": "mid-high density, stratum 4, mixed commercial",
  "recommendations": [
    {
      "category": "healthy_food",
      "label": "Healthy food / juice bar",
      "opportunity_score": 0.87,
      "reason": "Present in 91% of similar zones, underrepresented here"
    },
    {
      "category": "tech_repair",
      "label": "Tech repair shop",
      "opportunity_score": 0.74,
      "reason": "High foot traffic zone with low tech service supply"
    }
  ],
  "saturated_categories": ["barbershop", "pharmacy"],
  "nearby_businesses_analyzed": 47
}
```

---

## Project Structure

```
bogota-store-recommender/
│
├── data/
│   ├── raw/                      # Downloaded OSMnx + DANE data
│   └── processed/                # Cleaned, geo-joined features per UPZ
│
├── src/
│   ├── data_loader.py            # Download businesses from OSMnx + DANE
│   ├── features.py               # Feature engineering per location/zone
│   ├── train.py                  # K-Means clustering + MLflow tracking
│   └── recommender.py            # Opportunity scoring logic
│
├── api/
│   └── main.py                   # FastAPI application
│
├── app/
│   └── streamlit_app.py          # Interactive Folium map + recommendations
│
├── tests/
│   └── test_recommender.py
│
├── notebooks/
│   └── eda_bogota.ipynb          # Exploratory analysis of Bogotá zones
│
├── .github/
│   └── workflows/
│       └── ci.yml                # Run tests on every push
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Clustering Results

The K-Means model identifies distinct commercial zone profiles across Bogotá's 117 UPZ zones:

| Cluster | Profile | Example zones | Dominant business types |
|---------|---------|---------------|------------------------|
| 0 | Low density, stratum 1-2, residential | Usme, Ciudad Bolívar | Tiendas de barrio, papelerías |
| 1 | Mid density, stratum 3, mixed | Kennedy, Bosa | Restaurantes, droguerías |
| 2 | Mid-high density, stratum 4, commercial | Chapinero, Teusaquillo | Cafés, servicios profesionales |
| 3 | High density, stratum 5-6, premium | Chicó, Rosales | Restaurantes gourmet, boutiques |
| 4 | High foot traffic, transit hubs | Centro, Suba | Comidas rápidas, minutos, misceláneas |

> *(Update cluster descriptions after running actual model)*

---

## Key Technical Decisions

### Why unsupervised learning instead of a classification model?
There are no labels for "successful business" — that data is not publicly available. K-Means clustering groups zones by commercial profile without needing outcome labels, allowing the system to infer opportunity from structural similarity between zones.

### Why 500m radius for feature extraction?
Walking distance in a Colombian urban context. Consumers in Bogotá typically patronize businesses within a 5–10 minute walk. A 500m radius captures the immediate competitive landscape without diluting the local signal with distant businesses.

### Why OSMnx over Google Places or Foursquare?
OSMnx is entirely free with no API rate limits, making it reproducible for anyone without a paid account. OpenStreetMap coverage in Bogotá is extensive enough for this use case. Foursquare could be added as an enrichment layer in a production version.

### Limitations
- OSMnx data completeness varies by neighborhood — informal businesses (tiendas de barrio) are underrepresented
- The model recommends *opportunity*, not guaranteed success — external factors like rent, foot traffic, and operator skill are not modeled
- Data reflects a snapshot in time; business landscape changes continuously

---

## Potential Real-World Applications

- **Entrepreneurs** deciding where to open a first business
- **Microcredit banks** (Bancóldex, Bancolombia) evaluating business viability before approving loans
- **Franchise chains** planning urban expansion in Bogotá
- **Urban planning** teams identifying commercial gaps in underserved zones

---

## Background

Bogotá has over 1 million registered businesses, yet there is no public tool to help entrepreneurs understand the commercial landscape of a specific neighborhood before investing. This project demonstrates that meaningful location intelligence can be built entirely from open data — no proprietary datasets required.

The methodology is designed to be extensible to Medellín, Cali, and other Colombian cities with minimal configuration changes.

---

## Author

**Juan Esteban Grimaldos** — Data Scientist / ML Engineer  
[LinkedIn](https://linkedin.com/in/YOUR_PROFILE) · [Portfolio](https://YOUR_PORTFOLIO) · est.juan.grimaldos@unimilitar.edu.co

---

## License

MIT