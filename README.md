# CRISPR-Pred 🧬
> Deep Learning for CRISPR-Cas9 Target Prediction

## 📌 Overview
CRISPR-Pred is a machine learning tool designed to predict CRISPR-Cas9 off-target effects and cleavage efficiency. It helps researchers analyze genomic sequences to find the most effective guide RNAs.

## 🚀 Getting Started
### Running in the Cloud
The easiest way to use this project is via Google Colab. https://colab.research.google.com/drive/1ZmTXzqCCkvNI4x2dwjKT4p9o1Di9fupg#scrollTo=0S0sptnQi-eD

### Local Installation
1. Clone the repo:
   ```bash
   git clone [https://github.com/sanjaypras/crispr_pred.git](https://github.com/sanjaypras/crispr_pred.git)


## Development Log

### Iteration 1: Model Baseline Comparison
* **Objective:** Establish baseline performance across multiple architectures.
* **Models Tested:** XGBoost, Linear Regression, and Multi-Layer Perceptron (Neural Network).
* **Key Findings:** XGBoost outperformed other models with a **Spearman Correlation of 0.57**.
* **Next Steps:** Investigate Neural Network architecture to close the performance gap.

### Iteration 2: Neural Network Optimization
* **Objective:** Hyperparameter tuning of the Neural Network to exceed baseline XGBoost performance.
* **Experiments:** Tested various optimizers (SGD, Adam, AdamW, RMSprop, Adagrad).
* **Results:** AdamW showed the most promise in convergence stability(it was consistently hitting 0.5), though performance currently remains below the 0.57 threshold.
* **Next Steps:** Explore alternative activation functions, dropout rates, and learning rate scheduling.
