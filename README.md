# SIADS 696 Milestone II — Team 17

## Healthcare Access Characteristics: Predicting Routine Checkup Rates and Identifying County Access–Vulnerability Profiles

**Team Members:** Sara Jahanian, Samantha Salazar, Gina Sheesley  
**Course:** SIADS 696 — Milestone II  
**Program:** University of Michigan, Master of Applied Data Science

---

## Project Overview

This project examines how healthcare-resource availability, geographic access, provider shortage, and social vulnerability relate to preventive healthcare utilization across U.S. counties.

The project contains two complementary analyses:

1. **Supervised learning:** predict county-level age-adjusted routine-checkup prevalence using healthcare-resource and access-related characteristics.
2. **Unsupervised learning:** identify county healthcare-access and social-vulnerability profiles without using routine-checkup prevalence to construct the clusters.

The analyses use a harmonized national county-level dataset combining CDC PLACES, HRSA Area Health Resources Files (AHRF), HRSA Health Professional Shortage Area (HPSA) data, and the CDC/ATSDR Social Vulnerability Index (SVI).

The final shared analytical dataset contains **3,144 counties/county equivalents and 50 retained variables**. Nine Connecticut planning regions have structured historical AHRF missingness; they remain in the supervised analysis but are excluded from the primary clustering matrix, yielding **3,135 observations** for unsupervised learning.

### Main finding

> **Healthcare-resource availability alone does not fully explain county-level preventive healthcare utilization.**

The supervised analysis found that geographic/access context contributed more predictive information than individual provider measures, while the unsupervised analysis showed that healthcare-resource scarcity and social vulnerability are partly independent dimensions of county context.

---

## Research Questions

### Part A — Supervised Learning

**Primary question:**  
How well can county-level healthcare-resource availability and access characteristics predict the age-adjusted prevalence of adults receiving a routine checkup within the past year?

### Part B — Unsupervised Learning

**Primary question:**  
What healthcare-access and social-vulnerability profiles emerge among U.S. counties?

**Secondary question:**  
Do the independently identified county profiles differ in their distributions of routine-checkup utilization?

The PLACES routine-checkup outcome is **excluded from cluster construction** and is examined only after cluster assignments are finalized as an external descriptive comparison.

---

## Data Sources

### CDC PLACES

**Role:** supervised-learning outcome and post-clustering descriptive comparison

- 2024 county release
- target year: 2022
- outcome: age-adjusted prevalence of adults reporting a routine checkup within the past year
- project geographic universe: 3,144 counties/county equivalents after excluding the U.S. aggregate

Source:  
https://www.cdc.gov/places/

### HRSA Area Health Resources Files (AHRF)

**Role:** healthcare-resource, infrastructure, rurality, and population-density features

Final engineered measures include:

- primary-care physicians per 100,000
- nurse practitioners per 100,000
- physician assistants per 100,000
- hospitals per 100,000
- hospital beds per 1,000
- Federally Qualified Health Centers (FQHCs) per 100,000
- Rural Health Clinics per 100,000
- log population density
- rural/urban context

The full AHRF geographic extract contained 3,240 unique records. All 3,144 project counties/county equivalents were represented; 96 additional territorial, historical, or superseded geographies were outside the project universe.

Source:  
https://data.hrsa.gov/topics/health-workforce/nchwa/ahrf

### HRSA Health Professional Shortage Area (HPSA) Data

**Role:** provider-shortage features, sensitivity analysis, and cluster profiling

Current HPSA extracts were used to approximate geographic and population designations active on **December 31, 2022**. Designation-level records were aggregated to county-level measures including:

- designation presence
- designation count
- maximum recorded HPSA score

Primary Care, Mental Health, and Dental Health designations were retained where relevant. Because this is a project-defined historical reconstruction from current HRSA extracts rather than an official archived 2022 snapshot, designation presence is emphasized over current count/score values in primary interpretation.

Source:  
https://data.hrsa.gov/data/download?data=SHORT

### CDC/ATSDR Social Vulnerability Index (SVI)

**Role:** primary social-vulnerability representation for unsupervised learning and sensitivity analysis for supervised learning

The clustering workflow uses the **16 underlying 2022 SVI percentage indicators** rather than simultaneously including derived theme and overall percentile rankings. These indicators cover:

- socioeconomic status
- household characteristics
- racial and ethnic minority status and language
- housing and transportation

SVI `-999` sentinel values were converted to missing values before analysis.

Source:  
https://www.atsdr.cdc.gov/place-health/php/svi/

---

## Data Preparation

All sources were harmonized using standardized five-character county FIPS codes. County names were retained for interpretation and quality control but were not used as merge keys.

Key preprocessing decisions:

- preserve raw source files unchanged
- exclude the national PLACES aggregate
- validate incremental merges as one-to-one
- convert relevant provider/facility counts to population-standardized rates
- log-transform population density
- simplify Rural-Urban Continuum Codes to metro/nonmetro status for the primary supervised model
- convert SVI `-999` sentinel values to missing
- reconstruct approximate year-end 2022 HPSA status using designation and withdrawal dates
- perform supervised imputation, scaling, and encoding inside model pipelines
- standardize continuous clustering features before K-Means/GMM fitting
- exclude routine-checkup prevalence from all clustering features

The resulting shared analytical dataset contains **3,144 rows and 50 retained variables**.

Nine Connecticut planning regions contain structured missingness in historical AHRF resource measures. They are retained in the shared dataset and supervised workflow but excluded from the primary clustering matrix, producing an unsupervised sample of **3,135 counties/county equivalents**.

---

## Part A — Supervised Learning

### Primary Feature Set

The primary supervised specification contains **10 predictors** across four domains:

- **Provider supply:** primary-care physicians, nurse practitioners, and physician assistants per 100,000
- **Healthcare infrastructure:** hospitals, FQHCs, and Rural Health Clinics per 100,000; hospital beds per 1,000
- **Geographic/access context:** log population density and metro/nonmetro status
- **Provider shortage:** primary-care HPSA designation presence

The full SVI feature set is excluded from the primary supervised specification because some SVI demographic and socioeconomic constructs overlap conceptually with information used in PLACES small-area estimation. SVI is therefore evaluated separately as a sensitivity specification.

### Models

The supervised workflow compares:

- **DummyRegressor** — median-prediction baseline
- **Elastic Net Regression** — regularized linear model
- **Support Vector Regression (SVR)** — nonlinear kernel-based model
- **Gradient Boosting Regression** — nonlinear tree ensemble

Hyperparameters for the substantive model families were tuned using cross-validation within the training data, with MAE as the refit criterion. The reserved holdout set was not used for tuning or model selection.

### Evaluation

Primary metric:

- **Mean Absolute Error (MAE)**

Secondary metrics:

- **Root Mean Squared Error (RMSE)**
- **R²**

Validation design:

- 5-fold random cross-validation for primary model comparison
- state-grouped cross-validation for geographic robustness
- reserved holdout test set for final evaluation of the selected model

### Primary Cross-Validation Results

| Model | Random-CV MAE | State-grouped MAE |
|---|---:|---:|
| Dummy | 2.813 | 3.056 |
| Elastic Net | 2.329 | 2.569 |
| **SVR** | **2.105** | **2.455** |
| Gradient Boosting | 2.116 | 2.461 |

**Selected model:** SVR

SVR reduced random-CV MAE by **0.708 percentage points (25.2%)** relative to the Dummy baseline.

After model selection was finalized, the selected SVR was evaluated on the reserved holdout set:

- **Holdout MAE:** 2.053
- **Holdout RMSE:** 2.696
- **Holdout R²:** 0.427
- **95% bootstrap CI for holdout MAE:** 1.921–2.186

### Main Supervised Findings

- Log population density was the strongest individual permutation-importance feature.
- Geographic/access context produced the largest feature-group ablation loss.
- Healthcare infrastructure and provider supply added smaller unique predictive gains.
- Primary-care HPSA designation added essentially no unique random-CV benefit in the primary specification.
- Replacing log population density with raw density worsened performance.
- Adding the 16 SVI indicators improved SVR random-CV MAE to approximately **1.624**, but this remains a sensitivity analysis because of shared-input concerns with PLACES.
- State-grouped validation reduced performance, indicating weaker generalization to entirely unseen states.
- Failure analysis showed regression toward the mean, with several large errors among counties with unusually low or high routine-checkup prevalence.

---

## Part B — Unsupervised Learning

### Primary Feature Representation

The primary clustering matrix contains **24 continuous standardized features**:

- 8 healthcare-resource/geographic-context measures
- 16 underlying SVI indicators

Routine-checkup prevalence, HPSA flags, and geographic coordinates are excluded from the primary clustering matrix.

### Methods

Primary methods:

- **K-Means**
- **Gaussian Mixture Models (GMM)**

Supporting analyses:

- PCA for visualization and interpretation
- silhouette score
- Davies–Bouldin index
- Calinski–Harabasz index
- inertia
- GMM AIC/BIC
- random-seed stability
- cluster-size and interpretability checks
- feature-representation sensitivity analyses
- county-level mapping
- post-clustering routine-checkup comparisons

### Primary K-Means Solution

Candidate values `k = 2–10` were evaluated using multiple internal and substantive criteria.

**Selected primary solution:** K-Means, `k = 5`

- silhouette = **0.123**
- mean seed-stability ARI = **0.992**
- minimum seed-stability ARI = **0.984**

The five profiles are:

1. **Rural, older, hospital-anchored**
2. **Urban/suburban, provider-rich**
3. **Resource-scarce, low-vulnerability**
4. **High access barriers, young**
5. **Rural high-hardship, FQHC-reliant**

Silhouette scores were low overall, so the clusters are interpreted as **overlapping profiles along a continuum** rather than sharply separated county types.

### GMM Comparison

A full-covariance GMM was retained as the main probabilistic comparison.

- `k = 5`: mean stability ARI = **0.955**
- `k = 9`: mean stability ARI = **0.747**

Although the nine-component model achieved a lower BIC, it was less stable and less interpretable. The five-component GMM was therefore retained as the primary comparison model, with `k = 9` treated as exploratory evidence of finer substructure.

K-Means and GMM `k = 5` showed moderate assignment agreement (**ARI ≈ 0.29**).

### Sensitivity Analysis

Key sensitivity results:

- equal weighting of healthcare-resource and SVI domains vs. primary K-Means: **ARI ≈ 0.364**
- adding HPSA flags to K-Means: **ARI ≈ 0.403**
- adding HPSA flags to GMM: **ARI ≈ 0.028**

Broad access/vulnerability structure remained recognizable, but finer cluster boundaries—especially among high-vulnerability counties—depended on feature representation and clustering method.

### Routine-Checkup Comparison

Routine-checkup prevalence was reintroduced only after clustering was finalized.

Cluster medians ranged approximately **71.5%–75.7%**, compared with an overall clustering-sample median of **74.4%**. The lowest routine-checkup prevalence appeared in profiles characterized by high uninsurance and related access barriers rather than in the profile defined mainly by low provider supply.

Because distributions overlapped substantially, these differences are interpreted descriptively rather than causally.

---

## Integrated Findings

The supervised and unsupervised analyses converge on a multidimensional view of healthcare access.

- In supervised learning, geographic/access context contributed more predictive information than individual provider measures.
- Broader SVI characteristics substantially improved prediction in sensitivity analysis.
- In clustering, counties with the lowest provider supply were not necessarily those with the greatest vulnerability.
- The lowest routine-checkup prevalence was more consistently associated with high-uninsurance/access-barrier profiles than with provider scarcity alone.

Together, the results suggest that workforce and infrastructure matter, but preventive healthcare utilization is also associated with social, geographic, insurance, policy, and healthcare-system context.

These analyses identify predictive and descriptive patterns and **do not establish causal effects**.

---

## Important Limitations

- PLACES outcomes are model-based small-area estimates rather than direct measurements for every county.
- Some SVI demographic and socioeconomic constructs overlap conceptually with inputs used in PLACES estimation.
- Source years are not perfectly aligned.
- HPSA 2022 status is reconstructed approximately from current HRSA extracts rather than an official archived 2022 snapshot.
- Connecticut geography creates structured historical AHRF missingness.
- Counties are spatially and administratively correlated.
- State-grouped validation is a robustness check but does not replace explicit spatial or multilevel modeling.
- Healthcare resources are not randomly distributed.
- All findings are ecological and should not be generalized to individual residents.
- Predictive importance and cluster membership do not imply causality.

---

## Repository Structure

```text
milestone2-team17/
├── README.md
├── data/
    ├── raw/
        ├── ahrf/
            └── ahrf_county_2023_2024.zip
        ├── hpsa/
            ├── hpsa_dental_health_2026-09-25.csv
            ├── hpsa_mental_health_2026-09-25.csv
            └── hpsa_primary_care_2026-09-25.csv
        ├── places/
            └── places_county_2024.csv
        └── svi/
            └── svi_county_2022.csv
    ├── interim/
        └── master_county_audit.csv
    ├── processed/
        └── master_county_analytical.csv
    └── reference/
        ├── ahrf_candidate_variables_team17.csv
        └── feature_manifest.csv
├── docs/
    ├── DATA_DICTIONARY.md
    ├── DECISIONS.md
    └── data documentation/
        ├── ahrf/
            ├── ahrf_2023_2024_user_tech.zip
            └── ahrf_definition.pdf
        ├── hpsa/
            └── hpsa_datamart_metadata.XLSX
├── figures/
    ├── supervised/
    └── unsupervisesd/
├── notebooks/
    ├── 01_data_audit.ipynb
    ├── 02_cleaning_and_merging.ipynb
    ├── 03a_supervised_eda.ipynb
    └── 03b_unsupervised_eda.ipynb
├── results/
    ├── data_audit/
        ├── geographic_coverage.csv
        ├── merge_audit_summary.csv
        ├── ahrf_missingness.csv
        ├── county_missingness.csv
        ├── svi_missingness.csv
        ├── hpsa_designation_coverage.csv
        └── source_manifest.csv
    ├── cleaning/
        ├── feature_group_summary.csv
        └── raw_to_engineered_mapping.csv
    └── supervised/
        ├── best_params.json
        ├── cv_results_all.csv
        ├── eda_missingness.cv
        ├── eda_skew_summary.csv
        ├── failure_worst_counties.csv
        ├── group_ablation.csv
        ├── permutation_importance.csv
        ├── sensitivity_specifications.csv
        ├── state_grouped_errors.cssv
        ├── test_results.cssv
    └── unsupervised/
        ├── cluster_assignments.csv
        ├── gmm_metrics_by_covariance.csv
        ├── hpsa_rates_gmm_k5.csv
        ├── hpsa_rates_kmeans_k5.csv
        ├── kmeans_metrics_k2_10.csv
        ├── model_comparison_kmeans_gmm.csv
        ├── outcome_summary_gmm_k5.csv
        ├── outcome_summary_kmeans_k5.csv
        ├── sensitivity_domain_weighting_crosstab.csv
        └── sensitivity_summary.csv
```

### Folder Descriptions

- `data/raw/` — original source files retained unchanged
- `data/interim/` — validated intermediate datasets produced during auditing and integration
- `data/processed/` — final analysis/modeling-ready datasets
- `data/reference/` — feature manifests and curated variable-reference files
- `docs/` — methodological documentation and source-specific reference materials
- `docs/data documentation/` — supporting AHRF and HPSA documentation
- `notebooks/` — numbered notebooks following the end-to-end analysis workflow
- `figures/` — supervised and unsupervised figures and maps
- `results/` — saved analytical outputs and model diagnostics
- `results/data_audit/` — geographic coverage, merge validation, source manifests, and missingness audits
- `results/cleaning/` — feature-engineering summaries and raw-to-engineered mappings
- `results/supervised/` — tuning, cross-validation, feature importance, ablation, sensitivity, and failure-analysis outputs
- `results/unsupervised/` — cluster assignments, model diagnostics, HPSA profiling, outcome summaries, and sensitivity outputs

---

## Analysis Workflow

1. Archive raw public data and source documentation.
2. Audit schemas, reference years, county FIPS codes, and missingness.
3. Harmonize PLACES, AHRF, HPSA, and SVI into a common county-level geographic universe.
4. Validate one-to-one merges and geographic coverage.
5. Engineer population-standardized healthcare-resource measures and other analytical features.
6. Create the shared analytical dataset and feature manifest.
7. Conduct supervised and unsupervised exploratory analysis.
8. Tune and compare supervised model families using cross-validation.
9. Perform feature importance, ablation, sensitivity, geographic robustness, and failure analysis.
10. Construct and standardize the unsupervised feature matrix.
11. Fit and compare K-Means and GMM solutions.
12. Evaluate cluster stability, feature-representation sensitivity, and substantive interpretability.
13. Reintroduce routine-checkup prevalence for external post-clustering comparison.
14. Export report-critical tables, metrics, and figures.
15. Complete final report and reproducibility checks.

---

## Team Contributions

### Samantha Salazar

- data acquisition and audit
- geographic harmonization
- HPSA historical reconstruction
- multi-source integration
- cleaning and feature engineering
- analytical dataset and feature-manifest development
- methodological documentation
- policy and health-equity framing

### Sara Jahanian

- supervised EDA and modeling
- Elastic Net, SVR, and Gradient Boosting development
- model tuning and evaluation
- sensitivity analysis
- feature importance and feature-group ablation
- failure analysis
- supervised visualizations

### Gina Sheesley

- unsupervised EDA and modeling
- K-Means and GMM analysis
- clustering evaluation and stability analysis
- sensitivity analysis
- cluster interpretation
- geospatial and unsupervised visualizations
- routine-checkup comparisons across profiles

### Shared Contributions

- methodology review
- interpretation
- report writing
- reproducibility checks
- final project review

---

## Reproducibility Notes

- Raw source files remain unchanged.
- Data transformations are implemented through notebooks/code rather than manual edits to modeling datasets.
- Dataset versions, reference years, and HPSA reconstruction assumptions are documented.
- FIPS-based joins are validated to prevent lost or duplicated counties.
- Supervised preprocessing occurs inside model pipelines where appropriate to reduce leakage.
- Hyperparameter tuning is performed using training data and cross-validation only.
- The reserved holdout set is used only for final evaluation of the selected SVR model.
- Random seeds are fixed for reproducible modeling and clustering.
- Routine-checkup prevalence remains excluded from cluster construction.
- Report-critical tables and figures are exported from reproducible analytical outputs.

---

## Prior Work and Code Reuse

The project builds conceptually on prior team work involving healthcare access, social vulnerability, and chronic-disease indicators. However, this Milestone II project uses a newly constructed national analytical dataset, new research questions, and newly developed supervised and unsupervised learning workflows.

**No previously graded supervised or unsupervised learning code was reused for this project.** Any overlap with earlier work is limited to domain knowledge, familiarity with public data sources, and general data-preparation concepts.

---

## Related Work

The project is informed by prior work examining social vulnerability, healthcare access, preventive-care utilization, and county typologies, including:

- Bauer et al. (2022), *County-Level Social Vulnerability and Breast, Cervical, and Colorectal Cancer Screening Rates in the US, 2018*
- Al Rifai et al. (2022), *State-Level Social Vulnerability Index and Healthcare Access: The Behavioral Risk Factor Surveillance System Survey*
- Bowser et al. (2024), *American clusters: Using machine learning to understand health and health care disparities in the United States*

See the final report for complete citations and discussion of how this project extends prior work.

---

## Project Status

**Current stage:** Core data preparation, supervised modeling, unsupervised modeling, sensitivity analyses, interpretation, and report-critical visualizations are complete. Final report assembly, appendix organization, documentation cleanup, and reproducibility review are in progress.
