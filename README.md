# Diversity-Aware Agricultural Decision-Support System

## Arab Youth Space Hackathon: Challenge 813

A transparent proof of concept that combines Earth Observation, environmental information, synthetic farming scenarios, livestock-feed constraints, crop rotation, water compatibility, and regional crop diversity to generate crop rankings and land-allocation recommendations for the **Ben Ahmed study area in Morocco**.

> **Current implementation:** An open-data multispectral baseline based on Sentinel-2 Level-2A. Challenge 813 hyperspectral imagery is reserved for a future extension because suitable coverage of the study area was not available during development.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Research Questions](#research-questions)
- [Objectives](#objectives)
- [Study Area](#study-area)
- [Current Status](#current-status)
- [Data Sources](#data-sources)
- [Methodology](#methodology)
- [Candidate Crops](#candidate-crops)
- [Synthetic Farmer Scenarios](#synthetic-farmer-scenarios)
- [Recommendation Model](#recommendation-model)
- [Installation](#installation)
- [Data Preparation](#data-preparation)
- [How to Run the Notebook](#how-to-run-the-notebook)
- [Results](#results)
- [Validation and Reproducibility](#validation-and-reproducibility)
- [Assumptions and Uncertainty](#assumptions-and-uncertainty)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [Responsible Use](#responsible-use)
- [Team](#team)
- [Acknowledgements](#acknowledgements)

---

## Project Overview

Agricultural planning in water-constrained mixed crop-livestock systems cannot be reduced to a single suitability map. A practical recommendation must also account for irrigation access, crop-water demand, previous crops, crop rotation, livestock-feed requirements, farmer objectives, soil-health contribution, and crop concentration across neighboring farms.

This project develops a farmer-centered analytical workflow that:

1. establishes a two-date Sentinel-2 baseline;
2. applies quality screening using the Sentinel-2 Scene Classification Layer;
3. derives NDVI and NDMI as relative spectral indicators;
4. integrates NASA POWER climate information, SoilGrids properties, and FAO-IIASA GAEZ v5 suitability evidence;
5. creates a database of eight candidate crops;
6. evaluates twelve synthetic farmer scenarios;
7. estimates approximate livestock-feed demand and feed gaps;
8. ranks eligible crops using a transparent weighted model;
9. allocates each synthetic farm among a maximum of five crops; and
10. compares independent recommendations with diversity-aware regional allocation.

All execution results, tables, figures, maps, validation messages, and model outputs are contained directly in the project notebook. The README explains the project and how to reproduce it without duplicating every notebook result.

---

## Problem Statement

Farmers in the Ben Ahmed area may face several connected constraints:

- irregular rainfall and recurring water stress;
- unequal or unreliable irrigation access;
- differences in soil and field conditions;
- repeated cultivation of the same crop family;
- insufficient crop rotation;
- livestock-feed shortages;
- competition between food and forage production;
- underuse of crop residues and dual-purpose crops; and
- regional dependence on a limited number of crops.

Satellite observations provide useful spatial context, but they cannot reveal livestock numbers, stored feed, irrigation reliability, farmer objectives, previous crops, or available resources. The project therefore combines remote sensing with environmental reference data and explicit farm-level scenario inputs.

---

## Research Questions

### Current baseline

How can an open-data multispectral Earth Observation baseline, regional environmental information, and synthetic farm-level scenarios be combined to produce transparent and diversity-aware crop-allocation recommendations for Ben Ahmed while accounting for water constraints, crop rotation, livestock-feed needs, and regional crop concentration?

### Planned hyperspectral extension

When suitable Challenge 813 hyperspectral coverage becomes available, does narrow-band information provide useful and defensible information beyond the Sentinel-2 multispectral baseline?

---

## Objectives

- Define and verify a reproducible analytical boundary around Ben Ahmed.
- Process two Sentinel-2 Level-2A observations.
- Apply cloud, shadow, cirrus, defective-pixel, no-data, snow, and unclassified-pixel screening.
- Produce true-color, NDVI, NDMI, and two-date difference products.
- Integrate regional climate, soil, and crop-suitability evidence while preserving their original spatial and temporal scales.
- Distinguish observed, modelled, derived, assumed, and unavailable information.
- Estimate approximate livestock-feed demand and feed gaps.
- Generate interpretable crop scores and crop-exclusion explanations.
- Allocate the complete area of each synthetic farm while respecting allocation limits.
- Compare independent and diversity-aware regional outcomes.
- Establish a baseline for future comparison with hyperspectral features.

---

## Study Area

**Location:** Ben Ahmed, Settat Province, Casablanca-Settat region, Morocco  
**Approximate center:** `33.066° N, 7.242° W`

Analytical bounding box:

```text
Minimum longitude: -7.32
Minimum latitude:   33.00
Maximum longitude: -7.16
Maximum latitude:   33.13
```

The rectangular extent is an analytical proof-of-concept boundary. It is not an official administrative boundary, verified agricultural district, farm parcel, or indication that every included pixel represents agricultural land.

---

## Current Status

### Implemented

- [x] Study-area definition and spatial validation
- [x] Two Sentinel-2 Level-2A acquisitions
- [x] Band discovery, clipping, reflectance conversion, and alignment
- [x] True-color visualization
- [x] SCL-based quality screening
- [x] NDVI and NDMI calculation
- [x] Common-valid-pixel comparison between two dates
- [x] NASA POWER climate processing for 2001-2025
- [x] SoilGrids regional summaries for the upper 0-30 cm profile
- [x] FAO-IIASA GAEZ v5 crop-suitability screening
- [x] Data alignment and normalization checks
- [x] Missing-data and uncertainty handling
- [x] Eight-crop candidate database
- [x] Twelve synthetic farmer scenarios
- [x] Livestock-feed gap screening
- [x] Independent recommendation and allocation
- [x] Diversity-aware recommendation and allocation
- [x] Regional diversity evaluation

### Planned

- [ ] Challenge 813 hyperspectral-data ingestion
- [ ] Sensor-specific bad-band and quality screening
- [ ] Narrow-band and red-edge feature extraction
- [ ] Moisture-sensitive and exposed-soil feature extraction
- [ ] PCA and spectral clustering where justified
- [ ] Quantitative comparison with the Sentinel-2 baseline
- [ ] Validation using real parcels, farmers, soil samples, and field observations
- [ ] Sensitivity analysis for assumptions, weights, thresholds, and processing order

---

## Data Sources

### Sentinel-2 Level-2A

Two observations from tile `T29SPS` are used:

- 18 December 2025
- 16 March 2026

Required bands and layers:

- `B02`: blue, 10 m
- `B03`: green, 10 m
- `B04`: red, 10 m
- `B08`: near infrared, 10 m
- `B11`: shortwave infrared, 20 m
- `SCL`: Scene Classification Layer, 20 m

B11 is aligned to the 10 m reference grid for NDMI calculation. The categorical SCL layer is aligned using nearest-neighbor resampling.

The quality mask excludes pixels classified as:

- no data;
- defective or saturated;
- cloud shadow;
- unclassified;
- medium-probability cloud;
- high-probability cloud;
- thin cirrus; and
- snow or ice.

Derived indicators:

```text
NDVI = (NIR - Red) / (NIR + Red)
NDMI = (NIR - SWIR) / (NIR + SWIR)
```

NDVI is treated as a relative vegetation indicator. NDMI is treated as a relative moisture-sensitive spectral indicator. Neither is interpreted as a direct measurement of crop health, crop yield, irrigation, soil moisture, or the cause of stress.

### NASA POWER

Period: **2001-2025**

Variables include:

- precipitation;
- mean, minimum, and maximum air temperature at 2 m;
- relative humidity;
- root-zone soil wetness; and
- surface solar irradiance.

NASA POWER is used as regional climate context. Its values are not treated as measurements for individual Sentinel-2 pixels or farm parcels.

### SoilGrids

Properties:

- soil pH;
- clay content;
- sand content;
- silt content; and
- soil organic carbon.

Depths:

- 0-5 cm;
- 5-15 cm; and
- 15-30 cm.

Median predictions are summarized regionally and combined using thickness weighting for the 0-30 cm profile. Resampling SoilGrids data to a smaller pixel size does not create additional soil information.

### FAO-IIASA GAEZ v5

The selected GAEZ product represents historical, rain-fed, low-input suitability classes for 2001-2020. Ordinal suitability classes are converted into a transparent 0-1 screening scale.

Matching evidence is available for:

- wheat;
- barley;
- chickpea; and
- maize.

Matching layers were not used for:

- lentil;
- faba bean;
- forage oats; and
- alfalfa.

Missing GAEZ evidence is not interpreted as zero suitability. Unsupported crops retain clearly flagged provisional assumptions so that the complete prototype pipeline can be tested.

### Synthetic scenario data

Synthetic values are used for information that cannot be derived reliably from satellite imagery:

- farm area and point location;
- sheep and cattle numbers;
- pasture area;
- stored, purchased, and residue feed;
- irrigation availability and reliability;
- previous crop and crop family; and
- farmer objectives.

These records do not represent real farmers, real holdings, verified parcels, or observed decisions.

---

## Methodology

### 1. Earth Observation processing

1. Define the study center and analytical boundary.
2. Locate and extract the Sentinel-2 SAFE products.
3. Discover the required JP2 bands.
4. Validate the scene footprint and study-area overlap.
5. Clip the bands to the analytical boundary.
6. Convert digital values to reflectance using a factor of `1 / 10000`.
7. Align B11 and SCL to the 10 m reference grid.
8. Create an SCL-based invalid-pixel mask.
9. Calculate cloud-masked NDVI and NDMI.
10. Align both acquisition dates to a common analytical grid.
11. Compare only pixels that are valid on both dates.

### 2. Environmental data integration

- NASA POWER variables are summarized as regional climate indicators.
- SoilGrids properties are summarized regionally and depth-weighted over 0-30 cm.
- GAEZ classes are converted into crop-level screening scores.
- Crop names, source labels, confidence categories, and numeric types are standardized.
- Physical measurements remain in their original units.
- Only model-scoring indicators are constrained to the 0-1 range.

### 3. Missing-data and uncertainty handling

- Missing GAEZ evidence remains unavailable rather than being replaced with zero.
- Valid GAEZ zero scores are preserved as scenario-specific evidence.
- Assumption-based and external-source values are explicitly separated.
- GAEZ-supported crops receive medium evidence confidence.
- Assumption-supported crops receive low evidence confidence.
- Confidence factors are used for secondary sensitivity analysis and are not presented as statistical probabilities.

### 4. Livestock-feed screening

Approximate annual dry-matter-equivalent feed demand is calculated from livestock counts. Estimated contributions from pasture, stored feed, purchased feed, and crop residues are deducted.

```text
Feed gap = max(estimated demand - estimated usable feed, 0)
```

Prototype coefficients:

- sheep feed requirement: `0.55 t/year`;
- cattle feed requirement: `3.50 t/year`;
- pasture contribution: `1.20 t/ha`;
- usable stored feed: `90%`;
- usable purchased feed: `95%`;
- usable crop residues: `60%`; and
- forage output equivalent: `4.00 t/ha`.

These coefficients are workflow assumptions, not animal rations or veterinary recommendations.

### 5. Crop eligibility

A crop is excluded when:

1. it requires irrigation and the farmer has no irrigation; or
2. its aligned environmental-suitability score is below `0.40`.

The reason for every exclusion is retained for auditability.

### 6. Crop scoring

Each eligible farmer-crop combination receives a weighted score:

| Component | Weight |
|---|---:|
| Environmental suitability | 0.30 |
| Water compatibility | 0.20 |
| Farmer-objective compatibility | 0.15 |
| Feed contribution | 0.10 |
| Rotation benefit | 0.10 |
| Diversity benefit | 0.10 |
| Soil-health benefit | 0.05 |
| **Total** | **1.00** |

```text
Final score = sum(component score x component weight)
```

### 7. Land allocation

- A maximum of five eligible crops is selected for each farm.
- Initial allocation is proportional to crop scores.
- Feed-oriented crops receive a moderate adjustment when a farmer has a larger feed gap.
- Minimum and maximum crop fractions are enforced.
- The allocation must cover exactly 100% of each farm's area.

### 8. Diversity-aware allocation

Two approaches are compared.

#### Independent approach

Each farmer is scored independently using a neutral diversity value of `0.50`. Existing allocations on other farms do not influence the recommendation.

#### Diversity-aware approach

Farmers are processed sequentially. Current regional crop and crop-family shares influence the diversity component for the next farmer. Underrepresented eligible crops receive a greater diversity benefit.

```text
Diversity benefit =
    0.60 x (1 - current crop share)
  + 0.40 x (1 - current crop-family share)
```

This method is a transparent heuristic rather than a globally optimized regional solution. Results can depend on farmer processing order.

---

## Candidate Crops

The model evaluates eight crops:

| Crop | Family | Main prototype role | Irrigation required |
|---|---|---|---|
| Wheat | Cereal | Food and crop residues | No |
| Barley | Cereal | Dual-purpose food and feed | No |
| Chickpea | Legume | Food and soil improvement | No |
| Lentil | Legume | Food and soil improvement | No |
| Faba bean | Legume | Food, feed, and soil improvement | No |
| Forage oats | Cereal | Forage | No |
| Alfalfa | Forage legume | Forage and soil improvement | Yes |
| Maize | Cereal | Food, feed, or silage | Yes |

Crop attributes include environmental suitability, water demand, direct feed value, residue value, rotation benefit, soil-health benefit, income potential, household-food contribution, and allocation limits. Many of these attributes are prototype assumptions and require local calibration.

---

## Synthetic Farmer Scenarios

Twelve farmer profiles test contrasting combinations of:

- small, medium, and large farm areas;
- rain-fed, limited, moderate, and reliable irrigation conditions;
- no livestock, sheep only, cattle only, and mixed livestock;
- different previous crops and crop families;
- varying pasture and feed resources; and
- objectives such as household food, livestock feed, income, crop rotation, reduced water demand, and diversified production.

The combined synthetic farm area is **121.5 ha**. Coordinates are artificial test points located within the analytical boundary and do not identify real farms.

---

## Recommendation Model

### Water compatibility

For rain-fed farms, crops with lower relative water demand receive higher compatibility. For irrigated farms, crop-water demand is compared with an available-water score derived from irrigation reliability.

```text
None:     0.00
Limited:  0.35
Moderate: 0.65
Reliable: 1.00
```

### Rotation compatibility

The model discourages repetition of the same crop family and rewards cereal-legume or legume-cereal transitions. The family rule is combined with each crop's prototype rotation-benefit value.

### Objective compatibility

The farmer's primary objective selects or combines relevant crop attributes, including household-food contribution, feed contribution, income potential, rotation benefit, low water demand, or a balanced multi-purpose score.

### Regional diversity

The diversity-aware model uses current crop and crop-family area shares to favor eligible crops that are underrepresented in the existing regional allocation.

---

## Installation

### Google Colab

The notebook is designed primarily for Google Colab with input data stored in Google Drive.

```python
%pip install -q \
    pystac-client \
    planetary-computer \
    stackstac \
    rasterio \
    rioxarray \
    xarray \
    geopandas \
    shapely \
    matplotlib \
    pandas \
    numpy \
    scikit-learn \
    scipy \
    h5py \
    ipywidgets \
    leafmap \
    OWSLib \
    pyproj
```

Networking dependencies may also need to be updated in Colab:

```python
%pip install -q --upgrade certifi requests==2.32.4 urllib3
```

### Local environment

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install \
  numpy pandas matplotlib xarray rioxarray \
  rasterio geopandas shapely pyproj OWSLib \
  h5py scipy scikit-learn pystac-client \
  planetary-computer stackstac leafmap jupyter
```

Python 3.10 or newer is recommended. Geospatial packages may require additional system libraries depending on the operating system.

---

## Data Preparation

### Sentinel-2

Place the two Sentinel-2 Level-2A ZIP products in the folder referenced by the notebook.

Expected observations:

```text
2025-12-18, tile T29SPS, Level-2A
2026-03-16, tile T29SPS, Level-2A
```

### NASA POWER

Place the exported monthly climate CSV in the climate-data path configured in the notebook. The loader:

1. searches for the CSV file;
2. identifies the end of the metadata header;
3. cleans parameter names;
4. replaces standard missing-data codes; and
5. converts the monthly wide table into a date-indexed structure.

### GAEZ

Place the supported crop-suitability GeoTIFF files in the GAEZ path configured in the notebook. The code validates product type, crop code, raster readability, class values, and regional extraction.

### SoilGrids

SoilGrids layers are requested through the Web Coverage Service for the regional bounding box. If the service is unavailable, equivalent median prediction layers can be downloaded separately and loaded from local storage.

### Paths

Before running the notebook, review all paths beginning with:

```text
/content/drive/MyDrive/
```

Update them to match the actual Google Drive or local directory locations.

---

## How to Run the Notebook

1. Open the project notebook in Google Colab or Jupyter.
2. Use a Python runtime with enough memory for raster processing.
3. Mount Google Drive when using Colab.
4. Install the required packages.
5. update the input paths to match the location of the datasets.
6. Run the notebook from top to bottom.
7. Review the validation output after every major processing phase.
8. Do not run the recommendation engine before creating the aligned environmental table, candidate-crop table, synthetic farmer table, and feed-gap results.
9. Review the final tables, maps, charts, confidence flags, and exclusion explanations directly in the notebook.

Google Drive setup:

```python
from google.colab import drive
drive.mount('/content/drive')
```

The workflow uses the following reproducibility seed:

```python
RANDOM_STATE = 42
```

---

## Results

The executed notebook contains the complete numerical results, intermediate tables, validation messages, visualizations, maps, final rankings, and land-allocation outputs.

The principal evaluated outputs include:

- true-color Sentinel-2 views for both dates;
- SCL quality-screening maps;
- cloud-masked NDVI and NDMI maps;
- NDVI and NDMI difference maps;
- NASA POWER monthly and annual climate summaries;
- SoilGrids regional and depth-weighted summaries;
- GAEZ crop-suitability summaries;
- aligned environmental indicators;
- missing-data and uncertainty tables;
- candidate-crop records;
- synthetic farmer profiles;
- feed-demand and feed-gap results;
- independent crop scores and allocations;
- diversity-aware crop scores and allocations;
- regional crop and crop-family shares;
- Shannon diversity;
- effective number of crops;
- Herfindahl-Hirschman concentration; and
- farmer-level recommendation explanations.

Because these results already appear in the executed notebook, they are not repeated in full in this README. This keeps the README focused on the purpose, methodology, reproducibility, and correct interpretation of the project.

---

## Validation and Reproducibility

The workflow includes checks for:

- valid coordinate ranges and bounding-box order;
- study-center containment;
- Sentinel-2 footprint overlap;
- required bands and unique SAFE products;
- consistent raster dimensions and coordinate-reference systems;
- common valid pixels across observation dates;
- required environmental variables;
- valid GAEZ classes and crop mappings;
- model indicators within the 0-1 range;
- unique crop and farmer identifiers;
- non-negative livestock and feed values;
- approved irrigation categories;
- synthetic farmer points inside the analytical boundary;
- complete farmer-crop scoring combinations;
- minimum numbers of eligible crops;
- crop-allocation fractions summing to 1.0;
- allocated hectares matching each farm's area; and
- compliance with crop minimum and maximum allocation limits.

The random seed is fixed at `42`, although most current calculations are deterministic. Diversity-aware outputs also depend on the specified farmer processing order.

---

## Assumptions and Uncertainty

The project exposes assumptions instead of presenting them as observations.

### External or modelled evidence

- Sentinel-2 surface reflectance and SCL classes;
- NASA POWER regional climate estimates;
- SoilGrids median predictions; and
- FAO-IIASA GAEZ scenario-specific suitability classes.

### Derived values

- NDVI and NDMI;
- GAEZ class-to-score conversion;
- depth-weighted soil summaries;
- feed-gap indicators;
- weighted crop scores;
- allocation fractions; and
- diversity metrics.

### Prototype assumptions

- crop water-demand scores;
- feed and residue contributions;
- rotation, soil-health, household-food, and income indicators;
- livestock feed requirements and usability factors;
- model weights and eligibility threshold;
- crop allocation limits;
- suitability scores for crops without matching GAEZ evidence; and
- all synthetic farmer attributes.

Confidence factors are decision-support metadata. They are not probabilities, confidence intervals, or measured agronomic accuracy.

---

## Limitations

- The current baseline is multispectral rather than hyperspectral.
- Only two Sentinel-2 dates are used, so the comparison is descriptive and not a long-term trend analysis.
- NDVI and NDMI cannot independently diagnose crop health, disease, yield, irrigation, soil moisture, or a specific stress cause.
- Climate, soil, and GAEZ datasets have coarser native scales than Sentinel-2.
- Environmental summaries are regional rather than parcel-specific.
- Four crop-suitability values remain provisional assumptions.
- Most non-suitability crop attributes are prototype values.
- Feed calculations do not model animal age, weight, physiological stage, forage quality, seasonal availability, losses, or ration balance.
- Synthetic scenarios do not represent real farmers or parcels.
- The diversity-aware approach is sequential and sensitive to farmer processing order.
- The model does not forecast prices or guarantee financial returns.
- The project is not a yield model, irrigation scheduler, veterinary ration model, or operational farm-management platform.

---

## Future Work

1. Integrate suitable Challenge 813 hyperspectral imagery.
2. Inspect sensor metadata, wavelength centers, bandwidths, coordinate systems, quality flags, and study-area overlap.
3. Remove invalid, noisy, edge, and atmospheric-absorption bands using documented sensor criteria.
4. Extract defensible narrow-band, red-edge, moisture-sensitive, and exposed-soil features.
5. Compare hyperspectral features quantitatively with the Sentinel-2 baseline.
6. Add a longer multi-date Earth Observation series.
7. Integrate verified parcel boundaries and farmer-provided information.
8. Collect local soil samples and field observations.
9. Calibrate crop and feed coefficients using Moroccan agronomic evidence and expert review.
10. Conduct sensitivity analysis for model weights, thresholds, assumptions, and farmer processing order.
11. Compare the sequential diversity heuristic with mathematical optimization.
12. Add validated market, storage, transport, and buyer information.
13. Develop an interactive interface with maps, explanations, and confidence labels.

---

## Responsible Use

This project is a research and hackathon proof of concept. It must not replace:

- professional agronomic advice;
- veterinary or animal-nutrition advice;
- field inspection;
- representative soil sampling and laboratory analysis;
- verified water-resource and irrigation assessment;
- local market research;
- agricultural authority guidance; or
- farmer knowledge and final decision-making.

The recommendations are regional scenario-screening results, not prescriptions for real farms.

---

## Team

This project was developed by a multidisciplinary team for the **Arab Youth Space Hackathon, Challenge 813**:

- **Marwa Benhaddi**: Data Science student
- **Hibat Allah Bekkar**: Cybersecurity student
- **Amine Fatah**: Aerospace Engineering student

The team combines expertise in data analysis, secure digital systems, and aerospace and Earth Observation applications.

---

## Acknowledgements

This project uses data and services associated with:

- the European Union Copernicus Sentinel-2 mission;
- NASA POWER;
- ISRIC SoilGrids;
- FAO-IIASA GAEZ v5;
- Challenge 813 hyperspectral data for the planned extension; and
- the open-source Python geospatial and scientific-computing ecosystem.

---

## License and Data Availability

All datasets used in this project were obtained from publicly accessible sources, including Sentinel-2, NASA POWER, SoilGrids, and FAO-IIASA GAEZ v5.

Public availability does not necessarily mean that every dataset is released under the same license. Each dataset remains subject to the terms of use, attribution requirements, access conditions, and licensing rules defined by its original provider.

The project team does not claim ownership of the external datasets. The notebook, original source code, methodology, documentation, and project-specific outputs may be distributed under a separate software or content license selected by the team. Before publishing the repository, an explicit `LICENSE` file should be added, such as the MIT License if the team wants to allow broad reuse of its original code with attribution.
