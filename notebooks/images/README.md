# GRACE Groundwater Downscaling over Nigeria

**Status: work in progress.** The data pipeline, validation and a first 0.05° map for one held-out month are built. The results are modest and exploratory. See Limitations.

## Goal
Test whether rainfall, temperature and terrain can be used to downscale NASA GRACE terrestrial water storage anomalies (TWSA, ~1° / ~110 km grid) toward ~0.05° (~5.5 km) over Nigeria, and measure honestly how far the result can be trusted.

## Data (all via Google Earth Engine)
- GRACE land mass grids: `NASA/GRACE/MASS_GRIDS_V04/LAND`, band `lwe_thickness_csr` (target). Unit: **centimetres** of liquid water equivalent; native pixel ~111 km.
- CHIRPS daily rainfall: `UCSB-CHG/CHIRPS/DAILY`
- MODIS land surface temperature: `MODIS/061/MOD11A1`
- SRTM elevation and slope: `USGS/SRTMGL1_003`

Period: 2003-2016 (GRACE only; GRACE-FO is not used). Sample: 150 fixed random places (seed 42) x 141 months that have GRACE data (GRACE has gaps), 17,874 rows after lag and cumulative-rainfall features.

## Method
Random Forest regression on monthly features: rainfall, previous-month rainfall, cumulative rainfall (3, 6 and 12 months), land surface temperature, elevation and slope. The final model predicts TWSA *anomalies* (value minus the calendar-month average) and adds the monthly average back, which tests skill beyond the seasonal cycle.

## Validation
- **Spatial:** whole GRACE cells held out, then 3-degree blocks. Points in one GRACE cell share the same target value, so a random split leaks information.
- **Temporal:** train on 2003-2012, test on 2013-2016.
- **Baseline:** the average TWSA for each calendar month. A model must beat it to count.
- **Consistency check for the map:** average the 0.05° map inside each GRACE cell and compare with GRACE's own value.

## Results

![Validation ladder](validation_ladder.png)

**Early single-year table (300 points)**

| Test | R² |
|---|---|
| Random split | 0.98 (invalid: leakage) |
| Whole GRACE cells held out | 0.74 |
| 3-degree blocks held out | -0.20 |

**Monthly data**

| Test | Random Forest (raw target) | Baseline |
|---|---|---|
| Held-out 3° blocks (mean R²) | 0.65 | 0.81 |
| Held-out years 2013-2016 (R²) | 0.34 | 0.44 |

**Anomaly model**

| Test | Without cumulative rain | With cumulative rain |
|---|---|---|
| Held-out blocks, skill beyond seasons | 0.126 | 0.187 |
| Held-out blocks, total R² (baseline 0.806) | 0.833 | 0.846 |
| Held-out years, total R² (baseline 0.441) | 0.48 | 0.48 |

Most of GRACE's signal over Nigeria is the seasonal cycle. Predicting raw values loses to the baseline, but modelling what is left after removing the seasons adds a small, real gain on unseen regions. On held-out years the result is inconclusive: 2013-2014 were wetter than the 2003-2012 average (mean offset 0.046 against an anomaly spread of 0.032), a shift a seasonal baseline cannot capture, and the R² skill measure is distorted by that offset.

## 0.05° map (August 2015, held out of training)

![Downscaled map](downscaled_map.png)
![Consistency check](consistency_check.png)

Averaged back to 1° (94 GRACE cells), the map's RMSE was **0.033 cm** against **0.038 cm** for a constant "August normal" baseline, with a correlation of 0.51. The slope of predicted against true was 0.24, so the map compresses between-cell differences to roughly half of GRACE's. This is a coarse consistency check on one month. It does not validate fine-scale detail, which would need independent well data.

## Limitations
- One held-out month. The model trained on the same 150 places in other months, so this check is easier than the block test above.
- The check covers only the 1° average. Nothing here shows the 5.5 km patterns are right.
- No ground well data for validation. GRACE's true resolution is coarser than its 1° grid.
- For August 2015, about 47% of map pixels had no MODIS temperature and were filled with the 2003-2016 August average. Fifteen pixels with temperatures below 15 °C (suspected retrieval errors) were removed. Three training rows below 15 °C remain.
- Terrain for training places was sampled at 1 km, and for the map at the 5.5 km grid.
- GRACE has missing months, mostly in August-September and in later years.
- 150 sample places is a small sample.

## Not done yet
- Repeat the map check on more months.
- Year-by-year (leave-one-year-out) comparison against the baseline.
- Comparison with independent groundwater observations.
- A small web map of the output.

## Repository
- `notebooks/01_data_acquisition.ipynb`: data extraction, modelling and map (to be split into separate notebooks)
- `images/`: figures used above

## Reproduce
1. `pip install earthengine-api pandas numpy scikit-learn matplotlib`
2. Authenticate Earth Engine and set your own project ID.
3. Run the notebook top to bottom.

## Development note
Developed with AI assistance. I am rebuilding the notebook from a blank page, cell by cell, to understand and own every step.
