# 🚗 SwiftRide Analytics

> A live analytics dashboard for a fictional Pakistani ride-sharing company — powered by SQLite, pandas, Plotly, Streamlit, and scikit-learn.

![Streamlit](https://img.shields.io/badge/Streamlit-1.30+-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-3-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3+-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Database Schema](#-database-schema)
- [Dashboard Pages](#-dashboard-pages)
- [ML Fare Predictor](#-ml-fare-predictor)
- [Quick Start](#-quick-start)
- [Project Structure](#-project-structure)
- [Data Generation](#-data-generation)
- [Design Decisions](#-design-decisions)

---

## 🎯 Overview

SwiftRide Analytics is a **single-file Streamlit web application** that transforms a SQLite database of ride-sharing trip data into an interactive, production-grade analytics dashboard. It features:

- **4 dashboard pages** with 20+ interactive charts
- **Machine learning fare prediction** using Random Forest
- **Real-time SQL queries** against a local SQLite file
- **Zero server setup** — runs entirely on your machine

The app targets Pakistani ride-sharing data across **8 cities** with **150 drivers**, **800 riders**, and **7,000 trips** spanning 2023–2024.

---

## 🏗️ Architecture

### High-Level System Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                        USER'S BROWSER                           │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Streamlit Web Application (app.py)            │  │
│  │                                                            │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │  │
│  │  │  Executive   │  │     Trip     │  │     Driver       │ │  │
│  │  │  Overview    │  │   Analytics  │  │   Performance    │ │  │
│  │  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘ │  │
│  │         │                 │                    │           │  │
│  │  ┌──────▼─────────────────▼────────────────────▼─────────┐ │  │
│  │  │          ML Fare Predictor (Page 4)                   │ │  │
│  │  │   ┌─────────────────┐    ┌──────────────────────────┐ │ │  │
│  │  │   │  Live Inputs    │───▶│  RandomForest Model      │ │ │  │
│  │  │   │  (sliders, etc) │    │  .predict()              │ │ │  │
│  │  │   └─────────────────┘    └──────────────────────────┘ │ │  │
│  │  └───────────────────────────────────────────────────────┘ │  │
│  │                           │                                 │  │
│  │                    ┌──────▼──────┐                          │  │
│  │                    │  Plotly     │                          │  │
│  │                    │  Charts     │                          │  │
│  │                    └─────────────┘                          │  │
│  └───────────────────────────┬───────────────────────────────┘  │
└──────────────────────────────┼──────────────────────────────────┘
                               │
                          SQL Queries
                    (SELECT, JOIN, GROUP BY)
                               │
                               ▼
                    ┌─────────────────────┐
                    │   SQLite Database   │
                    │    swiftride.db     │
                    │                     │
                    │  ┌───────────────┐  │
                    │  │ 6 Tables:     │  │
                    │  │ cities        │  │
                    │  │ drivers       │  │
                    │  │ riders        │  │
                    │  │ trips         │  │
                    │  │ payments      │  │
                    │  │ reviews       │  │
                    │  └───────────────┘  │
                    └─────────────────────┘
```

### Data Flow Pipeline

```
┌──────────────┐     ┌───────────────┐     ┌──────────────┐     ┌──────────────┐
│  generate_   │────▶│   SQLite      │────▶│  Streamlit   │────▶│   Browser    │
│  data.py     │     │   Database    │     │  app.py      │     │   UI         │
│              │     │   (swiftride) │     │              │     │              │
│  150 drivers │     │               │     │  SQL Queries │     │  20+ Charts  │
│  800 riders  │     │  6 tables     │     │  pandas DF   │     │  KPI Cards   │
│  7000 trips  │     │  FK relations │     │  Plotly      │     │  ML Predict  │
│  seed=42     │     │  indexes      │     │  sklearn     │     │  Interactive  │
└──────────────┘     └───────────────┘     └──────────────┘     └──────────────┘
     PYTHON               FILE              PYTHON+WEB           RENDERING
```

### Request/Response Cycle

```
User clicks "Trip Analytics"
         │
         ▼
┌─────────────────────────────────┐
│  Streamlit re-reruns app.py     │
│  (stateful session persists)    │
└───────────────┬─────────────────┘
                │
    ┌───────────▼───────────┐
    │  @st.cache_data(ttl=60)│
    │  Check if query       │
    │  cached or run fresh  │
    └───────────┬───────────┘
                │
    ┌───────────▼───────────┐
    │  sqlite3.connect()    │
    │  pd.read_sql_query()  │
    │  Returns DataFrame    │
    └───────────┬───────────┘
                │
    ┌───────────▼───────────┐
    │  px.bar() / px.line() │
    │  go.Heatmap() etc.    │
    │  st.plotly_chart()    │
    └───────────┬───────────┘
                │
                ▼
     Interactive chart renders
     in browser (Plotly.js)
```

---

## 🧰 Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | Streamlit | Web UI framework — turns Python into a web app |
| **Database** | SQLite 3 | Serverless file-based relational database |
| **Data Processing** | pandas | SQL results → DataFrames for analysis |
| **Visualization** | Plotly | Interactive, zoomable charts (line, bar, scatter, heatmap, histogram) |
| **Machine Learning** | scikit-learn | Random Forest fare prediction |
| **Numerical** | NumPy | Random data generation, seed control, array ops |

### Why These Tools?

```
                    COMPLEXITY vs POWER

    High │                                    ● scikit-learn
         │                            ● Plotly
         │                    ● pandas
         │            ● NumPy
         │    ● Streamlit
         │● SQLite
    Low  └──────────────────────────────────────
         Simple                              Complex
                    SETUP REQUIRED
```

- **SQLite** — No server, no config, lives in a single file. Speaks standard SQL.
- **pandas** — Bridges database results to Python's data ecosystem.
- **Plotly** — One line of code per chart. Hoverable, zoomable, browser-rendered.
- **Streamlit** — No HTML/CSS/JS needed. Hot-reloads on save. Instant feedback.
- **scikit-learn** — Two-line ML interface: `.fit()` to train, `.predict()` to use.

---

## 🗄️ Database Schema

### Entity Relationship Diagram

```
┌──────────────────┐       ┌──────────────────┐
│     cities       │       │     drivers      │
│──────────────────│       │──────────────────│
│ PK city_id    PK │◄──────│ FK city_id       │
│    city_name     │       │    name          │
│    province      │       │    phone         │
│    population    │       │    vehicle_type  │──┐
│    is_active     │       │    rating        │  │
└────────┬─────────┘       └────────┬─────────┘  │
         │                          │            │
         │              ┌───────────┘            │
         │              │                        │
         ▼              ▼                        │
┌──────────────────────────────────┐            │
│            trips                 │            │
│──────────────────────────────────│            │
│ PK trip_id                       │            │
│ FK rider_id          ┌───────────┤            │
│ FK driver_id─────────┤          │            │
│ FK city_id───────────┤          │            │
│    pickup_area       │          │            │
│    dropoff_area      │          │            │
│    vehicle_type ─────┼──────────┘            │
│    distance_km       │                       │
│    duration_mins     │                       │
│    fare_pkr          │                       │
│    trip_date         │                       │
│    trip_hour         │                       │
│    day_of_week       │                       │
│    status            │                       │
│    is_raining        │                       │
│    is_peak_hour      │                       │
│    surge_multiplier  │                       │
└──────────┬───────────┘                       │
           │                                    │
           │         ┌──────────────────┐       │
           │         │     riders       │       │
           │         │──────────────────│       │
           └─────────│ FK city_id       │       │
                     │    name          │       │
                     │    phone         │       │
                     │    email         │       │
                     │    signup_date   │       │
                     │    total_trips   │       │
                     │    rating        │       │
                     └──────────────────┘       │
                                                │
           ┌──────────────────┐                │
           │    payments      │                │
           │──────────────────│                │
           │ PK payment_id    │                │
           │ FK trip_id ──────┤                │
           │    amount_pkr    │                │
           │    payment_method│                │
           │    payment_status│                │
           │    paid_at       │                │
           └──────────────────┘                │
                                                │
           ┌──────────────────┐                │
           │    reviews       │                │
           │──────────────────│                │
           │ PK review_id     │                │
           │ FK trip_id ──────┤                │
           │ FK rider_id      │                │
           │ FK driver_id ────┼────────────────┘
           │    rider_rating  │
           │    driver_rating │
           │    sentiment     │
           │    review_date   │
           └──────────────────┘
```

### Table Summary

| Table | Rows | Key Columns | Description |
|-------|------|-------------|-------------|
| `cities` | 8 | `city_id`, `city_name`, `province` | Pakistani cities with provinces |
| `drivers` | 150 | `driver_id`, `name`, `vehicle_type`, `rating` | Drivers across 8 cities |
| `riders` | 800 | `rider_id`, `name`, `email`, `city_id` | Riders with signup history |
| `trips` | 7,000 | `trip_id`, `fare_pkr`, `status`, `surge_multiplier` | Core trip records with pricing |
| `payments` | ~6,440 | `payment_id`, `amount_pkr`, `payment_method` | One per completed trip |
| `reviews` | ~4,186 | `review_id`, `rider_rating_given`, `sentiment` | 65% of completed trips |

### Vehicle Type Fare Logic

```
Base Fare = fixed + (distance × rate_per_km)

┌──────────┬────────┬───────────────┬──────────────────┐
│ Vehicle  │ Fixed  │ Rate/km (PKR) │ Typical Distance │
├──────────┼────────┼───────────────┼──────────────────┤
│ Bike     │  PKR 50│      25       │     1–8 km       │
│ Rickshaw │  PKR 60│      30       │     1–6 km       │
│ Car      │ PKR 100│      45       │     2–20 km      │
│ SUV      │ PKR 150│      60       │     3–25 km      │
└──────────┴────────┴───────────────┴──────────────────┘

Final Fare = Base Fare × Surge Multiplier + Noise (±20 PKR)
             Rounded to nearest 10 PKR
```

---

## 📊 Dashboard Pages

### Page 1 — Executive Overview

```
┌─────────────────────────────────────────────────────────────┐
│  🚗 SwiftRide — Executive Overview                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐┌──────────┐┌──────────┐┌──────────┐┌────────┐│
│  │ Total    ││ Total    ││Completed ││Completion││ Avg    ││
│  │ Revenue  ││ Trips    ││  Trips   ││   Rate   ││ Fare   ││
│  │PKR X,XXX ││  7,000   ││  6,440   ││  92.0%   ││PKR XXX ││
│  └──────────┘└──────────┘└──────────┘└──────────┘└────────┘│
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  📈 Monthly Revenue Trend (Line Chart)               │  │
│  │  ╱╲    ╱╲╱                                           │  │
│  │ ╱  ╲  ╱  ╲     ← vertical dashed line = latest month│  │
│  │╱    ╲╱    ╲╱                                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────┐  ┌─────────────────────────────┐ │
│  │  📊 Trips by City    │  │  🥧 Fleet Mix (Donut)       │ │
│  │  Karachi ████████    │  │    Car    ██████ 42%         │ │
│  │  Lahore  ███████     │  │    Bike   ████ 28%           │ │
│  │  Islmd   ██████      │  │    Rick.  ███ 18%            │ │
│  │  Rawal   ████        │  │    SUV    ██  12%            │ │
│  └──────────────────────┘  └─────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  📋 City-Level Summary Table                         │  │
│  │  City  │ Total Trips │ Revenue   │ Avg Fare │ Rate   │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Page 2 — Trip Analytics

```
┌─────────────────────────────────────────────────────────────┐
│  🗺️ Trip Analytics — Demand, Pricing, Weather Impact       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🔥 Trip Demand Heatmap (Hour × Day of Week)               │
│  ┌────────────────────────────────────────────────────┐    │
│  │     0  2  4  6  8 10 12 14 16 18 20 22 (Hours)     │    │
│  │ Mon ░░░░▒▒▒▒██████▓▓▓░░░░▒▒▒██████▓▓▓░░           │    │
│  │ Tue ░░░░▒▒▒▒██████▓▓▓░░░░▒▒▒██████▓▓▓░░           │    │
│  │ Wed ░░░░▒▒▒▒██████▓▓▓░░░░▒▒▒██████▓▓▓░░           │    │
│  │ Thu ░░░░▒▒▒▒██████▓▓▓░░░░▒▒▒██████▓▓▓░░           │    │
│  │ Fri ░░░░▒▒▒▒██████▓▓▓░░░░▒▒▒██████▓▓▓░░           │    │
│  │ Sat ░░░░░▒▒▒▒▒█████▓▓▓▓░░░▒▒▒▒████▓▓▓▓           │    │
│  │ Sun ░░░░░▒▒▒▒▒█████▓▓▓▓░░░▒▒▒▒████▓▓▓▓           │    │
│  └────────────────────────────────────────────────────┘    │
│  ░ = Low  ▒ = Medium  ▓ = High  █ = Peak                  │
│                                                             │
│  ┌──────────────────────┐  ┌─────────────────────────────┐ │
│  │  💰 Avg Fare/Vehicle │  │  📍 Fare vs Distance        │ │
│  │  Bike    ███ PKR 200 │  │  •    • •     •             │ │
│  │  Rick.   ████ PKR 230│  │   • •  •  ••                │ │
│  │  Car     █████ PKR 450│ │    •   •• •  •              │ │
│  │  SUV     ██████ PKR 620││   • •  •  •                 │ │
│  └──────────────────────┘  └─────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  📊 Peak vs Off-Peak Fares (Grouped Bar)             │  │
│  │  Bike   ██ Peak  ▓▓ Off-Peak                         │  │
│  │  Rick.  ███ Peak  ▓▓▓ Off-Peak                       │  │
│  │  Car    █████ Peak  ███ Off-Peak                     │  │
│  │  SUV    ██████ Peak  ████ Off-Peak                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ☔ Rain Impact                                              │
│  ┌──────────────┐┌──────────────┐┌──────────────┐┌────────┐│
│  │☔ Avg Fare(R) ││☀️ Avg Fare(D)││☔ Trips/Day(R)││☀️Trips/││
│  │  PKR XXX     ││  PKR XXX     ││    XX.X      ││ Day(XX)││
│  └──────────────┘└──────────────┘└──────────────┘└────────┘│
│  💡 Surge: 1.5× during rain+peak, 1.3× during peak only    │
└─────────────────────────────────────────────────────────────┘
```

### Page 3 — Driver Performance

```
┌─────────────────────────────────────────────────────────────┐
│  🏆 Driver Performance — Leaderboards & Ratings            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🏅 Top 10 Drivers — Earnings Leaderboard                  │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Rank │ Driver    │ City    │ Vehicle │ Earnings │ ⭐ │  │
│  │  1   │ Ahmed K.  │ Karachi │ Car     │ PKR XXK  │4.8 │  │
│  │  2   │ Fatima R. │ Lahore  │ SUV     │ PKR XXK  │4.9 │  │
│  │ ...                                                  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────┐  ┌─────────────────────────────┐ │
│  │  ⭐ Rating by City   │  │  💵 Earnings/Trip/Vehicle   │ │
│  │  Islmd ██████████ 4.8│  │  Bike  ███ PKR XXX          │ │
│  │  Lahore █████████ 4.7│  │  Rick. ████ PKR XXX         │ │
│  │  Karac █████████ 4.6 │  │  Car   █████ PKR XXX        │ │
│  │  Pesha ████████ 4.5  │  │  SUV   ██████ PKR XXX       │ │
│  └──────────────────────┘  └─────────────────────────────┘ │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  👥 Active Drivers Per Month (Line + Area)           │  │
│  │       ╱╲                                             │  │
│  │      ╱  ╲╱╲      ╱╲                                 │  │
│  │     ╱    ╱  ╲    ╱  ╲  ╱╲                           │  │
│  │  ──╱────╱────╲──╱────╲╱──╲──────                    │  │
│  │  Jan  Mar  May  Jul  Sep  Nov  Jan                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  📊 Distribution of Driver Ratings (Histogram)       │  │
│  │       █                                              │  │
│  │      ██    █                                         │  │
│  │     ███   ██   █                                     │  │
│  │    ████  ███  ██  █  █                               │  │
│  │   ──┬────┬────┬────┬────┬─                            │  │
│  │     1    2    3    4    5   Rating                    │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Page 4 — ML Fare Predictor

```
┌─────────────────────────────────────────────────────────────┐
│  🤖 ML Fare Predictor — Random Forest Model                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📊 Model Performance                                       │
│  ┌──────────────────┐┌──────────────────┐┌────────────────┐│
│  │  R² Score        ││  MAE             ││  RMSE          ││
│  │  0.XXXX          ││  PKR XX.X        ││  PKR XX.X      ││
│  │  ✅ Excellent    ││                  ││                ││
│  └──────────────────┘└──────────────────┘└────────────────┘│
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  🔬 What Drives Fare Prices? (Feature Importance)    │  │
│  │  Distance (km)     ████████████████████ 0.XXX        │  │
│  │  Duration (mins)   ████████████████ 0.XXX            │  │
│  │  Surge Multiplier  ████████████ 0.XXX                │  │
│  │  Vehicle: Car      ████████ 0.XXX                    │  │
│  │  Vehicle: SUV      ██████ 0.XXX                      │  │
│  │  Trip Hour         ████ 0.XXX                        │  │
│  │  Peak Hour         ██ 0.XXX                          │  │
│  │  Day of Week       █ 0.XXX                           │  │
│  │  Raining           █ 0.XXX                           │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  🔮 Predict a Fare                                          │
│  ┌─────────────────────┐  ┌─────────────────────────────┐  │
│  │  Vehicle: [Car  ▼]  │  │  ┌──────────────────────┐   │  │
│  │  Distance: [●───]   │  │  │  Predicted Fare      │   │  │
│  │  Trip Hour:  [●──]  │  │  │  PKR 450             │   │  │
│  │  Day:        [Mon▼] │  │  │  +PKR 30 above avg   │   │  │
│  │  ☔ Raining?  [ ]    │  │  └──────────────────────┘   │  │
│  │                     │  │  ┌──────────────────────┐   │  │
│  │  Derived:           │  │  │  Conditions          │   │  │
│  │  Peak Hour: Yes     │  │  │  🔴 Peak Hour (+30%) │   │  │
│  │  Surge: 1.3×        │  │  │  Distance: 8.0 km    │   │  │
│  │  Duration: 32 mins  │  │  │  Surge: 1.3×         │   │  │
│  └─────────────────────┘  └─────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  📈 Actual vs Predicted Fares (Test Set)             │  │
│  │        │  /                                        │  │
│  │      • │ / •  •  ← diagonal = perfect prediction    │  │
│  │    •  •│/  • • •                                    │  │
│  │   • • •├───•───•───                                 │  │
│  │  •  •  │  •   •                                     │  │
│  │  ──────┼────────────                                │  │
│  │        │                                          Actual│
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 🤖 ML Fare Predictor

### Model Architecture

```
┌──────────────────────────────────────────────────────┐
│              Random Forest Regressor                  │
│              n_estimators=100                        │
│              random_state=42                         │
├──────────────────────────────────────────────────────┤
│                                                      │
│  INPUT FEATURES (7 raw + 4 one-hot = 11 total)       │
│                                                      │
│  ┌────────────────────┐    ┌──────────────────────┐  │
│  │ Numeric Features   │    │ One-Hot Encoded      │  │
│  │ ├─ distance_km     │    │ ├─ vtype_Bike        │  │
│  │ ├─ duration_mins   │    │ ├─ vtype_Car         │  │
│  │ ├─ is_peak_hour    │    │ ├─ vtype_Rickshaw    │  │
│  │ ├─ is_raining      │    │ └─ vtype_SUV         │  │
│  │ ├─ surge_multiplier│    │                      │  │
│  │ ├─ day_of_week     │    └──────────────────────┘  │
│  │ └─ trip_hour       │                              │
│  └────────────────────┘                              │
│                                                      │
│  TRAIN/TEST SPLIT: 80% / 20% (random_state=42)       │
│                                                      │
│  ┌──────────────┐    ┌───────────────┐               │
│  │  X_train     │    │  X_test       │               │
│  │  ~5,152 rows │    │  ~1,288 rows  │               │
│  │  y_train     │    │  y_test       │               │
│  └──────┬───────┘    └───────┬───────┘               │
│         │                    │                        │
│    .fit()               .predict()                    │
│         │                    │                        │
│         │              ┌─────▼─────┐                 │
│         │              │ y_pred    │                 │
│         │              └─────┬─────┘                 │
│         │                    │                        │
│         └──────────┬─────────┘                        │
│                    ▼                                  │
│          ┌───────────────────┐                        │
│          │  Evaluation       │                        │
│          │  ├─ R² Score      │                        │
│          │  ├─ MAE (PKR)     │                        │
│          │  └─ RMSE (PKR)    │                        │
│          └───────────────────┘                        │
└──────────────────────────────────────────────────────┘
```

### Feature Engineering Pipeline

```
Raw SQLite Data                  Processed Features
─────────────────                ────────────────────
distance_km        ──────────▶   distance_km (float)
duration_mins      ──────────▶   duration_mins (float)
is_peak_hour       ──────────▶   is_peak_hour (int 0/1)
is_raining         ──────────▶   is_raining (int 0/1)
surge_multiplier   ──────────▶   surge_multiplier (float)
day_of_week        ──────────▶   day_of_week (int 0-6)
trip_hour          ──────────▶   trip_hour (int 0-23)
vehicle_type       ────┐
  "Car"               ├───▶   pd.get_dummies() ──▶  vtype_Car=1, others=0
  "Bike"              │                              vtype_Bike=1, others=0
  "SUV"              │                              vtype_SUV=1, others=0
  "Rickshaw"         ┘                              vtype_Rickshaw=1, others=0

fare_pkr             ──────────▶   TARGET (y)
```

---

## 🚀 Quick Start

### Prerequisites

- Python 3.10+
- pip (Python package manager)

### Installation & Run

```bash
# 1. Clone the repository
git clone <repo-url>
cd SwiftRide-Analytics

# 2. Install dependencies
pip install -r requirements.txt

# 3. Launch the dashboard
#    (swiftride.db is pre-generated and included)
streamlit run app.py
```

The app will open automatically at `http://localhost:8501`.

### First-Time Setup Diagram

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  git clone  │───▶ │ pip install  │───▶ │ streamlit    │
│  the repo   │     │ -r req.txt   │     │ run app.py   │
│             │     │              │     │              │
│  ~5 seconds │     │  ~30 seconds │     │  Opens browser │
└─────────────┘     └──────────────┘     └──────────────┘
     STEP 1              STEP 2              STEP 3

                                      ✅ Dashboard Live!
                               (swiftride.db included)
```

---

## 📁 Project Structure

```
Streamlit-app/
│
├── app.py                      # Main Streamlit application (918 lines)
│   ├── Page 1: Executive Overview
│   │   ├── KPI Cards (5 metrics)
│   │   ├── Monthly Revenue Trend (line chart)
│   │   ├── Trips by City + Fleet Mix (bar + pie)
│   │   └── City Summary Table
│   │
│   ├── Page 2: Trip Analytics
│   │   ├── Demand Heatmap (hour × day)
│   │   ├── Fare by Vehicle + Scatter (bar + scatter)
│   │   ├── Peak vs Off-Peak (grouped bar)
│   │   └── Rain Impact Analysis (metrics + info)
│   │
│   ├── Page 3: Driver Performance
│   │   ├── Top 10 Leaderboard (table)
│   │   ├── Rating by City + Earnings (bar charts)
│   │   ├── Active Drivers Over Time (line + area)
│   │   └── Rating Distribution (histogram)
│   │
│   └── Page 4: ML Fare Predictor
│       ├── Model Training (RandomForest, cached)
│       ├── Performance Metrics (R², MAE, RMSE)
│       ├── Feature Importance (horizontal bar)
│       ├── Live Fare Predictor (interactive inputs)
│       └── Actual vs Predicted (scatter + diagonal)
│
├── generate_data.pdf           # Data generation specification (PDF, 4 pages)
│                               # Describes schema, logic, and requirements
│                               # for creating swiftride.db
│
├── swiftride.db                # SQLite database (pre-generated, ~1.2MB)
│
├── requirements.txt            # Python dependencies
├── prompt.pdf                  # Original project specification (4 pages)
└── README.md                   # This file
```

---

## 📊 Data Generation

### Reproducibility

The database was generated using **NumPy seed 42** for deterministic output.
The specification lives in `generate_data.pdf` — a 4-page document describing
the exact schema, data distribution, and fare logic used to create `swiftride.db`.

If you need to regenerate the database from scratch, follow the spec in
`generate_data.pdf` and use `prompt.pdf` for the full project requirements.

### Database Contents (as generated per spec)

```
┌──────────────────────────────────────────────────────┐
│         Database Specification (seed=42)             │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1. cities  ──────────▶  8 rows                     │
│     Karachi, Lahore, Islamabad, Rawalpindi,          │
│     Peshawar, Quetta, Multan, Faisalabad             │
│                                                      │
│  2. drivers ──────────▶  150 rows                   │
│     85% active, realistic Pakistani names,           │
│     vehicles distributed by type                     │
│                                                      │
│  3. riders  ──────────▶  800 rows                   │
│     Distributed across cities, ratings 3.0-5.0       │
│                                                      │
│  4. trips   ──────────▶  7,000 rows                 │
│     92% completed, 5% cancelled, 3% no-show          │
│     Realistic fare logic with surge pricing          │
│     Seasonal patterns (more winter evenings)         │
│     Karachi + Lahore = 35% of trips combined         │
│                                                      │
│  5. payments ─────────▶  ~6,440 rows                │
│     One per completed trip, 60% cash, 25% card       │
│                                                      │
│  6. reviews  ─────────▶  ~4,186 rows                │
│     65% of completed trips get reviews               │
│     70% positive, 20% neutral, 10% negative          │
│                                                      │
└──────────────────────────────────────────────────────┘
```

### City Distribution

```
Total Trips by City (approximate)

Karachi     ████████████████████  ~1,400 (20%)
Lahore      ██████████████████    ~1,260 (18%)
Islamabad   ████████████████      ~1,050 (15%)
Rawalpindi  ██████████████        ~910 (13%)
Peshawar    ████████████          ~840 (12%)
Multan      ██████████            ~700 (10%)
Faisalabad  ████████              ~560 (8%)
Quetta      ██████                ~280 (4%)
```

---

## 🎨 Design Decisions

### Why Streamlit?

```
Traditional Web App                    Streamlit App
─────────────────                    ───────────────
                                     
  ┌──────────┐                        ┌──────────────┐
  │ Frontend │                        │  app.py      │
  │ React/Vue│                        │  (Python)    │
  │ HTML/CSS │                        └──────┬───────┘
  │ JS/TS    │                               │
  └────┬─────┘                        ┌──────▼───────┐
       │                              │  SQLite      │
  ┌────▼─────┐                        │  swiftride.db│
  │ Backend  │                        └──────────────┘
  │ FastAPI/ │                               
  │ Flask    │                        Result: 1 file, 0 config
  └────┬─────┘                        Runs with: streamlit run app.py
       │                              
  ┌────▼─────┐                        
  │ Database │                        
  │ SQLite   │                        
  └──────────┘                        

  Result: 10+ files, complex setup
  Requires: npm, pip, config, routing
```

### Caching Strategy

```
┌─────────────────────────────────────────────────────┐
│                 Caching Layers                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  @st.cache_data(ttl=60)                             │
│  ├── Applied to: query(sql) function                │
│  ├── Caches: SQL query results as DataFrames        │
│  ├── TTL: 60 seconds (auto-invalidates)             │
│  └── Benefit: avoids re-querying unchanged data     │
│                                                     │
│  @st.cache_resource                                 │
│  ├── Applied to: train_model()                      │
│  ├── Caches: trained RandomForest model + metrics   │
│  ├── Lifetime: entire session                       │
│  └── Benefit: trains once, predicts instantly       │
│                                                     │
└─────────────────────────────────────────────────────┘
```

### Fare Pricing Model

```
                    SURGE PRICING DECISION TREE
                    
                    ┌─────────────────┐
                    │  Is it peak     │
                    │  hour? (7-9,    │
                    │  17-20)         │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
               Yes  │                 │  No
              ┌─────┤     PEAK?      ├─────┐
              │     │                 │     │
              ▼     └─────────────────┘     ▼
    ┌─────────────────┐           ┌─────────────────┐
    │ Is it raining?  │           │  surge = 1.0    │
    └────────┬────────┘           │  (standard)     │
             │                    └─────────────────┘
    ┌────────▼────────┐
Yes │                 │ No
┌───┤  RAIN+PEAK?     ├───┐
│   │                 │   │
▼   └─────────────────┘   ▼
1.5×                   1.3×
surge                  surge

Final Fare = (fixed + distance × rate) × surge + noise(±20)
             Rounded to nearest 10 PKR
```

---

## 📄 License

MIT License — feel free to use, modify, and distribute.

---

## 🙏 Acknowledgments

Built with [Claude](https://claude.ai) as a database lab project demonstrating the full stack from data generation to interactive analytics with machine learning.
