# Sr ISOSCAPE - Strontium Isotope Maps for Central Europe

![Status](https://img.shields.io/badge/status-production--ready-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

## Overview

Sr ISOSCAPE generates high-resolution Strontium isotope ratio (87Sr/86Sr) maps for Central Europe using Random Forest ML with 15 environmental covariates.

## Key Features
- Random Forest (500 trees, 15 covariates)
- 30,765 LUCAS EU23 training profiles
- Resolution: ~1.5 km × 1 km
- Extent: 4°E-26°E, 47°N-56°N
- Output: GeoTIFF rasters + Shapefiles + 10 maps

## Status: COMPLETE & PRODUCTION-READY

## Quick Start

```bash
git clone https://github.com/budziszewskiadam-rgb/Sr-ISOSCAPE-Analysis.git
```

```r
install.packages(c("terra", "viridis", "sf", "dplyr", "randomForest"))
source("R_scripts/99_complete_workflow.R")
```

## Documentation
- WORKFLOW.md
- HOW_TO.md
- METHODS.md
- DATA_SOURCES.md
- CONTRIBUTING.md

## License
MIT License

## Author
Adam Budziszewski