# EGFR Mutation Prediction Using DenseNet201 and ABMIL

This research project investigates the prediction of EGFR mutation status (Mutant or Wild Type) in Lung Adenocarcinoma (TCGA-LUAD) using Hematoxylin and Eosin (H&E)-stained Whole Slide Images (WSIs).

The proposed approach combines **DenseNet201**, used as a frozen feature extractor, and **Attention-Based Multiple Instance Learning (ABMIL)**, used to aggregate patch-level features into slide-level representations for binary classification.

> **Disclaimer:** This project is intended for academic research and documentation purposes only. It is not a diagnostic tool and must not be used for clinical decision-making.

## Repository Contents

| File | Description |
|---|---|
| `TAHAP_PRAPROSES.ipynb` | WSI acquisition and selection from GDC, dataset preparation, tissue detection, patch extraction, quality filtering, and Macenko stain normalization. |
| `FINALL_DENSENET201+ABMIL.ipynb` | DenseNet201 feature extraction, cross-validation, ABMIL training, hold-out evaluation, and supplementary analyses. |

## Dataset

The dataset consists of **200 diagnostic H&E WSIs** from the TCGA-LUAD cohort, comprising:

- 50 EGFR Mutant cases (25%).
- 150 EGFR Wild Type cases (75%).
- One diagnostic WSI per patient.

WSI data were obtained from the [GDC Data Portal](https://portal.gdc.cancer.gov/), while EGFR mutation labels were obtained from cBioPortal (TCGA PanCancer Atlas 2018).

Only eligible diagnostic slides scanned at native 40× magnification were included. When multiple slides were available for a patient, one slide was selected, prioritizing the larger file.

The dataset was split at the patient level using stratified hold-out sampling (80:20), with random seed 42:

| Dataset | Mutant | Wild Type | Total |
|---|---:|---:|---:|
| Development set | 40 | 120 | 160 |
| Hold-out test set | 10 | 30 | 40 |
| **Total** | **50** | **150** | **200** |

The hold-out test set was kept separate from model development and used only for final evaluation.

## Methodology

The proposed pipeline consists of the following stages:

1. **WSI Preprocessing**
   - Tissue detection using image thresholding and morphological operations.
   - Patch extraction at native 40× magnification.
   - Patch size of 224 × 224 pixels with a stride of 224 pixels.
   - Tissue and blur-based quality filtering.
   - Macenko stain normalization.

2. **Feature Extraction**
   - DenseNet201 pretrained on ImageNet is used as a frozen feature extractor.
   - Each patch is represented by a 1,920-dimensional feature embedding.
   - Patch embeddings from each WSI are grouped into a slide-level feature bag.

3. **Model Development**
   - ABMIL aggregates patch-level features using attention-based weighting.
   - Stratified 5-fold cross-validation is performed on the 160-case development set.
   - Binary Cross-Entropy (BCE), Class-Weighted BCE, and Focal Loss are compared to address class imbalance.
   - ABMIL is compared with mean-pooling and max-pooling.

4. **Final Evaluation**
   - The final model is trained using the selected configuration on the complete development set.
   - Evaluation is performed once on the 40-case hold-out test set.
   - Classification metrics include AUROC, accuracy, precision, recall, F1-score, and confusion matrix.
   - A 95% bootstrap confidence interval is calculated for hold-out AUROC using 1,000 resampling iterations.

## Main Experimental Results

The following results represent the main evaluation of the DenseNet201-ABMIL model.

### 5-Fold Cross-Validation

| Aggregation Method | Mean AUROC ± SD |
|---|---:|
| ABMIL | **0.827 ± 0.089** |
| Max-pooling | 0.785 ± 0.087 |
| Mean-pooling | 0.703 ± 0.042 |

ABMIL achieved the highest mean AUROC among the three aggregation methods. Its improvement over mean-pooling was consistent across all five folds, while its advantage over max-pooling was less conclusive.

### Hold-out Test Evaluation

The final model was evaluated on 40 WSIs (10 Mutant and 30 Wild Type), using a classification threshold of 0.5.

| Metric | Result |
|---|---:|
| AUROC | **0.713** |
| AUROC 95% Bootstrap CI | 0.496–0.911 |
| Accuracy | 75.00% |
| Precision | 50.00% |
| Recall (Sensitivity) | 60.00% |
| Specificity | 80.00% |
| F1-score | 0.545 |
| Average Precision | 0.532 |

The confusion matrix produced 6 true positives, 4 false negatives, 6 false positives, and 24 true negatives.

The hold-out results indicate that the model can distinguish EGFR Mutant and Wild Type cases to some extent. However, the confidence interval is wide because the test set contains only 10 Mutant cases. Therefore, the results should be interpreted as preliminary research findings rather than evidence of clinical performance.

## Computational Environment

The experiments were conducted using the following environments:

- **WSI preprocessing:** Shared research server with parallel processing.
- **Feature extraction:** Google Colab with NVIDIA Tesla T4 GPU.
- **Model training and evaluation:** Google Colab with NVIDIA A100-SXM4-80GB GPU.
- **Implementation:** Python, PyTorch, OpenSlide, and scikit-learn.

## How to Run

### 1. Clone the Repository

```bash
git clone https://github.com/Natasya47/EGFR-Mutation-Prediction-DenseNet201-ABMIL.git
cd EGFR-Mutation-Prediction-DenseNet201-ABMIL
```

### 2. WSI Preprocessing

Open `TAHAP_PRAPROSES.ipynb` in a suitable Python or Google Colab environment.

Install the required dependencies:

```bash
pip install notebook numpy pandas openpyxl requests opencv-python-headless openslide-python scikit-image matplotlib
```

OpenSlide system libraries must also be installed where required.

Prepare the EGFR label file and the WSI data according to the notebook configuration. Run the preprocessing stages sequentially, then store the normalized patch outputs in Google Drive.

### 3. Feature Extraction and Model Training

Open `FINALL_DENSENET201+ABMIL.ipynb` using Google Colab with GPU acceleration enabled.

Install additional dependencies if necessary:

```python
!pip install timm statsmodels huggingface_hub
```

Configure `DATA_ROOT` in the notebook to point to the preprocessed feature or patch directory required by the pipeline.

Run the notebook cells in sequence. Keep the hold-out confirmation setting disabled during cross-validation and enable it only when performing the final hold-out evaluation.

### 4. Supplementary Analyses

Additional experiments are provided in the notebook for exploratory evaluation. These analyses are supplementary and are not used to select or modify the primary model after hold-out evaluation.

Some supplementary experiments, such as UNI feature extraction, require Hugging Face access and acceptance of the corresponding model license.

## Research Limitations

- The dataset is relatively small and has an imbalanced class distribution.
- The hold-out test set contains only 10 EGFR Mutant cases, resulting in substantial uncertainty in performance estimates.
- The evaluation is based on TCGA-LUAD data and has not been independently validated on an external cohort.
- Attention weights indicate the relative contribution of patches to model predictions and do not establish biological or pathological significance.
- The model is intended for research purposes and is not validated for clinical diagnosis.

## Research Outputs

The complete experimental outputs, logs, and results are available in the following Google Drive folder:

[Access Research Results – Google Drive](https://drive.google.com/drive/folders/1GCs4Temrh8msLWRrR-2YODpAjYisohRs?usp=sharing)

## Author

**Natasya Haidar Romadhoni**  
Information Systems Study Program  
Universitas Trunojoyo Madura, Indonesia.
