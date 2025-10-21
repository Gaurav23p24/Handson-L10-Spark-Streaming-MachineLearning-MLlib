# Spark Structured Streaming + MLlib: Real-Time Ride-Sharing Fare Prediction

![Spark](https://img.shields.io/badge/Apache%20Spark-3.5.0-E25A1C?style=for-the-badge&logo=apache-spark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Machine Learning](https://img.shields.io/badge/MLlib-Linear%20Regression-FF6F00?style=for-the-badge)

## 📋 Project Overview

This project implements a **real-time analytics pipeline** for a ride-sharing platform using **Apache Spark Structured Streaming** and **MLlib**. The system processes live streaming data, trains machine learning models, and performs real-time fare predictions with anomaly detection capabilities.

### Key Features

- 🚗 Real-time ride-sharing data processing
- 🤖 Machine Learning-based fare prediction using Linear Regression
- 📊 Time-series trend analysis with windowed aggregations
- 🔍 Anomaly detection through deviation analysis
- ⚡ Continuous streaming data ingestion and prediction

---

## 🏗️ Architecture
```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│ Data Generator  │────────▶│  Spark Streaming │────────▶│   ML Models     │
│   (Socket)      │         │   (Port 9999)    │         │    (MLlib)      │
└─────────────────┘         └──────────────────┘         └─────────────────┘
                                     │                             │
                                     ▼                             ▼
                            ┌──────────────────┐         ┌─────────────────┐
                            │ Real-Time Predict│         │  Fare Trends    │
                            │   & Deviation    │         │  (5-min window) │
                            └──────────────────┘         └─────────────────┘
```

---

## 📁 Project Structure
```
spark-streaming-mllib-fare-prediction/
│
├── task4.py                          # Real-time fare prediction script
├── task5.py                          # Time-based trend prediction script
├── data_generator.py                 # Streaming data simulator
├── training-dataset.csv              # Historical training data
│
├── models/
│   ├── fare_model/                   # Task 4: Trained Linear Regression model
│   └── fare_trend_model_v2/          # Task 5: Temporal prediction model
│
├── screenshots/
│   ├── task4_output.png              # Task 4 results
│   └── task5_output.png              # Task 5 results
│
└── README.md                         # This file
```

---

## 🔧 Prerequisites

### Required Software

- **Python**: 3.8 or higher
- **Apache Spark**: 3.5.0 (via PySpark)
- **Java**: 11 or higher (for Spark)

### Python Libraries
```bash
pip install pyspark
```

---

## 🚀 Installation & Setup

### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/spark-streaming-mllib-fare-prediction.git
cd spark-streaming-mllib-fare-prediction
```

### 2. Install Dependencies
```bash
pip install pyspark
```

### 3. Verify Training Data

Ensure `training-dataset.csv` exists in the root directory with columns:
- `trip_id`
- `driver_id`
- `distance_km`
- `fare_amount`
- `timestamp`

---

## 📊 Task 4: Real-Time Fare Prediction Using MLlib Regression

### 🎯 Objective

Build a real-time system that predicts ride fares based on trip distance and detects anomalies by comparing predicted vs. actual fares.

### 🧠 Approach

#### Phase 1: Offline Model Training

1. **Data Loading**: Load historical ride data from `training-dataset.csv`
2. **Feature Engineering**: Use `VectorAssembler` to transform `distance_km` into feature vectors
3. **Model Training**: Train a `LinearRegression` model to learn the distance-to-fare relationship
4. **Model Persistence**: Save the trained model to `models/fare_model/` for reuse

#### Phase 2: Real-Time Streaming Inference

1. **Stream Ingestion**: Connect to socket stream (`localhost:9999`) to receive live ride data
2. **Data Parsing**: Parse incoming JSON/CSV formatted ride records
3. **Feature Transformation**: Apply the same `VectorAssembler` to streaming data
4. **Prediction**: Use the pre-trained model to predict `fare_amount` based on `distance_km`
5. **Anomaly Detection**: Calculate deviation (absolute difference) between actual and predicted fares
6. **Output**: Stream results to console showing predictions and deviations

### 📝 Implementation Details

**Key Components:**
- **Model**: Linear Regression (single feature: `distance_km`)
- **Feature Vector**: `[distance_km]` → `features`
- **Label**: `fare_amount`
- **Deviation Formula**: `|actual_fare - predicted_fare|`
- **Output Mode**: Append (show each new record)

### 🎬 How to Run

**Terminal 1: Start Data Generator**
```bash
python data_generator.py
```

**Terminal 2: Run Task 4 Script**
```bash
python task4.py
```

### 📸 Output

#### Sample Output Screenshot
```
-------------------------------------------
Batch: 60
-------------------------------------------
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+
|trip_id                             |driver_id|distance_km|fare_amount|predicted_fare   |deviation        |
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+
|71bc38a1-dfaf-409e-a449-47bbc95438c8|66       |4.12       |126.66     |97.13669918168897|29.52330081831103|
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+

-------------------------------------------
Batch: 61
-------------------------------------------
+------------------------------------+---------+-----------+-----------+----------------+----------------+
|trip_id                             |driver_id|distance_km|fare_amount|predicted_fare  |deviation       |
+------------------------------------+---------+-----------+-----------+----------------+----------------+
|8820fea5-4a89-434b-9878-c6899c01d3ab|52       |11.78      |147.81     |95.2537635243963|52.5562364756037|
+------------------------------------+---------+-----------+-----------+----------------+----------------+

-------------------------------------------
Batch: 62
-------------------------------------------
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+
|trip_id                             |driver_id|distance_km|fare_amount|predicted_fare   |deviation        |
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+
|87cd5fb5-be8e-4061-82f9-1a5e5a4f6a15|55       |46.86      |130.48     |86.63060649334847|43.84939350665152|
+------------------------------------+---------+-----------+-----------+-----------------+-----------------+
```

### 🔍 Results Analysis

**Key Observations:**

- The model successfully predicts fares in real-time for incoming ride data
- High deviation values (29.5, 52.5, 43.8) indicate potential anomalies:
  - Surge pricing during peak hours
  - Special route conditions (tolls, traffic)
  - Promotional discounts
  - Driver-initiated pricing adjustments
- The linear relationship between distance and fare is established, but real-world factors create variance
- Deviation metric enables real-time anomaly detection for fraud prevention

---

## 📈 Task 5: Time-Based Fare Trend Prediction

### 🎯 Objective

Predict average fare trends using temporal patterns by aggregating streaming data into 5-minute windows and utilizing cyclical time features.

### 🧠 Approach

#### Phase 1: Offline Model Training

1. **Data Loading**: Load `training-dataset.csv` with timestamp information
2. **Temporal Aggregation**: Group data into 5-minute windows using `window()` function
3. **Feature Engineering**:
   - Extract `hour_of_day` from window start time (0-23)
   - Extract `minute_of_hour` from window start time (0-59)
   - These cyclical features capture time-based patterns (rush hours, off-peak times)
4. **Aggregation**: Calculate `avg_fare` for each 5-minute window
5. **Model Training**: Train Linear Regression on `[hour_of_day, minute_of_hour]` → `avg_fare`
6. **Model Persistence**: Save to `models/fare_trend_model_v2/`

#### Phase 2: Real-Time Streaming Inference

1. **Stream Ingestion**: Connect to socket stream for live ride data
2. **Windowed Aggregation**: Apply 5-minute tumbling windows to streaming data
3. **Feature Engineering**: Extract the same time-based features for each window
4. **Prediction**: Predict `avg_fare` for upcoming windows based on temporal patterns
5. **Output**: Display window boundaries, actual average fare, and predicted trend

### 📝 Implementation Details

**Key Components:**
- **Window Size**: 5 minutes (tumbling window)
- **Features**: `hour_of_day`, `minute_of_hour`
- **Target**: `avg_fare` (average fare per window)
- **Model**: Linear Regression (temporal pattern learning)
- **Output Mode**: Complete (show all active windows with updates)

### 🎬 How to Run

**Terminal 1: Start Data Generator**
```bash
python data_generator.py
```

**Terminal 2: Run Task 5 Script**
```bash
python task5.py
```

### 📸 Output

#### Sample Output Screenshot
```
-------------------------------------------                                     
Batch: 45
-------------------------------------------
+-------------------+-------------------+-----------------+-----------------------+
|window_start       |window_end         |avg_fare         |predicted_next_avg_fare|
+-------------------+-------------------+-----------------+-----------------------+
|2025-10-21 23:23:00|2025-10-21 23:28:00|76.60976588628762|92.39431034482759      |
|2025-10-21 23:27:00|2025-10-21 23:32:00|73.70322033898304|92.39431034482759      |
|2025-10-21 23:25:00|2025-10-21 23:30:00|74.96340782122905|92.39431034482759      |
|2025-10-21 23:26:00|2025-10-21 23:31:00|77.19815126050419|92.39431034482759      |
|2025-10-21 23:24:00|2025-10-21 23:29:00|77.4539330543933 |92.39431034482759      |
+-------------------+-------------------+-----------------+-----------------------+

-------------------------------------------                                     
Batch: 46
-------------------------------------------
+-------------------+-------------------+------------------+-----------------------+
|window_start       |window_end         |avg_fare          |predicted_next_avg_fare|
+-------------------+-------------------+------------------+-----------------------+
|2025-10-21 23:23:00|2025-10-21 23:28:00|76.54679999999999 |92.39431034482759      |
|2025-10-21 23:27:00|2025-10-21 23:32:00|76.24742424242424 |92.39431034482759      |
|2025-10-21 23:25:00|2025-10-21 23:30:00|75.81876344086022 |92.39431034482759      |
|2025-10-21 23:28:00|2025-10-21 23:33:00|104.35333333333331|92.39431034482759      |
|2025-10-21 23:26:00|2025-10-21 23:31:00|78.33666666666666 |92.39431034482759      |
|2025-10-21 23:24:00|2025-10-21 23:29:00|78.02979674796747 |92.39431034482759      |
+-------------------+-------------------+------------------+-----------------------+

-------------------------------------------                                     
Batch: 47
-------------------------------------------
+-------------------+-------------------+-----------------+-----------------------+
|window_start       |window_end         |avg_fare         |predicted_next_avg_fare|
+-------------------+-------------------+-----------------+-----------------------+
|2025-10-21 23:27:00|2025-10-21 23:32:00|74.48555555555555|92.39431034482759      |
|2025-10-21 23:25:00|2025-10-21 23:30:00|75.17145833333333|92.39431034482759      |
|2025-10-21 23:28:00|2025-10-21 23:33:00|79.72916666666666|92.39431034482759      |
|2025-10-21 23:26:00|2025-10-21 23:31:00|77.2806818181818 |92.39431034482759      |
|2025-10-21 23:24:00|2025-10-21 23:29:00|77.48396825396826|92.39431034482759      |
+-------------------+-------------------+-----------------+-----------------------+
```

### 🔍 Results Analysis

**Key Observations:**

- Multiple overlapping windows are processed simultaneously, showing Spark's windowed aggregation capability
- The model predicts a consistent fare trend of ~92.39 for the 23:00 hour time period
- Actual `avg_fare` values (74-78 range) show natural variance around the prediction
- **Batch 46 anomaly**: Window 23:28:00-23:33:00 shows spike to 104.35, indicating:
  - Possible end-of-day surge pricing
  - Special event nearby
  - Data clustering effect
- The temporal model captures general time-of-day patterns but may need additional features (day_of_week, weather, events) for better accuracy

**Business Insights:**

- Late evening hours (23:00+) maintain relatively stable fare patterns
- The prediction baseline (~92.39) can be used for dynamic pricing decisions
- Deviations from predictions help identify demand surges in real-time

---

## 🎓 Key Learnings

### Technical Skills Developed

1. **Apache Spark Structured Streaming**: Real-time data processing with socket-based ingestion
2. **Spark MLlib**: Machine learning model training, persistence, and deployment
3. **Feature Engineering**:
   - Vector assembly for ML pipelines
   - Temporal feature extraction (cyclical time features)
   - Windowed aggregations for time-series analysis
4. **Streaming Analytics**: Stateful stream processing with continuous predictions
5. **Model Deployment**: Loading pre-trained models into streaming contexts

### Challenges & Solutions

| Challenge | Solution |
|-----------|----------|
| Real-time latency | Used Spark's micro-batch processing with optimized window sizes |
| Model persistence | Saved models to disk and loaded them in streaming context |
| Temporal features | Engineered cyclical time features instead of raw timestamps |
| Anomaly detection | Implemented deviation metrics to identify outliers |
| Stream-batch integration | Trained on static data, deployed on streaming data seamlessly |

---

## 📝 License

This project is licensed under the MIT License.