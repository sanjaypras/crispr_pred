# CRISPR_Pred - Machine Learning Tool Designed to Predict CRISPR-Cas9 Off-Target Effects and Cleavage Efficiency.

---

### **Problem statement**
The goal of this project is to successfully predict the outcome of a CRISPR-Cas9 cut, a tool that cuts and fixes DNA. The model uses information like what gene is being targeted in the cut and at what location in the gene the cut is happening, and tries to use that to predict the success of the cut. The performance metric that the Azimuth page gave us was to use spearman correlation, and the target Spearman score was 0.6.

### **Model / approach**
For regression, we tried to predict a cut's `Percent Rank` based on their features (e.g., `Percent Peptide`, `Strand`, `Gene Symbol`, etc.) under two distinct feature selection strategies:
* **Method 1 (Fewer Features):** Our baseline feature set focusing primarily on core sequence and genetic features to establish initial model performance.
* **Method 2 (More Features):** An expanded feature set introduced to see if adding high-dimensional metadata and positional context (`Transcript`, `Annotation`, `Nucleotide cut position`, `Amino Acid Cut position`) would help the models capture more complex biological relationships and improve predictive power.

We evaluated both methods across our top-performing architectures:
* **Linear Regression:** Used to see if the relationship was simple and linear, and as a baseline for more complex models. 
* **XGBoost:** Used for its skill in handling relationships between categorical features and sequence data. 
* **Neural Network (NN):** Used to see if model could find any niche high-dimensional relationships between features.

### **Data**
**Dataset:** `V1_suppl_data.txt` (attached) from [https://github.com/MicrosoftResearch/Azimuth]. This dataset has a list of a 30 character sequence of DNA, along with some other statistics, and how well the cut worked. 

**Features:**
* `Spacer Sequence` *(Dropped in preprocessing)*
* `Extended Spacer(NNNN[20nt]NGGNNNNNNN)` — **[Both Methods]**
* `Strand` — **[Both Methods]**: either 'sense' or 'antisense', represents which side of the DNA helix the cut was made.
* `Transcript` — **[Method 2 Only]**: an identifier for a specific mRNA isoform that accounts for isoform-specific expression levels and physical interference from RNA polymerase during Cas9 binding.
* `Gene Symbol` — **[Both Methods]**: Gene that the cut was targeting.
* `Nucleotide cut position` — **[Method 2 Only]**: Index of the cut in nucleotides.
* `Amino Acid Cut position` — **[Method 2 Only]**: Index of the cut in amino acids.
* `Percent Peptide` — **[Both Methods]**: Percent of gene being cut.
* `Annotation` — **[Method 2 Only]**: labels a specific DNA sequence with its biological function.
* `Activity` — **[Both Methods]**: Activity remaining in the sample after cut, a lot of activity is good, and means the cut preserved the life of the sample.
* `Percent Rank` — **[Both Methods] [Target Variable]**: Activity ranked from 0-1, higher the better.

**Preprocessing:** To prepare the data, we dropped `Spacer Sequence` (we already had a longer version of this feature). Also, since we had a ~30 character sequence of DNA, we one-hot encoded each character so we had ~120 different binary columns representing each DNA sequence. We also one-hot encoded `Transcript` & `Annotation` for the models utilizing Method 2 features. Train set was 80%, and test set was 20% of the data.

### **Training setup**
The hyperparameters and architecture layouts were held constant across both feature sets to ensure a fair comparison:

* **XGBoost (Method 1 & Method 2):** `n_estimators=1000, learning_rate=0.05, max_depth=6, subsample=0.8, random_state=42`
* **Neural Network (Method 1 - Fewer Features):** `256, 128, 64, 16 arrangement of neurons, l2 regularizer(0.01), dropout of 0.3, BatchNorm, relu activation` using an Adam optimizer.
* **Neural Network (Method 2 - More Features):** `256, 64, 32, 16 arrangement of neurons, l2 regularizer(1e-4), dropout of 0.3, BatchNorm, relu activation` using an Adam optimizer.

### **Results**
| Model Framework | Feature Setup | Spearman Score |
| :--- | :--- | :--- |
| Linear Regression | **Control** (All Features) | 0.44 |
| Neural Network | **Method 1** (Fewer Features) | 0.55 |
| Neural Network | **Method 2** (More Features) | 0.51 |
| XGBoost | **Method 1** (Fewer Features) | 0.57 |
| **XGBoost** | **Method 2** (More Features) | **0.68** |

We next wanted to look at feature importance. We used SHAP analysis for this, and we used it on the XGBoost model since that had the best performance. The most important feature was `Percent Peptide`, which we could tell by looking at the beeswarm plot, and after that, none of the others were standout, but they all contributed more or less equally. Next, we looked at the waterfall plots for individual examples. `Percent Peptide` was consistently in the top 3, but otherwise, nothing really stood out.

### **What worked / what didn't**
With the XGBoost model achieving a Spearman correlation of 0.682 using Method 2 features, exceeding the benchmark of 0.60, the model was overall very successful. Linear Regression showed the data had a weak to moderate correlation, meaning there was definitely room to improve. XGBoost was a lot better passed the 0.6 threshold. The Neural Network was a little bit of a letdown since it decreased from XGB. The better performance of XGBoost over the Neural Network (0.68 vs 0.51 in the expanded feature set) shows that the dataset is better for decision trees. The NN probably struggled to find a clear correlation with only ~2,000 samples.

Comparing our feature strategies, we initially built Method 1 with fewer features to keep the input clean. We then introduced Method 2 to see if adding high-cardinality features like `Transcript` and `Annotation` would improve performance. This expansion proved highly successful for the XGBoost model, allowing it to jump from a 0.57 to a 0.68, successfully beating the target threshold of 0.60 set by the Azimuth documentation. 

However, adding these features was not as effective for the Neural Network, where performance actually dropped from 0.55 to 0.51. In a relatively small dataset of about 2,000 samples, drastically increasing the dimensionality of the input space caused the neural network to overfit to noise rather than learning the actual biological patterns. Pruning those extra features back down (Method 1) allowed the simpler network to generalize better. Also, the fact that this dataset was tabular made it better suited to a discrete model like XGBoost when handling the complex feature interactions of Method 2.

The clean nature of the dataset allowed for a smooth pipeline and there were no outliers in the data or in the prediction phases, which was very helpful for the data preparation. 

**Overall thoughts:** `Percent Peptide` is the most important feature; it showed for XGB how much it helps predict `Percent Rank` through feature importance, shown in the below image. Biologically, this makes sense because cuts made near the very beginning (low percent) or very end (high percent) of a gene might result in a partially functional protein or leave crucial domains intact. Cuts in the middle are more effective at completely knocking out the gene. The fact that we succeeded in passing the 0.6 mark using Method 2 with XGBoost was great, but I feel like we should be able to get the NN to hit 0.6. 

![SHAP Beeswarm Plot](SHAP%20Beeswarm%20on%20XGBoost%20\(CRISPR_pred\).png)

### **Next steps**
* Tune the NN more, maybe changing the optimizer, to hit that 0.6 that the XGB achieved.
