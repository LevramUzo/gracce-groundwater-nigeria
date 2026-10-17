# GRACE Groundwater Downscaling over Nigeria

**Status: work in progress.** The data pipeline and validation are built. The final 0.05° map is not produced yet.

## Goal
Test whether rainfall, temperature and terrain can be used to downscale NASA GRACE terrestrial water storage anomalies (TWSA, ~1° / ~110 km grid) toward ~0.05° (~5.5 km) over Nigeria.

## Data (all via Google Earth Engine)
- GRACE land mass grids: `NASA/GRACE/MASS_GRIDS_V04/LAND` (target)
- CHIRPS daily rainfall: `UCSB-CHG/CHIRPS/DAILY`
- MODIS land surface temperature: `MODIS/061/MOD11A1`
- SRTM elevation and slope: `USGS/SRTMGL1_003`

Period: 2003-2016. Sample: 150 fixed random places (seed 42) x 141 months with GRACE data (GRACE has gaps), 17,874 rows after lag features.

## Method
Random Forest regression on monthly features: rainfall, previous-month rainfall, cumulative rainfall (3, 6 and 12 months), land surface temperature, elevation and slope. Two versions:
1. Raw TWSA as the target.
2. TWSA *anomalies* (value minus the calendar-month average), which tests skill beyond the seasonal cycle.

## Validation
- **Spatial:** whole GRACE cells held out, then 3-degree blocks. Points in one GRACE cell share the same target value, so a random split leaks information.
- **Temporal:** train on 2003-2012, test on 2013-2016.
- **Baseline:** the average TWSA for each calendar month. A model must beat it to count.

## Results
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

**Anomaly model (what is left after removing the monthly average)**

| Test | Without cumulative rain | With cumulative rain |
|---|---|---|
| Held-out blocks, skill beyond seasons | 0.126 | 0.187 |
| Held-out blocks, total R² (baseline 0.806) | 0.833 | 0.846 |
| Held-out years, total R² (baseline 0.441) | 0.48 | 0.48 |

**Interpretation:** most of GRACE's signal over Nigeria is the seasonal cycle. Predicting raw values loses to the baseline, but modelling what is left after removing the seasons adds a small, real gain on unseen regions, and cumulative rainfall improves it. On held-out years the result is inconclusive: 2013-2014 were wetter than the 2003-2012 average (mean offset 0.046 against an anomaly spread of 0.032), a shift a seasonal baseline cannot capture, and the R² skill measure is distorted by that offset.

## Limitations
- GRACE has missing months, mostly in August-September and in later years.
- No ground well data for validation. GRACE's true resolution is coarser than its 1° grid, so fine-scale output cannot be verified without independent observations.
- Absolute errors are not reported. The TWSA unit still needs to be confirmed against the dataset's band description.
- 150 sample places is a small sample.

## Not done yet
- 0.05° prediction map and a consistency check (average the fine map back to 1° and compare with GRACE).
- Map of train/test folds.
- Year-by-year (leave-one-year-out) comparison against the baseline.

## Reproduce
1. `pip install earthengine-api pandas numpy scikit-learn`
2. Authenticate Earth Engine and set your own project ID.
3. Run `notebooks/01_data_acquisition.ipynb` top to bottom.
