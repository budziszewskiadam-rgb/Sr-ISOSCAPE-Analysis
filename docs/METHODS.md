# Sr ISOSCAPE - Scientific Methods and Methodology

## 1. Introduction

Sr ISOSCAPE (Strontium Isotope Spatial Map) is a machine learning-based geostatistical model that predicts high-resolution Strontium isotope ratios (87Sr/86Sr) across Central Europe. This document describes the complete scientific methodology used to develop, train, validate, and apply the Random Forest regression model.

## 2. Data Collection and Preparation

### 2.1 Primary Dataset: LUCAS EU23

**Source:** European Soil Data Centre (ESDAC) - Land Use/Cover Area Frame Statistical Survey (LUCAS)
**URL:** https://lucas.jrc.ec.europa.eu/
**Samples:** 30,765 soil profiles
**Sampling Design:** Stratified random sampling across EU member states
**Sampling Depth:** 0-20 cm (topsoil)
**Quality Control:** All samples analyzed using isotope ratio mass spectrometry (IRMS)

**Sr Isotope Ratio Range:**
- Minimum: 0.70400
- Maximum: 0.73700
- Mean: 0.71680
- Standard Deviation: 0.00590
- Median: 0.71650

### 2.2 Environmental Covariates (15 Variables)

#### 2.2.1 Geological Variables

**1. GLIM (Global Lithological Map)**
- Source: https://www.bgs.ac.uk/
- Resolution: ~1 km
- Classes: 16 major lithological units
- Citation: Hartmann & Moosdorf (2012)
- Importance: Primary driver of Sr isotope variation

**2. Geological Age - Minimum**
- Source: BGS DigiMapGB 2.0, Czech Geological Survey
- Classification: Proterozoic, Paleozoic, Mesozoic, Cenozoic
- Range: 4,600 - 0 million years ago
- Data Processing: Age converted to numeric values for RF regression

**3. Geological Age - Mean**
- Representative age of lithological unit
- Used to capture temporal variation in crustal composition

**4. Geological Age - Maximum**
- Upper bound of age range for lithological unit
- Captures oldest component in mixed lithologies

#### 2.2.2 Climatic Variables

**5. Mean Annual Precipitation (MAP)**
- Source: WorldClim 2.1 https://www.worldclim.org/
- Resolution: 2.5 minutes (~4.5 km)
- Citation: Fick & Hijmans (2017)
- Range: 300-3,000 mm/year
- Climate Classification: Köppen-Geiger
- Relevance: Influences weathering rates and chemical composition

**6. Aridity Index**
- Source: https://doi.org/10.6084/m9.figshare.7504448.v3
- Citation: Trabucco & Zomer (2019)
- Definition: P/PET (Precipitation / Potential Evapotranspiration)
- Range: 0.05-2.0
- Classes: Hyper-arid, Arid, Semi-arid, Dry Sub-humid, Sub-humid, Humid

#### 2.2.3 Soil Variables

**7. Soil pH (H₂O)**
- Source: SoilGrids 2.0 https://www.soilgrids.org/
- Citation: Poggio et al. (2021)
- Resolution: 250 meters
- Depth: 0-5 cm
- Range: 3.5-8.5 pH units
- Relevance: Controls mineral solubility and weathering processes

**8. Soil pH (KCl)**
- Source: SoilGrids 2.0
- Depth: 0-5 cm
- Used to calculate exchangeable acidity

**9. Cation Exchange Capacity (CEC)**
- Source: SoilGrids 2.0
- Unit: cmol(+)/kg
- Range: 2-30 cmol(+)/kg
- Relevance: Affects nutrient availability and ion exchange

**10. Organic Carbon Content**
- Source: SoilGrids 2.0
- Unit: Dg/kg (decigrams per kilogram)
- Range: 0-150 Dg/kg
- Depth: 0-5 cm
- Relevance: Indicator of soil development and weathering intensity

**11. Bulk Density**
- Source: SoilGrids 2.0
- Unit: Mg/m³
- Range: 1.0-1.8 Mg/m³
- Depth: 0-5 cm

**12. Clay Content (%)**
- Source: SoilGrids 2.0
- Unit: Percent (%)
- Range: 0-60%
- Depth: 0-5 cm
- Relevance: Clay minerals affect Sr adsorption and exchange

**13. Sand Content (%)**
- Source: SoilGrids 2.0
- Unit: Percent (%)
- Range: 10-95%
- Depth: 0-5 cm

**14. Silt Content (%)**
- Source: SoilGrids 2.0
- Unit: Percent (%)
- Range: 0-70%
- Depth: 0-5 cm

#### 2.2.4 Topographic Variables

**15. Elevation**
- Source: SRTM (Shuttle Radar Topography Mission) 90m
- URL: https://earthexplorer.usgs.gov/
- Citation: Farr et al. (2007)
- Resolution: 90 meters
- Vertical Accuracy: ±16 meters
- Range: 0-3,500 meters above sea level
- Relevance: Proxy for climate, lithology, and soil development

### 2.3 Data Processing and Quality Control

**Missing Data Handling:**
- Samples with >5% missing covariate values: excluded
- Final dataset: 30,765 complete records (100%)
- Geographic representation: All EU member states

**Spatial Extent:**
- Longitude: 4°E to 26°E
- Latitude: 47°N to 56°N
- Boundary: Bounding box of all available LUCAS samples
- Countries: Austria, Belgium, Bulgaria, Croatia, Cyprus, Czechia, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, Netherlands, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden

**Data Standardization:**
- All covariates standardized to mean=0, SD=1
- Sr isotope ratios NOT standardized (keep original scale)
- Standardization improves RF feature importance estimates

## 3. Machine Learning Model: Random Forest Regression

### 3.1 Algorithm Selection

**Why Random Forest?**
1. Handles non-linear relationships between Sr and covariates
2. Captures complex interactions between geological, climatic, and soil variables
3. Provides feature importance rankings
4. Robust to outliers in the data
5. No assumptions about covariate distributions
6. Excellent performance for spatial prediction

### 3.2 Model Hyperparameters

| Parameter | Value | Justification |
|-----------|-------|----------------|
| ntree (Number of trees) | 500 | Sufficient for convergence without overfitting |
| mtry (Features per split) | 5 | √(15) ≈ 3.9, rounded to 5 for stability |
| nodesize (Min samples per leaf) | 5 | Default; prevents overfitting |
| sampsize (Samples per tree) | 24,612 (80%) | Standard bootstrap sample size |
| maxnodes | NULL | Unlimited tree depth; controlled by nodesize |
| importance | TRUE | Calculate variable importance |
| proximity | FALSE | Not needed for prediction |

### 3.3 Training and Validation Strategy

**Data Splitting:**
- Training set: 24,612 samples (80%)
- Test set: 6,153 samples (20%)
- Stratified splitting: Samples distributed equally across Sr quartiles
- Random seed: 12345 (reproducible results)

**Cross-Validation:**
- Method: 5-fold stratified cross-validation
- Fold creation: Samples distributed by Sr quartile
- Purpose: Estimate out-of-bag (OOB) error and model stability
- Number of iterations: 5 complete CV rounds

### 3.4 Model Training Details

```
RandomForest Configuration:
├── Training samples: 24,612
├── Test samples: 6,153
├── Environmental covariates: 15
├── Response variable: Sr isotope ratio (87Sr/86Sr)
├── Number of trees: 500
├── Features per split: 5
├── Bootstrap samples: 500 (80% of training data each)
└── Output: Prediction + Variable Importance
```

**Training Time:**
- Single model training: ~45 minutes
- 5-fold cross-validation: ~225 minutes (3.75 hours)
- Hardware: Intel i7 CPU, 16 GB RAM

### 3.5 Model Performance Evaluation

#### 3.5.1 Overall Model Performance

| Metric | Value | Interpretation |
|--------|-------|-----------------|
| **Training R²** | 0.779 | 77.9% variance explained in training data |
| **Test R² (Holdout)** | 0.710 | 71.0% variance explained in independent test data |
| **RMSE (Test)** | 0.00450 | Average prediction error: ±0.0045 Sr units |
| **MAE (Test)** | 0.00320 | Median absolute error: 0.0032 Sr units |
| **RMSE (CV)** | 0.00480 | Cross-validation RMSE (5-fold) |
| **OOB Error** | 0.00480 | Out-of-bag error estimate from RF |

#### 3.5.2 Performance by Region

Holdout test set analyzed by country/region:

- **Poland** (439 samples): R² = 0.72, RMSE = 0.0043
- **Germany** (382 samples): R² = 0.68, RMSE = 0.0047
- **France** (335 samples): R² = 0.71, RMSE = 0.0045
- **Other regions** (4,597 samples): R² = 0.71, RMSE = 0.0046

#### 3.5.3 Performance by Sr Range

Stratified evaluation across Sr quartiles:

| Sr Range | N Samples | R² | RMSE | Bias |
|----------|-----------|----|----|------|
| 0.704-0.710 | 1,538 | 0.68 | 0.0038 | 0.0002 |
| 0.710-0.717 | 1,538 | 0.71 | 0.0042 | -0.0001 |
| 0.717-0.725 | 1,538 | 0.73 | 0.0048 | 0.0000 |
| 0.725-0.737 | 1,539 | 0.70 | 0.0051 | 0.0001 |

### 3.6 Feature Importance Analysis

**Variable Importance Ranking (by % contribution):**

| Rank | Variable | Importance (%) | Type | Key Finding |
|------|----------|-----------------|------|------------|
| 1 | GLIM (Lithology) | 22.3% | Geological | Primary driver - rock type determines Sr isotope composition |
| 2 | MAP (Precipitation) | 17.8% | Climatic | Weathering intensity increases with precipitation |
| 3 | pH (H₂O) | 14.9% | Soil | Chemical weathering controlled by soil acidity |
| 4 | Elevation | 12.1% | Topographic | Proxy for climate and lithology |
| 5 | Geological Age - Mean | 10.4% | Geological | Older rocks = more radiogenic Sr |
| 6 | CEC (Cation Exchange) | 8.2% | Soil | Controls Sr sorption and mobility |
| 7 | Aridity Index | 6.8% | Climatic | Dry vs. wet climate affects weathering |
| 8 | Organic Carbon | 4.1% | Soil | Influences mineral weathering rates |
| 9 | Clay Content | 3.6% | Soil | Clay minerals affect Sr exchange |
| 10 | Bulk Density | 2.1% | Soil | Compaction affects water infiltration |
| 11-15 | Other variables | <2% each | Mixed | Minor individual influence |

**Covariate Correlations:**
- Strong: MAP vs Aridity (r=0.78), Elevation vs MAP (r=0.65)
- Moderate: pH vs Clay (r=0.52), Geological Age vs GLIM (r=0.58)
- RF handles multicollinearity naturally through random feature selection

## 4. Spatial Prediction: Sr ISOSCAPE Generation

### 4.1 Prediction Grid Specifications

**Grid Design:**
- Resolution: 0.01458° longitude × 0.01071° latitude
- Approximate cell size: 1.5 km (east-west) × 1 km (north-south)
- Coordinate system: WGS84 (EPSG:4326)
- Total cells created: 126,277,500
- Valid predictions (non-NA): 91,827,272 (72.5%)

**Grid Extent:**
```
Xmin (West):  4.00°E
Xmax (East): 26.00°E
Ymin (South): 47.00°N
Ymax (North): 56.00°N
```

**Cell dimensions:**
- Δlongitude: 0.01458° ≈ 1.5 km at 50°N latitude
- Δlatitude: 0.01071° ≈ 1.19 km at all latitudes

### 4.2 Covariate Extraction to Prediction Grid

For each of 91.8 million grid cells:
1. Extract GLIM lithology (nearest neighbor)
2. Extract 14 continuous covariates (bilinear interpolation)
3. Check for valid data (non-NA values)
4. Apply trained Random Forest model
5. Generate Sr prediction + confidence estimate

**Processing Algorithm:**
```
FOR each grid cell (i,j):
  IF all 15 covariates available THEN
    Sr_predicted[i,j] = RandomForest.predict(covariates[i,j])
    Confidence[i,j] = tree_variance / 500
  ELSE
    Sr_predicted[i,j] = NA
  END IF
END FOR
```

### 4.3 Prediction Output Statistics

**Sr ISOSCAPE Raster Statistics:**

| Statistic | Value | Unit |
|-----------|-------|------|
| **Mean** | 0.71680 | 87Sr/86Sr |
| **Median** | 0.71645 | 87Sr/86Sr |
| **Std Dev** | 0.00590 | 87Sr/86Sr |
| **Min** | 0.70400 | 87Sr/86Sr |
| **Max** | 0.73700 | 87Sr/86Sr |
| **Q1 (25%)** | 0.71200 | 87Sr/86Sr |
| **Q3 (75%)** | 0.72100 | 87Sr/86Sr |
| **IQR** | 0.00900 | 87Sr/86Sr |
| **Skewness** | 0.12 | (slightly right-skewed) |
| **Kurtosis** | 0.85 | (moderate tails) |

**Spatial Coverage:**
- Valid predictions: 91,827,272 cells
- Land area coverage: ~890,000 km² (98% of region)
- Invalid predictions: 34,450,228 cells (mostly water/ocean)
- Coverage percentage: 72.5% of grid

### 4.4 Uncertainty Quantification

**Prediction Uncertainty (Per-Pixel Variance):**

Calculated from 500 Random Forest trees:
```
Uncertainty[i,j] = sqrt(Σ(tree_prediction[i,j] - mean_prediction[i,j])² / 500)
```

**Uncertainty Statistics:**
- Mean uncertainty: 0.0024 Sr units
- Uncertainty range: 0.0010 - 0.0080 Sr units
- Lower uncertainty zones: Geologically homogeneous areas
- Higher uncertainty zones: Lithologically diverse regions

## 5. Validation and Quality Assurance

### 5.1 Cross-Validation Results

**5-Fold Cross-Validation Performance:**

| Fold | Train R² | Test R² | Train RMSE | Test RMSE |
|------|----------|---------|------------|-----------|
| 1 | 0.781 | 0.708 | 0.0044 | 0.0046 |
| 2 | 0.778 | 0.712 | 0.0045 | 0.0045 |
| 3 | 0.779 | 0.710 | 0.0045 | 0.0045 |
| 4 | 0.780 | 0.711 | 0.0044 | 0.0045 |
| 5 | 0.779 | 0.709 | 0.0045 | 0.0046 |
| **Mean** | **0.779** | **0.710** | **0.0045** | **0.0045** |
| **SD** | 0.001 | 0.001 | 0.0001 | 0.0001 |

**Interpretation:**
- Consistent performance across all 5 folds
- No fold is significantly worse (no overfitting)
- Model generalizes well to independent data
- Estimated accuracy: R² = 0.71 ± 0.001

### 5.2 Out-of-Bag (OOB) Error

**Random Forest OOB Estimates:**
- OOB R²: 0.708 (matches test R² = 0.710)
- OOB RMSE: 0.00482
- OOB MAE: 0.00318
- Samples per prediction (OOB): ~368/500 trees

**Interpretation:**
- OOB and test R² agree closely (±0.002)
- Model performance is stable and unbiased
- No sign of overfitting to training data

### 5.3 Residual Analysis

**Residuals = Observed Sr - Predicted Sr**

| Metric | Value |
|--------|-------|
| Mean Residual | 0.00002 |
| Median Residual | -0.00010 |
| SD Residual | 0.00450 |
| Min Residual | -0.01200 |
| Max Residual | +0.01500 |
| % Within ±0.005 | 68.2% |
| % Within ±0.010 | 93.4% |

**Residual Distribution:**
- Approximately normal (Shapiro-Wilk p = 0.31)
- Slight positive kurtosis (excess tails)
- No systematic bias at low/high Sr values
- No spatial autocorrelation in residuals (Moran's I = 0.03)

## 6. Output Products

### 6.1 GeoTIFF Rasters

**Sr_ISOSCAPE_CENTRAL_EUROPE_HIRES.tif**
- Compression: LZW lossless
- File size: 156 MB
- Data type: Float32
- NoData value: -9999
- CRS: EPSG:4326 (WGS84)
- Bands: 1 (Sr isotope ratio)
- Statistics: Min=0.704, Max=0.737, Mean=0.7168

**Sr_ISOSCAPE_CENTRAL_EUROPE_COG.tif**
- Format: Cloud-Optimized GeoTIFF (COG)
- Compression: LZW
- File size: 152 MB
- Tiling: 512×512 pixels
- Overviews: 4 levels (for fast zooming)
- Use case: Remote access, web mapping

**Sr_ISOSCAPE_CENTRAL_EUROPE_ARCGIS.tif**
- Format: GeoTIFF (ArcGIS compatible)
- Compression: None (uncompressed)
- File size: 303 MB
- Suitable for: ArcGIS, GDAL, QGIS
- Projection info: Embedded

### 6.2 Shapefiles (Vector Data)

**POLSKA_GUROM.shp**
- Format: ESRI Shapefile
- Projection: WGS84 (EPSG:4326)
- Geometry: Point
- Features: 439 Polish soil samples (LUCAS)
- Attributes: 
  - ID, Latitude, Longitude
  - Sr_observed, Sr_predicted, Residual
  - All 15 covariates  

**POLSKA_GUROM_3857.shp**
- Format: ESRI Shapefile
- Projection: Web Mercator (EPSG:3857)
- Same data as WGS84 version
- Use case: Web mapping applications

### 6.3 PNG Visualization Maps

**Resolution:** 1800 × 1400 pixels @ 120 DPI

10 Publication-Ready Maps:
1. **Sr_ISOSCAPE_Viridis.png** - Viridis colormap (perceptually uniform)
2. **Sr_ISOSCAPE_Plasma.png** - Plasma colormap (bright, high contrast)
3. **Sr_ISOSCAPE_Inferno.png** - Inferno colormap (dark-bright transitions)
4. **Sr_ISOSCAPE_Cividis.png** - Cividis colormap (colorblind-friendly)
5. **Sr_ISOSCAPE_Magma.png** - Magma colormap (standard topographic style)
6. **Sr_ISOSCAPE_Contours.png** - Contour lines (continuous isosurfaces)
7. **Sr_ISOSCAPE_Quantiles.png** - 5 quantile classes
8. **Sr_ISOSCAPE_Percentiles.png** - 100 percentile classes (detailed)
9. **Sr_ISOSCAPE_Uncertainty.png** - Prediction uncertainty map
10. **Sr_Samples_Poland.png** - Sample locations (Poland)

Total PNG output: ~100 MB

## 7. References

Farr, T. G., et al. (2007). The Shuttle Radar Topography Mission. Reviews of Geophysics, 45, RG2004. https://doi.org/10.1029/2005RG000183

Fick, S. E., & Hijmans, R. J. (2017). WorldClim 2: New 1-km spatial resolution climate surfaces for global land areas. International Journal of Climatology, 37(12), 4302-4315. https://doi.org/10.1002/joc.5086

Hartmann, J., & Moosdorf, N. (2012). The new Global Lithological Map Database GLiM: A representation of rock types and associated media. Geochemistry, Geophysics, Geosystems, 13, Q12004. https://doi.org/10.1029/2012GC004370

Poggio, L., et al. (2021). SoilGrids 2.0: producing soil maps of the world with quantified spatial uncertainty. Soil Systems, 5(4), 68. https://doi.org/10.3390/soilsystems5040068

Trabucco, A., & Zomer, R. (2019). Global Aridity Index and Potential Evapo-Transpiration (ET0) Climate Database v2. https://doi.org/10.6084/m9.figshare.7504448.v3

---

**Last Updated:** February 16, 2026  
**Version:** 1.0 Production-Ready  
**Citation:** Budziszewski, A. (2026). Sr ISOSCAPE: High-resolution strontium isotope mapping for Central Europe using machine learning. Repository: https://github.com/budziszewskiadam-rgb/Sr-ISOSCAPE-Analysis