
# Brain Aging: Structural MRI-Based Brain Age Estimation and Cross-Dataset Generalization

A computational neuroimaging study of brain aging, structural brain morphometry, brain-age prediction, and the generalizability of neuroimaging-derived age estimates across heterogeneous datasets.

---

## Abstract

Brain aging is a complex biological process characterized by progressive changes in brain structure, organization, and function across the lifespan. These changes are not uniform across individuals or brain regions. Some individuals exhibit patterns of brain structure that appear relatively preserved for their chronological age, whereas others show patterns associated with accelerated structural aging.

Brain-age estimation provides a computational framework for studying these differences. A machine-learning model is trained to estimate an individual's chronological age from neuroimaging-derived features. The difference between the predicted age and chronological age is commonly referred to as the brain-age gap ($\text{BAG}$) and has been investigated as a potential marker of deviation from population-level patterns of brain aging.

However, brain-age prediction is not simply a problem of maximizing predictive accuracy. Neuroimaging datasets differ in scanner hardware, acquisition protocols, preprocessing pipelines, demographic composition, age distributions, and population characteristics. Consequently, a model can achieve strong performance within one dataset while failing to generalize to another.

This project investigates brain-age estimation using regional structural MRI-derived morphometric features, with particular emphasis on feature compatibility, cohort construction, model development, validation, and cross-dataset generalization. The analysis uses neuroimaging resources including **OpenBHB** and **OASIS-3**, with regional morphometric measurements derived from **FreeSurfer** forming the primary feature space.

> **Central Premise:** A useful brain-age model should not only predict chronological age accurately within its training cohort; its predictions should remain interpretable, calibrated, and reasonably robust when confronted with changes in cohort composition and dataset characteristics.

---

## Table of Contents
- [1. Background](#1-background)
- [2. The Brain-Age Gap](#2-the-brain-age-gap)
- [3. Problem Formulation & Research Gaps](#3-problem-formulation--research-gaps)
- [4. Research Questions & Hypotheses](#4-research-questions--hypotheses)
- [5. Methodological Workflow & Safeguards](#5-methodological-workflow--safeguards)
- [6. Datasets & Feature Harmonization](#6-datasets--feature-harmonization)
- [7. Modeling Strategy & Evaluation](#7-modeling-strategy--evaluation)
- [8. Repository Structure](#8-repository-structure)
- [9. Reproducibility, Governance & Limitations](#9-reproducibility-governance--limitations)
- [10. Author & Citation](#10-author--citation)

---

## 1. Background

### 1.1 What Is Brain Aging?
Brain aging refers to the biological and structural changes that occur in the nervous system across the lifespan. Unlike chronological aging, which is simply the passage of time, biological brain aging reflects changes in the physical and functional properties of neural tissue.

These changes encompass multiple structural dimensions:
* Cortical thickness, surface area, and volume
* Subcortical gray matter volumes and ventricular enlargement
* Regional tissue composition and white-matter integrity
* Macro-scale network organization and functional connectivity
* Vascular, metabolic, and cognitive profiles

Brain aging is multidimensional: no single anatomical measurement completely captures how an individual's brain ages.

### 1.2 Chronological Age vs. Biological Brain Age
* **Chronological Age ($A_i$):** The actual elapsed time since birth.
* **Brain Age ($\widehat{A}_i$):** A statistical estimate of age inferred from neuroimaging features via a trained model $f$:

$$\widehat{A}_i = f(X_i)$$

where $X_i$ represents the individual's neuroimaging-derived feature vector. The model is optimized such that:

$$f(X_i) \approx A_i$$

The resulting output is not a direct measurement of biological age, but a statistical prediction based on multivariate structural morphology.

---

## 2. The Brain-Age Gap

The difference between predicted and chronological age defines the **Brain-Age Gap ($\text{BAG}$)**:

$$\text{BAG}_i = \widehat{A}_i - A_i$$

### Interpretation
* $\text{BAG}_i > 0$: Predicted brain age is older than chronological age (apparent accelerated structural aging).
* $\text{BAG}_i < 0$: Predicted brain age is younger than chronological age (apparent preserved structural aging).
* $\text{BAG}_i \approx 0$: Predicted brain age aligns with chronological age.

$$\text{Raw BAG} \neq \text{Pure Biological Aging Signal}$$

Raw brain-age gaps are systematically confounded by:
* Regression-to-the-mean bias
* Cohort age distributions and edge effects
* Scanner hardware and acquisition protocols
* Preprocessing and segmentation software versions
* Unmodeled demographic factors and feature measurement errors

---

## 3. Problem Formulation & Research Gaps

### 3.1 Key Challenges
1. **High Predictive Performance $\neq$ Generalizability:** Models often learn site-, scanner-, or demographic-specific signatures rather than purely invariant biological aging trajectories.
2. **Dataset Heterogeneity:** Divergent field strengths, voxel resolutions, FreeSurfer versions, and inclusion criteria induce domain shift.
3. **Demographic Imbalance:** Models trained on young-skewed cohorts often saturate or fail when extrapolating to geriatric cohorts.

### 3.2 Research Gaps Addressed
* How stable are regional morphometric relationships under severe dataset shift?
* Which specific morphometric features remain invariant and informative across cohorts?
* How can $\text{BAG}$ be calibrated to separate true deviation from methodological bias?

---

## 4. Research Questions & Hypotheses

### Research Questions
* **RQ1 (Age Information):** To what extent does regional structural brain morphometry predict chronological age?
* **RQ2 (Feature Representation):** Which regional measurements provide the most stable age-predictive signal?
* **RQ3 (Model Complexity):** How do non-linear and regularized linear models compare in predictive accuracy and out-of-distribution stability?
* **RQ4 (Dataset Shift):** How robust are brain-age models to scanner and protocol differences?
* **RQ5 (Generalization):** Does development-cohort performance transfer to independent external cohorts?
* **RQ6 (Gap Calibration):** Under what statistical conditions can $\text{BAG}$ deviations be interpreted meaningfully?

### Hypotheses
* **H1 (Information Content):** Morphometric features outperform a naive mean-age baseline: $\text{Performance}(\text{Model}) > \text{Performance}(\text{Mean Baseline})$.
* **H2 (Non-Linearity):** Non-linear models (SVR, Random Forest) capture localized lifespan dynamics not fully captured by linear regularized models.
* **H3 (Sensitivity to Shift):** Out-of-distribution performance degrades under scanner and demographic shifts.
* **H4 (Cohort Composition):** Training age distribution dictates calibration fidelity across evaluation sub-bands.
* **H5 (Feature Parity):** Explicit feature mapping is necessary to ensure biological equivalence across differing pipelines.

---

## 5. Methodological Workflow & Safeguards


```

Dataset Discovery ──► Feature Inventory ──► Cohort Definition & Matching
│
┌──────────────────────────────────────────────────┘
▼
Feature Compatibility Assessment (Parity Mapping)
│
▼
Regional Morphometric Feature Matrix ($N \times M$)
│
▼
Disjoint Participant-Level Split (70% Train / 15% Val / 15% Test)
│
▼
Model Ladder Training & Validation (Mean Baseline ──► Ridge ──► Elastic Net ──► SVR ──► Random Forest)
│
▼
Development Bias Calibration ($\mathbb{E}[\text{BAG}] = \alpha + \beta A$)
│
▼
Held-Out Internal Evaluation ──► External Cross-Dataset Generalization (OASIS-3)

```

### Methodological Safeguards
* **Disjoint Participant Splitting:** Partitions are assigned at the subject level ($\text{Train} \cap \text{Val} \cap \text{Test} = \emptyset$) to eliminate identity leakage.
* **Encapsulated Preprocessing:** Imputation and scaling parameters are estimated strictly on training folds inside isolated pipeline objects.
* **Independent Calibration:** Age-bias calibration lines are fitted exclusively on development data (Train + Val) and applied frozen to unseen test sets.
* **Explicit Feature Mapping:** Columns are aligned via verified anatomical correspondences rather than unverified header matching.

---

## 6. Datasets & Feature Harmonization

### 6.1 Cohort Summary
* **OpenBHB (Reference Development Cohort):**
  * $3,240$ participants in the relaxed cohort[cite: 1]
  * $3,227$ represented in the FreeSurfer feature inventory
  * $2,598$ verified successfully matched participants[cite: 1]
  * Age span: $6 \le \text{Age} \le 83$ years ($\text{Mean} \approx 20.7$ years)[cite: 1]
* **OASIS-3 (External Evaluation Cohort):**
  * Independent cohort used for feature compatibility audits, baseline distribution locking, and cross-dataset transportability evaluation[cite: 1, 2].

### 6.2 Feature Parity Mapping
The primary feature space comprises **136 harmonized Option B regional cortical measurements** derived from the Desikan–Killiany atlas ($34$ bilateral regions)[cite: 1, 8]:
* $68$ regional cortical thickness measurements ($\text{mm}$)
* $68$ regional cortical gray matter volumes ($\text{mm}^3$)

### 6.3 Modeling Feature Matrix Structure
| Participant ID | Cortical Thickness (ROI 1) | Cortical Thickness (ROI 2) | ... | Gray Matter Volume (ROI 68) | Chronological Age ($A_i$) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `sub-0001` | $2.451$ | $2.812$ | ... | $3412.0$ | $21.4$ |
| `sub-0002` | $2.310$ | $2.695$ | ... | $3120.5$ | $34.8$ |
| `sub-0003` | $2.553$ | $2.901$ | ... | $3890.2$ | $19.2$ |

---

## 7. Modeling Strategy & Evaluation

### 7.1 Model Ladder
1. **Naive Mean Baseline:** Benchmark floor predicting $\widehat{A}_i = \bar{A}_{\text{train}}$.
2. **Ridge Regression ($L_2$):** Penalizes collinear morphometric parameters:
   $$\min_{\beta} \left[ \Vert{}y - X\beta\Vert{}_2^2 + \lambda \Vert{}\beta\Vert{}_2^2 \right]$$
3. **Elastic Net ($L_1 + L_2$):** Balanced feature shrinkage and sparse selection:
   $$\min_{\beta} \left[ \Vert{}y - X\beta\Vert{}_2^2 + \lambda \left( \alpha \Vert{}\beta\Vert{}_1 + (1 - \alpha)\Vert{}\beta\Vert{}_2^2 \right) \right]$$
4. **Support Vector Regression (SVR):** Non-linear mapping using an RBF kernel: $K(x, x') = \exp(-\gamma \Vert{}x - x'\Vert{}^2)$.
5. **Random Forest:** Bagged ensemble capturing non-linear interactions and regional thresholds.

### 7.2 Evaluation Metrics
* **Mean Absolute Error (MAE):** $\text{MAE} = \frac{1}{N} \sum_{i=1}^N \vert{}\widehat{A}_i - A_i\vert{}$
* **Root Mean Squared Error (RMSE):** $\text{RMSE} = \sqrt{\frac{1}{N} \sum_{i=1}^N (\widehat{A}_i - A_i)^2}$
* **Coefficient of Determination ($R^2$):** $R^2 = 1 - \frac{\sum_i (A_i - \widehat{A}_i)^2}{\sum_i (A_i - \bar{A})^2}$
* **Pearson Correlation ($r$):** Linear alignment between chronological and predicted age.
* **Residual Bias Correlation:** $r(\text{BAG}_{\text{corr}}, A)$, targeting $r \approx 0.00$.

---

## 8. Repository Structure

```text
Brain-Aging/
│
├── configs/
│   └── pipeline_config.json
│
├── data/
│   ├── 02_brainage_model_development.ipynb
│   ├── BrainAge_Model_Development.ipynb
│   ├── Experiment2_OpenBHB_CrossValidated.ipynb
│   ├── Feature_Compatibility_Assessment.ipynb
│   ├── OASIS-3_feature_inventory.ipynb
│   ├── OASIS3_FeatureMatrix_and_Baseline.ipynb
│   ├── OASIS3_Filtering.ipynb
│   ├── OpenBHB_Feature_Inventory.ipynb
│   └── OpenBHB_Filtering.ipynb
│
├── manifests/
│   ├── feature_parity/
│   │   └── openBHB_OASIS3_phase0_regional_measurement_compatibility.csv
│   ├── openbhb/
│   └── oasis3/
│
├── models/
│   ├── frozen_openbhb_normative_pipeline.joblib
│   └── frozen_openbhb_model_config.json
│
├── reports/
│   ├── figures/
│   └── model_card.md
│
├── requirements.txt
└── README.md

```

---

## 9. Reproducibility, Governance & Limitations

### Reproducibility Standards

To reproduce these experiments, execute the notebooks in sequence or import the frozen artifacts using Python:

```python
import joblib
import pandas as pd

# Load the frozen normative modeling pipeline
bundle = joblib.load("models/frozen_openbhb_normative_pipeline.joblib")
model = bundle["pipeline"]
features = bundle["feature_names"]
alpha = bundle["bias_calibration"]["intercept_alpha"]
beta = bundle["bias_calibration"]["slope_beta"]

# Predict and calibrate on new feature matrix X (DataFrame)
y_pred_raw = model.predict(X[features])
bag_raw = y_pred_raw - chronological_age
bag_corrected = bag_raw - (alpha + beta * chronological_age)
y_pred_calibrated = chronological_age + bag_corrected

```

### Data Governance

This repository contains code, configuration metadata, and statistical manifests only. Raw neuroimaging data and direct participant identifiers are not distributed here. Access to primary scans must be requested through official data portals:

* **OpenBHB Consortium:** [OpenBHB Portal](https://www.google.com/search?q=https://neurospin-projects.org)
* **OASIS-3:** [NITRC / Central Neuroimaging Resource](https://www.oasis-brains.org)

### Key Limitations

* **Demographic Concentration:** OpenBHB reference distributions skew young (mean $\approx 20.7$ years); external evaluation on geriatric cohorts requires domain-shift adjustments.


* **Statistical vs. Biological Age:** Predicted brain age reflects a multivariate statistical property of structural MRI scans, not an absolute index of biological senescence.
* **Non-Diagnostic Nature:** $\text{BAG}$ is a continuous research metric and cannot be used as an independent clinical diagnosis.

---


---

*Brain aging is not simply a number predicted from an MRI. The scientific challenge is understanding what that number represents.*

```

```
