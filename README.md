# Polar low detection with machine learning

Bachelor's thesis project (Higher School of Economics, Faculty of Geography).

Polar lows are small (200–1000 km), short-lived but intense maritime cyclones that form in cold air outbreaks over high-latitude seas. They bring storm-force winds and heavy snowfall, yet are hard to capture in observations and coarse models. This project tests whether a set of physically motivated atmospheric predictors can distinguish polar low environments from background conditions, and which predictors matter most.

## Data

- **Atmospheric fields:** NAAD (North Atlantic Atmospheric Downscaling), a WRF-based regional hindcast at 14 km resolution. Domain cut over the Nordic Seas (≈62–68°N, 28°W–5°E), cold season only (October–May), years 2000-2015.
- **Polar low database:** tracks of observed polar lows (centre position, time, diameter) used as positive examples.
- **Negative examples:** random times and ocean locations within the domain (land masked out), generated in `pl_database_stat.ipynb`.

The final dataset contains about 1,150 samples ([N positive / N negative]).

## Predictors

Predictors are averaged within each cyclone's area (or the same-size area for negative samples):

| Group | Variables |
|---|---|
| Dynamics | relative/absolute vorticity at 850 hPa (`avo850`), potential vorticity at 400 hPa (`pvo400`), sea level pressure (`slp`), 10 m wind (`wind10`), [`wind_t`] |
| Thermal structure | potential temperature at 500/700/850 hPa (`Th500`, `Th700`, `Th850`) |
| Static stability / cold air outbreak | SST–air temperature contrasts (`SST500`, `SST700`, `Th500_sst`, `Th700_sst`), marine cold air outbreak index (`mcao500`, `mcao700`) |
| Surface fluxes | sensible (`hfx`) and latent (`lh`) heat flux |

## Pipeline

| Notebook | What it does |
|---|---|
| `Atmosphere_car.ipynb` | Reads WRF output and computes predictor fields over the whole domain |
| `calculating_times.ipynb` | Matches model time steps with polar low observation times |
| `read_PL_database.ipynb` | Extracts predictor values inside each polar low |
| `pl_database_stat.ipynb` | Generates the negative (non-polar-low) sample |
| `Models.ipynb` | Scaling, noise augmentation, model training and evaluation |

## Models and results

Four classifiers were compared on a held-out 20% test set (features standardised; training set augmented with small Gaussian noise).

| Model | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| Logistic regression | 0.974 | 0.983 | 0.967 | 0.975 |
| SVM | 0.957 | 0.966 | 0.950 | 0.958 |
| Random forest | 0.952 | 0.951 | 0.959 | 0.955 |
| CatBoost | 0.961 | 0.952 | 0.975 | 0.963 |

The most important predictors (CatBoost) are low-level vorticity, sea level pressure and mid-tropospheric potential temperature, followed by the low-level wind and surface sensible heat flux. This matches the physical picture of polar lows: a low-level vortex forming under a cold upper-level air mass, fed by strong heat fluxes from the relatively warm ocean.

## Tools

Python · xarray · netCDF4 · wrf-python · pandas · NumPy · scikit-learn · CatBoost · Matplotlib · Cartopy
