# Project-Bangladesh
District-Level Disease Mapping with Household Survey Data: A Case Study in Bangladesh
 
---
## Project Overview:

This project investigates the prevalence of childhood diarrhea in Bangladesh using the 2019 Multiple Indicator Cluster Survey (MICS), combined with external demographic and climate covariates. The goal is to estimate district-level diarrhea risk, evaluate predictive models, and produce interactive risk maps to support public health planning. Childhood diarrhea remains a leading cause of morbidity and mortality in low- and middle-income countries. Using MICS 2019 microdata, this project computes district-level prevalence of diarrhea among children under five, integrates external features (population density, rainfall, temperature), and builds predictive models. Results are visualized through interactive, map-based dashboards.



## Data Sources: 
- MICS 2019 SPSS datasets  
  - `hh.sav` → Households (water, sanitation, dwelling, location)  
  - `hl.sav` → Household members (age, sex, education)  
  - `wm.sav` → Women 15–49 (fertility, health, education)  
  - `bh.sav` → Birth history (live births, survival, immunization)  
  - `ch.sav` → Children under 5 (illness, feeding, diarrhea in last 2 weeks)  
  - `fs.sav` → Children 5–17 (schooling, child labor)  

- External Covariates  
  - Population density: [WorldPop 2019](https://www.worldpop.org/) raster (`bgd_ppp_2019.tif`)  
  - Climate data: [NASA POWER](https://power.larc.nasa.gov/) daily precipitation & temperature (2019, aggregated to annual means)  

- District boundaries: [GADM v4.1](https://gadm.org/) Level-2 shapefiles (Bangladesh districts)



## Data Collection and Cleaning
1. Converted raw SPSS (`.sav`) files → CSV using `pandas.read_spss`  
2. Created a master child-level dataset (`master_children_u5.csv`) with outcome, WASH, caregiver, location features  
   - Outcome: Diarrhea in past 2 weeks (CA1)  
   - Household WASH: water source, sanitation, urban/rural  
   - Caregiver: age, sex, education  
3. Re-coded missing values (e.g., -99, -77, "DON’T KNOW") → `NaN`  
4. Linked household, roster, and women’s data using IDs (HH1, HH2, LN, UF4)

   

## Data Exploration
- Survey EDA
  - Explored distributions of outcome & covariates at the child level  
  - Verified survey weights (`chweight`) and prevalence calculations  
  - Computed district-level weighted prevalence of diarrhea:  
    - Effective sample size, confidence intervals  
    - Output: `district_weighted_prevalence.csv`  
- Spatial EDA
  - Cleaned and harmonized district names (e.g., “Chittagong” → “Chattogram”) 
  - Joined MICS prevalence with GADM shapefiles
  - Produced choropleth maps of observed prevalence (`district_prevalence_merged.geojson`) 
  - Built interactive Folium maps for zoomable, district-level exploration 

- Feature EDA
  - Population features: Aggregated WorldPop raster into district totals, persons per pixel, and population density (people/km²) 
  - Climate features: Processed NASA POWER daily precipitation & temperature into annual means (2019)  
  - Created `district_features.csv`  
  - Merged into a unified dataset: `district_model_data.csv` / `.geojson` (64 districts, ~38 predictors)


## Modeling
- Approach
  - Target: `wt_prev_diarrhea` (survey-weighted prevalence, 0–1)  
  - Predictors: Population density, rainfall, temperature, and additional district-level covariates  
  - Cross-validation strategy: Leave-One-Division-Out (LODO) to test generalization across Bangladesh’s divisions  

- Models
  - ElasticNet Regression: linear and regularized)  
  - XGBoost Gradient Boosting: nonlinear and tree-based  

- Results
  - ElasticNet: RMSE ≈ 3.8e-05, R² ≈ 0.9999 → likely overfitting due to collinearity/leakage 
  - XGBoost: RMSE ≈ 0.03, MAE ≈ 0.008, R² ≈ 0.48 → more realistic performance

- Outputs:  
  - `model_scores_elasticnet.csv`, `model_scores_xgb.csv`  
  - `district_predictions_elasticnet.csv`, `district_predictions_xgb.csv`  
  - Final map-ready predictions: `district_predictions_xgb.geojson`

## Producing Risk Maps
- Observed prevalence maps: weighted diarrhea prevalence (MICS survey) 
- Predicted prevalence maps: district-level predictions from ElasticNet and XGBoost 
- Residual maps: highlight over- and under-predicted areas  
- Outputs saved as interactive Folium HTML maps (`map_observed_prevalence.html`, `map_risk_layers_observed_predicted_residuals.html`)  

These maps allow users to zoom, pan, and explore district-level variation interactively.  


## Key Takeaways:
- MICS survey microdata was transformed into clean, district-level estimates of diarrhea prevalence.  
- External covariates (population & climate) were integrated to model spatial risk factors.  
- Cross-validation (LODO) exposed the challenges of generalization across divisions.  
- Interactive maps provide intuitive, visual insights for researchers and policymakers.  
