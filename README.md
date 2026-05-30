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


## log

### v1:
started, xgb, linear reg, neural netwrok
xgb was the best - spearman corr of 0.57
need to tune nn

### v2
tried to tune nn
tried diff optimizers, adamw is getting there but still cant hit 0.57
try again later to see what i can tune
