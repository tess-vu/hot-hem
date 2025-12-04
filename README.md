# Hot Hẻm: Sài Gòn Giữa Cái Nóng Hổng Công Bằng

## Optimization for Suffering: The Hottest Route Available

**Hot Hẻm: Where algorithm meets asphalt—a routing tool not for the fastest nor the coolest walking path, but the hottest walking path.**

Finds the hottest walking path instead of the fastest or the coolest, using machine learning (ML) methods to account for tree shade, building shadows, and sun position in Sài Gòn, Việt Nam (Hồ Chí Minh City / HCMC). In hotter climates or heatwaves, existing literature notes shaded routes can significantly improve pedestrian comfort, but there's lacking emphasis that the onus falls on local government to provide resilient, cool, and green infrastructure—this is a byproduct communicated by shade-finding algorithms that present coolest routes. Regardless of intention, they present as alternatives rather than tools to assist with building solutions, implying health, wellbeing, and heat-stress mitigation is a choice among locals, and not a prevailing systemic and infrastructural issue that will exacerbate with global warming. This project aims to fill that gap by seeking the *hottest routes* as a government tool, where a ML optimization can recommend routes *minimizing shade* and *maximizing sun exposure*, revealing the hottest paths as potential candidates for shaded infrastructure or future tree canopies, demonstrating how ML can help enhance urban resilience to extreme heat.

This project is antithetical to climate resilience framed as a choice, and is not just an algorithmic exploration of heat and power, but also a portrait of urbanization's unequal sunlight.

In this downscaled study, I focus on the city's disparate districts 1, 2, and 8.

-   [**District 1**](https://maisonoffice.vn/en/news/map-of-district-1/) is the central business hub and also the densest considering the built environment and population.

    -   **Ward Bến Thành:** Dense commercial zone, historic market.

    -   **Ward Co Giang:**

-   [**District 2**](https://thesuperprime.com/research/vietnam-a-fast-rising-luxury-real-estate-and-lifestyle-destination-of-the-super-rich/) is widely considered the most [affluent and verdant area](https://www.april-international.com/en/long-term-international-health-insurance/guide/best-places-to-live-in-ho-chi-minh-city-for-expats) with many foreign expats and wealthy locals.

    -   **Ward An Khanh:**

    -   **Ward Thảo Điền:** Green and suburban.

-   [**District 8**](https://saigoneer.com/saigon-environment/26586-as-infrastructure-lags-behind,-saigon-s-poorest-hardest-hit-by-worsening-flooding) is considered the most infrastructurally lacking and socioeconomically struggling.

    -   **Ward 5:** Canals, high built density, low vegetation.

    -   **Ward 6:** Old mixed-use area with narrow roads.

[**Hẻm**](https://en.wikipedia.org/wiki/Hem_(alleyway))**:** "Narrow streets branching off of main roads in Vietnam. Characterized by narrow width and aligned with narrow, multistory buildings known as tube houses, creating a dense and vertical urban form." Southern Vietnamese dialect for *"alleyway"* in English terms.

**NOTE:** Administrative districts were removed for the country and were refined to smaller administrative wards as of summer 2025.

## Directory Structure and Data Flow

The structure is organized by data stage (inputs, processing, outputs) with clear district/ward hierarchies for image data.

### Directory Structure

Made using [ASCII Text Tree Generator](https://www.text-tree-generator.com/).

```         
hot_hem/
├── data/
│   ├── inputs/
│   │   ├── boundaries/
│   │   │   └── aoi_wards.geojson                    # Ward boundary polygons.
│   │   └── raster/
│   │       ├── LANDSAT_composite_raster.tif         # LST, NDVI, emissivity.
│   │       ├── JAXA_PALSAR-2_2024_composite_bands.tif
│   │       ├── JAXA_DSM_ALPSMLC30_N010_composite_bands.tif
│   │       └── JAXA_LULC_N10E106_2020_v23.09_10m.tif
│   │
│   ├── processing/
│   │   ├── network/
│   │   │   ├── hcmc_pedestrian_network.graphml      # OSMnx network graph.
│   │   │   ├── network_nodes.csv                    # Node coordinates.
│   │   │   └── network_edges.csv                    # Edge attributes.
│   │   │
│   │   ├── gsv/
│   │   │   ├── metadata.csv                         # GSV point metadata.
│   │   │   ├── checkpoint.json                      # Download progress.
│   │   │   ├── gsv_sample_points.geojson            # GSV points with geometry.
│   │   │   └── gsv_thumbnails.html                  # Visual reference.
│   │   │
│   │   └── images/
│   │       ├── district_1/
│   │       │   ├── ben_thanh/
│   │       │   │   ├── original/                    # gsv_#####.jpg.
│   │       │   │   ├── segmented/                   # class_#####.png.
│   │       │   │   └── superclass/                  # superclass_#####.png.
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
│   │
│   └── outputs/
│       ├── features/
│       │   ├── gsv_gvi_svi_bvi.csv                  # Visual indices per GSV point.
│       │   ├── gsv_with_raster_features.csv         # Training dataset.
│       │   ├── superclass_metrics.csv               # Pixel counts per superclass.
│       │   └── network_nodes_with_raster_features.csv
│       │
│       └── predictions/
│           └── network_nodes_with_predictions.csv   # Final predicted LST.
│
├── models/
│   ├── xgboost_full_model.pkl                     # Full model with GSV features.
│   ├── xgboost_deployment_model.pkl               # Deployment model (raster only).
│   └── feature_importance.csv                     # Feature rankings.
│
└── notebooks/
    ├── 01_download_gsv.ipynb
    ├── 02_segmentation_pspnet.ipynb
    ├── 03_merge_segmentation_classes.ipynb
    ├── 04_compute_bvi_gvi_svi.ipynb
    ├── 05a_extract_gsv_features.ipynb
    ├── 05b_extract_network_features.ipynb
    ├── 06_train_XGBoost.ipynb
    └── 07_node_prediction.ipynb
```

### File Naming Conventions

| Stage                             | Prefix       | Example              |
|-----------------------------------|--------------|----------------------|
| Original GSV images               | gsv\_        | gsv_12345.jpg        |
| Segmented masks (ADE20K classes)  | class\_      | class_12345.png      |
| Superclass masks (merged classes) | superclass\_ | superclass_12345.png |

### Data Flow Summary

#### Download GSV (01_download_gsv.ipynb)

-   **Input**: boundaries/aoi_wards.geojson
-   **Output**:
    -   images/district_X/ward_Y/original/gsv\_#####.jpg
    -   processing/gsv/metadata.csv
    -   processing/gsv/gsv_sample_points.geojson
    -   processing/network/*.graphml,* .csv

#### Segmentation (02_segmentation_pspnet.ipynb)

-   **Input**: images/district_X/ward_Y/original/gsv\_#####.jpg
-   **Output**: images/district_X/ward_Y/segmented/class\_#####.png

#### Merge Classes (03_merge_segmentation_classes.ipynb)

-   **Input**: images/district_X/ward_Y/segmented/class\_#####.png
-   **Output**:
    -   images/district_X/ward_Y/superclass/superclass\_#####.png
    -   outputs/features/superclass_metrics.csv

#### Compute Visual Indices (04_compute_bvi_gvi_svi.ipynb)

-   **Input**:
    -   outputs/features/superclass_metrics.csv
    -   processing/gsv/metadata.csv
-   **Output**: outputs/features/gsv_gvi_svi_bvi.csv

#### ArcGIS Composite Rasters (Intermediate Preparation)

-   **Input**:
    -   inputs/raster/2020_JAXA_LANDCOVER_YEAR/\*.tif
    -   inputs/raster/2024_2025_LANDSAT_DEC_APR/\*.tif
    -   inputs/raster/2024_JAXA_PALSAR-2_MOSAIC/N11E106_24_MOS_F02DAR/\*.tif
    -   inputs/raster/JAXA_DSM_N010E106/\*.tif
-   **Output**:
    -   inputs/raster/ALPSMLC30_N010_composite_bands.tif
    -   inputs/raster/JAXA_LULC_N10E106_2020_v23.09_10m.tif
    -   inputs/raster/LANDSAT_composite_raster.tif
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
-   **Output**: models/\*.pkl, models/feature_importance.csv

#### Node Prediction (07_node_prediction.ipynb)

-   **Input**:
    -   outputs/features/network_nodes_with_raster_features.csv
    -   models/xgboost_deployment_model.pkl
-   **Output**: outputs/predictions/network_nodes_with_predictions.csv

### ArcGIS v3.4 Data Flow Summary

#### Geoprocessing Tools

1.  [Create Mosaic Dataset (Data Management)](https://pro.arcgis.com/en/pro-app/3.4/tool-reference/data-management/create-mosaic-dataset.htm)
2.  [Make Mosaic Layer (Data Management)](https://pro.arcgis.com/en/pro-app/3.4/tool-reference/data-management/make-mosaic-layer.htm)
3.  [Cell Statistics (Spatial Analyst)](https://pro.arcgis.com/en/pro-app/3.3/tool-reference/spatial-analyst/cell-statistics.htm)
4.  [Raster Calculator (Spatial Analyst)](https://pro.arcgis.com/en/pro-app/3.3/tool-reference/spatial-analyst/raster-calculator.htm)
5.  [Copy Raster (Data Management)](https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/copy-raster.htm)
6.  [Composite Bands (Data Management)](https://pro.arcgis.com/en/pro-app/latest/tool-reference/data-management/composite-bands.htm)

#### Satellite Raster Bands

-   **JAXA DSM**:

-   **JAXA LULC**:

-   **JAXA PALSAR-2**:

-   **Landsat**:

### Data Downloads
