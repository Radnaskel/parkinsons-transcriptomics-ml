# Machine Learning for Blood Transcriptomic Biomarkers of Parkinson’s Disease

## Overview

This project investigates whether peripheral blood transcriptomic profiles can be used to distinguish patients with Parkinson’s disease (PD) from healthy controls using machine learning.

The work consists of two complementary studies:

Within-cohort modeling (**GSE99039**) – development and evaluation of machine learning models using cross-validation.
Cross-cohort external evaluation (GSE6613) – exploratory assessment of model transferability across an independent cohort after gene-level harmonization and batch-effect correction.

---
## Key Results

* Internal validation (GSE99039): ROC-AUC ≈ 0.71
* External validation (GSE6613): ROC-AUC ≈ 0.63
* L1 Logistic Regression selected 499 probes but did not improve performance
* 13,237 shared genes were identified across cohorts
* ComBat batch correction substantially improved cross-cohort generalization
* PTGDS, CD14, ICAM1, KIR2DL1, and KIR2DL3 emerged as candidate genes

---
## Objective

To evaluate whether blood transcriptomic profiles can distinguish Parkinson’s disease patients from healthy controls, identify candidate biomarkers, and assess model generalization across independent cohorts.

---

## Datasets

### Training Cohort — GSE99039

* Source: GEO (Gene Expression Omnibus):
  https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE99039
  (GSE99039_series_matrix.txt)
* Accession: GSE99039
* Platform: Affymetrix GPL570
* Samples: 438 (205 PD, 233 controls)
* Features: ~54,000 gene expression probes

### External Validation Cohort — GSE6613

* Source: GEO (Gene Expression Omnibus):
  https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE6613
  (GSE6613_series_matrix.txt)
* Accession: GSE6613
* Platform: Affymetrix GPL96
* Samples: 72 (50 PD, 22 controls)
* Features: ~22,000 gene expression probes

---

## Methods

* Data preprocessing and cohort selection (PD vs control)
* Logistic Regression baseline
* 5-fold stratified cross-validation
* Model comparison:

  * Logistic Regression
  * Support Vector Machine (SVM)
  * Random Forest
  * L1 Logistic Regression
  * Elastic Net Logistic Regression
* External validation using an independent cohort (GSE6613)
* Gene-level harmonization across microarray platforms
* ComBat batch-effect correction
* Feature selection (ANOVA / SelectKBest)
* Model interpretation using coefficients


---
## Study 1: Within-Cohort Modeling (GSE99039)

Notebook:
`notebook_GSE99039.ipynb`

### Key Steps

* preprocessing and cohort selection (PD vs control),
* 5-fold stratified cross-validation,
* comparison of Logistic Regression, SVM, Random Forest, L1, and Elastic Net models,
* ANOVA-based feature selection,
* probe-to-gene mapping and biological interpretation of top genes.

### Results

| Model | Mean ROC-AUC | Std |
|---|---:|---:|
| SVM | 0.711 | 0.024 |
| Logistic Regression | 0.707 | 0.022 |
| L1 Logistic Regression | 0.658 | 0.037 |
| Random Forest | 0.650 | 0.042 |
| Elastic Net | 0.637 | 0.032 |

Model Performance

![ROC Curve](roc_auc.png)

---

### Candidate Genes

Top-ranked unique genes included PTGDS, LGR6, ZMYM2, RAB1A, KIR2DL1, VMO1, RUFY2, LMOD3, and CCSAP.

Top features (genes) identified by the model:

![Top Features](feature_importance.png)

These genes represent the strongest contributors to the model’s classification of PD vs control.

Key findings:

* Blood transcriptomic data contains a moderate but reproducible predictive signal
* An initial 80/20 train-test split produced ROC-AUC ≈ 0.64, while 5-fold cross-validation provided a more stable estimate of model performance.
* Linear models outperform nonlinear models in this high-dimensional setting
* ANOVA/SelectKBest with top 500 probes reduced Logistic Regression performance to ROC-AUC ≈ 0.623 ± 0.046.
* L1 Logistic Regression selected approximately 499 probes out of 54,675.
* Neither L1 regularization nor Elastic Net improved predictive performance.
* Results suggest that the Parkinson’s disease transcriptomic signal is distributed across many weak features rather than a small number of dominant genes.

---
## Study 2: External Validation (GSE6613)

Notebook:
`notebook_2_ext_val_GSE6613.ipynb`

To evaluate model generalization, an independent Parkinson’s disease cohort (GSE6613) was used for external validation.

Key steps included:

* probe-to-gene conversion using GPL570 and GPL96 annotations,
* identification of 13,237 shared genes between cohorts,
* Principal Component Analysis (PCA),
* ComBat batch-effect correction,
* ANOVA feature selection,
* model comparison across multiple algorithms.


### Summary of Model Performance

| Pipeline                                | External ROC-AUC |
| --------------------------------------- | ---------------: |
| Probe-level harmonization               |             0.46 |
| Gene-level harmonization                |             0.50 |
| Gene-level + ComBat                     |             0.58 |
| Gene-level + Feature Selection          |             0.59 |
| Gene-level + ComBat + Feature Selection |         **0.63** |


### Batch Effect Before Correction

![PCA Before](pca_before_combat.png)
PC1 = 36.7%, PC2 = 14.0%

### Batch Effect After Correction

![PCA After](pca_after_combat.png)
PC1 = 28.1%, PC2 = 7.2%

### Final External Validation ROC Curve

![External Validation ROC Curve](external_validation_roc_auc.png)

### Model Comparison

| Model                           | External ROC-AUC |
| ------------------------------- | ---------------: |
| Logistic Regression             |        **0.634** |
| Elastic Net Logistic Regression |            0.599 |
| Random Forest                   |            0.591 |
| Linear SVM                      |            0.576 |
| L1 Logistic Regression          |            0.565 |

Among the evaluated classifiers, standard Logistic Regression achieved the highest external validation performance after gene-level harmonization, ComBat correction, and feature selection.
More complex models, including Random Forest and SVM, did not improve external generalization, suggesting that the transferable disease signal is largely linear in the selected feature space.

**These results indicate that preprocessing and harmonization had a larger impact on performance than classifier choice**.


### Top Genes from the Final Logistic Regression Model

The final Logistic Regression model was trained using:

* gene-level harmonization,
* ComBat batch correction,
* ANOVA feature selection,
* top 500 selected genes.

Top-ranked genes from the final externally validated model included PPP2R1A, CD14, DNASE2, ICAM1, KIR2DL3, TMEM185B, KIR2DL1, NACC2, COTL1, CACNA1I, TRAFD1, PTGDS, CHST15, P2RX1, and HPR.

The top 15 genes ranked by absolute coefficient magnitude were:

![Top 15 Genes External Validation](top_15_genes_external_validation.png)


Key findings:

* Internal cross-validation on GSE99039 produced a mean ROC-AUC of approximately **0.70**, indicating the presence of a reproducible transcriptomic signal associated with Parkinson’s disease.
* Direct external validation initially failed, with performance close to random prediction (ROC-AUC ≈ **0.50**), highlighting substantial cross-study differences.
* Principal Component Analysis revealed a strong batch effect between cohorts.
* Application of **ComBat batch correction** substantially improved cohort alignment and increased external ROC-AUC.
* Additional feature selection further improved external validation performance.

This approach achieved an external ROC-AUC of approximately **0.63** on the independent GSE6613 cohort.

Model interpretation identified several biologically plausible candidate genes, including **PTGDS**, **CD14**, **ICAM1**, **KIR2DL1**, **KIR2DL3**, **ITGAM**, and **PTK2B**. Many of these genes are involved in immune regulation, innate immunity, and inflammatory signaling pathways.

---

## Biological Interpretation

### Literature Validation

To assess biological plausibility, candidate genes identified across both studies were compared with published literature on Parkinson’s disease, neuroinflammation, and immune regulation.

Several genes emerged as particularly noteworthy because they were either supported by existing Parkinson’s disease literature or belonged to biological pathways previously implicated in disease pathogenesis.

| Gene        | Evidence in Parkinson’s Disease                                                | Biological Relevance                          | Interpretation                                              |
| ----------- | ------------------------------------------------------------------------------ | --------------------------------------------- | ----------------------------------------------------------- |
| **PTGDS**   | Reported as differentially expressed in blood-based PD studies                 | Prostaglandin metabolism, neuroinflammation   | Strongest candidate; identified in both Study 1 and Study 2 |
| **KIR2DL1** | Limited direct PD evidence                                                     | NK-cell receptor, immune regulation           | Identified in both studies; supports immune involvement     |
| **CD14**    | Associated with immune alterations in PD patients                              | Monocyte activation, innate immunity          | Strong immune-related candidate                             |
| **ICAM1**   | Reported among inflammatory markers in PD                                      | Leukocyte adhesion and inflammatory signaling | Consistent with neuroinflammatory mechanisms                |
| **KIR2DL3** | Limited direct PD evidence                                                     | NK-cell signaling                             | Supports a role for innate immune dysregulation             |
| **P2RX1**   | Limited direct PD evidence                                                     | Purinergic signaling and inflammation         | Biologically plausible immune-related candidate             |
| **PTK2B**   | Associated with neurodegenerative and inflammatory pathways                    | Immune and neuronal signaling                 | Moderate evidence supporting relevance                      |
| **PSAP**    | Linked to lysosomal biology and neurodegeneration                              | Lysosomal function                            | Consistent with known PD-related pathways                   |
| **RAB1A**   | Previously implicated in vesicular trafficking and α-synuclein toxicity models | Intracellular trafficking                     | Biologically plausible exploratory candidate                |
| **LGR6**    | Limited PD evidence                                                            | Stem-cell and signaling pathways              | Exploratory candidate requiring validation                  |
| **ZMYM2**   | No established PD association                                                  | Transcriptional regulation                    | Exploratory candidate                                       |
| **RUFY2**   | No established PD association                                                  | Intracellular transport pathways              | Exploratory candidate                                       |


Across both studies, the strongest and most reproducible signal originated from genes associated with immune regulation and neuroinflammation rather than classical neuronal markers.

Two genes (**PTGDS** and **KIR2DL1**) emerged independently in both the discovery and external validation analyses, providing additional confidence that their identification was not driven solely by dataset-specific effects.

The external validation study further highlighted immune-related genes including **CD14**, **ICAM1**, **KIR2DL3**, **P2RX1**, and **PTK2B**, suggesting that peripheral immune activation may contribute substantially to the transcriptomic signature observed in Parkinson’s disease blood samples.

Several additional genes identified in the discovery cohort (**LGR6**, **ZMYM2**, **RUFY2**, **VMO1**, **LMOD3**, and **CCSAP**) currently lack strong literature support in Parkinson’s disease and should be considered exploratory candidates for future investigation.

---
## Limitations

* High dimensionality vs small sample size
* Moderate predictive performance (ROC-AUC ≈ 0.63–0.71)
* Limited cohort sizes
* Cross-platform differences between microarray datasets
* Potential residual batch effects
* ComBat correction was applied using both cohorts simultaneously, which may introduce transductive information from the external dataset.
* The external validation cohort (GSE6613) was also used for preprocessing strategy comparison and should therefore be considered an exploratory external validation rather than a completely untouched holdout dataset.
---

## Future Work

* Pathway and enrichment analysis
* SHAP and permutation-based interpretability methods
* Integration of additional Parkinson’s disease cohorts
* Multi-cohort training strategies
---

## References

1. Gene Expression Omnibus (GEO).
   GSE99039: Blood-based transcriptomic profiling of Parkinson’s disease patients and healthy controls.

2. Gene Expression Omnibus (GEO).
   GSE6613: Independent whole-blood Parkinson’s disease cohort used for external validation.

3. Singleton, A. B., Farrer, M. J., & Bonifati, V. (2013).
   *The genetics of Parkinson’s disease: progress and therapeutic implications.*
   The Lancet Neurology, 12(5), 500–512.

4. Makarious, M. B., Leonard, H. L., Vitale, D., et al. (2022).
   *Blood-based RNA biomarkers for Parkinson’s disease identified in discordant monozygotic twins.*
   https://pubmed.ncbi.nlm.nih.gov/36010675/

5. Hirsch, E. C., & Standaert, D. G. (2021).
   *Ten unsolved questions about neuroinflammation in Parkinson’s disease.*
   Movement Disorders, 36(1), 16–24.

6. Tansey, M. G., & Romero-Ramos, M. (2019).
   *Immune system responses in Parkinson’s disease: Early and dynamic.*
   European Journal of Neuroscience, 49(3), 364–383.

7. Vivier, E., Raulet, D. H., Moretta, A., et al. (2011).
   *Innate or adaptive immunity? The example of natural killer cells.*
   Science, 331(6013), 44–49.

8. Abel, A., & Funk, C. D. (2016).
   *Metabolism of prostaglandins and the role of prostaglandin D synthase (PTGDS) in neurological disease.*
   Journal of Lipid Research.

9. Cooper, A. A., Gitler, A. D., Cashikar, A., et al. (2006).
   *Alpha-synuclein blocks ER-Golgi traffic and Rab1 rescues neuron loss.*
   Science, 313(5785), 324–328.

10. NCBI Gene Database.
    Gene annotations used for biological interpretation.

11. UniProt Knowledgebase (UniProtKB).
    Functional annotations of candidate genes.

12. Affymetrix GPL570 and GPL96 platform annotation files.
    Used for probe-to-gene mapping and cross-platform harmonization.

---

## Acknowledgements

This project was developed with the assistance of AI tools, including ChatGPT, which was used to support code structuring, debugging, and explanation of machine learning concepts.

All modeling decisions, analysis, and interpretations were performed by the author.
