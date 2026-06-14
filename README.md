# Household Electricity Consumption Segmentation using RECS 2020

This repository contains the complete implementation for clustering U.S. households into electricity-consumption tiers using the 2020 Residential Energy Consumption Survey (RECS). The project applies unsupervised clustering, internal validation, supervised prediction, regression analysis, robustness checks, and inequality measurement to understand household electricity-use heterogeneity.

## Project Overview

Residential electricity consumption varies substantially across households due to structural, socioeconomic, and behavioral factors. This project segments households using annual electricity consumption and then evaluates whether those segments can be partially recovered from household characteristics.

The analysis supports policy-oriented interpretation by identifying Low, Moderate, High, and Very High electricity-consumption tiers.

## Research Objectives

1. Segment households into interpretable electricity-consumption groups.
2. Validate the cluster structure using internal validity metrics and statistical tests.
3. Evaluate robustness across clustering algorithms.
4. Predict consumption-tier membership using household and housing characteristics.
5. Identify important predictors of high electricity use.
6. Measure inequality in electricity consumption using the Gini coefficient.

## Dataset

The project uses the 2020 U.S. Residential Energy Consumption Survey dataset.

Primary target variable:

- `KWH`: Annual household electricity consumption in kilowatt-hours.

Clustering is performed using only `KWH` to avoid feature leakage. Predictive modeling uses non-energy household, demographic, and housing characteristics.

## Methodology

### 1. Data Preprocessing

- Load RECS 2020 household-level data.
- Clean missing and invalid values.
- Separate electricity-use variables from household explanatory variables.
- Standardize variables where required.
- Create interpretable ordered labels after clustering.

### 2. Clustering

The main segmentation is performed using K-Means clustering on annual electricity consumption.

Candidate values of `K` are evaluated using:

- Silhouette coefficient
- Calinski-Harabasz index
- Davies-Bouldin index

Although `K=2` maximizes the silhouette score, it produces an overly coarse segmentation. Solutions with `K>4` create finer partitions with limited interpretability gains. Therefore, `K=4` is selected as a compromise between statistical quality and policy interpretability.

Final tiers:

- Low consumption
- Moderate consumption
- High consumption
- Very High consumption

### 3. Statistical Validation

Cluster distinctiveness is evaluated using:

- One-way ANOVA
- Tukey HSD post-hoc tests

These tests verify whether electricity-consumption levels differ significantly across the selected tiers.

### 4. Robustness Checks

The clustering solution is compared across multiple algorithms, including:

- K-Means
- Gaussian Mixture Models
- Agglomerative Clustering

Adjusted Rand Index is used to measure agreement across clustering methods and random initializations.

### 5. Supervised Prediction

Several machine-learning classifiers are trained to predict cluster membership from non-energy household attributes.

Models include:

- Logistic Regression
- Random Forest
- XGBoost
- LightGBM
- Support Vector Machine
- k-Nearest Neighbors
- Naive Bayes

Evaluation metrics include:

- Accuracy
- Macro-F1 score
- Class-level precision, recall, and F1-score

The prediction task is leakage-aware: direct electricity-consumption variables are excluded from the supervised feature set.

### 6. Regression Analysis

OLS regression and principal component regression are used to examine the relationship between household characteristics and annual electricity consumption.

These models are used for explanatory support rather than as the main predictive contribution.

### 7. Inequality Analysis

The Gini coefficient is computed to quantify inequality in household electricity consumption.

This helps show whether a small share of households accounts for a disproportionate share of total electricity demand.
