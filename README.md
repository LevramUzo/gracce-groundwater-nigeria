# High-Resolution Satellite Groundwater Downscaling over Nigeria Using Tree-Based Machine Learning Ensembles

## 🌍 Project Overview
This repository hosts a cloud-native geospatial pipeline designed to monitor and downscale groundwater storage variations across the Federal Republic of Nigeria. Due to the high cost and sparse distribution of physical monitoring wells in Sub-Saharan Africa, tracking localized aquifer depletion remains a critical challenge. 

This project solves this data-scarcity bottleneck by using machine learning to downscale **NASA GRACE (Gravity Recovery and Climate Experiment)** Terrestrial Water Storage Anomalies (TWSA) from their native coarse resolution of **1° (~110 km)** down to a high-resolution, actionable **0.05° (~5.5 km)** grid.

## 🚀 Key Features & Methodological Rigor
- **Cloud-Native Ingestion:** Built entirely using the **Google Earth Engine (GEE) API** to seamlessly ingest planetary-scale hydroclimatic grids without local data overhead.
- **Multi-Sensor Predictor Suite:** Integrates hydroclimatic inputs (CHIRPS Precipitation, MODIS NDVI, MODIS Land Surface Temperature), land surface modeling states (GLDAS Soil Moisture and Evapotranspiration), and terrain geometry (NASA SRTM Elevation and Slope).
- **Leakage-Protected Machine Learning:** Utilizes advanced tree-based regression models (Random Forest / XGBoost) trained with spatial block-validation strategies to completely eliminate temporal data leakage.
- **Regional Adaptability:** Tailored specifically to capture the distinct hydrological behaviors of Nigeria's crystalline Basement Complex and deep Sedimentary Basins across an intense north-to-south climate gradient.

## 📁 Repository Structure
```unset
├── README.md               # Project documentation and setup guide
├── requirements.txt         # Required Python packages
├── data/
│   ├── raw/                # Extracted regional satellite parameters (git-ignored)
│   └── processed/          # Downscaled groundwater anomaly grids
└── notebooks/
    ├── 01_data_acquisition.ipynb   # GEE satellite time-series harvesting
    └── 02_model_downscaling.ipynb   # Machine learning model training & validation
```

## 🛰️ Target Dataset Specifications
- **Study Area:** Nigeria (~923,768 km²)
- **Temporal Domain:** January 2003 – December 2016 (Core GRACE Baseline)
- **Coarse Resolution:** 1.0° Grid (Native GRACE)
- **Target Downscaled Resolution:** 0.05° Grid (~5.5 km)
