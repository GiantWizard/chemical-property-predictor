# Chemical Property Predictor

Given a molecule's structure, this predicts seven of its physical and chemical properties. The model is CatBoost, tuned with Optuna, trained on RDKit 2D descriptors plus Morgan fingerprints.

I tried adding Mordred 3D conformer descriptors early on, but the geometry-optimization step kept hitting a multiprocessing bug, so that part got cut and the current notebook runs without it.

## Results

The numbers below come from a single end-to-end rerun of the notebook after the pipeline change described above (Mordred removal, Optuna trials cut to 15). Optuna is unseeded, so a subsequent rerun may land a few percent off these exact figures.

| Property | n | R² | MAPE | RMSE |
|---|---|---|---|---|
| Melting point (K) | 3175 | 0.647 | 0.149 | 73.4 |
| Boiling point (K) | 2922 | 0.831 | 0.098 | 106.0 |
| Heat of fusion (J/mol) | 2728 | 0.837 | 0.077 | 13642 |
| Heat of vaporization (J/mol) | 1403 | 0.734 | 0.157 | 59069 |
| Critical temperature (K) | 2846 | 0.936 | 0.048 | 68.3 |
| Critical pressure (Pa) | 2950 | 0.924 | 0.075 | 398634 |
| Flash point (K) | 267 | 0.801 | 0.060 | 29.9 |

## Pipeline

### 1. Data filtering
Inorganic compounds and salts are removed, keeping only carbon-containing organic molecules. Salts show up as a "." in their SMILES string, marking two disconnected pieces in one structure, so that pattern is used to filter them out.

### 2. Feature engineering
Two feature sets are computed for each molecule and concatenated into one table:

- **RDKit 2D descriptors**: calculated straight from the SMILES string, covering electronic, topological, and physical properties of the molecule
- **Morgan fingerprints**: 2048-bit circular fingerprints (radius 2) that encode which small substructures surround each atom

3D conformer descriptors (via Mordred) were tried at an earlier stage of this project but dropped because of a multiprocessing bug in the geometry optimization step. They are not part of the current pipeline.

### 3. Preprocessing
`VarianceThreshold` drops any feature that has the same value across every molecule, since a feature like that carries no information. `StandardScaler` then rescales what's left so every feature sits on a comparable numeric range.

### 4. Hyperparameter tuning
Optuna runs 15 trials per target property, searching over the number of boosting iterations, the learning rate, tree depth, L2 regularization strength (a penalty that keeps the model from overfitting to noise), and how much of the feature set each tree samples. Every trial is scored against a held-out validation set.

### 5. Training and evaluation
Each property's final model is retrained using the best parameters Optuna found for it. All seven properties are trained independently on their own 70/15/15 train/validation/test split, and outliers, meaning values far outside the normal range for that property, are removed using the IQR method before training begins.

## Dataset

[Physical and Chemical Properties of Organic Substances](https://www.kaggle.com/datasets/ivanyakovlevg/physical-and-chemical-properties-of-substances): 4,343 organic compounds with experimentally measured properties.

Download the CSV and place it at:
```
chemical_property_predictor/physical_chemical_properties_of_organic_substances.csv
```

## Setup

```bash
pip install catboost scikit-learn rdkit seaborn optuna
```

Recommended: run this on Google Colab or a machine with several CPU cores, since descriptor computation in Phase 1 can take a while on the full dataset.

## Usage

Open `chemical_property_predictor/chemical_property_predictor.ipynb` from within the `chemical_property_predictor/` directory.

**Phase 1: descriptor computation (run once):** Cells 1-10 compute all features and cache them to `descriptors.csv`. This is the slow step.

**Phase 2: training (run per target):** Set `property_to_predict` in cell 11, then run the remaining cells. Takes around 10-15 minutes per property with Optuna tuning.

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

**Melting point is the hardest target (R²=0.647, per the results table above).** Melting point depends heavily on crystal packing and intermolecular forces, and molecular graph descriptors don't fully capture either of those. Other papers on this kind of prediction run into the same wall, so it looks like a limitation of the approach rather than something specific to this dataset.

**The flash point dataset is very sparse.** Only about 270 valid rows survive after cleaning. The R²=0.801 result in the table above is promising, but given how small that sample is, it should be treated as a rough signal rather than a settled number.

**Structural class features are still untapped.** The dataset includes more than 20 precomputed binary flags, things like `is_aromatic`, `is_alcohol`, and `is_ketone`, that could be added as features at no extra computation cost.

**Multi-task learning could help.** The seven targets are physically correlated (boiling point and critical temperature, for instance, both reflect intermolecular forces), so a single multi-output model could exploit those correlations instead of training seven separate models. It would also be much faster to train one model than seven.

**A graph neural network baseline is worth trying.** Message-passing networks such as Chemprop operate directly on the molecular graph instead of relying on hand-crafted descriptors, and they tend to outperform descriptor-based models, especially on sparse targets like flash point.
