
# Forecasting Future Transcriptional States Using Time-Course scRNA-seq

## Project Overview

This research investigates whether machine learning models can forecast future transcriptional states using earlier time-course single-cell RNA sequencing (scRNA-seq) measurements.

Six computational architectures were benchmarked across two biological systems:

- **Fibroblast senescence:** Hydrogen-peroxide-treated human fibroblasts undergoing oxidative-stress-associated senescence.
- **Epithelial-to-mesenchymal transition (EMT):** TGF-β-treated human MCF10A breast epithelial cells undergoing EMT.

The central challenge is predicting a completely held-out future cell population using only earlier observations. Beyond evaluating population-level expression accuracy, the study examines whether models preserve the distributional properties and cell-to-cell variability of observed populations.

The research was conducted through the Polygence Research Program with guidance from a research mentor.

## Research Objectives

The study addresses three primary questions:

1. Can computational models forecast future transcriptional states from earlier scRNA-seq population measurements?
2. How do different machine learning architectures compare when predicting completely unseen future populations?
3. How do optimal-transport versus random cell-pairing strategies influence forecasting accuracy and preservation of cellular variability?

## Repository Structure

The repository contains the two primary experimental pipelines:

| File | Description |
|---|---|
| `FibroblastDatasetModellingPipeline.ipynb` | Fibroblast senescence forecasting, model training, validation, and evaluation |
| `EMTDatasetModellingPipeline.ipynb` | EMT forecasting, model training, validation, and evaluation |
| `README.md` | Research overview, methodology, and results |
| `.gitignore` | Excludes unnecessary local files and generated artifacts |

Each notebook contains the corresponding dataset's computational workflow and experimental analyses.

## Models Evaluated

Six machine learning architectures were implemented and compared:

| Model | Approach |
|---|---|
| Ridge Regression | Regularized linear regression |
| Multi-Layer Perceptron (MLP) | Feedforward neural network |
| XGBoost Regressor | Gradient-boosted decision trees |
| Random Forest Regressor | Ensemble of decision trees |
| Neural Ordinary Differential Equation (Neural ODE) | Learned continuous-time dynamics |
| Temporal-Token Transformer | Attention-based temporal prediction |

Models use PCA-based representations of gene expression and a time-interval input to predict future transcriptional states.

## Datasets and Forecasting Tasks

### 1. Human Fibroblast Senescence

The primary dataset consists of time-course scRNA-seq measurements from human fibroblasts undergoing oxidative-stress-associated senescence following hydrogen peroxide treatment.

- **Forecasting task:** Predict the Day-7 population from Day-0 cells.
- **Held-out endpoint:** Day 7.
- **Validation endpoint:** Day 4.
- **Biological process:** Cellular senescence.

### 2. Epithelial-to-Mesenchymal Transition

The secondary dataset contains time-course scRNA-seq measurements from TGF-β-treated human MCF10A breast epithelial cells undergoing EMT.

- **Forecasting task:** Predict the Day-8 population from Day-0 cells.
- **Held-out endpoint:** Day 8.
- **Validation endpoint:** Day 6.
- **Biological process:** Epithelial-to-mesenchymal transition.

The same six model architectures and overall experimental framework were applied to both datasets. PCA dimensionality and optimal-transport pairing settings were selected independently using each dataset's earlier validation time points.

## Methodology

### 1. Data Preprocessing

Gene-expression measurements were processed using the following framework:

1. Normalize and log-transform expression measurements.
2. Select 2,000 highly variable genes (HVGs).
3. Standardize expression using training-derived statistics.
4. Apply principal component analysis (PCA).
5. Represent cells in reduced-dimensional transcriptional space.

Preprocessing parameters were fitted without access to the final held-out endpoint population.

### 2. PCA Dimensionality Selection

PCA dimensionality was evaluated using earlier validation endpoints rather than the final forecasting targets.

Each model was evaluated across multiple PCA dimensionalities, and the dimensionality producing the strongest validation performance was selected.

This allowed different architectures to operate at dimensionalities suited to their predictive behavior rather than imposing a single fixed representation across every model.

### 3. Cell Correspondence Construction

Because scRNA-seq measures different cells at different time points, direct cell-to-cell correspondences are generally unavailable.

Two training-pair construction strategies were compared:

**Optimal-transport pairing**

Entropic Sinkhorn optimal transport was used to estimate correspondences between observed cell populations in PCA space. The resulting transport relationships were converted into source-target training pairs using a target-use cap.

**Capped-random pairing**

Source cells were paired with randomly selected observed target cells under a comparable target-use constraint.

Comparing these approaches allowed the study to investigate whether transport-informed cell correspondences improved forecasting performance relative to random pairings.

### 4. Model Training and Future-State Prediction

Models were trained using earlier observed time points and corresponding forecasting intervals.

For final evaluation, each model received held-out Day-0 cells and predicted their transcriptional representations at the terminal endpoint.

Predicted PCA representations were inverse-transformed into gene-expression space for evaluation against the observed terminal population.

The Day-7 fibroblast and Day-8 EMT populations were excluded from model training, preprocessing fitting, pairing construction, and model selection.

### 5. Model Evaluation

Forecasting performance was assessed using complementary population-level and distributional metrics:

| Metric | Purpose |
|---|---|
| Pearson r² | Measures agreement between predicted and observed mean gene-expression profiles |
| Endpoint R² | Evaluates additional population-level expression accuracy |
| 2-Wasserstein distance (W₂) | Measures distributional discrepancy between predicted and observed cell populations |
| Mean gene-expression variance | Measures predicted cell-to-cell variability |
| Mean distance from population center | Evaluates population dispersion |
| PCA-space variance preservation | Compares predicted and observed variability in PCA space |

Gene-expression accuracy was evaluated using all 2,000 selected HVGs and the top 100 most variable genes.

Distributional metrics were included because accurately predicting average gene expression does not necessarily mean reproducing the diversity of individual cell states.

## Results

### Fibroblast Senescence

All six models outperformed the Day-0 persistence baseline when forecasting the held-out Day-7 population.

Under optimal-transport pairing, **Random Forest achieved the highest population-level Pearson r²**:

| Evaluation | Pearson r² |
|---|---:|
| All 2,000 HVGs | 0.9314 |
| Top 100 HVGs | 0.8181 |

The results demonstrated that strong mean-expression accuracy did not necessarily correspond to accurate preservation of observed cell-to-cell variability.

### Epithelial-to-Mesenchymal Transition

All six architectures also outperformed the persistence baseline when forecasting the held-out Day-8 population.

Under optimal-transport pairing, XGBoost achieved the strongest All-HVG Pearson r², while the Temporal-Token Transformer achieved the strongest Top-100-HVG Pearson r².

| Evaluation | Best model | Pearson r² |
|---|---|---:|
| All 2,000 HVGs | XGBoost | 0.9737 |
| Top 100 HVGs | Temporal-Token Transformer | 0.9436 |

### Main Findings

- Models successfully forecast population-level transcriptional states beyond the observed training period in both biological systems.
- No single architecture consistently dominated across all datasets and evaluation metrics.
- More complex neural architectures did not consistently outperform simpler regression and tree-based methods.
- Optimal-transport pairing did not universally improve forecasting accuracy relative to capped-random pairing.
- Strong mean-expression predictions did not guarantee preservation of the observed population's cellular variability.

These findings highlight the importance of evaluating both expression-level accuracy and distributional similarity when forecasting future scRNA-seq populations.

## Reproducibility

The experiments use controlled random seeds, standardized training and evaluation procedures, and independent validation and testing endpoints.

The final endpoint populations were excluded from preprocessing fitting, model training, correspondence construction, and model selection to reduce the risk of data leakage.

The two Jupyter notebooks provide the primary implementations for the fibroblast and EMT forecasting experiments.

Raw datasets and intermediate experimental outputs are not included in this repository.

## Limitations and Future Work

The study was conducted using two biological systems and a limited collection of computational architectures and hyperparameter configurations.

Although the models demonstrated strong population-level forecasting performance, the results do not establish that individual predicted cells follow biologically correct trajectories.

Future work could investigate additional biological datasets, longer forecasting horizons, improved distribution-aware training objectives, and more extensive model optimization.

## Research Contributions and AI Usage

I directed the computational research workflow, selected and evaluated methodologies, designed experiments, compared model performance, interpreted results, and developed the scientific conclusions with guidance from my research mentor.

AI-assisted tools were used extensively to generate, debug, and refine the Python implementations and visualizations. I guided the implementation requirements, evaluated the outputs, and reviewed the resulting analyses.

The research decisions, experimental evaluation, and scientific interpretations were conducted under my direction.

## Research Manuscript

The experimental analyses and research manuscript are complete. The manuscript is undergoing final citation review, reference formatting, and preparation for submission.

A finalized manuscript may be added to this repository following the remaining review.

## Research Context

This project was developed through Polygence as an independent computational biology research study.

Its primary objective is to evaluate computational methods for forecasting future transcriptional states from time-course scRNA-seq data and to identify how model architecture and cell-pairing strategy influence predictive performance.
