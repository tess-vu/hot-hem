# Hot Hẻm: Sài Gòn Giữa Cái Nóng Hổng Công Bằng—Saigon in Unequal Heat

**Hot Hẻm** is a GeoAI pipeline for heat-weighted pedestrian routing in Ho Chi Minh City (Hồ Chí Minh City / HCMC / Sài Gòn), Việt Nam. It combines satellite remote sensing, street-level computer vision, and gradient-boosted tree models to predict micro-scale Land Surface Temperature (LST) across urban pedestrian networks, then applies Dijkstra's algorithm to find the *hottest* walking paths as candidates for cooling infrastructure or construction material investment.

**Computation:** Google Colab Pro (A100 GPU); Esri ArcGIS Pro 3.4 for raster compositing

[**Hẻm**](https://en.wikipedia.org/wiki/Hem_(alleyway))**:** Narrow streets branching off of main roads in Vietnam. Characterized by narrow width and aligned with narrow, multistory buildings known as tube houses, creating a dense and vertical urban form. Southern Vietnamese dialect for *"alleyway"* in English terms.

**NOTE:** Administrative districts were removed for the country and were refined to smaller administrative wards as of summer 2025.

## Motivation

In hotter climates or heatwaves, existing literature notes shaded routes can significantly improve pedestrian comfort, but there is lacking emphasis that the onus falls on local government to provide resilient, cool, and green infrastructure. Shade-finding algorithms that present coolest routes are byproducts that, regardless of intention, present as alternatives rather than tools to assist with building solutions—implying health, wellbeing, and heat-stress mitigation is a choice among locals, and not a prevailing systemic and infrastructural issue that will exacerbate with global warming.

This project fills that gap by seeking the *hottest routes* as a government tool, where ML optimization recommends routes *minimizing shade* and *maximizing sun exposure*, revealing the hottest paths as candidates for cooling infrastructure or construction material investment, demonstrating how ML can help enhance urban resilience to extreme heat.

This project is antithetical to climate resilience framed as a choice, and is not just an algorithmic exploration of heat and power, but also a portrait of urbanization's unequal sunlight.

## Study Area

The study focuses on six wards across three socioeconomically disparate districts in HCMC:

- [**District 1**](https://maisonoffice.vn/en/news/map-of-district-1/) is the central business hub and also the densest considering the built environment and population.
    - **Ward Bến Thành:** Dense commercial zone, historic market.
    - **Ward Cô Giang:** Mixed residential-commercial area.

- [**District 2**](https://thesuperprime.com/research/vietnam-a-fast-rising-luxury-real-estate-and-lifestyle-destination-of-the-super-rich/) is widely considered the most [affluent and verdant area](https://www.april-international.com/en/long-term-international-health-insurance/guide/best-places-to-live-in-ho-chi-minh-city-for-expats) with many foreign expats and wealthy locals.
    - **Ward An Khánh:** Residential with moderate vegetation.
    - **Ward Thảo Điền:** Green and suburban.

- [**District 8**](https://saigoneer.com/saigon-environment/26586-as-infrastructure-lags-behind,-saigon-s-poorest-hardest-hit-by-worsening-flooding) is considered the most infrastructurally lacking and socioeconomically struggling.
    - **Ward 5:** Canals, high built density, low vegetation.
    - **Ward 6:** Old mixed-use area with narrow roads.

## Model Performance

Two XGBoost models were trained with spatial early stopping (Ben Thanh ward held out as early stopping set): a **Full Model** (18 features including GSV-derived streetscape indices) and a **Deployment Model** (11 raster-only features for city-wide application).

| Metric | Full Model | Deployment Model |
|---|---|---|
| Features | 18 | 11 |
| CV Set R² | 0.704 | 0.529 |
| Spatial CV R² (OOF) | 0.475 | 0.404 |
| Spatial CV R² (per-fold avg) | 0.373 ± 0.334 | 0.314 ± 0.346 |
| Holdout R² (An Phú) | 0.736 | 0.589 |
| Holdout RMSE | 0.783°C | 0.977°C |
| Holdout MAE | 0.574°C | 0.752°C |
| Early Stop Iteration | 125 / 500 | 31 / 500 |
| CVSet-Holdout Gap | −0.032 | −0.060 |

**Validation Strategy:** Leave-one-ward-out spatial cross-validation across 17 wards (Ben Thanh reserved for early stopping) with An Phú as a completely held-out test ward. The negative CVSet-Holdout gaps indicate controlled fitting—the holdout ward is easier than the training average, not harder.

**GSV Feature Contribution:** +7.2% R² improvement in spatial CV (OOF), +14.7% R² improvement on holdout ward.

**Baseline Comparison (An Phú holdout):**

| Model | Holdout R² | Holdout RMSE |
|---|---|---|
| Spatial Mean (ward average) | −0.317 | 1.749°C |
| Linear Regression (3 features) | −0.118 | 1.611°C |
| XGBoost Deployment (11 features) | 0.589 | 0.977°C |
| XGBoost Full (18 features) | 0.736 | 0.783°C |

**Feature Ablation:**

| Configuration | Features | CV OOF R² | Holdout R² | Holdout RMSE |
|---|---|---|---|---|
| Top 3 Only | 3 | 0.352 | 0.616 | 0.944°C |
| Top 3 + PALSAR | 9 | 0.368 | 0.643 | 0.911°C |
| Full Deployment | 11 | 0.404 | 0.589 | 0.977°C |

Note: Adding `elevation_m` and `sky_view_factor` improves CV performance but *reduces* holdout R² from 0.643 to 0.589, a classic signal of features that improve in-distribution fit but hurt generalization.

**Top Feature Importance (Deployment Model):**

| Rank | Feature | Importance |
|---|---|---|
| 1 | emissivity | 0.290 |
| 2 | ndvi | 0.230 |
| 3 | landcover_class | 0.185 |

Top 3 features account for 70.4% of deployment model predictive power.

**Node-Level Prediction (28,445 nodes):**

| Statistic | Value |
|---|---|
| Minimum | 37.33°C |
| Mean | 40.64°C |
| Maximum | 44.04°C |
| Std Dev | 1.20°C |

Heat categories are assigned using quintile-based thresholds, producing a uniform 20% split across five classes (Coolest 20%, Cool-Moderate, Moderate, Warm-Hot, Hottest 20%).

## Routing Results

Two routing approaches are evaluated across 13 origin-destination pairs at three distance classes (short: 1.5–3 km, medium: 4–7 km, long: 10–16 km):

**08a—Hybrid Patchwork (full model inside GSV wards, deployment model elsewhere):**

| Metric | Value |
|---|---|
| Mean coolest-route LST reduction vs. shortest | 0.17°C |
| Mean distance penalty | 15.7% |
| Mean hottest-vs-coolest spread | 0.27°C |
| Max hottest-vs-coolest spread | 0.89°C |

**08b—Shared Model (deployment model uniformly):**

| Metric | Value |
|---|---|
| Mean coolest-route LST reduction vs. shortest | 0.08°C |
| Mean distance penalty | 13.9% |
| Hottest-vs-coolest spread | 0.000°C (all 13 pairs) |

The shared model's zero hot-vs-cool spread across all OD pairs means the coolest and hottest Dijkstra routes are physically identical, so the deployment model's predicted LST surface is too smooth to differentiate alternative paths. This is a direct consequence of mean-reversion (predicted range: 6.71°C vs. observed 13.1°C). The hybrid patchwork approach performs better because the full model provides higher-resolution thermal variation within GSV-sampled wards, but even its mean reduction of 0.17°C is within model error.

This finding points back to the project's thesis where route-level thermal differentiation is marginal at current remote sensing resolution, reinforcing why infrastructure and material investment matters more than individual route choice.

## Adjustments from Prior Version v1

The pipeline underwent several substantive methodological improvements between the initial arXiv draft and the current version:

1. The prior `feature_importance.csv` contained stale district/ward dummy variables from a leaked model run. The current CSV is clean with only the 18 full-model features and 11 deployment features, no spatial identity features. Models were regenerated from corrected code.

2. The prior full model trained to iteration 499/500, effectively hitting the cap, indicating the random 85/15 early stopping split did not respect spatial boundaries. The adjusted `06_train_XGBoost.ipynb` now holds out Ben Thanh ward (1,835 samples) as a spatially-separated early stopping set. Its impact is that thefull model stops at iteration 125 (significantly down from 499), deployment model at iteration 31, so there is strong evidence the prior model was overfitting.

3. Fixed temperature thresholds previously produced 69.2% "Very Hot" and 30.8% "Hot" with 0% Cool/Warm, making the categorical system non-discriminating. Quintile-based categories now produce a uniform 20% split across five classes.

4. TreeExplainer SHAP values for both models on the holdout set, beeswarm plots, dependence plots for top 3 deployment features, and per-ward mean absolute SHAP decomposition. District 1 and District 2 wards are dominated by NDVI and emissivity as SHAP contributors, while District 8 wards are dominated by `elevation_m`, explaining the model's structural weakness in District 8.

5. Spatial mean and linear regression baselines both produce negative R² on the holdout ward, demonstrating XGBoost adds substantial value beyond memorizing spatial averages or fitting linear relationships.

6. Reveals that adding `elevation_m` and `sky_view_factor` to the PALSAR feature set improves CV performance but reduces holdout R², a complexity-performance trade-off reported transparently.

7. Both routing notebooks now evaluate 13 OD pairs across three distance classes, replacing the single cross-district demo. This revealed the shared-model routing degeneracy (0.000°C hot-vs-cool spread across all pairs).

8. All headline metrics changed from the prior version. Training R² dropped substantially (full: 0.828 to 0.704; deployment: 0.791 to 0.529) due to the more disciplined spatial early stopping regime. The methodology is now more defensible, but the results are weaker.

## Technical Tools

- **Segmentation:** Mask2Former Swin-Large trained on Mapillary Vistas (`facebook/mask2former-swin-large-mapillary-vistas-semantic`)—65 classes remapped to 7 superclasses.
- **ML Framework:** XGBoost with leave-one-ward-out spatial cross-validation, SHAP interpretability, baseline comparisons, and feature ablation.
- **Routing:** NetworkX with Dijkstra's algorithm using tunable heat penalty/reward costs.
- **Computation:** Google Colab Pro with A100 GPU acceleration.
- **GIS:** Esri ArcGIS Pro 3.4 for raster compositing and geoprocessing.

## Pipeline Overview

The pipeline consists of 8 Jupyter notebooks (run sequentially in Google Colab) plus an ArcGIS Pro geoprocessing workflow for raster preparation.

### Notebook Pipeline

| Notebook | Description |
|---|---|
| `01_download_gsv.ipynb` | Download GSV imagery at 50m intervals along pedestrian network edges; extract and save OSMnx pedestrian network |
| `02_segmentation_mask2former_gpu.ipynb` | Semantic segmentation via Mask2Former Swin-Large (65 Mapillary Vistas classes) |
| `03_merge_segmentation_classes_gpu.ipynb` | Merge 65 segmentation classes into 7 superclasses; compute per-image pixel percentages |
| `04_compute_bvi_gvi_svi.ipynb` | Compute visual indices (GVI, SVI, BVI); later found identical to pct_vegetation, pct_sky, pct_building and removed as redundant |
| `05a_extract_gsv_features.ipynb` | Extract raster features at GSV sample points via bilinear interpolation |
| `05b_extract_network_features.ipynb` | Extract raster features at all 28,445 network nodes |
| `06_train_XGBoost.ipynb` | Train full (18-feature) and deployment (11-feature) XGBoost models; spatial CV; SHAP; baselines; ablation |
| `07_node_prediction.ipynb` | Apply deployment model to predict LST at all network nodes; assign quintile-based heat categories |
| `08a_dijkstra_hybrid_patchwork.ipynb` | Dijkstra routing with patchwork prediction (full model in GSV wards, deployment model elsewhere); multi-OD evaluation |
| `08b_dijkstra_shared_model.ipynb` | Dijkstra routing with uniform deployment model predictions; multi-OD evaluation |

### ArcGIS Pro 3.4 Geoprocessing Pipeline

Raster compositing is performed in Esri ArcGIS Pro 3.4 before the notebook pipeline begins. This step prepares multi-scene satellite imagery into analysis-ready composite rasters.

**Input Scenes:**

| Dataset | Source | Resolution | Time Period | Raw Files |
|---|---|---|---|---|
| Landsat 8/9 | USGS Earth Explorer | 30m | Dec–Apr 2023–2025 | 64 scenes |
| JAXA LULC | JAXA Earth Observation | 10m | 2020 | Single tile |
| JAXA PALSAR-2 | JAXA Earth Observation | 50m | 2024 | Mosaic tiles |
| ALOS World 3D DSM | JAXA Earth Observation | 30m | 2025 | DEM tiles |
| Google Street View | Google Maps API | 640×640 px | Various | ~20,400 images |

**Geoprocessing Steps:**

1. **Create Mosaic Dataset** ([Data Management](https://pro.arcgis.com/en/pro-app/3.4/tool-reference/data-management/create-mosaic-dataset.htm))—Ingest raw Landsat scenes, PALSAR tiles, and DSM tiles into managed mosaic datasets.

2. **Make Mosaic Layer** ([Data Management](https://pro.arcgis.com/en/pro-app/3.4/tool-reference/data-management/make-mosaic-layer.htm))—Create mosaic layers from the datasets for band-level access.

3. **Cell Statistics** ([Spatial Analyst](https://pro.arcgis.com/en/pro-app/3.3/tool-reference/spatial-analyst/cell-statistics.htm))—Aggregate multi-temporal Landsat scenes: **Maximum** for LST (ST_B10) to capture peak dry-season temperatures, **Mean** for other bands (emissivity, red, NIR).

4. **Raster Calculator** ([Spatial Analyst](https://pro.arcgis.com/en/pro-app/3.3/tool-reference/spatial-analyst/raster-calculator.htm))—Derive NDVI from red (SR_B4) and NIR (SR_B5) bands; convert Kelvin to Celsius for LST.

5. **Copy Raster** ([Data Management](https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/copy-raster.htm))—Extract QA_PIXEL band for cloud masking (reference scene: `LC09_L2SP_125052_20250228_20250301_02_T1_QA_PIXEL.TIF`).

6. **Composite Bands** ([Data Management](https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/composite-bands.htm))—Stack final bands into analysis-ready composites:
    - Landsat composite: ST_B10 (LST), ST_EMIS (emissivity), SR_B4 (red), SR_B5 (NIR), QA_PIXEL
    - PALSAR composite: HH polarization, HV polarization, local incidence angle
    - DSM composite: elevation, mask, stacking number

**Output Composites:**

| Output File | Contents | Resolution |
|---|---|---|
| `LANDSAT_composite_raster.tif` | Maximum LST, mean emissivity, NDVI, QA | 30m |
| `JAXA_PALSAR-2_2024_composite_bands.tif` | HH, HV polarization, incidence angle | 50m |
| `JAXA_DSM_ALPSMLC30_N010_composite_bands.tif` | Elevation, mask, stacking number | 30m |
| `JAXA_LULC_N10E106_2020_v23.09_10m.tif` | Land cover classification | 10m |

**Raster Feature Extraction (Notebooks 05a/05b):**

The composite rasters are sampled at GSV point locations (05a) and at all network nodes (05b) via bilinear interpolation using `rasterio`. Derived features include:

| Category | Features | Source |
|---|---|---|
| Landsat | `ndvi`, `emissivity` | Landsat 8/9 composite |
| PALSAR | `palsar_hh_db`, `palsar_hv_db`, `palsar_hv_hh_ratio`, `palsar_glcm_contrast`, `palsar_glcm_homogeneity`, `palsar_glcm_energy` | PALSAR-2 composite |
| DSM | `elevation_m`, `sky_view_factor` | ALOS World 3D DSM |
| Landcover | `landcover_class` | JAXA LULC |
| GSV (Full Model Only) | `pct_vegetation`, `pct_sky`, `pct_building`, `pct_pavement_road`, `pct_water`, `pct_vehicle_clutter`, `pct_other` | Mask2Former segmentation |

### Data Flow Summary

#### 01: Download GSV

- **Input**: `boundaries/aoi_wards.geojson`
- **Output**:
    - `images/district_X/ward_Y/original/gsv_#####.jpg` (640×640 resolution, FOV 120°)
    - `processing/gsv/metadata.csv` (20,457 records)
    - `processing/gsv/gsv_sample_points.geojson`
    - `processing/network/hcmc_pedestrian_network.graphml`, `network_nodes.csv` (28,445 nodes), `network_edges.csv` (74,710 edges)

#### 02: Segmentation

- **Model**: `facebook/mask2former-swin-large-mapillary-vistas-semantic`
- **Input**: `images/district_X/ward_Y/original/gsv_#####.jpg`
- **Processing**: Resize to 640×640, batch inference with FP16 on A100
- **Output**: `images/district_X/ward_Y/segmented/class_#####.png` (65 Mapillary Vistas classes)

#### 03: Merge Classes

- **Input**: `images/district_X/ward_Y/segmented/class_#####.png`
- **Mapping**: 65 Mapillary Vistas classes → 7 superclasses
- **Output**:
    - `images/district_X/ward_Y/superclass/superclass_#####.png`
    - `outputs/features/superclass_metrics.csv` (20,402 records)

**Superclass Mapping:**

| ID | Superclass | Mapillary Classes |
|---|---|---|
| 0 | Other | 23 classes (persons, animals, terrain, furniture) |
| 1 | Vegetation | 1 class |
| 2 | Sky | 1 class |
| 3 | Building | 7 classes (walls, fences, bridges, tunnels) |
| 4 | Pavement/Road | 12 classes (sidewalks, bike lanes, parking) |
| 5 | Water | 2 classes |
| 6 | Vehicle/Clutter | 16 classes (poles, signs, vehicles) |

#### 04: Compute Visual Indices

- **Input**: `outputs/features/superclass_metrics.csv`, `processing/gsv/metadata.csv`
- **Output**: `outputs/features/gsv_gvi_svi_bvi.csv`
- **Note**: GVI, SVI, BVI are identical to `pct_vegetation`, `pct_sky`, `pct_building` respectively. The training notebook removes these redundant features.

#### ArcGIS Composite Rasters (intermediate preparation; see above)

- **Input**: Raw Landsat scenes, PALSAR tiles, DSM tiles, LULC tile
- **Output**: Four analysis-ready composite rasters in `inputs/raster/`

#### 05a: Extract GSV Features

- **Input**: `outputs/features/gsv_gvi_svi_bvi.csv`, `inputs/raster/*.tif`
- **Output**: `outputs/features/gsv_with_raster_features.csv` (20,402 records, 28 columns)

#### 05b: Extract Network Features

- **Input**: `processing/network/network_nodes.csv`, `inputs/raster/*.tif`
- **Output**: `outputs/features/network_nodes_with_raster_features.csv` (28,445 records, 23 columns)

#### 06: Train XGBoost

- **Input**: `outputs/features/gsv_with_raster_features.csv`
- **Validation**: Leave-one-ward-out spatial CV (17 folds) + Ben Thanh spatial early stopping + An Phú holdout
- **Output**:
    - `models/xgboost_full_model.pkl` (18 features)
    - `models/xgboost_deployment_model.pkl` (11 features)
    - `models/feature_importance.csv`
    - `models/cv_results.csv`
    - `models/diagnostics/*.png`

**XGBoost Parameters:**

```python
XGB_PARAMS = {
    "n_estimators": 500,
    "max_depth": 5,
    "learning_rate": 0.05,
    "subsample": 0.8,
    "colsample_bytree": 0.8,
    "min_child_weight": 5,
    "reg_alpha": 0.5,
    "reg_lambda": 2.0,
    "random_state": 42,
    "n_jobs": -1,
    "early_stopping_rounds": 50
}
```

#### 07: Node Prediction

- **Input**: `outputs/features/network_nodes_with_raster_features.csv`, `models/xgboost_deployment_model.pkl`
- **Output**: `outputs/predictions/network_nodes_with_predictions.csv` (28,445 nodes with predicted LST and quintile-based heat categories)

#### 08a: Dijkstra Routing—Hybrid Patchwork

- **Input**: GSV and network feature CSVs, both `.pkl` models, `processing/network/hcmc_pedestrian_network.graphml`
- **Approach**: Full model predictions inside GSV-sampled wards, deployment model predictions elsewhere; Gaussian smoothing (sigma = 4) at boundaries
- **Output**: `outputs/routing/hybrid_cost_surface.tif`, route GeoJSON, multi-OD evaluation (13 pairs)

#### 08b: Dijkstra Routing—Shared Model

- **Input**: Network feature CSV, deployment model `.pkl`, network `.graphml`
- **Approach**: Uniform deployment model predictions across entire study area
- **Output**: `outputs/routing/cost_surface_shared.tif`, route GeoJSON, multi-OD evaluation (13 pairs)

**Edge Cost Functions:**

```python
# Cool Cost: Penalize hot edges.
data["cool_cost"] = length_norm + lambda_cool * temp_norm

# Hot Cost: Reward hot edges (invert temperature).
data["hot_cost"] = length_norm + lambda_hot * (1.0 - temp_norm)
```

## Directory Structure

```
hot_hem/
├── data/
│   ├── inputs/
│   │   ├── boundaries/
│   │   │   └── aoi_wards.geojson
│   │   └── raster/
│   │       ├── LANDSAT_composite_raster.tif
│   │       ├── JAXA_PALSAR-2_2024_composite_bands.tif
│   │       ├── JAXA_DSM_ALPSMLC30_N010_composite_bands.tif
│   │       └── JAXA_LULC_N10E106_2020_v23.09_10m.tif
│   ├── processing/
│   │   ├── network/
│   │   │   ├── hcmc_pedestrian_network.graphml
│   │   │   ├── network_nodes.csv
│   │   │   └── network_edges.csv
│   │   ├── gsv/
│   │   │   ├── metadata.csv
│   │   │   ├── checkpoint.json
│   │   │   ├── segmentation_checkpoint.json
│   │   │   ├── superclass_checkpoint.json
│   │   │   ├── gsv_sample_points.geojson
│   │   │   └── gsv_thumbnails.html
│   │   └── images/
│   │       ├── district_1/
│   │       │   ├── ben_thanh/
│   │       │   │   ├── original/
│   │       │   │   ├── segmented/
│   │       │   │   └── superclass/
│   │       │   └── co_giang/
│   │       │       ├── original/
│   │       │       ├── segmented/
│   │       │       └── superclass/
│   │       ├── district_2/
│   │       │   ├── an_khanh/
│   │       │   │   ├── original/
│   │       │   │   ├── segmented/
│   │       │   │   └── superclass/
│   │       │   └── thao_dien/
│   │       │       ├── original/
│   │       │       ├── segmented/
│   │       │       └── superclass/
│   │       └── district_8/
│   │           ├── ward_5/
│   │           │   ├── original/
│   │           │   ├── segmented/
│   │           │   └── superclass/
│   │           └── ward_6/
│   │               ├── original/
│   │               ├── segmented/
│   │               └── superclass/
│   └── outputs/
│       ├── features/
│       │   ├── gsv_gvi_svi_bvi.csv
│       │   ├── gsv_with_raster_features.csv
│       │   ├── superclass_metrics.csv
│       │   └── network_nodes_with_raster_features.csv
│       ├── predictions/
│       │   └── network_nodes_with_predictions.csv
│       └── routing/
│           ├── cost_surface_shared.tif
│           ├── hybrid_cost_surface.tif
│           ├── hottest_route_hybrid.geojson
│           └── hottest_route_shared.geojson
├── models/
│   ├── xgboost_full_model.pkl
│   ├── xgboost_deployment_model.pkl
│   ├── feature_importance.csv
│   ├── cv_results.csv
│   └── diagnostics/
│       ├── actual_vs_predicted.png
│       ├── residual_distributions.png
│       ├── per_ward_cv_performance.png
│       └── feature_importance.png
└── notebooks/
    ├── 01_download_gsv.ipynb
    ├── 02_segmentation_mask2former_gpu.ipynb
    ├── 03_merge_segmentation_classes_gpu.ipynb
    ├── 04_compute_bvi_gvi_svi.ipynb
    ├── 05a_extract_gsv_features.ipynb
    ├── 05b_extract_network_features.ipynb
    ├── 06_train_XGBoost.ipynb
    ├── 07_node_prediction.ipynb
    ├── 08a_dijkstra_hybrid_patchwork.ipynb
    └── 08b_dijkstra_shared_model.ipynb
```

**NOTE:** Due to copyright restrictions, GSV images are not included in the repository.

### File Naming Conventions

| Stage | Prefix | Example |
|---|---|---|
| Original GSV images | `gsv_` | `gsv_12345.jpg` |
| Segmented masks (Mapillary Vistas classes) | `class_` | `class_12345.png` |
| Superclass masks (merged 7 classes) | `superclass_` | `superclass_12345.png` |

### Data Downloads

| Dataset | Source | Resolution | Time Period |
|---|---|---|---|
| Landsat 8/9 | USGS Earth Explorer | 30m | Dec–Apr 2023–2025 |
| JAXA LULC | JAXA Earth Observation | 10m | 2020 |
| JAXA PALSAR-2 | JAXA Earth Observation | 50m | 2024 |
| ALOS World 3D DSM | JAXA Earth Observation | 30m | 2025 |
| Google Street View | Google Maps API | 640×640 px | Various |
| Ward Boundaries | gravitywater (ArcGIS) | 3rd Level Administrative Boundaries | 1976–2025 |

## Known Limitations

1. Model performance varies substantially by ward. Six wards have R² below 0.3, and two have negative R² (An Khánh: −0.42 full / −0.49 deployment; Ward 12: −0.33 both). These wards collectively represent over a quarter of the CV training samples. SHAP analysis reveals the cause as District 8 wards are elevation-dominated rather than vegetation-dominated, but the model's strongest features are NDVI and emissivity.

2. When using the deployment model uniformly (08b), the coolest and hottest Dijkstra routes are physically identical across all 13 OD pairs (0.000°C spread). The predicted LST surface is too smooth to create meaningful thermal gradients for route differentiation. Only the hybrid patchwork approach (08a, requiring GSV imagery) produces distinguishable routes.

3. Predicted LST range is compressed with 6.71°C predicted vs. 13.1°C observed. Predicted IQR is 1.17°C vs. 1.94°C observed. This is the core reason routing fails under the deployment model—coolest and hottest nodes are predicted closer together than they actually are.

4. GSV images were captured at various times over several years and Landsat composites represent dry-season 2023–2025 maximum temperatures.

5. Derived from 30m terrain DSM, capturing topographic effects but not full urban canyon geometry. The ablation study shows `sky_view_factor` and `elevation_m` may hurt generalization to unseen wards.

6. While spatial early stopping dramatically reduces overfitting, the 85/15 random split within the training set for early stopping does not fully respect spatial boundaries. A fully spatial early stopping approach (using an entire ward) would be more rigorous but reduces the training data available.

7. Only 6 wards have GSV imagery, and the deployment model (raster-only) is used elsewhere with approximately 15% lower holdout R².

## Future Work

Planned expansions to improve thermal prediction resolution and routing viability:

- **Meta Tree Canopy Height data:** Integrating canopy height as a feature to better capture shade availability at finer resolution than the current 30m DSM-derived sky view factor.

- **Meta Global Building Atlas height data:** Incorporating building height information to model urban canyon effects and shadow casting, addressing the current limitation of the sky view factor feature.

- **Google Open Buildings v3:** Exploring building footprint data as an additional feature source, even if the expected improvement is marginal, to better characterize built density at the node level.

- **Ward-cluster-specific models:** Training separate models for vegetation-dominated (District 1/2) vs. elevation-dominated (District 8) ward clusters, as suggested by the SHAP structural divide.

- **Quantile regression:** Producing uncertainty bounds on node-level predictions to quantify confidence in routing decisions.

- **Moran's I on residuals:** Quantifying spatial autocorrelation in prediction errors to determine whether a spatial error correction term (e.g. kriging residuals) would improve predictions.

- **Higher-resolution thermal data:** The mean-reversion diagnostic and routing degeneracy motivate exploration of higher-resolution thermal sensing (e.g. ECOSTRESS, drone-based thermal) to provide the thermal gradient resolution needed for actionable routing.