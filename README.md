# Chemical Property Predictor

Predicts seven physical and chemical properties of organic molecules directly from molecular structure. Uses gradient-boosted trees (CatBoost) with Optuna hyperparameter tuning and a feature set combining RDKit 2D descriptors, Mordred 3D conformer descriptors, and Morgan fingerprints.

## Results

| Property | n | R² | MAPE | RMSE |
|---|---|---|---|---|
| Melting point (K) | 3175 | 0.674 | 0.142 | 70.6 |
| Boiling point (K) | 2922 | 0.827 | 0.100 | 107.3 |
| Heat of fusion (J/mol) | 2728 | 0.852 | 0.072 | 13009 |
| Heat of vaporization (J/mol) | 1403 | 0.755 | 0.143 | 56752 |
| Critical temperature (K) | 2846 | 0.939 | 0.045 | 66.6 |
| Critical pressure (Pa) | 2950 | 0.920 | 0.079 | 410057 |
| Flash point (K) | 267 | 0.843 | 0.055 | 26.6 |

## Pipeline

### 1. Data filtering
Inorganic compounds and salts (dot-notation SMILES) are removed, keeping only carbon-containing organic molecules.

### 2. Feature engineering
Three complementary feature sets are computed and concatenated (~2,400+ total features):

- **RDKit 2D descriptors**: computed directly from SMILES: electronic, topological, and physicochemical properties
- **Mordred 3D descriptors**: molecules are embedded as 3D conformers using MMFF/UFF geometry optimization, then 3D shape and spatial descriptors are computed
- **Morgan fingerprints**: 2048-bit circular fingerprints (radius 2) encoding local substructure patterns

### 3. Preprocessing
`VarianceThreshold` removes zero-variance features. `StandardScaler` then normalizes the remaining features.

### 4. Hyperparameter tuning
Optuna runs 50 trials per target property, searching over iterations, learning rate, tree depth, L2 regularization, and feature subsampling. Trials are evaluated on a held-out validation set.

### 5. Training and evaluation
The final model is then trained with best Optuna params. Each of the seven properties is trained independently on a 70/15/15 train/val/test split. IQR-based outlier removal applied per target before training.

## Dataset

[Physical and Chemical Properties of Organic Substances](https://www.kaggle.com/datasets/ivanyakovlevg/physical-and-chemical-properties-of-substances) — 4,343 organic compounds with experimentally measured properties.

Download the CSV and place it at:
```
chemical_property_predictor/physical_chemical_properties_of_organic_substances.csv
```

## Setup

```bash
pip install catboost scikit-learn rdkit shap mordredcommunity seaborn deepchem optuna
```

Recommended: run on Google Colab or a machine with multiple CPU cores. The 3D conformer generation step (Phase 1) takes about 20–30 minutes on Colab.

## Usage

Open `chemical_property_predictor/chemical_property_predictor.ipynb` from within the `chemical_property_predictor/` directory.

**Phase 1: descriptor computation (run once):** Cells 1–10 compute all features and cache them to `descriptors.csv`. This is the expensive step.

**Phase 2: training (run per target):** Set `property_to_predict` in cell 11, then run the remaining cells. Takes around 10–15 minutes per property with Optuna tuning.

```python
property_to_predict = 'boiling_point_K'  # swap to train a different target
```

Available targets: `melting_point_K`, `boiling_point_K`, `heat_of_fusion`, `heat_of_vaporization`, `critical_temperature`, `critical_pressure`, `flash_point`

## Output

Each run produces:
- Printed metrics: RMSE, MAPE, MAE, R²
- Actual vs predicted scatter plot
- Error analysis table sorted by worst-predicted molecules
- Feature importance ranking

## Notes and future improvements

**Melting point is the hardest target (R²=0.63).** Melting point depends heavily on crystal packing and intermolecular forces that molecular graph descriptors don't fully encode. This is apparently a known challenge across literature.

**Flash point dataset is very sparse.** Only ~270 valid rows after cleaning. While promising, the R²=0.81 result should be treated cautiously given the relatively small sample size.

**Structural class features are untapped.** The dataset includes 20+ precomputed binary flags (`is_aromatic`, `is_alcohol`, `is_ketone`, etc.) that could be added as features with no extra computation cost.

**Multi-task learning could help.** The seven targets are physically correlated (e.g., boiling point and critical temperature both reflect intermolecular forces). A multi-output model could exploit these correlations instead of training seven independent models. This can help significantly with speed.

**Graph neural network baseline.** Message-passing networks such as Chemprop operate directly on the molecular graph without hand-crafted descriptors and tend to outperform descriptor-based models, especially on sparse targets like flash point.
