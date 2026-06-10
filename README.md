# CRISPR_Pred - Machine Learning Tool Designed to Predict CRISPR-Cas9 Off-Target Effects and Cleavage Efficiency.

---

### **Problem statement**
The goal of this project is to successfully predict the outcome of a CRISPR-Cas9 cut, a tool that cuts and fixes DNA. The model uses information like what gene is being targeted in the cut and at what location in the gene the cut is happening, and tries to use that to predict the success of the cut. The performance metric that the Azimuth page gave us was to use spearman correlation, and the target Spearman score was 0.6.

### **Data**
**Dataset:** `V1_suppl_data.txt` (attached) from [https://github.com/MicrosoftResearch/Azimuth]. This dataset has a list of a 30 character sequence of DNA, along with some other statistics, and how well the cut worked. 

**Features:**
* `Spacer Sequence`, `Extended Spacer(NNNN[20nt]NGGNNNNNNN)`, `Strand`, `Transcript`, `Gene Symbol`, `Nucleotide cut position`, `Amino Acid Cut position`, `Percent Peptide`, `Annotation`, `Activity`, `Percent Rank`
* **Strand:** either 'sense' or 'antisense', represents which side of the DNA helix the cut was made.
* **Transcript:** an identifier for a specific mRNA isoform that accounts for isoform-specific expression levels and physical interference from RNA polymerase during Cas9 binding.
* **Gene Symbol:** Gene that the cut was targeting.
* **Nucleotide cut position:** Index of the cut in nucleotides.
* **Amino acid cut position:** Index of the cut in amino acids.
* **Percent Peptide:** Percent of gene being cut.
* **Annotation:** labels a specific DNA sequence with its biological function.
* **Activity:** Activity remaining in the sample after cut, a lot of activity is good, and means the cut preserved the life of the sample.
* **Percent Rank[target variable]:** Activity ranked from 0-1, higher the better.

**Preprocessing:** To prepare the data, we dropped `Spacer Sequence` (we already had a longer version of this feature), `Transcript` & `Annotation` (dropped to focus on sequence-specific predictors) and `Nucleotide cut position` & `Amino Acid Cut position` (already explained in `Percent Peptide`). Also, since we had a ~30 character sequence of DNA, we one-hot encoded each character so we had ~120 different binary columns representing each DNA sequence. Train set was 80%, and test set was 20% of the data.

### **Model / approach**
For regression, we tried to predict a cut's `Percent Rank` based on their features (e.g., `Percent Peptide`, `Strand`, `Gene Symbol`, etc.)

* **Linear Regression:** Used to see if the relationship was simple and linear, and as a baseline for more complex models. 
* **XGBoost:** Used for its skill in handling relationships between categorical features and sequence data. 
* **Neural Network (NN):** Used to see if model could find any niche high-dimensional relationships between features.

### **Training setup**
* **XGBoost:** `n_estimators=1000, learning_rate=0.05, max_depth=6, subsample=0.8, random_state=42`
* **Neural Network:** `256, 128, 64, 16 arrangement of neurons, l2 regularizer(0.01), dropout of 0.3, BatchNorm, relu activation` using an Adam optimizer.

### **Results**
| Model | Spearman Score |
| :--- | :--- |
| Linear Regression | ~0.42 |
| Neural Network | 0.55 |
| **XGBoost** | **0.575** |

We next wanted to look at feature importance. We used SHAP analysis for this, and we used it on the XGBoost model since that had the best performance. The most important feature was `Percent Peptide`, which we could tell by looking at the beeswarm plot, and after that, none of the others were standout, but they all contributed more or less equally. Next, we looked at the waterfall plots for individual examples. `Percent Peptide` was consistently in the top 3, but otherwise, nothing really stood out.

### **What worked / what didn't**
With the XGBoost model achieving a Spearman correlation of 0.575, closely approaching the benchmark of 0.60, the model was overall successful. Linear Regression showed the data had a weak to moderate correlation, meaning there was definitely room to improve. XGBoost was much better and almost at the 0.6 threshold. The Neural Network was a little bit of a letdown since it decreased from XGB. The better performance of XGBoost over the Neural Network (0.575 vs 0.55) shows that the dataset is better for decision trees. The NN probably struggled to find a clear correlation with only ~2,000 samples. The clean nature of the dataset allowed for a smooth pipeline and there were no outliers in the data or in the prediction phases, which was very helpful for the data preparation. 

**Overall thoughts:** `Percent Peptide` is the most important feature; it showed for XGB how much it helps predict `Percent Rank` through feature importance. I felt like there was still a way to hit 0.6, as it had been done before, we need to try a different model, or maybe tweak the XGB a little bit more. 

### **Next steps**
* To hopefully increase the Spearman metric, we could possibly include the other columns.
* Tweak the XGB a little bit more to improve the model and hit the 0.6 threshold.
