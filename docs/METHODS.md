# Sr ISOSCAPE - Scientific Methods

## Data
30,765 soil profiles from LUCAS EU23
15 environmental covariates

## Model
Random Forest Regression
Trees: 500, mtry: 5, CV: 5-fold

## Performance
Training R²: 0.779
Test R²: 0.710
RMSE: 0.0045
MAE: 0.0032

## Prediction
Grid: 1.5 km × 1 km
Valid pixels: 91,827,272

## Sr Statistics
Min: 0.704
Max: 0.737
Mean: 0.7168
SD: 0.0059

## Feature Importance
1. GLIM (22%)
2. MAP (18%)
3. pH (15%)
4. Elevation (12%)
5. Geological Age (10%)