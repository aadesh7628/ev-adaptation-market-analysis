# 🚗 EV Adaptation & Market Analysis

## 📊 Project Overview

This project analyzes Electric Vehicle (EV) registration trends in India from 2020-2025 using a **Medallion Architecture** (Bronze-Silver-Gold) on Databricks. The analysis provides insights into:

* EV adoption rates and penetration across Indian states
* Top EV manufacturers and their market share
* Vehicle category distribution (2-wheelers, 3-wheelers, cars, etc.)
* Monthly and yearly EV registration trends
* Comparative analysis of EV vs Non-EV registrations

**Live Dashboard**: The project includes an interactive Lakeview dashboard visualizing key metrics and trends.

---

## 🗂️ Repository Structure

```
ev-adaptation-market-analysis/
│
├── Bronze_Layer.ipynb                    # Data ingestion from raw source
├── Silver_Layer.ipynb                    # Data cleaning and transformation
├── Gold_EV_State.ipynb                   # State-level EV aggregations
├── Gold_EV_Manufacturer.ipynb            # Manufacturer-level EV aggregations
├── Gold_EV_Monthly.ipynb                 # Monthly EV trend aggregations
├── Gold_EV_Vehicle_Category.ipynb        # Vehicle category aggregations
├── India EV Landscape Dashboard.lvdash.json  # Lakeview dashboard configuration
└── README.md                             # This file
```

---

## 📥 Dataset Setup

### 1. Download the Dataset

The raw dataset is sourced from Kaggle:

**Dataset**: [Indian Vehicle Registration Data (2020-25)](https://www.kaggle.com/datasets/aatifahmad123/indian-vehicle-registration-data-202025)

**File Size**: ~78 MB (CSV format)

#### Option A: Download via Kaggle Website
1. Go to the [dataset page](https://www.kaggle.com/datasets/aatifahmad123/indian-vehicle-registration-data-202025)
2. Click **Download** (requires Kaggle account)
3. Extract the CSV file

#### Option B: Download via Kaggle CLI
```bash
# Install Kaggle CLI
pip install kaggle

# Configure Kaggle API credentials (get from kaggle.com/account)
kaggle datasets download -d aatifahmad123/indian-vehicle-registration-data-202025

# Unzip the file
unzip indian-vehicle-registration-data-202025.zip
```

### 2. Upload the Raw Data to Databricks

You need to create a Unity Catalog table from the raw CSV file.

#### Step 1: Create the Catalog and Schema

Run the following SQL commands in a Databricks notebook or SQL editor:

```sql
-- Create the catalog (if it doesn't exist)
CREATE CATALOG IF NOT EXISTS ev_registration;

-- Create the default schema for raw data
CREATE SCHEMA IF NOT EXISTS ev_registration.default;
```

#### Step 2: Upload the CSV File

**Using Databricks UI:**

1. Go to **Data** → **Create Table**
2. Select **Upload File**
3. Choose the downloaded CSV file
4. Set the target table name as: `ev_registration.default.vehicle_registrations_500_k`
5. Let Databricks auto-detect schema
6. Click **Create Table**

**Using PySpark (Alternative):**

```python
# Read the CSV file (adjust path to where you uploaded it)
raw_df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("/path/to/your/uploaded/file.csv")

# Save as Delta table
raw_df.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("ev_registration.default.vehicle_registrations_500_k")
```

---

## 🏗️ Unity Catalog Structure

This project uses the following Unity Catalog hierarchy:

```
ev_registration (Catalog)
│
├── default (Schema) - Raw Data Layer
│   └── vehicle_registrations_500_k (Table)
│       └── Source: Raw CSV file from Kaggle
│
├── bronze (Schema) - Bronze Layer
│   └── bronze_vehicle_registrations (Table)
│       └── Raw data + ingestion timestamp
│
├── silver (Schema) - Silver Layer
│   └── silver_vehicle_registrations (Table)
│       └── Cleaned, standardized, and enriched data
│       └── Features: is_ev flag, ev_type, parsed dates
│
└── gold (Schema) - Gold Layer (Business-Level Aggregations)
    ├── gold_ev_state (Table)
    │   └── State-level EV registrations, penetration %, rankings
    │
    ├── gold_ev_manufacturer (Table)
    │   └── Manufacturer-level EV registrations, market share
    │
    ├── gold_ev_monthly (Table)
    │   └── Monthly EV/Non-EV registration trends
    │
    └── gold_ev_vehicle_category (Table)
        └── Vehicle category-wise EV registrations
```

### Key Columns in Silver Layer

| Column | Description |
|--------|-------------|
| `registration_year` | Year of vehicle registration |
| `registration_month` | Month name (Jan, Feb, Mar...) |
| `registration_month_number` | Month number (1-12) |
| `maker` | Vehicle manufacturer name |
| `state` | Indian state where registered |
| `rto_code` | Regional Transport Office code |
| `vehicle_category` | Type of vehicle (Two-Wheeler, Car, etc.) |
| `fuel` | Fuel type (PETROL, DIESEL, PURE EV, etc.) |
| `vehicle_count` | Number of vehicles registered |
| `is_ev` | Binary flag: 1 = EV, 0 = Non-EV |
| `ev_type` | EV category: PURE_EV, BATTERY_EV, PLUG_IN_HYBRID, etc. |
| `ingestion_timestamp` | When the data was ingested |

---

## 🚀 Running the Pipeline

### Prerequisites

* Databricks workspace with Unity Catalog enabled
* SQL Warehouse or Compute cluster attached
* Raw data loaded into `ev_registration.default.vehicle_registrations_500_k`

### Execution Steps

Run the notebooks **in order**:

#### 1. Bronze Layer
```
Run: Bronze_Layer.ipynb
```
* **Input**: `ev_registration.default.vehicle_registrations_500_k`
* **Output**: `ev_registration.bronze.bronze_vehicle_registrations`
* **Purpose**: Ingest raw data with minimal transformation (adds `ingestion_timestamp`)

#### 2. Silver Layer
```
Run: Silver_Layer.ipynb
```
* **Input**: `ev_registration.bronze.bronze_vehicle_registrations`
* **Output**: `ev_registration.silver.silver_vehicle_registrations`
* **Purpose**: Clean and standardize data
  * Trim whitespace from string columns
  * Parse registration month into month name and number
  * Flag EV vehicles (`is_ev` column)
  * Categorize EV types (`ev_type` column)
  * Filter out invalid registration years

#### 3. Gold Layer (Run all 4 notebooks)

**a) State-Level Aggregations**
```
Run: Gold_EV_State.ipynb
```
* **Output**: `ev_registration.gold.gold_ev_state`
* **Metrics**: Total registrations, EV registrations, EV penetration %, rankings by state

**b) Manufacturer-Level Aggregations**
```
Run: Gold_EV_Manufacturer.ipynb
```
* **Output**: `ev_registration.gold.gold_ev_manufacturer`
* **Metrics**: EV registrations by manufacturer, market share %

**c) Monthly Trend Aggregations**
```
Run: Gold_EV_Monthly.ipynb
```
* **Output**: `ev_registration.gold.gold_ev_monthly`
* **Metrics**: Monthly EV/Non-EV registrations, EV penetration % over time

**d) Vehicle Category Aggregations**
```
Run: Gold_EV_Vehicle_Category.ipynb
```
* **Output**: `ev_registration.gold.gold_ev_vehicle_category`
* **Metrics**: EV registrations by vehicle category (2-wheeler, 3-wheeler, cars, etc.)

---

## 📊 Dashboard Setup

### Importing the Dashboard

The project includes a pre-built Lakeview dashboard: `India EV Landscape Dashboard.lvdash.json`

#### Steps to Import:

1. **Go to Databricks AI/BI Dashboards**
   * Navigate to **Dashboards** in the Databricks sidebar

2. **Import Dashboard**
   * Click **Create** → **Import dashboard**
   * Select the `India EV Landscape Dashboard.lvdash.json` file from this repository
   * Click **Import**

3. **Verify Data Connections**
   * The dashboard connects to the following Gold tables:
     * `ev_registration.gold.gold_ev_monthly`
     * `ev_registration.gold.gold_ev_state`
     * `ev_registration.gold.gold_ev_manufacturer`
     * `ev_registration.gold.gold_ev_vehicle_category`
   * Ensure these tables exist by running all Gold layer notebooks

4. **Refresh & Publish**
   * Click **Refresh** to load the latest data
   * Click **Publish** to make the dashboard available

### Dashboard Features

The dashboard includes:

* **KPI Cards**: Total registrations, EV registrations, Non-EV registrations, EV penetration %
* **Trend Charts**: EV vs Non-EV registration trends over time, EV adoption growth
* **State Analysis**: Top 10 states by EV registrations and EV penetration %
* **Manufacturer Insights**: Top 10 EV manufacturers, market share distribution
* **Vehicle Category Breakdown**: EV registrations by vehicle type

---

## 🛠️ Tech Stack

* **Platform**: Databricks (Unity Catalog, Lakehouse)
* **Language**: PySpark (Python)
* **Storage Format**: Delta Lake
* **Visualization**: Lakeview Dashboards
* **Architecture**: Medallion (Bronze-Silver-Gold)

---

## 📈 Key Insights (Sample)

Based on the analysis, the project reveals:

* **EV Penetration**: Overall EV penetration rate in India
* **Top States**: Leading states in EV adoption (e.g., Karnataka, Maharashtra, Delhi)
* **Dominant Manufacturers**: Major players like Ola Electric, TVS, Bajaj, Ather Energy
* **Vehicle Preference**: 2-wheelers dominate the EV market in India
* **Growth Trend**: Month-over-month and year-over-year EV adoption growth

---

## 🤝 Contributing

Contributions are welcome! To contribute:

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/new-analysis`)
3. Commit your changes (`git commit -m 'Add new analysis'`)
4. Push to the branch (`git push origin feature/new-analysis`)
5. Open a Pull Request

---

## 📝 License

This project is for educational and analytical purposes. The dataset is sourced from Kaggle and is subject to its terms of use.

---

## 📧 Contact

For questions or feedback, please open an issue in this repository.

---

## 🙏 Acknowledgments

* **Dataset Source**: [Aatif Ahmad on Kaggle](https://www.kaggle.com/datasets/aatifahmad123/indian-vehicle-registration-data-202025)
* **Platform**: Databricks Community Edition / Databricks Lakehouse

---

**Happy Analyzing! 🚀**