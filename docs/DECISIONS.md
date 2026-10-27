# Project Decisions

This document records the major methodological, analytical, and data-management decisions for the SIADS 696 Milestone II project. It preserves the rationale behind choices made during data acquisition, auditing, preprocessing, exploratory analysis, modeling, evaluation, sensitivity analysis, and final reporting.

**Last updated:** 2026-10-26

---

## 1. Project Scope

### Decision

Use U.S. counties and county equivalents as the unit of analysis.

### Rationale

All four core data sources can be aligned at the county/county-equivalent level using standardized five-character FIPS codes.

The project combines:

- CDC PLACES
- CDC/ATSDR Social Vulnerability Index (SVI)
- HRSA Area Health Resources Files (AHRF)
- HRSA Health Professional Shortage Area (HPSA) data

The final shared analytical dataset contains **3,144 counties/county equivalents and 50 retained variables**.

---

## 2. Supervised Learning Research Question

### Decision

Use county-level healthcare-resource and access characteristics to predict routine-checkup utilization.

### Primary outcome

2022 age-adjusted prevalence of adults reporting a routine checkup within the past year from the CDC PLACES 2024 county release.

### Rationale

Routine-checkup prevalence represents preventive healthcare utilization rather than healthcare access itself. Project language therefore distinguishes between:

- healthcare-resource/access characteristics as predictors; and
- routine-checkup utilization as the outcome.

The analysis is predictive and descriptive rather than causal.

---

## 3. Age-Adjusted PLACES Outcome

### Decision

Use the age-adjusted routine-checkup prevalence as the primary supervised-learning target rather than crude prevalence.

### Rationale

Age adjustment improves comparability across counties with different population age structures and reduces the extent to which geographic differences are driven solely by county age composition.

Age-adjusted prevalence is treated as a standardized county-level population estimate rather than as a direct individual-level measurement.

---

## 4. Project Geographic Universe

### Decision

Use the **3,144 county/county-equivalent FIPS codes** represented in the county-level PLACES target as the authoritative project geographic universe.

### Audit result

After removing the national aggregate:

- PLACES contains 3,144 unique counties/county equivalents.
- SVI contains the same 3,144 project FIPS codes.
- AHRF contains all 3,144 project counties plus 96 additional territorial, historical, or superseded geographic records.
- HPSA records are aggregated onto the same 3,144-county universe.

### Rationale

PLACES contains the supervised-learning outcome and therefore defines the geographic observations eligible for the shared analytical dataset.

---

## 5. National PLACES Aggregate

### Decision

Exclude the PLACES record with `StateAbbr = "US"`.

### Rationale

The record represents the national U.S. aggregate rather than a county-level observation. Including it would mix national- and county-level units and violate the project's one-row-per-county design.

---

## 6. Geographic Join Key

### Decision

Use standardized five-character county FIPS codes as the authoritative geographic join key.

### Rationale

County names vary across sources and may include differences in county, borough, census-area, or planning-region terminology.

County names are retained for interpretation and quality control but are not used as merge keys.

---

## 7. AHRF Geographic Handling

### Decision

Retain only AHRF records whose FIPS codes occur in the 3,144-county PLACES geographic universe after documenting additional AHRF geographies.

### Audit result

The full AHRF extract contains:

- 3,240 unique geographic records
- 3,144 project counties/county equivalents
- 96 additional geographic/historical records

All 3,144 PLACES geographies are represented in AHRF.

### Rationale

The additional AHRF records include territories and historical or superseded county-equivalent geographies outside the project universe.

---

## 8. Connecticut Geography

### Decision

Treat Connecticut-related AHRF gaps as **structured feature missingness**, not failed geographic matches.

### Final analytical treatment

- Retain the nine Connecticut planning regions in the shared analytical dataset.
- Retain them in supervised learning, where missing values are handled inside the modeling pipeline.
- Exclude the nine planning regions from the primary clustering matrix because most relevant historical AHRF healthcare-resource measures are unavailable for those geographies.

### Result

- Shared/supervised sample: **3,144 observations**
- Primary unsupervised sample: **3,135 observations**

### Rationale

PLACES and SVI use current Connecticut planning-region county equivalents, and AHRF contains matching planning-region records. The problem is therefore not geographic alignment; it is structured historical feature availability following Connecticut's transition from counties to planning regions.

---

## 9. AHRF Feature Selection

### Decision

Use a deliberately interpretable subset of AHRF-derived healthcare-resource and geographic-context measures rather than all available AHRF variables.

### Final retained AHRF-derived resource/context measures

- `primary_care_physicians_per_100k`
- `nurse_practitioners_per_100k`
- `physician_assistants_per_100k`
- `hospitals_per_100k`
- `hospital_beds_per_1000`
- `fqhcs_per_100k`
- `rural_health_clinics_per_100k`
- `log_population_density`

The primary supervised specification additionally uses:

- `rucc_metro_status`

### Selection criteria

Features were retained based on:

- substantive relevance
- temporal alignment
- county coverage
- missingness
- interpretability
- redundancy
- distributional behavior
- compatibility with the supervised or unsupervised method

### Rationale

The project prioritizes interpretable healthcare-access constructs rather than maximizing the number of available predictors.

---

## 10. AHRF Count Standardization and Transformations

### Decision

Preserve raw source counts for traceability, but use population-standardized or transformed measures for modeling where appropriate.

### Final transformations

- provider/facility counts are converted to rates per 100,000 population where appropriate
- hospital beds are represented per 1,000 population
- population density is represented as `log(1 + density)` in the primary specification
- detailed Rural-Urban Continuum Codes are collapsed to metro/nonmetro status for the primary supervised model

### Sensitivity treatment

- raw population density is evaluated as an alternative specification
- detailed rurality is evaluated as an alternative specification

### Result

Raw population density materially worsened predictive performance relative to log-transformed density, supporting the log-density representation.

---

## 11. Shared-Input Circularity

### Decision

Exclude the full SVI feature set from the **primary supervised specification**.

### Rationale

PLACES small-area estimates are partly constructed using demographic and socioeconomic information. Using strongly overlapping demographic inputs as primary supervised predictors could partially reproduce the PLACES estimation process rather than primarily test the predictive contribution of healthcare-resource characteristics.

### Final treatment

The 16 underlying SVI indicators are evaluated in a clearly separated supervised sensitivity analysis.

### Result

Adding the 16 SVI indicators substantially improved predictive performance, but the SVI-expanded model remains a sensitivity analysis because of the conceptual shared-input concern.

---

## 12. SVI Role

### Decision

Use SVI primarily in the unsupervised analysis and secondarily as a supervised sensitivity specification.

### Final representation

Use the **16 underlying SVI percentage indicators** rather than simultaneously including derived theme and overall percentile rankings.

### Rationale

Including composite rankings together with all constituent indicators would double-count related information.

### Missing-value treatment

SVI sentinel values such as `-999` are converted to missing values before analysis.

---

## 13. HPSA Designation Types

### Decision

For county-level HPSA representation, retain:

- Geographic HPSA
- High Needs Geographic HPSA
- HPSA Population

Exclude facility-specific designations from the primary county shortage representation.

### Rationale

The project is intended to characterize broader county/population healthcare shortages rather than shortage status tied to an individual facility.

---

## 14. HPSA Historical Reconstruction

### Decision

Approximate HPSA designation status as of **December 31, 2022** using the current HRSA extract.

A designation is treated as active at year-end 2022 when:

- `HPSA Designation Date <= 2022-12-31`; and
- `Withdrawn Date` is missing or later than `2022-12-31`.

### Limitation

This reconstruction is **not an official archived 2022 HRSA snapshot**.

The final report describes it as an approximate historical reconstruction from current HRSA records.

---

## 15. HPSA Deduplication

### Decision

Deduplicate HPSA data using:

`HPSA ID + county FIPS`

before county aggregation.

### Rationale

A single HPSA designation may contain multiple component records associated with the same county. Deduplication prevents the same designation from being counted repeatedly.

---

## 16. HPSA County-Level Features

### Decision

Create county-level HPSA measures for Primary Care, Mental Health, and Dental Health using:

- designation presence
- count of unique qualifying designations
- maximum recorded HPSA score among qualifying designations

### Final analytical roles

- **Primary supervised model:** Primary Care HPSA designation presence (`pc_hpsa_any`)
- **Supervised sensitivity:** HPSA count and maximum score alternatives
- **Primary unsupervised matrix:** HPSA variables excluded
- **Unsupervised sensitivity/profiling:** Primary Care, Mental Health, and Dental Health designation flags

### Rationale

Designation presence is more defensible for the approximate historical reconstruction than count or current score. Current scores may not equal exact historical 2022 values.

---

## 17. Counties Without HPSA Designations

### Decision

For HPSA binary and count features:

- no qualifying designation = `0`

This is treated as structural absence rather than missingness.

### HPSA maximum score

If no qualifying designation exists, the maximum score is not automatically treated as a true score of zero.

### Rationale

Absence of a designation and a score of zero are not equivalent concepts.

---

## 18. HPSA Shortage/FTE Fields

### Decision

Do not sum HPSA shortage-FTE or estimated underserved-population fields across designation records.

### Rationale

Designation-level values may repeat across county components and could be inflated by naïve aggregation.

---

## 19. Missing-Data Handling

### Decision

Do not apply one global missing-data rule across all sources or models.

Missingness is classified as:

1. true missingness;
2. structural missingness; or
3. encoded missingness.

### Final supervised treatment

Numeric imputation, scaling, transformations, and categorical encoding are performed inside scikit-learn pipelines so preprocessing parameters are estimated from training data/folds only.

### Final unsupervised treatment

A complete clustering matrix is constructed before fitting. Structured Connecticut missingness results in exclusion of nine planning regions from the primary clustering matrix. Limited missingness among retained observations is handled using the documented unsupervised preprocessing strategy before standardization.

### Rationale

Method-specific handling reduces leakage and avoids treating structurally different forms of missingness as equivalent.

---

## 20. SVI Sentinel Values

### Decision

Convert SVI encoded sentinel values such as `-999` to missing values before analysis.

### Rationale

Sentinel values represent unavailable information and must not be interpreted as valid extreme numeric observations.

Raw source files remain unchanged.

---

## 21. Incremental Merge Strategy

### Decision

Use PLACES as the base table and merge sources incrementally:

1. PLACES + AHRF
2. add county-level HPSA
3. add SVI

All merges use:

- standardized FIPS
- left joins
- `validate="one_to_one"`

### Audit requirement

Each merge must preserve:

- 3,144 rows
- 3,144 unique FIPS
- zero duplicate county observations

### Result

All four sources were integrated without losing or multiplying project counties.

---

## 22. Analytical Dataset and Audit Handoff

### Decision

Use:

`data/interim/master_county_audit.csv`

as the validated audit-stage handoff and:

`data/processed/master_county_analytical.csv`

as the analysis-ready shared dataset.

### Final analytical dataset

- **3,144 rows**
- **50 retained variables**

### Rationale

The interim file preserves source-level audit information and documented missingness, while the processed file contains the engineered variables used in downstream analysis.

---

## 23. Final Supervised Feature Specification

### Decision

Use a **10-predictor primary supervised specification** spanning four domains.

### Provider supply

- primary-care physicians per 100,000
- nurse practitioners per 100,000
- physician assistants per 100,000

### Healthcare infrastructure

- hospitals per 100,000
- hospital beds per 1,000
- FQHCs per 100,000
- Rural Health Clinics per 100,000

### Geographic/access context

- log population density
- metro/nonmetro status

### Provider shortage

- Primary Care HPSA designation presence

### Rationale

This specification emphasizes healthcare-resource and access characteristics while preserving conceptual separation from the demographic inputs used in PLACES estimation.

---

## 24. Supervised Model Families

### Decision

Compare:

- DummyRegressor baseline
- Elastic Net Regression
- Support Vector Regression (SVR)
- Gradient Boosting Regression

### Rationale

The substantive models represent distinct learning frameworks:

- regularized linear
- nonlinear kernel-based
- nonlinear tree ensemble

The DummyRegressor provides a constant median-prediction baseline appropriate for MAE comparison.

---

## 25. Supervised Hyperparameter Tuning

### Decision

Tune the substantive supervised models using cross-validation **within the training data**, with MAE as the refit/model-selection criterion.

### Final approach

Hyperparameter searches evaluate:

- Elastic Net regularization strength, mixing ratio, and feature-rate representation
- SVR `C`, `epsilon`, `gamma`, kernel configuration, and feature-rate representation
- Gradient Boosting learning rate, number of trees, tree depth, minimum leaf size, subsampling, and loss function

### Holdout protection

The reserved holdout test set is not used for hyperparameter tuning or model selection.

---

## 26. Supervised Validation Design

### Decision

Use three distinct evaluation stages:

1. **5-fold random cross-validation** for primary model comparison
2. **state-grouped cross-validation** for geographic robustness
3. **reserved holdout test set** for final evaluation of the selected model only

### Primary metric

- MAE

### Secondary metrics

- RMSE
- R²

### Rationale

Random CV estimates performance when counties are mixed across folds. State-grouped CV provides a more difficult robustness check by holding entire states out of training.

The holdout set remains separate from model selection.

---

## 27. Final Supervised Model Selection

### Decision

Select **Support Vector Regression (SVR)** as the primary supervised model using the predefined random-CV MAE criterion.

### Primary cross-validation result

- SVR random-CV MAE: **2.105**
- Dummy random-CV MAE: **2.813**
- improvement vs. Dummy: **0.708 percentage points (25.2%)**

Gradient Boosting was nearly tied under random CV but was not selected.

### Rationale

SVR achieved the lowest mean random-CV MAE under the predefined selection rule.

---

## 28. Holdout-Test Reporting

### Decision

Use the reserved holdout test set **only to evaluate the selected SVR model** in final reporting.

Do not use holdout performance from non-selected candidate models to reconsider or compare model selection.

### Selected SVR holdout result

- MAE: **2.053**
- RMSE: **2.696**
- R²: **0.427**
- 95% bootstrap CI for MAE: **1.921–2.186**

### Rationale

Once SVR was selected using cross-validation, comparing candidate models again on the holdout set would make the holdout function as an additional model-selection dataset. Restricting final holdout reporting to the selected model preserves the intended separation between model selection and final evaluation.

---

## 29. Supervised Feature Importance and Ablation

### Decision

Interpret the selected SVR using both permutation importance and feature-group ablation.

### Main findings

- log population density is the strongest individual permutation-importance feature
- removing geographic/access context produces the largest deterioration in MAE
- healthcare infrastructure contributes additional unique predictive information
- provider supply contributes a smaller unique gain
- Primary Care HPSA designation provides essentially no unique random-CV benefit in the primary specification

### Interpretation rule

Predictive importance does **not** imply causal effect.

---

## 30. Supervised Sensitivity Analysis

### Decision

Evaluate alternative representations of key supervised features rather than treating the primary specification as uniquely correct.

### Final sensitivity specifications

- HPSA designation count
- HPSA maximum score
- detailed rurality
- raw population density
- addition of all 16 SVI indicators

### Main findings

- HPSA count/score alternatives produced small changes
- detailed rurality produced small changes
- raw population density worsened performance
- adding 16 SVI indicators produced the largest predictive improvement

### Interpretation

The SVI-expanded specification remains a sensitivity analysis because of conceptual overlap with demographic and socioeconomic inputs used in PLACES estimation.

---

## 31. Supervised Failure Analysis

### Decision

Use out-of-fold predictions to identify and examine specific counties with large prediction errors.

### Rationale

Failure analysis should identify where the selected model struggles rather than relying only on aggregate metrics.

### Main pattern

SVR tends to compress predictions toward the middle of the outcome distribution:

- unusually low routine-checkup values are often overpredicted
- unusually high values are often underpredicted

Specific county examples are used in the final report to illustrate distinct failure contexts.

---

## 32. Primary Unsupervised Feature Matrix

### Decision

Use **24 continuous standardized features** for the primary clustering analysis:

- 8 healthcare-resource/geographic-context measures
- 16 underlying SVI indicators

### Excluded from primary cluster construction

- routine-checkup prevalence
- HPSA designation flags
- latitude/longitude
- SVI composite theme/overall rankings

### Sample size

**3,135 counties/county equivalents**

### Rationale

The representation allows healthcare-resource and vulnerability structure to emerge independently while avoiding direct inclusion of the outcome, binary shortage flags, geographic coordinates, or redundant SVI composites.

---

## 33. Unsupervised Methods

### Decision

Use two primary clustering families:

- K-Means
- Gaussian Mixture Models (GMM)

Use PCA for visualization and interpretation rather than as the primary clustering representation.

### Rationale

K-Means provides an interpretable distance-based solution with centroid profiles.

GMM provides a probabilistic alternative that allows components to differ in covariance structure, size, and shape.

Using both helps distinguish broad structures that persist across modeling assumptions from boundaries that are method-dependent.

---

## 34. K-Means Candidate Evaluation

### Decision

Evaluate candidate values `k = 2–10` using multiple criteria rather than selecting `k` from a single metric.

### Criteria

- silhouette score
- Davies–Bouldin index
- Calinski–Harabasz index
- inertia
- stability
- cluster size
- substantive interpretability

### Rationale

Internal clustering metrics do not necessarily identify the most substantively useful solution, particularly when the data form overlapping profiles rather than sharply separated groups.

---

## 35. Final K-Means Solution

### Decision

Select **K-Means, `k = 5`** as the primary unsupervised model.

### Evidence

- silhouette score: **0.123**
- mean seed-stability ARI: **0.992**
- minimum seed-stability ARI: **0.984**
- lowest Davies–Bouldin score among the evaluated candidate solutions
- local silhouette maximum
- five interpretable and reasonably balanced profiles

### Interpretation

The low silhouette score indicates that the five clusters should be interpreted as **overlapping county profiles along a continuum**, not as objectively discrete county types.

---

## 36. K-Means Profile Interpretation

### Decision

Interpret clusters using mean standardized feature values.

### Interpretation thresholds

- `|z| >= 0.5`: notable feature
- `|z| >= 1.0`: defining feature

### Final profiles

1. Rural, older, hospital-anchored access, low vulnerability
2. Urban/suburban, provider-rich, low vulnerability
3. Resource-scarce, low vulnerability
4. High access barriers, young, average healthcare resources
5. Rural high-hardship, FQHC-reliant

### Key substantive finding

Healthcare-resource scarcity and social vulnerability are related but distinct dimensions. The most provider-scarce profile is not the most socially vulnerable profile.

---

## 37. GMM Model Selection

### Decision

Use a **full-covariance GMM with `k = 5`** as the primary probabilistic comparison model.

Treat the full-covariance `k = 9` solution as exploratory.

### Evidence

- GMM `k = 5` mean stability ARI: **0.955**
- GMM `k = 5` minimum stability ARI: **0.921**
- GMM `k = 9` mean stability ARI: **0.747**
- GMM `k = 9` minimum stability ARI: **0.578**

The nine-component model achieved a lower BIC but was substantially less stable and more difficult to interpret.

### Rationale

Model-fit criteria such as BIC are considered alongside stability, component size, separation, and substantive interpretability rather than used alone.

---

## 38. K-Means vs. GMM Comparison

### Decision

Use K-Means as the **primary unsupervised model** and GMM as the **comparison model**.

### Evidence

- K-Means `k = 5` silhouette: **0.123**
- GMM `k = 5` silhouette: **0.023**
- K-Means mean stability ARI: **0.992**
- GMM mean stability ARI: **0.955**
- agreement between K-Means and GMM assignments: **ARI ≈ 0.29**

### Interpretation

Both methods recover several broad county structures, but finer boundaries—particularly among high-vulnerability counties—are method-dependent.

---

## 39. Unsupervised Sensitivity Analysis

### Decision

Evaluate sensitivity to both random initialization and feature representation.

### Final checks

- repeated-seed stability for K-Means `k = 5`
- repeated-seed stability for GMM `k = 5`
- exploratory GMM `k = 9`
- add HPSA designation flags
- equal weighting of healthcare-resource and SVI domains

### Key results

- K-Means seed stability: **ARI 0.992**
- GMM `k = 5` seed stability: **ARI 0.955**
- GMM `k = 9` seed stability: **ARI 0.747**
- K-Means + HPSA flags vs. primary: **ARI 0.403**
- GMM + HPSA flags vs. primary: **ARI 0.028**
- equal domain weighting vs. primary K-Means: **ARI 0.364**

### Interpretation

Random initialization has little effect on the selected five-cluster solutions, but finer cluster boundaries are sensitive to feature representation.

HPSA designation remains a complementary profiling/sensitivity measure rather than a primary clustering feature.

---

## 40. PLACES Outcome and Clustering

### Decision

Do **not** include routine-checkup prevalence in the clustering feature matrix.

Reintroduce it only after cluster assignments are finalized.

### Rationale

Clusters should emerge independently from healthcare-resource and social-vulnerability characteristics.

### Final use

Routine-checkup prevalence is used as an external descriptive comparison across independently identified profiles.

### Main finding

Cluster medians range approximately **71.5%–75.7%**, compared with an overall clustering-sample median of **74.4%**.

The lowest median prevalence occurs in profiles characterized by high uninsurance and related access barriers rather than in the profile defined primarily by provider scarcity.

These differences are descriptive, not causal.

---

## 41. Geographic Coordinates

### Decision

Do not include latitude or longitude in the primary clustering feature set.

### Rationale

The unsupervised research question concerns similarity in healthcare-access and vulnerability profiles rather than geographic proximity.

Maps are used for interpretation of the resulting profiles, not for defining cluster membership.

---

## 42. PCA Role

### Decision

Use PCA as a visualization and interpretation tool, not as the primary clustering feature representation.

### Rationale

Clustering on the standardized original variables preserves direct interpretability of the healthcare-resource and SVI features.

PCA is used to visualize broad structure, overlap, and differences between K-Means and GMM assignments.

---

## 43. Causal Interpretation

### Decision

Do not make causal claims from the supervised or unsupervised analyses.

### Appropriate language

Use:

- predicts
- is associated with
- differs across
- is related to
- characterizes
- is consistent with

Avoid:

- causes
- leads to
- increases because of
- would improve if changed

### Rationale

Healthcare resources, provider locations, utilization, insurance, social vulnerability, and community characteristics are jointly determined by many factors not controlled by the project design.

Predictive importance, cluster membership, and between-profile differences therefore do not identify causal effects.

---

## 44. Ethical Interpretation

### Decision

Treat both prediction and clustering outputs as descriptive/decision-support evidence rather than stand-alone allocation rules.

### Supervised concern

Predictions or feature importance could be misinterpreted as causal evidence or used to rank communities without accounting for omitted context and model error.

### Unsupervised concern

Labels such as “high vulnerability,” “high hardship,” or “resource scarce” may stigmatize communities if treated as inherent or fixed characteristics.

### Cross-cutting concern

County-level averages can conceal within-county inequality and must not be generalized to individual residents.

---

## 45. Prior Work and Code Reuse

### Decision

Document conceptual continuity with earlier team projects while distinguishing the present project from previously graded work.

### Final disclosure

The project uses:

- a newly constructed national county-level analytical dataset
- new supervised and unsupervised research questions
- newly developed modeling and evaluation workflows

**No previously graded supervised or unsupervised learning code was reused for this project.**

Any overlap with prior work is limited to domain knowledge, familiarity with public data sources, and general data-preparation concepts.

---

## 46. Reproducibility and Output Preservation

### Decision

Preserve raw data and save report-critical analytical outputs.

### Reproducibility rules

- raw source files remain unchanged
- transformations are implemented through code/notebooks
- dataset versions and reference years are documented
- HPSA reconstruction assumptions are documented
- FIPS-based merges are validated
- supervised preprocessing occurs inside pipelines where appropriate
- hyperparameter tuning uses training data/cross-validation only
- random seeds are fixed where applicable
- routine-checkup prevalence remains excluded from cluster construction
- report-critical metrics, tables, and figures are exported to project folders

---

## 47. Team Workflow

### Decision

Use a shared data-audit and analytical-dataset stage followed by parallel supervised and unsupervised analyses.

### Final workflow

`Raw data → source audit → geographic harmonization → cleaning/feature engineering → shared analytical dataset → parallel supervised/unsupervised EDA → model development → evaluation/sensitivity analysis → integrated interpretation → final report`

### Responsibility structure

- Samantha Salazar: data acquisition/audit, geographic harmonization, HPSA reconstruction, multi-source integration, cleaning/feature engineering, shared analytical dataset, feature manifest, methodological documentation, policy/health-equity framing
- Sara Jahanian: supervised EDA/modeling, tuning/evaluation, sensitivity analysis, feature importance/ablation, failure analysis, supervised visualizations
- Gina Sheesley: unsupervised EDA/modeling, clustering evaluation/stability, sensitivity analysis, cluster interpretation, geospatial/unsupervised visualizations, routine-checkup comparisons
- Shared: methodology review, interpretation, report writing, reproducibility, and final review

---

# Finalized Decisions Summary

The items previously listed as pending are now resolved as follows:

| Previously pending item | Final resolution |
|---|---|
| Final supervised feature set | 10 predictors across provider supply, healthcare infrastructure, geographic/access context, and Primary Care HPSA presence |
| Final clustering feature set | 24 continuous standardized features: 8 healthcare-resource/context measures + 16 SVI indicators |
| Model-specific missing-data treatment | Supervised preprocessing inside pipelines; structured Connecticut missingness excluded from primary clustering; remaining clustering missingness resolved before standardization |
| Connecticut AHRF missingness | Retained for supervised analysis; nine planning regions excluded from primary clustering |
| HPSA sensitivity specifications | Presence retained for primary supervised shortage representation; count/score used for supervised sensitivity; flags used for unsupervised sensitivity/profiling |
| Skewed feature transformations | Population-standardized rates retained; log population density used in primary specification; raw density tested and performed worse |
| Redundancy / multicollinearity | Conceptual and empirical screening used; SVI composites excluded when component indicators are present |
| Final number of clusters | K-Means `k = 5` selected as primary; full-covariance GMM `k = 5` retained as comparison; GMM `k = 9` exploratory |
| Final clustering preprocessing | Complete 3,135-county matrix; transformed where appropriate; standardized before fitting; PCA used for visualization |
| Hyperparameter-search strategy | Model-specific CV tuning within training data; MAE refit criterion |
| Supervised validation design | 5-fold random CV + state-grouped CV + reserved holdout evaluation of selected SVR only |
| Geographic robustness analysis | State-grouped cross-validation retained and reported |
| Supervised SVI sensitivity | 16 SVI indicators materially improve prediction but remain a sensitivity specification due to shared-input concerns |

---
