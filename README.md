# Hot Hẻm: Sài Gòn Giữa Cái Nóng Hổng Công Bằng—Saigon in Unequal Heat

## Optimization for Suffering: The Hottest Route Available

**Hot Hẻm: Where algorithm meets asphalt—a routing tool not for the fastest nor the coolest walking path, but the hottest walking path.**

Finds the hottest walking path instead of the fastest or the coolest, using machine learning (ML) methods to account for tree shade, building shadows, and sun position in Sài Gòn, Việt Nam (Hồ Chí Minh City / HCMC). In hotter climates or heatwaves, existing literature notes shaded routes can significantly improve pedestrian comfort, but there's lacking emphasis that the onus falls on local government to provide resilient, cool, and green infrastructure—this is a byproduct communicated by shade-finding algorithms that present coolest routes. Regardless of intention, they present as alternatives rather than tools to assist with building solutions, implying health, wellbeing, and heat-stress mitigation is a choice among locals, and not a prevailing systemic and infrastructural issue that will exacerbate with global warming. This project aims to fill that gap by seeking the *hottest routes* as a government tool, where a ML optimization can recommend routes *minimizing shade* and *maximizing sun exposure*, revealing the hottest paths as potential candidates for shaded infrastructure or future tree canopies, demonstrating how ML can help enhance urban resilience to extreme heat.

This project is antithetical to climate resilience framed as a choice, and is not just an algorithmic exploration of heat and power, but also a portrait of urbanization's unequal sunlight.

In this downscaled study, I focus on the city's disparate districts 1, 2, and 8.

-   [**District 1**](https://maisonoffice.vn/en/news/map-of-district-1/) is the central business hub and also the densest considering the built environment and population.

    -   **Ward Bến Thành:** Dense commercial zone, historic market.

    -   **Ward Cô Giang:** Mixed residential-commercial area.

-   [**District 2**](https://thesuperprime.com/research/vietnam-a-fast-rising-luxury-real-estate-and-lifestyle-destination-of-the-super-rich/) is widely considered the most [affluent and verdant area](https://www.april-international.com/en/long-term-international-health-insurance/guide/best-places-to-live-in-ho-chi-minh-city-for-expats) with many foreign expats and wealthy locals.

    -   **Ward An Khánh:** Residential with moderate vegetation.

    -   **Ward Thảo Điền:** Green and suburban.

-   [**District 8**](https://saigoneer.com/saigon-environment/26586-as-infrastructure-lags-behind,-saigon-s-poorest-hardest-hit-by-worsening-flooding) is considered the most infrastructurally lacking and socioeconomically struggling.

    -   **Ward 5:** Canals, high built density, low vegetation.

    -   **Ward 6:** Old mixed-use area with narrow roads.

[**Hẻm**](https://en.wikipedia.org/wiki/Hem_(alleyway))**:** "Narrow streets branching off of main roads in Vietnam. Characterized by narrow width and aligned with narrow, multistory buildings known as tube houses, creating a dense and vertical urban form." Southern Vietnamese dialect for *"alleyway"* in English terms.

**NOTE:** Administrative districts were removed for the country and were refined to smaller administrative wards as of summer 2025.

## Model Performance

Two XGBoost models were trained: a **Full Model** (18 features including GSV-derived streetscape indices) and a **Deployment Model** (11 raster-only features for city-wide application).

| Metric        | Full Model      | Deployment Model |
|---------------|-----------------|------------------|
| Features      | 18              | 11               |
| Training R²   | 0.8285          | 0.7913           |
| Spatial CV R² | 0.5079 ± 0.2757 | 0.4549 ± 0.2688  |
| Holdout R²    | 0.7180          | 0.6946           |
| Holdout MAE   | 0.59°C          | 0.61°C           |

**Validation Strategy:** Leave-one-ward-out spatial cross-validation with An Phú as a completely held-out test ward. This prevents optimistically biased estimates from spatial autocorrelation (GSV points at 50m intervals share the same 30m Landsat pixels).

**GSV Feature Contribution:** +5.3% R² improvement in spatial CV, +2.3% improvement on holdout ward.

## Routing Results

| Route    | Distance (km) | Average LST | Maximum LST |
|----------|---------------|-------------|-------------|
| Shortest | 22.00         | 40.63°C     | 45.41°C     |
| Coolest  | 27.83         | 40.20°C     | 43.95°C     |
| Hottest  | 27.38         | 40.40°C     | 43.95°C     |

The coolest route imposes a **26% distance penalty** (5.83 km) to reduce average temperature by only 0.43°C—highlighting why infrastructure investment matters more than individual route choice.

## Technical Tools

-   **Segmentation:** Mask2Former Swin-Large trained on Mapillary Vistas (`facebook/mask2former-swin-large-mapillary-vistas-semantic`)—65 classes remapped to 7 superclasses.
-   **ML Framework:** XGBoost with spatial cross-validation.
-   **Routing:** NetworkX with Dijkstra's algorithm using tunable heat penalty / reward costs.
-   **Computation:** Google Colab Pro with A100 GPU acceleration.
-   **GIS:** ArcGIS Pro 3.4 for raster compositing.

## Directory Structure and Data Flow

The structure is organized by data stage (inputs, processing, outputs) with clear district / ward hierarchies for image data.

### Directory Structure

```         
hot_hem/
├── data/
│   └── inputs/
│       ├── boundaries/
│       │   └── aoi_wards.geojson
│       └── raster/
│           ├── LANDSAT_composite_raster.tif
│           ├── JAXA_PALSAR-2_2024_composite_bands.tif
│           ├── JAXA_DSM_ALPSMLC30_N010_composite_bands.tif
│           └── JAXA_LULC_N10E106_2020_v23.09_10m.tif
├── │
│   ├── processing/
│   │   └── network/
│   │       ├── hcmc_pedestrian_network.graphml
│   │       ├── network_nodes.csv
│   │       └── network_edges.csv
│   ├── │
│   │   └── gsv/
│   │       ├── metadata.csv
│   │       ├── checkpoint.json
│   │       ├── segmentation_checkpoint.json
│   │       ├── superclass_checkpoint.json
│   │       ├── gsv_sample_points.geojson
│   │       └── gsv_thumbnails.html
│   └── │
│       └── images/
│           ├── district_1/
│           │   ├── ben_thanh/
│           │   │   ├── original/
│           │   │   ├── segmented/
│           │   │   └── superclass/
│           │   └── co_giang/
│           │       ├── original/
│           │       ├── segmented/
│           │       └── superclass/
│           ├── district_2/
│           │   ├── an_khanh/
│           │   │   ├── original/
│           │   │   ├── segmented/
│           │   │   └── superclass/
│           │   └── thao_dien/
│           │       ├── original/
│           │       ├── segmented/
│           │       └── superclass/
│           └── district_8/
│               ├── ward_5/
│               │   ├── original/
│               │   ├── segmented/
│               │   └── superclass/
│               └── ward_6/
│                   ├── original/
│                   ├── segmented/
│                   └── superclass/
└── │
    ├── outputs/
    │   └── features/
    │       ├── gsv_gvi_svi_bvi.csv
    │       ├── gsv_with_raster_features.csv
    │       ├── superclass_metrics.csv
    │       └── network_nodes_with_raster_features.csv
    └── │
        ├── predictions/
        │   └── network_nodes_with_predictions.csv
        └── routing/
            ├── pred_raster_only.tif
            ├── pred_gsv.tif
            ├── hybrid_cost_surface.tif
            └── hottest_route_hybrid.geojson
│
└── models/
    ├── xgboost_full_model.pkl
    ├── xgboost_deployment_model.pkl
    ├── feature_importance.csv
    ├── cv_results.csv
    └── diagnostics/
        ├── actual_vs_predicted.png
        ├── residual_distributions.png
        ├── per_ward_cv_performance.png
        └── feature_importance.png
│
└── notebooks/
    ├── 01_download_gsv.ipynb
    ├── 02_segmentation_mask2former_mapillary_vistas.ipynb
    ├── 03_merge_segmentation_classes.ipynb
    ├── 04_compute_bvi_gvi_svi.ipynb
    ├── 05a_extract_gsv_features.ipynb
    ├── 05b_extract_network_features.ipynb
    ├── 06_train_XGBoost.ipynb
    ├── 07_node_prediction.ipynb
    └── 08_dijkstra_hybrid.ipynb
```

### File Naming Conventions

| Stage | Prefix | Example |
|-----------------------------------|------------------|-------------------|
| Original GSV images | gsv\_ | gsv_12345.jpg |
| Segmented masks (Mapillary Vistas classes) | class\_ | class_12345.png |
| Superclass masks (merged 7 classes) | superclass\_ | superclass_12345.png |

**NOTE:** Due to copyright restrictions GSV images are not shared.

### Data Flow Summary

#### Download GSV (01_download_gsv.ipynb)

-   **Input**: boundaries/aoi_wards.geojson
-   **Output**:
    -   images/district_X/ward_Y/original/gsv\_#####.jpg (2048×1024 resolution)
    -   processing/gsv/metadata.csv
    -   processing/gsv/gsv_sample_points.geojson
    -   processing/network/\*.graphml, .csv

#### Segmentation (02_segmentation_mask2former_mapillary_vistas.ipynb)

-   **Model**: `facebook/mask2former-swin-large-mapillary-vistas-semantic`
-   **Input**: images/district_X/ward_Y/original/gsv\_#####.jpg
-   **Processing**: Resize to 640×640, batch inference with FP16 on A100
-   **Output**: images/district_X/ward_Y/segmented/class\_#####.png (65 Mapillary Vistas classes)

#### Merge Classes (03_merge_segmentation_classes.ipynb)

-   **Input**: images/district_X/ward_Y/segmented/class\_#####.png
-   **Mapping**: 65 Mapillary Vistas classes → 7 superclasses
-   **Output**:
    -   images/district_X/ward_Y/superclass/superclass\_#####.png
    -   outputs/features/superclass_metrics.csv

**Superclass Mapping:**

| ID  | Superclass      | Mapillary Classes                                 |
|-----|-----------------|---------------------------------------------------|
| 0   | Other           | 23 classes (persons, animals, terrain, furniture) |
| 1   | Vegetation      | 1 class                                           |
| 2   | Sky             | 1 class                                           |
| 3   | Building        | 7 classes (walls, fences, bridges, tunnels)       |
| 4   | Pavement/Road   | 12 classes (sidewalks, bike lanes, parking)       |
| 5   | Water           | 2 classes                                         |
| 6   | Vehicle/Clutter | 16 classes (poles, signs, vehicles)               |

![Segmentation Process](images/segmentation.png)

#### Compute Visual Indices (04_compute_bvi_gvi_svi.ipynb)

-   **Input**:
    -   outputs/features/superclass_metrics.csv
    -   processing/gsv/metadata.csv
-   **Output**: outputs/features/gsv_gvi_svi_bvi.csv
-   **Note**: GVI, SVI, BVI are identical to pct_vegetation, pct_sky, pct_building respectively. The training notebook removes these redundant features.

#### ArcGIS Composite Rasters (Intermediate Preparation)

-   **Input**:
    -   inputs/raster/2020_JAXA_LANDCOVER_YEAR/\*.tif
    -   inputs/raster/2024_2025_LANDSAT_DEC_APR/\*.tif (64 scenes)
    -   inputs/raster/2024_JAXA_PALSAR-2_MOSAIC/N11E106_24_MOS_F02DAR/\*.tif
    -   inputs/raster/JAXA_DSM_N010E106/\*.tif
-   **Output**:
    -   inputs/raster/ALPSMLC30_N010_composite_bands.tif
    -   inputs/raster/JAXA_LULC_N10E106_2020_v23.09_10m.tif
    -   inputs/raster/LANDSAT_composite_raster.tif (maximum LST, average other bands)
    -   inputs/raster/N11E106_2024_composite_bands.tif

#### Extract GSV Features (05a_extract_gsv_features.ipynb)

-   **Input**:
    -   outputs/features/gsv_gvi_svi_bvi.csv
    -   inputs/raster/\*.tif
-   **Output**: outputs/features/gsv_with_raster_features.csv

#### Extract Network Features (05b_extract_network_features.ipynb)

-   **Input**:
    -   processing/network/network_nodes.csv
    -   inputs/raster/\*.tif
-   **Output**: outputs/features/network_nodes_with_raster_features.csv

#### Train XGBoost (06_train_XGBoost.ipynb)

-   **Input**: outputs/features/gsv_with_raster_features.csv
-   **Validation**: Leave-one-ward-out spatial CV + An Phú holdout
-   **Output**:
    -   models/xgboost_full_model.pkl (18 features)
    -   models/xgboost_deployment_model.pkl (11 features)
    -   models/feature_importance.csv
    -   models/cv_results.csv
    -   models/diagnostics/\*.png

**XGBoost Parameters:**

``` python
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

**Feature Sets:**

| Category | Features |
|------------------------------------|------------------------------------|
| Landsat | `ndvi`, `emissivity` |
| PALSAR | `palsar_hh_db`, `palsar_hv_db`, `palsar_hv_hh_ratio`, `palsar_glcm_contrast`, `palsar_glcm_homogeneity`, `palsar_glcm_energy` |
| DSM | `elevation_m`, `sky_view_factor` |
| Landcover | `landcover_class` |
| GSV (Full Model Only) | `pct_vegetation`, `pct_sky`, `pct_building`, `pct_pavement_road`, `pct_water`, `pct_vehicle_clutter`, `pct_other` |

#### Node Prediction (07_node_prediction.ipynb)

-   **Input**:
    -   outputs/features/network_nodes_with_raster_features.csv
    -   models/xgboost_deployment_model.pkl
-   **Output**: outputs/predictions/network_nodes_with_predictions.csv

**Prediction Statistics:**

| Statistic | Value   |
|-----------|---------|
| Minimum   | 35.44°C |
| Mean      | 40.84°C |
| Maximum   | 46.28°C |
| Std Dev   | 1.76°C  |

#### Dijkstra Routing (08_dijkstra_hybrid.ipynb)

-   **Input**:
    -   outputs/features/gsv_with_raster_features.csv
    -   outputs/features/network_nodes_with_raster_features.csv
    -   models/xgboost_full_model.pkl
    -   models/xgboost_deployment_model.pkl
    -   processing/network/network.graphml
-   **Output**:
    -   outputs/routing/pred_raster_only.tif
    -   outputs/routing/pred_gsv.tif
    -   outputs/routing/hybrid_cost_surface.tif
    -   outputs/routing/hottest_route_hybrid.geojson

**Patchwork Approach:** Uses full model predictions inside GSV-sampled wards, deployment model predictions elsewhere. Creates hybrid cost surface with Gaussian smoothing (sigma = 4).

**Edge Cost Functions:**

``` python
# Cool Cost: Penalize hot edges.
data["cool_cost"] = length_norm + lambda_cool * temp_norm

# Hot Cost: Reward hot edges (invert temperature).
data["hot_cost"] = length_norm + lambda_hot * (1.0 - temp_norm)
```

### ArcGIS v3.4 Data Flow Summary

#### Geoprocessing Tools

1.  [Create Mosaic Dataset (Data Management)](https://pro.arcgis.com/en/pro-app/3.4/tool-reference/data-management/create-mosaic-dataset.htm)
2.  [Make Mosaic Layer (Data Management)](https://pro.arcgis.com/en/pro-app/3.4/tool-reference/data-management/make-mosaic-layer.htm)
3.  [Cell Statistics (Spatial Analyst)](https://pro.arcgis.com/en/pro-app/3.3/tool-reference/spatial-analyst/cell-statistics.htm)—Maximum for LST, Mean for other bands
4.  [Raster Calculator (Spatial Analyst)](https://pro.arcgis.com/en/pro-app/3.3/tool-reference/spatial-analyst/raster-calculator.htm)—NDVI calculation, Kelvin to Celsius conversion
5.  [Copy Raster (Data Management)](https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/copy-raster.htm)—LC09_L2SP_125052_20250228_20250301_02_T1_QA_PIXEL.TIF
6.  [Composite Bands (Data Management)](https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/composite-bands.htm)
    1.  ST_B10
    2.  ST_EMIS
    3.  SR_B4
    4.  SR_B5
    5.  QA_PIXEL

#### Satellite Raster Bands

-   **ALOS World 3D DSM (30m)**: Elevation, Mask, Stacking Number

-   **JAXA LULC (10m)**: Land Cover Classification

-   **JAXA PALSAR-2 (50m)**: HH Polarization, HV Polarization, Local Incidence Angle

-   **Landsat 8/9 (30m)**: LST (ST_B10), Emissivity (ST_EMIS), Red (SR_B4), NIR (SR_B5), QA_PIXEL

### Data Downloads

| Dataset | Source | Resolution | Time Period |
|-----------------|-----------------|-----------------------|-----------------|
| Landsat 8/9 | USGS Earth Explorer | 30m | Dec–Apr 2023–2025 |
| JAXA LULC | JAXA Earth Observation | 10m | 2020 |
| JAXA PALSAR-2 | JAXA Earth Observation | 50m | 2024 |
| ALOS World 3D DSM | JAXA Earth Observation | 30m | 2025 |
| Google Street View | Google Maps API | 640×640 | Various |
| Districts | gravitywater (ArcGIS) | 2nd Level Administrative Boundaries | 1976–2025 |
| Wards | gravitywater (ArcGIS) | 3rd Level Administrative Boundaries | 1976–2025 |

## Known Limitations

1.  **Spatial Transferability:** Model performance varies by ward (CV Standard Deviation = ± 0.27 R²). Some wards with unique urban morphology are harder to predict.

2.  **Temporal Mismatch:** GSV images captured at various times over several years; Landsat composites represent dry-season 2023–2025 maximum temperatures.

3.  **Sky View Factor:** Derived from 30m terrain DSM, captures topographic effects but not full urban canyon geometry.

4.  **Coverage Gaps:** Only 6 wards have GSV imagery; deployment model (raster-only) used elsewhere with \~2-3% lower accuracy.

## Potential Improvements

