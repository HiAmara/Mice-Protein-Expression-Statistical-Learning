# Mice Protein Expression: Probabilistic Classification of Down Syndrome Genotype

**Statistical Learning — Master in Statistics for Data Science**

## Overview

This project builds a classification pipeline to distinguish trisomic (Down Syndrome) from control mice based on the mices protein expression profiles. Using probabilistic classifiers like **Logistic Regression**, **Gaussian Naive Bayes**, **LDA**, and **QDA** we identify which of 77 measured proteins carry the strongest biological signal for genotype classification.

Link to the dataset: [Mice Protein Expression&](https://archive.ics.uci.edu/dataset/342/mice+protein+expression)

## Methodology

**Leakage-safe preprocessing.** All imputation and scaling parameters are derived exclusively from the training set. This includes a imputation strategy that computes missing protein values from genotype × treatment subgroup medians, rather than naive global averages.

**Missing-data handling.** Features with >5% missingness are dropped entirely rather than imputed, based on the reasoning that high missingness in proteomic data likely means the data is missing for a specific reason *Missing Not at Random* (MNAR), for example the protein expression could be too small to be detectable. Imputing these with the Median would introduce bias.

**Multicollinearity control.** Highly correlated proteins (r > 0.70) are pruned using `findCorrelation`, empirically justified by QDA's numerical instability above this threshold which is directly motivated by covariance matrix invertibility requirements.

**Stability of the feature selection.** The 5-fold cross-validated ANOVA-based feature selection (`SelectKBest`) is stabilized with ranking features by how consistently they're selected across folds, not just by a single split. Which reduces selection bias. We additionally map QDA's empirical breakdown point (rank deficiency beyond ~15 features) to pin down a safe, optimal K = 9.

**Effect size over raw differences.** Beyond p-values, Cohen's *d* is used to quantify biological relevance of each protein's discriminative power.

## Exploratory Findings

- Target Classes are balances (52.8% / 47.2%), we can use accuracy as a metric
- Reasonable normality of top discriminative features (validated via Q-Q plots), supporting Gaussian model assumptions
- PCA reveals substantial overlap between classes in low dimensions (10–17 PCs needed for 80–90% variance) which justifies the use of multivariate probabilistic models over simple linear separators

<div style="display: flex; justify-content: space-around;">
  <img src="Graphs/proportion of NA" alt="Image 1" style="width: 25%;">
  <img src="Graphs/Features Removed Cutoff" alt="Image 2" style="width: 24%;">
  <img src="Graphs/Correlation Heatmap across Features" alt="Image 3" style="width: 25%;">
</div>

<div style="display: flex; justify-content: space-around;">
  <img src="Graphs/Example Proteins Density and QQ Plot" alt="Image 4" style="width: 25%;">
  <img src="Graphs/Espression Profiles of the top ten predictor proteins" alt="Image 5" style="width: 25%;">
</div>



# Second Part of the Assignment: Classification with k-NN, SVM, Random Forests and Neural Network 

## Overview

In the first part we established a baseline using Gaussian models (LDA, QDA, Naive Bayes, Logistic Regression), reaching a maximum accuracy of 88.3%. This second part extends the analysis with the additional model families **k-NN**, **SVM**, and **tree-based ensembles** as well as a **Neural Network**, and evaluates whether cost-sensitive learning improves performance in a medical screening context.

## Key Results

| Model | Accuracy | AUC | Expected Cost |
|---|---|---|---|
| QDA (Assignment 1, best baseline) | 0.883 | – | – |
| k-NN | 0.994 | 0.9997 | 0.0123 |
| **SVM (RBF, cost-weighted)** | **1.000** | **1.000** | **0.000** |
| Random Forest (Standard) | 0.991 | 0.9995 | 0.0278 |
| Neural Network | 0.978 | 0.998 | 0.0401 |

Every model introduced in this part outperformed the best previous baselines by more than 10 percentage points, with SVM achieving perfect classification on the held-out test set.

## Methodology

**Rigorous hyperparameter tuning.** k-NN and SVM were tuned via 5-fold cross-validated grid search. Rather than selecting the configuration with the highest raw accuracy which can be prone to favoring unstable, overfit models such as k=1, we applied the **One-Standard-Error Rule**. We select the simplest model whose accuracy remained within one standard error of the best result.

**Model-appropriate feature handling.** k-NN and SVM reused the ANOVA-based feature selection pipeline from Assignment 1, as distance- and kernel-based methods are sensitive to irrelevant features. Tree-based methods were excluded from this step, since they perform feature selection internally through their splitting criterion. All proteins were retained, and variable importance scores served as a data-driven feature ranking.

**Cost-sensitive learning, evaluated critically.** Given that a missed diagnosis (false negative) carries a higher clinical cost than a false alarm, a 3:1 cost penalty was tested across all models. Contrary to expectations, this did not improve outcomes: as the classes are already highly separable, increasing model caution mainly introduced additional false positives without meaningfully reducing false negatives. 

**Overfitting diagnostics.** Given the near-perfect scores obtained, additional validation steps were carried out to confirm the results reflect genuine signal rather than overfitting:
- **Learning curves** show test accuracy converging toward training accuracy as sample size increases, consistent with genuine generalization.
- **Train-test gaps** remained consistently below 2–3% across all models.
- **Cross-model agreement**: four independent model families (k-NN, SVM, Random Forest, Neural Network) each achieved accuracy above 97–99%, a pattern unlikely to arise if any single model were merely overfitting to noise.

**Biological validation of top features.** The Random Forest variable importance ranking identified **APP_N** and **ITSN1_N** as the most discriminative proteins which are both located on chromosome 21, the chromosome triplicated in Down Syndrome. This provides independent biological confirmation that the models capture genuine underlying signal rather than statistical artifacts.

## Explaining the Performance Improvement

1. **Non-linear decision boundaries.** Unlike LDA and QDA, the models introduced here can capture the complex, non-linear class structure suggested by the PCA visualization in the first part.
2. **Model-specific feature tuning.** Rather than applying a single fixed feature count across all models, the optimal number of features was tuned individually for each method.
3. **Strong underlying biological signal.** Down Syndrome results from an entire triplicated chromosome rather than a subtle mutation. Combined with tightly controlled experimental conditions (genetically identical mice, standardized measurement protocols), this produces unusually low-noise data, explaining the convergence of multiple independent methods on near-perfect accuracy.

**Limitation.** These results are based on a single controlled study comprising 324 test samples. Prior to any real-world deployment, independent validation on mice from different laboratories or colonies would be required to confirm generalizability.
