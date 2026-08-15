# 🇮🇩 Indonesian Hoax News Classification

A high-dimensional text classification project that investigates whether **feature selection and regularization** can identify useful linguistic signals for distinguishing Indonesian **hoax** and **valid** news.

The project compares:

* Adaptive Best Subset Selection (`abess`)
* LASSO Logistic Regression
* Ridge Logistic Regression

The main objective is to understand the trade-off between **predictive performance and model interpretability**.

## 🎯 Problem

Text classification often creates a high-dimensional feature space.

When news articles are represented using Bag-of-Words features, the number of vocabulary variables can become much larger than the number of observations.

This project addresses the following question:

> **Can we classify Indonesian hoax news accurately while reducing thousands of text features to a much smaller and more interpretable subset?**

The analysis focuses on three objectives:

1. Identify informative linguistic features associated with hoax and valid news.
2. Compare feature-selection and regularization methods.
3. Evaluate the trade-off between predictive performance and model sparsity.

## 📊 Dataset

The project uses:

`hoax_news_indonesia_postprocess.csv`

The dataset contains **500 observations and 6,685 columns**: one binary target variable and **6,684 Bag-of-Words features**.

### Target Variable

| Variable        | Description                |
| --------------- | -------------------------- |
| `news_category` | News classification target |

The target contains two classes:

| Class   | Observations | Share |
| ------- | -----------: | ----: |
| `hoax`  |          250 |   50% |
| `valid` |          250 |   50% |

The dataset is therefore balanced between the two classes.

### Predictor Variables

The predictors are **6,684 continuous Bag-of-Words token-frequency features**.

Each feature represents the frequency of a particular token in the news corpus.

### Data Quality

* **500 observations**
* **6,684 text features**
* **0% reported missing observations**
* Balanced binary target

## 🔬 Method

The analysis follows a high-dimensional binary classification workflow.

### 1. Exploratory Data Analysis

The dataset is first inspected for:

* Dimensions
* Variable structure
* Missing values
* Class distribution
* Token frequency patterns

The project also visualizes the distribution of the target variable and generates word clouds to explore differences in token usage between hoax and valid news.

### 2. Adaptive Best Subset Selection

The `abess` package is used to perform **adaptive best subset selection** for a binomial classification problem.

Support sizes from:

```text
0 → 100 variables
```

are evaluated using:

* 10-fold cross-validation
* Bayesian Information Criterion (BIC)

This approach searches for a sparse subset of features that can explain the classification outcome.

### 3. LASSO Logistic Regression

LASSO uses an L1 penalty:

```text
α = 1
```

Unlike ordinary logistic regression, LASSO can shrink some coefficients exactly to zero.

This makes it particularly useful for:

* Feature selection
* Dimensionality reduction
* Sparse modeling
* Interpretability

Two 10-fold cross-validation procedures are performed using:

* Deviance
* AUC

The `lambda.1se` solution is used for the reported sparse model.

### 4. Ridge Logistic Regression

Ridge uses an L2 penalty:

```text
α = 0
```

Ridge shrinks coefficients toward zero but generally retains all predictors.

It is therefore useful as a comparison against LASSO:

> **Does retaining all text features produce better predictive performance than aggressively selecting features?**

The model is tuned using 10-fold cross-validation with deviance and AUC metrics.

### 5. Model Comparison

The final comparison focuses on:

* Mean AUC
* Number of retained features
* Model sparsity
* Interpretability

This allows predictive performance to be evaluated alongside model complexity.

## 📈 Results

### Model Comparison

| Model     | Penalty |  Mean AUC | Selected Features | Feature Reduction |
| --------- | ------- | --------: | ----------------: | ----------------: |
| **LASSO** | L1      | **0.902** |           **146** |         **97.8%** |
| **Ridge** | L2      | **0.937** |             6,684 |                0% |

The reported cross-validation results show that **Ridge achieves the highest AUC**, while **LASSO produces a much sparser model**.

### LASSO

LASSO retains:

**146 of 6,684 features**

This represents approximately:

**97.8% dimensionality reduction.**

Its mean AUC is:

**0.902**

### Ridge

Ridge retains:

**6,684 of 6,684 features**

with a mean AUC of:

**0.937**.

### The Accuracy–Interpretability Trade-off

The results demonstrate a clear trade-off:

```text
Predictive Performance
        ↑
        │                     ● Ridge
        │
        │
        │          ● LASSO
        │
        └────────────────────────────→
             Model Complexity

        LASSO                  Ridge
       146 vars             6,684 vars
       AUC 0.902             AUC 0.937
```

Ridge improves AUC by approximately **0.035**, but requires retaining the entire 6,684-feature vocabulary.

LASSO sacrifices some predictive performance in exchange for a dramatically smaller and more interpretable feature space.

## 📊 Visualization

### 1. Class Distribution

![News Category Distribution](unnamed-chunk-3-1.png)

The dataset contains an equal number of hoax and valid news observations.

### 2. Word Frequency Visualization

![Word Cloud](unnamed-chunk-3-2.png)

The word-cloud analysis provides a qualitative overview of frequently occurring tokens across the two news categories.

### 3. Feature Selection

![Forward Selection CV](unnamed-chunk-4-1.png)

The `abess` cross-validation curve shows how model performance changes as additional features are introduced.

### 4. BIC-Based Selection

![BIC Feature Selection](unnamed-chunk-4-2.png)

BIC provides a model-complexity-aware criterion for selecting a parsimonious feature subset.

### 5. LASSO Coefficient Paths

![LASSO Coefficient Paths](unnamed-chunk-5-1.png)

The coefficient paths demonstrate how LASSO progressively shrinks feature coefficients as the regularization strength changes.

### 6. Ridge Coefficient Paths

![Ridge Coefficient Paths](unnamed-chunk-6-1.png)

The Ridge coefficient paths illustrate shrinkage without aggressive feature elimination.

## 💡 Conclusion

This project demonstrates that high-dimensional Indonesian text classification can be approached through **feature selection and regularized logistic regression**.

### Key Takeaways

**1. The text representation is extremely high-dimensional.**

The dataset contains 6,684 predictors for only 500 observations, creating a classic high-dimensional modeling problem.

**2. Ridge achieves the highest reported AUC.**

With a mean AUC of **0.937**, Ridge provides the strongest predictive performance among the two regularized models evaluated.

**3. LASSO provides substantially greater interpretability.**

LASSO reduces the feature space from 6,684 variables to just **146**, while retaining a relatively strong mean AUC of **0.902**.

**4. Model selection depends on the objective.**

If maximizing predictive performance is the primary objective, Ridge is preferable in this experiment.

If interpretability, computational efficiency, and identifying a small set of informative tokens are more important, LASSO provides a compelling alternative.

### Overall Recommendation

> **LASSO offers the strongest balance between predictive performance and interpretability when a sparse linguistic representation is required.**

However, Ridge remains the better-performing model according to the reported cross-validation AUC.

## ⚠️ Limitations

Several limitations should be considered.

### High-Dimensional, Small-Sample Setting

The dataset contains far more predictors than observations:

```text
p = 6,684
n = 500
```

This makes model selection sensitive to the specific sample and preprocessing pipeline.

### Bag-of-Words Representation

Bag-of-Words features capture token frequency but do not fully represent:

* Word order
* Context
* Semantics
* Sarcasm
* Negation
* Long-range linguistic relationships

### Model Evaluation

The reported AUC values come from 10-fold cross-validation, which is appropriate for model tuning but does not provide a completely independent external validation set.

### Generalization

The results should not be interpreted as evidence that these models will perform equally well on future Indonesian news from different publishers or time periods.

### Future Improvements

Potential extensions include:

* Stratified train/test evaluation
* Nested cross-validation
* Elastic Net
* Linear SVM
* Naive Bayes
* XGBoost
* Indonesian stemming and linguistic preprocessing
* TF-IDF instead of raw BOW frequencies
* n-gram features
* Word embeddings
* IndoBERT
* External temporal validation
* Model calibration
* Precision-recall analysis
* SHAP-based interpretation

## 🛠️ Technologies

* **R**
* **R Markdown**
* **Tidyverse**
* **vroom**
* **DataExplorer**
* **ggplot2**
* **ggpubr**
* **ggwordcloud**
* **abess**
* **glmnet / glmnetUtils**
* **broom**
* **10-fold Cross-Validation**
* **Logistic Regression**
* **LASSO**
* **Ridge Regression**
* **Feature Selection**
* **Natural Language Processing**

### Methods

`High-Dimensional Modeling` `Feature Selection` `LASSO` `Ridge` `Logistic Regression` `Regularization` `Text Classification` `NLP` `Cross-Validation`

## 📁 Repository Structure

```text
indonesian-hoax-news-classification/
│
├── data/
│   └── hoax_news_indonesia_postprocess.csv
│
├── figures/
│   ├── class-distribution.png
│   ├── word-frequency.png
│   ├── forward-selection-cv.png
│   ├── forward-selection-bic.png
│   ├── lasso-coefficient-path.png
│   └── ridge-coefficient-path.png
│
├── indonesian-hoax-news-classification.Rmd
├── indonesian-hoax-news-classification.md
└── README.md
```

## 📌 Topics

`R` `RMarkdown` `NLP` `Text Classification` `Hoax Detection` `Feature Selection` `LASSO` `Ridge` `Logistic Regression` `Regularization` `Machine Learning` `Indonesian NLP` `Data Science`
