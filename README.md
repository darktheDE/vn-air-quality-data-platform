<h1 align="center">🌤️ VN Air Quality Data Platform (VNAQ-DP)</h1>
<h3 align="center">Real-time PM2.5 Pollution Spike Detection & Analytics</h3>

<p align="center">
  <img src="https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white" />
  <img src="https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white" />
  <img src="https://img.shields.io/badge/Apache_Kafka-231F20?style=for-the-badge&logo=apache-kafka&logoColor=white" />
  <img src="https://img.shields.io/badge/dbt-FF694B?style=for-the-badge&logo=dbt&logoColor=white" />
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" />
</p>

> **Note:** This project is the Final Capstone Project for the **Data Engineering Zoomcamp 2026**. It is designed to evaluate an end-to-end data pipeline integrating streaming ingestion, data warehousing, transformations, and visualization.

---

## 📖 1. Problem Statement & Business Value

**The Context (Vietnam 2026):**
Air pollution, specifically PM2.5, remains one of the most critical public health crises in major Vietnamese cities like Hanoi and Ho Chi Minh City. While current systems provide daily averages or long-term forecasts, they often fail to capture **short-term, hyper-local pollution spikes** caused by sudden thermal inversions (early morning) or localized traffic congestion.

**The Solution:**
**VNAQ-DP** is an end-to-end streaming data pipeline that ingests real-time air quality metrics and meteorological data. By integrating PM2.5 data with humidity and wind speed, the platform provides:
1. A clear correlation between weather patterns and pollution spikes.
2. A scalable data lakehouse architecture serving as a foundation for future Machine Learning forecasting and B2B API delivery.

---

## 🏗️ 2. Architecture & Technologies

![Architecture Diagram](https://via.placeholder.com/1000x400.png?text=Please+Upload+Your+Draw.io+Architecture+Diagram+Here)
*(Update this image with your actual architecture diagram)*

### 🛠️ Tech Stack & Zoomcamp Criteria Mapping:
* **Cloud:** Google Cloud Platform (GCP) - *Storage & Compute*
* **Infrastructure as Code (IaC):** Terraform - *Automated provisioning of GCS and BigQuery*
* **Streaming Ingestion:** Apache Kafka & Python - *Real-time data streaming from OpenAQ and OpenWeather APIs*
* **Data Lake:** Google Cloud Storage (GCS) - *Raw data storage (Parquet files)*
* **Workflow Orchestration:** Kestra - *Managing the ELT workflow from GCS to BigQuery*
* **Data Warehouse:** Google BigQuery - *Optimized analytics storage*
* **Data Transformation:** dbt (Data Build Tool) - *Data modeling, cleaning, and testing*
* **Dashboard:** Looker Studio - *Interactive visualization*

---

## ⚙️ 3. Pipeline Deep Dive

### A. Data Ingestion (Streaming)
Instead of static batch files, this project implements a **Streaming Pipeline**. 
* A Python `Producer` fetches data from OpenAQ API (PM2.5) and OpenWeather API (Humidity, Temperature, Wind) every 15 minutes and publishes it to Kafka topics.
* A Python `Consumer` subscribes to these topics, applies micro-batching, and writes `.parquet` files directly to the GCS Data Lake.

### B. Data Warehouse (Partitioning & Clustering)
To ensure cost-efficiency and high performance in BigQuery, the `fact_air_quality` table is optimized:
* **Partitioning:** Partitioned by `measurement_timestamp` (DATE). Since environmental analysis heavily relies on time-series queries (e.g., "show me last week's data"), this drastically reduces the volume of data scanned.
* **Clustering:** Clustered by `station_id` and `city`. This optimizes queries filtering by specific geographic locations, which is the most common filter in our BI dashboard.

### C. Transformation (dbt)
The transformation layer is built with dbt, following modular data modeling:
1. **Staging (`stg_`):** Cleans raw JSON, casts data types, and standardizes column names.
2. **Core (`fact_` & `dim_`):** Joins air quality metrics with corresponding weather data based on timestamp and geographic proximity.
3. **Marts (`mart_`):** Aggregates data to calculate the *3-hour Moving Average of PM2.5* and flags *Pollution Spikes* (e.g., PM2.5 jumps > 30% in one hour).

---

## 📊 4. Dashboard Visualizations

The final dashboard is built using Looker Studio. You can access the live dashboard here: **[Link to Looker Studio Dashboard]**

**Tiles Included:**
1. **Categorical Distribution (Bar Chart):** Compares the Average AQI / PM2.5 levels across different stations/districts in Hanoi and HCMC.
2. **Temporal Distribution (Line/Combo Chart):** A time-series graph displaying PM2.5 concentration against Humidity/Wind Speed over the last 72 hours, highlighting correlation trends (e.g., high humidity + low wind = PM2.5 peaks).

![Dashboard Screenshot](https://via.placeholder.com/1000x500.png?text=Dashboard+Screenshot)

---

## 🚀 5. Reproducibility (How to Run)

I have containerized the environment and provided a `Makefile` to make the replication process seamless.

### Prerequisites:
* Docker & Docker Compose installed
* GCP Account & Project (with a downloaded Service Account `.json` key)
* Terraform installed
* API Keys from OpenAQ and OpenWeatherMap

### Step-by-step Setup:

**Step 1: Clone the repo and setup credentials**
```bash
git clone https://github.com/your-username/vn-air-quality-data-platform.git
cd vn-air-quality-data-platform
# Place your GCP service account key in the root folder and rename to credentials.json
export GOOGLE_APPLICATION_CREDENTIALS="./credentials.json"
```

**Step 2: Provision Cloud Infrastructure (Terraform)**
```bash
make setup-infra
```
*This command runs `terraform init`, `plan`, and `apply` to create the GCS bucket and BigQuery dataset.*

**Step 3: Start Kafka and Orchestration Services**
```bash
make up
```
*Spins up Zookeeper, Kafka Brokers, and Kestra via Docker Compose.*

**Step 4: Start the Streaming Pipeline**
```bash
make stream
```
*Triggers the Kafka Producer & Consumer to fetch API data and upload Parquet to GCS.*

**Step 5: Run dbt Transformations**
```bash
make run-dbt
```
*Executes `dbt run` and `dbt test` to build the data marts in BigQuery.*

---

## 🌟 6. Going the Extra Mile

To elevate this project to enterprise standards, I have implemented the following optional features:
* **CI/CD Pipeline:** Configured GitHub Actions (`.github/workflows/`) to automatically run Python Linters (`pylint`) and `dbt compile/test` on every Pull Request.
* **Data Quality Testing:** Implemented `dbt tests` (`not_null`, `unique`, and custom accepted values) to ensure corrupted API data (e.g., negative PM2.5 values) is flagged.
* **Make Utility:** Used `Makefile` to abstract complex Docker and Terraform commands into simple, single-line executions.

---

## 🔮 7. Future Roadmap (Phase B)
While this project satisfies the DE Zoomcamp requirements, it serves as the foundation for a larger Product:
1. **Predictive Modeling:** Integrate an ML model (LSTM/XGBoost) to forecast PM2.5 levels 6 hours ahead based on current weather data.
2. **Alerting Bot:** Deploy a Serverless Cloud Function that queries the BigQuery data mart hourly and pushes Telegram/Zalo alerts to users if a pollution spike is detected.

---
*Developed with ☕ and data passion by[Your Name/LinkedIn Profile]*
```
