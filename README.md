# Data-Driven Framework for Segmentation, Prediction, and Inequality Analysis of U.S. Residential Electricity Use

This repository contains the code for our research on how electricity use differs across U.S. households. Using data from the **2020 Residential Energy Consumption Survey (RECS)**, we group households by annual electricity consumption, explore how well household characteristics can predict those groups, and examine differences in electricity use across the sample.

The work was presented at **IEEE ISTAS 2026**.

## Project Overview

Not every household uses electricity in the same way or at the same level. Home size, building characteristics, location, and socioeconomic factors can all be related to how much electricity a household consumes.

We begin by dividing households into four groups—**Low, Moderate, High, and Very High**—based only on their annual electricity use. We then test whether demographic and housing information can help predict which group a household belongs to. Finally, we use regression and inequality measures to look beyond the four groups and better understand variation in consumption.

## Research Objectives

1. Group households into meaningful electricity-consumption tiers.
2. Check how well the clusters are separated using validation metrics and statistical tests.
3. Test whether the clustering results remain consistent across different methods and random initializations.
4. Predict consumption tiers using household and housing information, without using electricity consumption as a predictor.
5. Explore which household characteristics are most useful for predicting consumption tiers.
6. Examine continuous electricity consumption and measure how unevenly it is distributed across households.

## Dataset

We use the **2020 U.S. Residential Energy Consumption Survey (RECS)** from the U.S. Energy Information Administration (EIA). After preprocessing, the analysis includes **18,496 households**.

The main variable used to form the consumption groups is:

- `KWH` — annual household electricity consumption, measured in kilowatt-hours.

For the prediction stage, we use ten household and housing variables: `HHAGE`, `EMPLOYHH`, `MONEYPY`, `REGIONC`, `TYPEHUQ`, `TOTSQFT_EN`, `YEARMADERANGE`, `WALLTYPE`, `ROOFTYPE`, and `ADQINSUL`.

The code expects a prepared dataset named `recs_cleaned_for_analysis.csv` by default. This is the analysis-ready file, not necessarily the original filename provided by EIA. You can change the local file location using `RECS_DATA_PATH`.

**An important distinction:** We use `KWH` to create the consumption tiers, but we leave it out of the supervised prediction inputs. Otherwise, the models would be given the same information used to define the labels they are trying to predict.

## Methodology

### 1. Data Preprocessing

We start with the prepared RECS household data and check for missing or invalid values. We keep annual electricity consumption separate from the household characteristics used later for prediction.

The main preprocessing steps are:

- Load the cleaned household-level CSV and handle missing or invalid records as required by the analysis.
- Standardize `KWH` before applying K-Means clustering.
- For the classification models, fill missing numeric values (`HHAGE` and `TOTSQFT_EN`) using the median and scale these variables.
- Fill missing categorical values using the most frequent category and convert categorical variables into model-ready features through one-hot encoding.
- Name the final clusters in ascending order of their average electricity consumption.

### 2. Clustering

We use **K-Means** to group households based on standardized annual electricity consumption (`KWH`) alone.

To choose the number of clusters, we compare solutions from **K = 2 to K = 8** using three measures:

- Silhouette coefficient
- Calinski–Harabasz index
- Davies–Bouldin index

The two-cluster solution gives the highest silhouette score, but it combines households into groups that are too broad for our purpose. We choose **four clusters** because they provide a useful balance between separation and interpretability.

| Consumption tier | Households | Average annual electricity use (kWh) |
| --- | ---: | ---: |
| Low | 7,991 | 5,301 |
| Moderate | 6,750 | 11,409 |
| High | 3,149 | 19,396 |
| Very High | 606 | 33,346 |

The four-cluster K-Means solution has a **silhouette score of 0.5535**.

### 3. Statistical Validation

After forming the four groups, we examine how their electricity-consumption levels differ.

We use **one-way ANOVA** and **Tukey HSD** pairwise comparisons on `log(1 + KWH)`. These tests describe the differences between the consumption tiers. Because the tiers were created from electricity consumption itself, these results should be understood as a check on the resulting groups, not as independent proof that the groups exist.

### 4. Robustness Checks

We also compare the K-Means groups with clusters produced by other methods:

- K-Medoids
- Agglomerative clustering (Ward linkage)
- Gaussian Mixture Model (GMM)

We use the **Adjusted Rand Index (ARI)** to measure how closely the different methods agree with the K-Means labels.

| Clustering method | ARI compared with K-Means |
| --- | ---: |
| K-Medoids | 0.496 |
| Agglomerative (Ward) | 0.590 |
| GMM | 0.834 |

We also repeat K-Means with different initializations to check its stability. The **mean pairwise ARI is 0.944**, with a standard deviation of **0.040**.

### 5. Supervised Prediction

Once the four consumption tiers are defined, we ask a different question: **Can we predict a household's tier from its demographic and housing characteristics, without knowing its electricity consumption?**

We use a stratified **70/30 training/test split** and compare six classification models:

1. Logistic Regression
2. Support Vector Machine (SVM, RBF kernel)
3. Decision Tree
4. Random Forest
5. XGBoost
6. LightGBM

We evaluate the models using accuracy, macro-precision, macro-recall, macro-F1, Matthews correlation coefficient (MCC), and macro one-vs-rest ROC–AUC. We also examine performance by class and review a confusion matrix.

| Model | Test accuracy | Macro-F1 |
| --- | ---: | ---: |
| SVM (RBF) | 0.4799 | **0.4012** |
| LightGBM | 0.4851 | 0.3966 |
| XGBoost | **0.5516** | 0.3959 |
| Random Forest | 0.5313 | 0.3837 |
| Logistic Regression | 0.4592 | 0.3810 |
| Decision Tree | 0.4493 | 0.3543 |

**XGBoost** has the highest overall test accuracy (**55.16%**). **SVM** has the highest macro-F1 (**0.4012**), which gives equal importance to each consumption tier when averaging performance.

We use permutation importance, scored with macro-F1, to explore which predictors contribute most to the best macro-F1 model. Geographic region and dwelling floor area stand out as particularly informative variables in the reported analysis.

### 6. Regression Analysis

Classification tells us how well we can predict the four consumption groups, but it does not directly model the amount of electricity a household uses. To explore that relationship, we also study **continuous annual electricity consumption** on a log-transformed scale.

The regression analysis includes:

- **Ordinary Least Squares (OLS)** regression, with variance inflation factor (VIF) checks for multicollinearity.
- **Principal Component Regression (PCR)**, using principal components that explain 95% of the variation in the predictors.

The reported PCR results are **R² = 0.321**, **MAE = 0.420**, and **RMSE = 0.552**, evaluated on the log-consumption scale. These results provide an additional view of the relationship between household characteristics and electricity use; they are not classification accuracy scores.

### 7. Inequality Analysis

Finally, we calculate the **unweighted Gini coefficient** to understand how unevenly annual electricity consumption is distributed across households.

The overall Gini coefficient is **0.335**. Within the four consumption tiers, the Gini values are lower, ranging from approximately **0.085 to 0.205**.

A Gini coefficient closer to zero indicates a more even distribution, while a higher value indicates greater concentration of consumption. Here, the measure describes **electricity-use inequality**, not income inequality or energy affordability.

Because these calculations are **unweighted**, they describe the households in our analytical sample. They should not be interpreted as survey-weighted estimates for all U.S. households.
