# Variable Selection Analysis on Indonesian Hoax News Dataset

---

## 📌 Project Overview & Objective

High-dimensional text data (Bag-of-Words / TF-IDF representations) typically suffers from the **curse of dimensionality** ($p \gg n$), where the number of vocabulary features far exceeds the number of observations. This can lead to model overfitting, high computational complexity, and poor interpretability.

The **primary objective** of this project is to perform variable selection and regularized classification on an Indonesian hoax news dataset to:
1. Identify key linguistic tokens/words that differentiate **hoax** news from **valid** news.
2. Evaluate and compare high-dimensional feature selection and shrinkage methods:
   * **Adaptive Best Subset Selection / Forward Selection (`abess`)**
   * **LASSO Regression ($\alpha = 1$)**
   * **Ridge Regression ($\alpha = 0$)**
3. Select an optimal model balancing high classification performance (AUC) with model parsimony (interpretability).

---

## 📊 Dataset Description & EDA

* **File:** `hoax_news_indonesia_postprocess.csv`
* **Dimensions:** 500 rows (observations) $\times$ 6,685 columns (features)
* **Target Variable:** `news_category`
  * `hoax` (50%, 250 samples)
  * `valid` (50%, 250 samples)
* **Predictor Variables:** 6,684 continuous Bag-of-Words (BOW) token frequency features.
* **Data Integrity:** 100% complete rows with zero missing values ($0.000\%$ missing observations).

---

## 🔬 Methodology & Analysis

### 1. Adaptive Best Subset Selection (Forward Selection via `abess`)
* **Objective:** Find the optimal subset size ($k \in [0, 100]$) using 10-fold Cross-Validation (CV) and Bayesian Information Criterion (BIC).
* **Cross-Validation Result:** Minimum CV error achieved at low support sizes ($k \approx 2$).
* **BIC Result:** BIC reached its optimal minimum around support size $k = 22-25$, demonstrating strong penalization against complex models.

### 2. Regularized Logistic Regression (`glmnetUtils`)

#### A. LASSO Penalty ($\alpha = 1$)
* Performs both **shrinkage** and **feature selection** by setting redundant feature coefficients strictly to zero.
* Fitted using 10-fold cross-validation with **Deviance** and **AUC** metrics.
* **Optimal Hyperparameters ($\lambda.1se$):**
  * $\lambda = 0.0253$
  * **Selected Features (`nzero`):** 146 active word tokens out of 6,684.
  * **Mean AUC:** **0.902**

#### B. Ridge Penalty ($\alpha = 0$)
* Performs **shrinkage only** without feature elimination.
* **Optimal Hyperparameters ($\lambda.1se$):**
  * $\lambda = 81.1$
  * **Selected Features (`nzero`):** 6,684 (all features retained).
  * **Mean AUC:** **0.937**

---

## 📈 Model Performance & Comparison

| Model | Penalty ($\alpha$) | Best $\lambda$ ($\text{1SE}$) | Mean AUC | Selected Features (`nzero`) | Model Sparsity / Interpretability |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **LASSO** | $1.0$ | $0.0253$ | **0.902** | **146** | **High** (97.8% dimensionality reduction) |
| **Ridge** | $0.0$ | $81.1000$ | **0.937** | **6,684** | **Low** (Keeps all features) |

### Sample Prediction Verification
Both models were tested on a sample of 6 unseen data points (3 hoax, 3 valid news articles):
* **LASSO Prediction:** Correctly classified all 6 instances (Hoax probabilities: `0.743`, `0.993`, `0.831`; Valid probabilities: `0.277`, `0.165`, `0.277`).
* **Ridge Prediction:** Correctly classified all 6 instances (Hoax probabilities: `0.902`, `0.885`, `0.793`; Valid probabilities: `0.249`, `0.304`, `0.246`).

---

## 💡 Key Conclusions

1. **Trade-off Between Accuracy and Interpretability:**
   * **Ridge Regression** yields a slightly higher overall predictive AUC (**0.937** vs. **0.902**), but requires retaining all 6,684 word tokens, making it computationally heavy and uninterpretable for qualitative analysis.
   * **LASSO Regression** achieves near-equivalent predictive performance (**AUC = 0.902**) while eliminating over **97.8% of irrelevant features**, reducing the feature space down to just **146 critical indicator words**.

2. **Practical Recommendation:**
   * For **real-time production deployment and qualitative linguistic analysis**, **LASSO** is the recommended algorithm due to its extreme model parsimony, low memory footprint, and ability to extract clear diagnostic keywords for hoax detection.

---

## 📦 Required Packages

```R
library(tidyverse)
library(vroom)
library(skimr)
library(DataExplorer)
library(ggwordcloud)
library(ggpubr)
library(abess)
library(glmnetUtils)
library(broom)
library(tictoc)
