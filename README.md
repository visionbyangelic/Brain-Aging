# Brain Aging: Structural MRI-Based Brain Age Estimation and Cross-Dataset Generalization

A computational neuroimaging study of brain aging, structural brain morphometry, brain-age prediction, and the generalizability of neuroimaging-derived age estimates across heterogeneous datasets.

-----

## Abstract

Brain aging is a complex biological process characterized by progressive changes in brain structure, organization, and function across the lifespan. These changes are not uniform across individuals or brain regions. Some individuals exhibit patterns of brain structure that appear relatively preserved for their chronological age, whereas others show patterns associated with accelerated structural aging.

Brain-age estimation provides a computational framework for studying these differences. A machine-learning model is trained to estimate an individual’s chronological age from neuroimaging-derived features. The difference between the predicted age and chronological age is commonly referred to as the brain-age gap (BAG) and has been investigated as a potential marker of deviation from population-level patterns of brain aging.

However, brain-age prediction is not simply a problem of maximizing predictive accuracy. Neuroimaging datasets differ in scanner hardware, acquisition protocols, preprocessing pipelines, demographic composition, age distributions, and population characteristics. Consequently, a model can achieve strong performance within one dataset while failing to generalize to another.

This project investigates brain-age estimation using regional structural MRI-derived morphometric features, with particular emphasis on feature compatibility, cohort construction, model development, cross-validation, cross-dataset generalization, and the diagnosis of a real cohort-linkage defect that was silently discarding most of the available training data. The analysis uses neuroimaging resources including **OpenBHB** and **OASIS-3**, with regional morphometric measurements derived from **FreeSurfer** forming the primary feature space.

> **Central Premise:** A useful brain-age model should not only predict chronological age accurately within its training cohort; its predictions should remain interpretable, calibrated, and reasonably robust when confronted with changes in cohort composition and dataset characteristics. Where they are not, that failure should be reported as clearly as the successes.

-----

## Table of Contents

- [1. Background](#1-background)
- [2. The Brain-Age Gap](#2-the-brain-age-gap)
- [3. Problem Formulation & Research Gaps](#3-problem-formulation--research-gaps)
- [4. Research Questions & Hypotheses](#4-research-questions--hypotheses)
- [5. Methodological Workflow & Safeguards](#5-methodological-workflow--safeguards)
- [6. Datasets & Feature Harmonization](#6-datasets--feature-harmonization)
- [7. Modeling Strategy & Evaluation](#7-modeling-strategy--evaluation)
- [8. Results](#8-results)
- [9. Repository Structure](#9-repository-structure)
- [10. Reproducibility, Governance & Limitations](#10-reproducibility-governance--limitations)
- [11. Related Work](#11-related-work)
- [12. Author & Citation](#12-author--citation)

-----

## 1. Background

### 1.1 What Is Brain Aging?

Brain aging refers to the biological and structural changes that occur in the nervous system across the lifespan. Unlike chronological aging, which is simply the passage of time, biological brain aging reflects changes in the physical and functional properties of neural tissue.

These changes encompass multiple structural dimensions:

- Cortical thickness, surface area, and volume
- Subcortical gray matter volumes and ventricular enlargement
- Regional tissue composition and white-matter integrity
- Macro-scale network organization and functional connectivity
- Vascular, metabolic, and cognitive profiles

Brain aging is multidimensional: no single anatomical measurement completely captures how an individual’s brain ages.

### 1.2 Chronological Age vs. Biological Brain Age

- **Chronological Age (A_i):** The actual elapsed time since birth.
- **Brain Age (A_hat_i):** A statistical estimate of age inferred from neuroimaging features via a trained model f:
  
  A_hat_i = f(X_i)
  
  where X_i represents the individual’s neuroimaging-derived feature vector. The model is optimized such that f(X_i) ≈ A_i.

The resulting output is not a direct measurement of biological age, but a statistical prediction based on multivariate structural morphology.

-----

## 2. The Brain-Age Gap

The difference between predicted and chronological age defines the **Brain-Age Gap (BAG)**:

BAG_i = A_hat_i − A_i

### Interpretation

- BAG_i > 0: Predicted brain age is older than chronological age (apparent accelerated structural aging).
- BAG_i < 0: Predicted brain age is younger than chronological age (apparent preserved structural aging).
- BAG_i ≈ 0: Predicted brain age aligns with chronological age.

Raw BAG is not a pure biological aging signal. Raw brain-age gaps are systematically confounded by:

- Regression-to-the-mean bias
- Cohort age distributions and edge effects
- Scanner hardware and acquisition protocols
- Preprocessing and segmentation software versions
- Unmodeled demographic factors and feature measurement errors

Section 8 documents a case, found in this project’s own pipeline, where a corrected model still produced BAG values pointing in a direction inconsistent with the expected biological signal, and traces that inconsistency to demographic mismatch rather than treating it as a positive result.

-----

## 3. Problem Formulation & Research Gaps

### 3.1 Key Challenges

1. **High Predictive Performance ≠ Generalizability:** Models often learn site-, scanner-, or demographic-specific signatures rather than purely invariant biological aging trajectories.
1. **Dataset Heterogeneity:** Divergent field strengths, voxel resolutions, FreeSurfer versions, and inclusion criteria induce domain shift.
1. **Demographic Imbalance:** Models trained on young-skewed cohorts often saturate or fail when extrapolating to geriatric cohorts.
1. **Silent Data-Pipeline Defects:** Cohort-to-feature linkage errors can silently discard the majority of available training data without raising an exception, producing a model that appears to have trained successfully while using a small, unrepresentative fraction of the intended cohort. This project encountered exactly this failure mode; see Section 8.2.

### 3.2 Research Gaps Addressed

- How stable are regional morphometric relationships under severe dataset shift?
- Which specific morphometric features remain invariant and informative across cohorts?
- How can BAG be calibrated to separate true deviation from methodological bias?
- How much of a brain-age model’s reported accuracy is attributable to the model itself versus the integrity of the cohort-to-feature linkage that feeds it?

-----

## 4. Research Questions & Hypotheses

### Research Questions and current status

|#  |Question                                                                                                         |Status                                                                                                                                                                                                                                                                                                                                                                |
|---|-----------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|RQ1|To what extent does regional structural brain morphometry predict chronological age?                             |**Answered.** Cross-validated MAE of 2.41 years (R² = 0.88) within the OpenBHB reference cohort. See 8.3.                                                                                                                                                                                                                                                             |
|RQ2|Which regional measurements provide the most stable age-predictive signal?                                       |Open. Feature-level importance analysis has not yet been run on the corrected cohort.                                                                                                                                                                                                                                                                                 |
|RQ3|How do non-linear and regularized linear models compare in predictive accuracy and out-of-distribution stability?|**Partially answered.** SVR outperforms Ridge, Elastic Net, and Random Forest in-distribution; Random Forest is more resistant to extrapolation failure than SVR out-of-distribution (Sections 8.4–8.5).                                                                                                                                                              |
|RQ4|How robust are brain-age models to scanner and protocol differences?                                             |Open. Not yet isolated from the demographic-shift confound.                                                                                                                                                                                                                                                                                                           |
|RQ5|Does development-cohort performance transfer to an independent external cohort?                                  |**Answered, with an important caveat.** Rank-order transfers well (Pearson r up to 0.978 on OASIS-3); absolute calibration does not transfer cleanly across a ~47-year gap in mean age (Section 8.5).                                                                                                                                                                 |
|RQ6|Under what statistical conditions can BAG deviations be interpreted meaningfully?                                |**Answered negatively for the current pipeline.** The clinical-association test run in this project (Section 8.6) produced a BAG difference in the direction opposite to the expected biological signal, traced to out-of-distribution kernel collapse rather than a genuine clinical effect. BAG is not yet interpretable for clinical comparison with this pipeline.|

### Hypotheses

- **H1 (Information Content):** Morphometric features outperform a naive mean-age baseline. **Supported**, with a documented caveat: the naive baseline collapsed to a mathematically degenerate MAE of 0.000 under the cross-validated calibration procedure used here (a constant predictor is trivially “correctable”; see Section 8.3), so the comparison is reported for transparency but should not be read as a real baseline.
- **H2 (Non-Linearity):** Non-linear models (SVR, Random Forest) capture localized lifespan dynamics not fully captured by linear regularized models. **Supported in-distribution**; SVR and Random Forest both outperform Ridge and Elastic Net under cross-validation.
- **H3 (Sensitivity to Shift):** Out-of-distribution performance degrades under scanner and demographic shifts. **Supported**, and more severely than anticipated: calibrated R² on the external cohort remained negative even after the linkage bug was fixed, despite a strong Pearson correlation.
- **H4 (Cohort Composition):** Training age distribution dictates calibration fidelity across evaluation sub-bands. **Supported.** See Section 8.5–8.6.
- **H5 (Feature Parity):** Explicit feature mapping is necessary to ensure biological equivalence across differing pipelines. **Supported** by the Phase 0 feature-compatibility audit (Section 6.2).

-----

## 5. Methodological Workflow & Safeguards

```
Dataset Discovery -> Feature Inventory -> Cohort Definition & Matching
   |
   +-> Feature Compatibility Assessment (Parity Mapping)
   |
   v
Regional Morphometric Feature Matrix (N x M)
   |
   v
Disjoint Participant-Level Split / 5-Fold Grouped Cross-Validation
   |
   v
Model Ladder Training & Validation (Mean Baseline -> Ridge -> Elastic Net -> SVR -> Random Forest)
   |
   v
Out-of-Fold Bias Calibration ( E[BAG] = alpha + beta * A )
   |
   v
Held-Out Internal Evaluation -> External Cross-Dataset Generalization (OASIS-3) -> Clinical-Association Probe
```

### Methodological Safeguards

- **Disjoint Participant Splitting:** Partitions are assigned at the subject level (Train ∩ Val ∩ Test = ∅) to eliminate identity leakage.
- **Encapsulated Preprocessing:** Imputation and scaling parameters are estimated strictly on training folds inside isolated pipeline objects.
- **Independent Calibration:** Age-bias calibration lines are fitted exclusively on development data (out-of-fold training predictions) and applied frozen to unseen test sets.
- **Explicit Feature Mapping:** Columns are aligned via verified anatomical correspondences rather than unverified header matching.

These safeguards did not prevent every defect; Section 8.2 documents one that slipped past them at the manifest-loading stage, before any of the above pipeline logic ran.

-----

## 6. Datasets & Feature Harmonization

### 6.1 Cohort Summary

- **OpenBHB (Reference Development Cohort):**
  - 3,240 participants in the approved “Relaxed Tim Trio” manifest, the cohort definition formally locked for this project
  - 3,227 participants represented in the FreeSurfer feature inventory
  - 2,598 participants successfully matched between the two once the correct manifest was used (see Section 8.2 for why an earlier run used only 221)
  - Age span: 6–83 years (mean ≈ 20.7 years)
- **OASIS-3 (External Evaluation Cohort):**
  - Used for feature-compatibility auditing, an independent within-dataset baseline (Section 8.1), and as the external transportability target for the OpenBHB-trained model (Sections 8.5–8.6)
  - Healthy-control subset: 1,049 participants (CDRTOT = 0), mean age ≈ 67.8 years
  - Cognitively impaired subset used for the clinical-association probe: 266 participants (CDRTOT > 0), no overlap with the healthy-control subset

### 6.2 Feature Parity Mapping

The primary feature space comprises **136 harmonized “Option B” regional cortical measurements** derived from the Desikan–Killiany atlas (34 bilateral regions):

- 68 regional cortical thickness measurements (mm)
- 68 regional cortical gray matter volumes (mm³)

Column names, units, and anatomical correspondence between OpenBHB and OASIS-3 were verified explicitly rather than assumed from matching header strings; feature-name matching alone does not guarantee the same anatomy or processing pipeline.

### 6.3 Modeling Feature Matrix Structure

|Participant ID|Cortical Thickness (ROI 1)|Cortical Thickness (ROI 2)|… |Gray Matter Volume (ROI 68)|Chronological Age|
|:-------------|:-------------------------|:-------------------------|:-|:--------------------------|:----------------|
|`sub-0001`    |2.451                     |2.812                     |… |3412.0                     |21.4             |
|`sub-0002`    |2.310                     |2.695                     |… |3120.5                     |34.8             |
|`sub-0003`    |2.553                     |2.901                     |… |3890.2                     |19.2             |

-----

## 7. Modeling Strategy & Evaluation

### 7.1 Model Ladder

1. **Naive Mean Baseline:** Benchmark floor predicting the training-set mean age for every participant.
1. **Ridge Regression (L2):** Penalizes collinear morphometric parameters.
1. **Elastic Net (L1 + L2):** Balanced feature shrinkage and sparse selection.
1. **Support Vector Regression (SVR):** Non-linear mapping using an RBF kernel.
1. **Random Forest:** Bagged ensemble capturing non-linear interactions and regional thresholds.

### 7.2 Evaluation Metrics

- **Mean Absolute Error (MAE):** average magnitude of prediction error, in years.
- **Root Mean Squared Error (RMSE):** error metric that penalizes large individual errors more heavily.
- **Coefficient of Determination (R²):** proportion of age variance explained; can be negative when a model performs worse than predicting the mean for every participant.
- **Pearson Correlation (r):** linear alignment between predicted and chronological age, independent of any constant offset.
- **Residual Bias Correlation:** correlation between calibrated BAG and chronological age, targeting r ≈ 0.00 after calibration.

R² and Pearson r are reported together throughout Section 8 because, as Section 8.5 shows directly, they can diverge sharply and each hides what the other reveals.

-----

## 8. Results

This section reports every experiment run in this repository, including the ones that did not work as hoped. A model-selection decision or a clinical-association test that fails is still a result, and is reported as one.

### 8.1 Experiment 1 — Within-OASIS-3 Healthy-Control Baseline

A reference model trained and evaluated entirely within OASIS-3 (N = 1,049 healthy controls, participant-level train/val/test split), independent of OpenBHB, to establish a same-distribution performance ceiling.

|Model                   |Held-out test MAE|Held-out test R²|
|------------------------|-----------------|----------------|
|Random Forest (selected)|**3.27 years**   |**0.76**        |

Status: complete, artifacts locked, not re-run.

### 8.2 The Cohort-Linkage Defect

An early run of the OpenBHB normative model (Experiment 2) trained on only 221 participants, despite 2,598 being correctly available and matched. The root cause: the cohort-freeze step loaded a “Strict” OpenBHB manifest (N = 248) instead of the formally locked “Relaxed Tim Trio” manifest (N = 3,240). The written project decision and the executed code disagreed, silently, with no exception raised, so the smaller, incorrect cohort trained the model without any error signal.

The fix relinked the correct manifest directly to the FreeSurfer feature table, recovering the full N = 2,598 cohort (split 1,818 / 390 / 390 for train / validation / test). No other part of the modeling approach was changed.

This defect, and its fix, is reported here in full because it materially affected every downstream result in Sections 8.3–8.6, and because a data-pipeline defect of this kind is easy to introduce and easy to miss without an explicit manifest-vs-code cross-check.

### 8.3 Experiment 2 — OpenBHB Normative Model, Corrected Cohort, Cross-Validated

With the cohort defect fixed, the model ladder was re-evaluated using 5-fold, participant-grouped cross-validation with calibration nested inside each fold (out-of-fold predictions only, no leakage).

|Model             |CV Raw MAE|CV Calibrated MAE|CV Calibrated RMSE|CV Calibrated R²|CV Calibrated Pearson r|
|------------------|----------|-----------------|------------------|----------------|-----------------------|
|Mean Baseline     |5.178     |~0.000*          |~0.000*           |1.000*          |0.000*                 |
|**SVR (selected)**|3.555     |**2.410**        |**3.067**         |**0.882**       |**0.943**              |
|Random Forest     |3.530     |2.960            |4.961             |0.691           |0.832                  |
|Elastic Net       |3.979     |3.407            |4.361             |0.761           |0.895                  |
|Ridge             |4.223     |3.723            |4.775             |0.714           |0.876                  |

* The Mean Baseline’s calibrated score is a mathematical artifact, not a real result: a constant predictor’s error is a perfectly linear function of age, so the out-of-fold bias-correction step used here fits it exactly and drives the residual to zero. It is included in the table for transparency, not as a legitimate baseline comparison.

SVR was selected on calibrated MAE and is the clear winner on every metric once proper cross-validation is used. An earlier, static single-split evaluation of the same model ladder (retained in `data/BrainAge_Model_Development.ipynb` for the record) had shown SVR and Random Forest in a near-tie (validation MAE 3.454 vs. 3.467, with Random Forest actually ahead on R²: 0.545 vs. 0.368), which cross-validation resolved.

Status: complete.

### 8.4 A Sample-Weighted SVR Variant (evaluated, not adopted)

A second version of the SVR model was trained with inverse age-bin frequency sample weighting (C = 10, versus C = 1.0 for the unweighted model) to address a mean-collapse tendency observed on OpenBHB’s own internal test set. This variant is preserved in `data/Experiment2_OpenBHB_CrossValidated_ipynb.ipynb` alongside the unweighted model for direct comparison, but is **not** the model used going forward, because it performed substantially worse on external validation. See Section 8.5.

### 8.5 Experiment 3 — External Validation on OASIS-3 Healthy Controls

The frozen OpenBHB pipeline, with its calibration, applied unchanged to OASIS-3 healthy controls (N = 1,049, mean age ≈ 67.8 years — a ~47-year gap from OpenBHB’s ≈20.7-year mean).

|Pipeline version                                   |Raw MAE|Calibrated MAE|Calibrated R²    |Calibrated Pearson r|
|---------------------------------------------------|-------|--------------|-----------------|--------------------|
|Pre-fix (N = 221 training cohort)                  |—      |30.43 y       |strongly negative|—                   |
|Post-fix, unweighted SVR, single-split calibration |45.39 y|10.47 y       |−0.419           |0.978               |
|Post-fix, unweighted SVR, CV-calibrated (reference)|—      |9.31 y        |−0.147           |0.972               |
|Post-fix, **sample-weighted** SVR (Section 8.4)    |—      |16.16 y       |−3.011           |0.452               |

Fixing the cohort-linkage defect alone cut external MAE by roughly two-thirds, with no change to the model. Pearson correlation of 0.97–0.98 shows the model reliably ranks participants by relative age even on a cohort it has never seen from a different country and a very different age range. Calibrated R² nonetheless remains negative in every corrected-cohort run: predictions carry a systematic offset that a linear bias-correction fitted on a ≈20.7-year-mean training population does not fully remove when applied to a ≈67.8-year-mean external population. This combination, strong r alongside negative R², is a known signature of demographic extrapolation failure in the brain-age literature (Section 11), not unique to this pipeline.

The sample-weighted variant (Section 8.4) is a clear regression on this metric and is documented here specifically so it is not mistaken for an improvement in any future reuse of this repository.

Status: complete, in the sense that the transportability question has a clear, honest answer (rank-order transfers, absolute calibration does not). Per this project’s own evaluation protocol, a result this far from a well-calibrated external fit should be treated as a signal to revisit feature harmonization or calibration strategy before drawing further downstream (clinical) conclusions — which is exactly what Section 8.6 tested, and why its result should not be over-read.

### 8.6 Experiment 4 — Clinical Association Probe (exploratory; not a validated finding)

An exploratory test of whether calibrated BAG differs between OASIS-3 healthy controls (N = 1,049) and cognitively impaired participants (CDRTOT > 0, N = 266), run twice, once with each model variant from Section 8.5.

|Pipeline version              |HC mean BAG|Impaired mean BAG|Difference|p-value                                     |
|------------------------------|-----------|-----------------|----------|--------------------------------------------|
|Sample-weighted SVR           |−15.72 y   |−18.98 y         |−3.27 y   |< 0.0001 (t-test and Mann-Whitney)          |
|Unweighted SVR (CV-calibrated)|−9.31 y    |−11.82 y         |−2.51 y   |1.4×10⁻²⁶ (t-test), 2.5×10⁻²⁶ (Mann-Whitney)|

Both statistically significant, and both in the **wrong direction**. Published work on brain-age gap in cognitive impairment and Alzheimer’s disease consistently reports an *elevated* (more positive) BAG in impaired groups relative to healthy controls (Section 11), reflecting accelerated apparent structural aging. Here, the impaired group’s BAG is consistently *more negative* than the healthy controls’, in both model variants.

The most likely explanation, consistent with Section 8.5, is that severely atrophied, out-of-distribution brain scans push the RBF kernel’s predictions further toward the OpenBHB training-set mean age (≈20.7 years) than a typical healthy older-adult scan does, producing a larger apparent age underestimate for the impaired group for reasons unrelated to any real aging-biomarker signal. That the pattern reproduces under two materially different model configurations makes it more likely to be a structural property of this pipeline’s current extrapolation behavior than a fluke of one run.

**This result is not presented as evidence for or against the brain-age-gap biomarker hypothesis.** It is presented as a diagnosed failure mode: this pipeline, in its current form, cannot yet support a clinical-association claim, and running this comparison before the Section 8.5 calibration gap is closed is itself a violation of this project’s own evaluation protocol (Section 5 and 8.5 status note). It is retained in the repository, and in this README, specifically so that this failure mode is documented and not silently rediscovered later.

Status: exploratory, not validated, held pending resolution of Section 8.5.

### 8.7 Summary

|Question                                                               |Outcome                                                                                                                      |
|-----------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
|Does structural morphometry predict age at all?                        |Yes — strongly, in-distribution (R² = 0.88, cross-validated).                                                                |
|Does a cohort-linkage bug matter more than model choice?               |Yes — a data fix alone cut external error by ~two-thirds; a model change (sample weighting) alone made it dramatically worse.|
|Does the model transfer to an unseen, demographically different cohort?|Partially — rank-order yes (r up to 0.98), absolute calibration no (R² < 0).                                                 |
|Does the current pipeline support a clinical brain-age-gap claim?      |No, not yet — the one test run pointed the wrong direction and is attributed to extrapolation failure, not biology.          |

-----

## 9. Repository Structure

This reflects what is actually present in the repository at the time of writing, not an aspirational layout.

```text
Brain-Aging/
│
├── README.md
│
└── data/
    ├── 02_brainage_model_development.ipynb            # Fix log: diagnosis and correction of the cohort-linkage defect (Section 8.2)
    ├── BrainAge_Model_Development.ipynb                # Original (pre-fix) model ladder, single train/val/test split (Section 8.3, static-split comparison)
    ├── Experiment2_OpenBHB_CrossValidated_ipynb.ipynb   # Cross-validated Experiment 2, sample-weighted variant, Experiments 3 and 4 (Sections 8.3-8.6)
    ├── Feature_Compatibility_Assessment.ipynb           # Phase 0 feature-parity audit and cohort-manifest investigation (Section 6.2)
    ├── OASIS-3 feature inventory.ipynb                  # OASIS-3 feature cataloguing
    ├── OASIS3_FeatureMatrix_and_Baseline.ipynb          # Experiment 1, within-OASIS-3 baseline (Section 8.1)
    ├── OASIS3_Filtering.ipynb                           # OASIS-3 cohort filtering and manifest construction
    ├── OpenBHB_Feature_Inventory.ipynb                  # OpenBHB feature cataloguing
    └── OpenBHB_Filtering.ipynb                          # OpenBHB cohort filtering and manifest construction
```

No `models/`, `manifests/`, `reports/`, or `configs/` directories currently exist in this repository; frozen model artifacts, manifests, and calibration parameters currently live inside the notebooks that produce them. Splitting these into standalone versioned artifacts is listed as future work in Section 10.

-----

## 10. Reproducibility, Governance & Limitations

### Reproducibility Notes

Run the notebooks in the order implied by their names and the workflow in Section 5: filtering and feature-inventory notebooks first, then `Feature_Compatibility_Assessment.ipynb`, then `OASIS3_FeatureMatrix_and_Baseline.ipynb` (Experiment 1), then `BrainAge_Model_Development.ipynb` and `02_brainage_model_development.ipynb` (the pre-fix model and the fix log), then `Experiment2_OpenBHB_CrossValidated_ipynb.ipynb` (Experiments 2 through 4). Several cells in the notebooks depend on variables defined in earlier cells within the same runtime session; running cells out of order, or in a fresh runtime without executing everything above them, will raise `NameError`s rather than silently producing wrong numbers, which is preferable but still worth knowing going in.

### Data Governance

This repository contains code and notebooks only. Raw neuroimaging data and direct participant identifiers are not distributed here. Access to primary scans must be requested through the official data portals:

- **OpenBHB:** <https://baobab.neurospin.info/openbhb/> (see also the OpenBHB Challenge paper, Section 11)
- **OASIS-3:** <https://www.oasis-brains.org>

### Key Limitations

- **Demographic concentration:** OpenBHB skews young (mean ≈ 20.7 years); evaluation against a geriatric external cohort (OASIS-3, mean ≈ 67.8 years) sits well outside the training distribution for a meaningful fraction of participants, and this is the primary driver of the calibration issues in Section 8.5.
- **Negative external R² despite strong correlation:** documented and explained, not resolved. Anyone extending this work should treat the current calibration as unfit for absolute-age claims on cohorts this age-mismatched, even though relative ranking is reliable.
- **No validated clinical association yet:** Section 8.6 is exploratory and reproduces a plausible artifact, not a biological finding. It should not be cited as evidence either for or against brain-age gap as a marker of cognitive impairment.
- **Model-selection instability across configurations:** the sample-weighted SVR variant was never re-validated against the full model ladder before being evaluated externally; Section 8.4-8.5 exists specifically to prevent that gap from being repeated silently.
- **Statistical, not biological, age:** predicted brain age reflects a multivariate statistical property of structural MRI, not a direct index of biological senescence.
- **Non-diagnostic:** BAG is a continuous research metric here, not an individual clinical diagnostic tool.
- **Population scope:** all data used to date is drawn from OpenBHB and OASIS-3. Neither includes African or Nigerian participants; external validation on that population is planned but not yet part of this repository.

### Suggested Next Steps

1. Resolve the Section 8.5 calibration gap (age-representative recalibration, non-linear bias correction, or restricting claims to the training-age range) before re-attempting Section 8.6.
1. Re-run the full model ladder with sample weighting applied consistently, rather than testing weighting only on the already-selected model.
1. Split frozen model artifacts, manifests, and calibration parameters out of the notebooks and into the `models/` / `manifests/` structure implied by earlier drafts of this document, for cleaner reuse.
1. Extend evaluation to a population-representative African/Nigerian cohort once available.

-----

## 11. Related Work

This project’s central empirical findings, that a model transfers in rank-order but not in absolute calibration across a large demographic gap, and that training-cohort age composition materially shapes both accuracy and interpretation, are consistent with a growing body of published work:

- Dufumier, B. et al. **OpenBHB: A Large-Scale Multi-Site Brain MRI Dataset for Age Prediction and Debiasing.** *NeuroImage*, 2022. The dataset used for model development here; its own challenge design explicitly separates internal (same-site) and external (unseen-site) test sets for exactly the reason this project rediscovered empirically. <https://www.sciencedirect.com/science/article/pii/S1053811922007522>
- **Bias and generalizability of brain age prediction models: A multi-cohort evaluation with anatomical and interpretability insights.** *Imaging Neuroscience*, 2026. Reports that brain-age predictions consistently regress toward the mean age of the training cohort, underestimating age in older-skewed external cohorts and overestimating it in younger-skewed ones, the same pattern documented in Section 8.5. <https://direct.mit.edu/imag/article/doi/10.1162/IMAG.a.1164/135350/Bias-and-generalizability-of-brain-age-prediction>
- **Brain Age Prediction: Deep Models Need a Hand to Generalize.** *(PMC)*. Notes that error rises sharply for participants outside a model’s training age range, and that the authors could not locate prior published work directly addressing this extrapolation problem, underscoring that Section 8.5’s finding is a real, still largely open, methodological gap in the field. <https://pmc.ncbi.nlm.nih.gov/articles/PMC12265028/>
- **Distribution Bias in Brain Age Research: Toward Age-Specific Interpretation of Brain Age Gaps.** *ScienceDirect*, 2026. Trained 400 models across differently age-skewed training distributions and found training-data age composition significantly changes both predictions and how brain-age gap should be interpreted clinically, directly relevant to Section 8.6’s caution against over-reading a single clinical-association run. <https://www.sciencedirect.com/science/article/pii/S2451902226000789>
- Franke, K. & Gaser, C. **BrainAGE in Mild Cognitive Impaired Patients: Predicting the Conversion to Alzheimer’s Disease.** *PLOS ONE*, 2013. Establishes the expected direction of the clinical signal (elevated, i.e. more positive, BAG in impaired participants) against which Section 8.6’s wrong-direction result is judged. <https://journals.plos.org/plosone/article?id=10.1371%2Fjournal.pone.0067346>
- **Analysis of Brain Age Gap across Subject Cohorts and Prediction Model Architectures.** *(PMC)*. Reports typical published BAG elevations of roughly 2–11 years across Parkinson’s disease, MCI, Alzheimer’s disease, and related conditions, useful context for the magnitude (not just direction) expected of a genuine clinical effect. <https://www.ncbi.nlm.nih.gov/pmc/articles/PMC11428686/>

-----

## 12. Author & Citation

Maintained by [@visionbyangelic](https://github.com/visionbyangelic). Author ORCID: [0009-0008-7279-9663](https://orcid.org/0009-0008-7279-9663).

If referencing this repository or its findings, please cite the repository directly (`github.com/visionbyangelic/Brain-Aging`) pending a formal publication, and separately cite the underlying datasets:

- OpenBHB: Dufumier et al., *NeuroImage* (2022) — <https://www.sciencedirect.com/science/article/pii/S1053811922007522>
- OASIS-3: LaMontagne et al., *OASIS-3: Longitudinal Neuroimaging, Clinical, and Cognitive Dataset for Normal Aging and Alzheimer’s Disease* — <https://www.oasis-brains.org>

-----

*Brain aging is not simply a number predicted from an MRI. The scientific challenge is understanding what that number represents, and being equally clear about the cases where the model does not yet know.*