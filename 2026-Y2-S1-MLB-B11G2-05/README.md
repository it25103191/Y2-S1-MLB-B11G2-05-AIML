# 2026-Y2-S1-MLB-B11G2-05 — UCI Letter Recognition (IT2011 Group Project)

**Module:** IT2011 — Artificial Intelligence and Machine Learning
**Group ID:** `2026-Y2-S1-MLB-B11G2-05`
**Phases covered:** Progress Review I (Data Preprocessing & EDA) and Final Evaluation (Model Implementation)

---

## 1. Project Overview

Our chosen problem is **Optical Character Recognition (OCR)**: classifying a scanned character's
numeric descriptors into one of the 26 uppercase English letters.

The project runs as one connected workflow:

```
data/raw/letter-recognition.data                      (original UCI dataset)
        │
        ▼  notebooks/group_pipeline.ipynb              (Progress Review I — six preprocessing stages)
        │
data/external/processed_letter_recognition.csv       (written by the pipeline)
        │
        ▼  notebooks_phase2/<member model>.ipynb       (Final Evaluation — reads that same file)
        │
results/model_metrics/*.json                          (each model's tuned results)
        │
        ▼  notebooks_phase2/Group_ComparisonOfAllModels.ipynb
        │
results/group_final_comparison.csv                    (group comparison of all six models)
```

Because both notebook folders sit directly inside `2026-Y2-S1-MLB-B11G2-05/`, the pipeline and every model use the
**same** `data/` folder: the file the pipeline writes is exactly the file the models read.

## 2. Dataset Details

- **Name:** UCI *Letter Recognition* Data Set
- **Source:** UCI Machine Learning Repository (donated by David J. Slate, Odesta Corporation, 1991)
- **Instances:** 20,000 raw (18,668 after Stage 1 duplicate removal)
- **Attributes:** 16 numeric predictors (statistical moments and edge counts, integer-scaled 0–15)
  + 1 categorical target (`lettr`, 26 classes, A–Z)
- **Missing values:** None — verified directly with `isnull().sum()` in
  `notebooks/IT25100513_Nirushiya_DuplicateDetection.ipynb`, not just assumed from the UCI docs.
- **Duplicate records:** **1,332 exact duplicate rows (6.66%)** were found and removed. Left in
  place, a duplicate can land in both the train and test split, letting a model be partly "tested"
  on data it already memorised and inflating reported accuracy.
- **Class balance:** Near-uniform — every letter has between ~730 and ~815 examples, and this
  balance is preserved after deduplication.

Raw data files are stored under `data/raw/`:
- `letter-recognition.data` — the dataset itself (comma-separated, no header row)
- `letter-recognition.names` — the original UCI documentation describing every attribute

## 3. Team Roles

| Member | Index No. | Preprocessing technique (Progress Review I) | Model (Final Evaluation) |
|---|---|---|---|
| Nirushiya A. | IT25100513 | Duplicate record detection & removal | Support Vector Machine |
| Thrishodaran V. | IT25102227 | Outlier removal | Decision Tree |
| Wijayagunasekara L.A.V.D.S. | IT25100064 | Encoding categorical variables | Logistic Regression |
| Almeida K.K.O. | IT25103191 | Normalization / scaling | K-Means Clustering |
| Danishiyan G.J. | IT25101436 | Feature engineering | Principal Component Analysis (PCA) → KNN |
| Muhunthan R. | IT25102618 | Feature selection & dimension reduction | Multi-Layer Perceptron |

## 4. Repository Structure

```
2026-Y2-S1-MLB-B11G2-05/
├── README.md
├── IT2011_Final_Evaluation_Report.docx          # Final Evaluation report
├── data/
│   ├── raw/                                     # Original, unmodified dataset files
│   │   ├── letter-recognition.data
│   │   └── letter-recognition.names
│   └── external/                                # Written by group_pipeline.ipynb, read by every model
│       └── processed_letter_recognition.csv
├── notebooks/                                   # PROGRESS REVIEW I — preprocessing
│   ├── IT25100513_Nirushiya_DuplicateDetection.ipynb
│   ├── IT25102227_Thrishodaran_OutlierRemoval.ipynb
│   ├── IT25100064_Wijayagunasekara_EncodingCategorical.ipynb
│   ├── IT25103191_Almeida_NormalizationScaling.ipynb
│   ├── IT25101436_Danishiyan_FeatureEngineering.ipynb
│   ├── IT25102618_Muhunthan_FeatureSelection.ipynb
│   └── group_pipeline.ipynb                     # Combined pipeline (group deliverable)
├── notebooks_phase2/                            # FINAL EVALUATION — models
│   ├── IT25100513_Nirushiya_SVMClassification.ipynb
│   ├── IT25102227_Thrishodaran_DecisionTreeClassification.ipynb
│   ├── IT25100064_Wijayagunasekara_LogisticRegressionClassification.ipynb
│   ├── IT25103191_Almeida_KMeansClustering.ipynb
│   ├── IT25101436_Danishiyan_PCADimensionalityReduction.ipynb
│   ├── IT25102618_Muhunthan_DeepLearningMLP.ipynb
│   └── Group_ComparisonOfAllModels.ipynb        # Group deliverable
└── results/
    ├── eda_visualizations/                      # PNG figures saved by every notebook (31)
    ├── logs/pipeline_log.txt                    # Stage-by-stage log from group_pipeline.ipynb
    ├── outputs/                                 # Intermediate CSV saved by Nirushiya's notebook
    ├── model_metrics/*.json                     # Each member's tuned model results
    └── group_final_comparison.csv               # Final group comparison table
```

## 5. How to Run

1. **Set up an environment** (Python 3.10+ recommended):
   ```bash
   pip install pandas numpy matplotlib seaborn scikit-learn scipy jupyter
   ```
2. **Launch Jupyter** from the `2026-Y2-S1-MLB-B11G2-05/` folder:
   ```bash
   jupyter notebook
   ```
3. **Run in this order.** Every notebook uses paths relative to its own folder
   (`../data/...`, `../results/...`), so always open notebooks from inside `notebooks/` or
   `notebooks_phase2/`.

   | Step | Notebook | Reads | Writes |
   |---|---|---|---|
   | 1 | `notebooks/group_pipeline.ipynb` | `data/raw/letter-recognition.data` | `data/external/processed_letter_recognition.csv`, `results/logs/pipeline_log.txt` |
   | 2 | Any model in `notebooks_phase2/` | `data/external/processed_letter_recognition.csv` | `results/model_metrics/<model>_results.json` |
   | 3 | `notebooks_phase2/Group_ComparisonOfAllModels.ipynb` | `results/model_metrics/*.json` | `results/group_final_comparison.csv` |

   The six individual preprocessing notebooks in `notebooks/` each load the raw data themselves and
   can be run in any order; they are not required before `group_pipeline.ipynb`.

   Approximate run times: the pipeline, PCA, Decision Tree and K-Means notebooks take seconds; the
   Logistic Regression, SVM and MLP notebooks take roughly 2–3 minutes each because of their grid
   searches.

## 6. Pipeline Order & Rationale

`group_pipeline.ipynb` combines all six preprocessing techniques in this order:

1. **Duplicate record detection & removal** (Nirushiya A.) — 20,000 → 18,668 rows, so every later
   stage and every model's train/test split works on a leak-free set of rows.
2. **Outlier capping** (Thrishodaran V.) — IQR-based capping (k = 1.5); values are clipped, no rows
   deleted.
3. **Feature engineering** (Danishiyan G.J.) — five shape/density features (`box_area`,
   `aspect_ratio`, `pixel_density`, `edge_ratio`, `total_variance`), 16 → 21 numeric columns.
4. **Encoding the categorical target** (Wijayagunasekara L.A.V.D.S.) — `lettr` (A–Z) → 0–25.
5. **Scaling / normalization** (Almeida K.K.O.) — `StandardScaler` on all 21 numeric columns.
6. **Feature selection & dimension reduction** (Muhunthan R.) — ANOVA F-ranking and PCA analysis on
   the fully-prepared, scaled feature matrix.

## 7. Final Evaluation Results

| Model | Member | Type | Result |
|---|---|---|---|
| Support Vector Machine (RBF) | Nirushiya A. | Classification | 96.46% accuracy |
| Multi-Layer Perceptron | Muhunthan R. | Deep Learning | 94.67% accuracy |
| Decision Tree | Thrishodaran V. | Classification | 86.05% accuracy |
| Logistic Regression | Wijayagunasekara L.A.V.D.S. | Classification | 77.32% accuracy |
| K-Means | Almeida K.K.O. | Clustering | k = 26, ARI 0.151, NMI 0.364 |
| PCA (13 components) → KNN | Danishiyan G.J. | Dimensionality Reduction + Classification (KNN) | 13/21 components, 95.18% variance retained, 38.1% dimensionality reduction; PCA → KNN: 90.33% accuracy, 90.37% macro-F1 |

All models use the same processed dataset (18,668 rows, 21 standardised features). The four
classifiers share an identical 80/20 stratified split with `random_state=42`.

## 8. Marking Rubric Coverage

- **Progress Review I — individual notebooks (20 marks each):** each explains its technique,
  justifies it against this dataset, implements it with executed code and output, and includes at
  least one EDA visualization with a written interpretation.
- **Progress Review I — group pipeline (5 marks):** `group_pipeline.ipynb` chains all six
  techniques with per-stage comments, printed checkpoints, a saved output and a log.
- **Final Evaluation — individual models (45 marks each):** each model notebook covers suitability,
  implementation, tuning, multiple varieties, evaluation metrics, validation, comparison and
  limitations.
- **Final Evaluation — group comparison (10 marks):** `Group_ComparisonOfAllModels.ipynb` compares
  the optimum result of all six models and discusses expected versus observed behaviour.

## 9. Data-Quality Correction (applied after the first submission)

Three of the five engineered features added in Stage 3 — `aspect_ratio`, `pixel_density` and
`edge_ratio` — are ratios, and a small number of rows have a denominator of exactly zero
(e.g. `high == 0`). The original implementation guarded this by adding a tiny epsilon before
dividing, which does **not** solve the problem: it converts a division by zero into a division by a
near-zero number, producing values in the hundreds of thousands. Those extreme values then distorted
the standardisation in Stage 5 and the principal components in Stage 6.

Each ratio now floors its denominator at 1 instead, which bounds every value to a sensible range
(verified in the pipeline log: max `aspect_ratio` 5.00, `pixel_density` 1.00, `edge_ratio` 7.00).

| | Before (epsilon) | After (floor at 1) |
|---|---|---|
| PCA components for 95% variance (Stage 6) | 14 | 13 |
| Top-ranked feature by ANOVA F-score | x2ybr (1085.1) | edge_ratio (1798.5) |

`edge_ratio`'s usefulness had been masked by the defect: the extreme values inflated its
within-group variance, which is exactly what an ANOVA F-test penalises. Once corrected, it becomes
the most discriminative feature in the whole set. All Final Evaluation results were produced on the
corrected dataset.

## 10. Note on Preprocessing Order

The shared pipeline fits its outlier-capping bounds and `StandardScaler` on all rows before the
train/test split. The PCA notebook's Appendix measures the effect by repeating the experiment with
both fitted on training rows only: KNN accuracy on PCA features moves from 90.33% to 90.36%, with
the component count and variance retained unchanged. The effect is negligible, and the reported
results stand; splitting first is recommended for future work.
