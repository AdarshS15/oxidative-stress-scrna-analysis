# Forecasting Future Transcriptional States Using Time-Course scRNA-seq

## Project Overview

This project investigates whether computational models can predict future transcriptional states from earlier time-course single-cell RNA sequencing (scRNA-seq) measurements.

Six machine learning architectures were benchmarked using two biological systems: hydrogen-peroxide-treated human fibroblasts undergoing senescence and TGF-β-treated human MCF10A breast epithelial cells undergoing epithelial-to-mesenchymal transition (EMT).

The study focuses on **forecasting completely held-out future cell populations**, comparing predictive performance across model architectures and evaluating whether accurate population-level predictions also preserve cell-to-cell variability.

This research was conducted as part of an independent computational biology research project through Polygence.

## Research Objectives

The project addresses three main questions:

1. Can computational models forecast future transcriptional states from earlier scRNA-seq population measurements?
2. How do different machine learning architectures compare in their ability to predict unseen future populations?
3. How do cell-pairing strategies and model architecture influence forecasting accuracy and preservation of cellular variability?

## Models Evaluated

Six computational architectures were implemented and compared:

| Model | Approach |
|---|---|
| Ridge Regression | Regularized linear regression |
| Multi-Layer Perceptron (MLP) | Feedforward neural network |
| XGBoost Regressor | Gradient-boosted decision trees |
| Random Forest Regressor | Ensemble of decision trees |
| Neural Ordinary Differential Equation (Neural ODE) | Continuous-time neural dynamics |
| Temporal-Token Transformer | Attention-based temporal prediction |

Each model predicts future cell states using principal-component representations of gene expression and the requested forecasting interval.

## Datasets

### 1. Human Fibroblast Senescence

The primary dataset contains time-course scRNA-seq measurements from human fibroblasts undergoing oxidative-stress-associated senescence following hydrogen peroxide treatment.

- **Prediction target:** Day 7
- **Evaluation:** Completely held-out Day-7 cell population
- **Model selection:** Earlier validation time points
- **Biological process:** Cellular senescence

### 2. Epithelial-to-Mesenchymal Transition (EMT)

The secondary dataset contains time-course scRNA-seq measurements from human MCF10A breast epithelial cells treated with TGF-β to induce EMT.

- **Prediction target:** Day 8
- **Evaluation:** Completely held-out Day-8 cell population
- **Biological process:** Epithelial-to-mesenchymal transition
- **Purpose:** Evaluate the forecasting framework in a second biological system

Model architectures and training hyperparameters were transferred from the primary experiment, while PCA dimensions and optimal-transport settings were selected independently for the EMT dataset.

## Methodology

### 1. Data Preprocessing

Gene-expression measurements were processed using a consistent preprocessing framework:

1. Normalize and log-transform expression values.
2. Select 2,000 highly variable genes.
3. Standardize expression using training-derived statistics.
4. Apply principal component analysis (PCA).
5. Represent cells in a reduced-dimensional transcriptional space.

Preprocessing and model selection were performed without using the final held-out forecasting endpoint.

### 2. Cell Correspondence Construction

Because time-course scRNA-seq consists of separate population measurements, direct cell-to-cell correspondences between time points are generally unavailable.

Two pairing strategies were compared:

- **Optimal-transport pairing:** Entropic Sinkhorn optimal transport was used to infer correspondences between observed cell populations, followed by assignment under a target-use cap.
- **Capped-random pairing:** Source cells were paired with randomly selected observed target cells under a comparable target-use constraint.

These strategies allowed the study to examine whether inferred cell correspondences improve downstream forecasting performance.

### 3. Model Training and Forecasting

Models were trained using earlier observed time points and corresponding time intervals.

The final forecasting task required models to predict an unseen future population beyond the training period.

Model-specific PCA dimensions were selected using validation data, and predicted PCA representations were transformed back into gene-expression space for evaluation.

### 4. Evaluation

Forecasting performance was evaluated using multiple complementary metrics:

- **Pearson r²:** Agreement between predicted and observed population-level mean gene expression.
- **Endpoint R²:** Additional measure of population-level expression accuracy.
- **Wasserstein distance:** Distributional difference between predicted and observed populations.
- **Mean gene-expression variance:** Variation across predicted cells.
- **Mean distance from population center:** Overall population dispersion.
- **PCA-space variance preservation:** Comparison of predicted and observed variance within the modeled PCA subspace.

Evaluations were performed across all 2,000 selected highly variable genes and a subset of the 100 most variable genes.

## Results

### Primary Dataset: Fibroblast Senescence

All six models outperformed the Day-0 persistence baseline when forecasting the held-out Day-7 population.

Under optimal-transport pairing, **Random Forest achieved the highest population-level Pearson r²**, with:

- **All 2,000 HVGs:** 0.9314
- **Top 100 HVGs:** 0.8181

The results also showed that high endpoint accuracy did not necessarily correspond to preservation of observed cell-to-cell variation.

### Secondary Dataset: EMT

All six architectures also outperformed the persistence baseline when forecasting the held-out Day-8 population.

Under optimal-transport pairing, **XGBoost achieved the highest All-HVG Pearson r²**, while the Temporal-Token Transformer achieved the highest Top-100-HVG Pearson r².

| Metric | Best model | Score |
|---|---|---|
| All-HVG Pearson r² | XGBoost | 0.9737 |
| Top-100-HVG Pearson r² | Temporal-Token Transformer | 0.9436 |

### Main Findings

- Future population-level transcriptional states could be forecast with substantially greater accuracy than the persistence baseline in both datasets.
- No single model architecture consistently dominated across both biological systems.
- More complex neural architectures did not consistently outperform simpler regression and tree-based methods.
- Optimal-transport pairing did not universally improve forecasting performance.
- High endpoint-expression accuracy did not necessarily indicate accurate preservation of observed population variability.

## Reproducibility

The experimental framework uses controlled random seeds, shared training and evaluation procedures, and held-out future endpoints.

To preserve the validity of forecasting evaluation, the final endpoint populations were excluded from preprocessing fitting, model training, correspondence construction, and hyperparameter selection.

The repository contains the code used for the forecasting experiments and associated analyses.

## Limitations and Future Work

The study evaluated two biological systems and a finite set of model architectures and hyperparameter configurations.

Although the results demonstrate strong population-level forecasting performance, they do not establish that predicted cells reproduce biologically correct individual-cell trajectories or subpopulations.

Future research could investigate additional biological datasets, longer forecasting horizons, more extensive model optimization, and distribution-aware forecasting approaches.

## Research Context

This project was developed through Polygence as an independent computational biology research study.

Its primary objective is to evaluate computational approaches for forecasting future transcriptional states from time-course scRNA-seq data and to identify how model architecture influences predictive performance.
