# Quark vs Gluon Jet Analysis — Lab and Rest Frames

**Author:** Avantika Pandey  
**Program:** Google Summer of Code 2026 — ML4SCI HEPSIM  
**Notebook:** `ML4SCI_HEPSIM_GSoC2026_AvantikaPandey.ipynb`

---

## Overview

This project analyzes **500,000 jets** from the Pythia8 Quark–Gluon dataset to investigate structural differences between quark- and gluon-initiated jets and to evaluate the effectiveness of rest-frame representations for jet classification.

Consistent with QCD predictions, gluon jets exhibit higher constituent multiplicities and broader radiation patterns, while quark jets tend to have more collimated energy flow. Using a compact set of physically motivated observables computed in the rest frame, a Logistic Regression classifier achieves an **AUC of 0.848**, with constituent multiplicity identified as the most discriminating feature.

---

## Physics Background

Jets are collimated sprays of hadrons produced when energetic quarks or gluons undergo parton showering and hadronization. In Quantum Chromodynamics (QCD), gluons carry a larger color factor (C_A = 3) compared to quarks (C_F = 4/3), leading to:

- **Higher particle multiplicities** in gluon jets
- **Broader angular radiation** patterns for gluons
- **More collimated energy flow** in quark jets

Quark–gluon discrimination is an important task in high-energy physics — improved separation can enhance signal sensitivity and suppress backgrounds in many analyses.

---

## Dataset

**Pythia8 Quark and Gluon Jets Dataset**  
Source: [Zenodo Record 3164691](https://zenodo.org/records/3164691)

### Setup

Download up to 5 files named `QG_jets_*.npz` and place them in a `data/` folder:

```
data/
├── QG_jets_0.npz
├── QG_jets_1.npz
├── ...
```

### Structure

Each `.npz` file contains:

| Key | Description |
|-----|-------------|
| `X` | Jet constituents of shape `(N_jets, N_particles, 4)` |
| `y` | Jet label — `0 = gluon`, `1 = quark` |

Each constituent has 4 features:

| Index | Feature | Description |
|-------|---------|-------------|
| 0 | p_T | Transverse momentum |
| 1 | y | Rapidity |
| 2 | φ | Azimuthal angle |
| 3 | pdgid | Particle ID |

> Jets are zero-padded to a fixed maximum multiplicity. Constituents with p_T = 0 are padding and are excluded from all calculations.

**Dataset summary:**
- Total jets: 500,000
- Quark jets: 50%
- Gluon jets: 50%

---

## Environment & Dependencies

- Python 3.13
- NumPy
- Matplotlib
- scikit-learn

Install dependencies:

```bash
pip install numpy matplotlib scikit-learn
```

---

## Notebook Structure

### 1. Data Loading & Exploration
- Load all `.npz` files using `glob`
- Mask zero-padded constituents using `pT > 0`
- Verify dataset balance and inspect constituent structure

### 2. Exploratory Analysis
- **Particle Multiplicity Distribution** — quark vs gluon jets; gluon jets consistently show higher constituent counts
- **Leading Constituent Kinematics** — pT and rapidity of the highest-pT particle per jet

### 3. Jet Observable Computation

Three substructure observables are computed for each jet:

#### (i) Jet Mass
Invariant mass of the jet computed from its constituent four-momenta. Gluon jets tend to have higher jet mass due to broader radiation.

#### (ii) Jet Width
pT-weighted mean angular distance of constituents from the jet axis:

$$w = \frac{\sum_i p_{T,i} \cdot \Delta R_i}{\sum_i p_{T,i}}$$

Gluon jets show a broader width distribution.

#### (iii) pT Dispersion (p_T^D)
Measures how evenly transverse momentum is shared among constituents:

$$p_T^D = \frac{\sqrt{\sum_i p_{T,i}^2}}{\sum_i p_{T,i}}$$

Quark jets (fewer, harder fragments) tend to have larger p_T^D; gluon jets (many softer particles) tend to have smaller values.

### 4. Rest-Frame Transformation

Each jet is boosted to its **center-of-mass (rest) frame** to isolate intrinsic geometric structure independent of the overall jet motion:
- Constituent 4-momenta are constructed from (pT, y, φ)
- The total jet 4-momentum is computed
- All constituents are Lorentz-boosted using the jet's β vector
- Rest-frame features (mass, width, pT dispersion, multiplicity) are extracted

This removes boost-related correlations and allows the classifier to focus on intrinsic radiation patterns.

### 5. Feature Extraction

Four rest-frame features are extracted per jet:

| Feature | Physical Meaning |
|---------|-----------------|
| Jet mass | Total radiation scale |
| Jet width | Angular spread of energy |
| pT dispersion | Fragmentation hardness |
| Multiplicity | Number of radiation particles |

### 6. Machine Learning Classification

- **Model:** Logistic Regression (scikit-learn)
- **Split:** 80% train / 20% test
- **Preprocessing:** StandardScaler normalization
- **Evaluation:** ROC curve, AUC, Confusion Matrix, feature coefficients

---

## Results

### Classification Performance

| Metric | Value |
|--------|-------|
| AUC | **0.848** |
| Model | Logistic Regression |
| Most discriminating feature | **Multiplicity** |

### Feature Importances (Logistic Regression Coefficients)

| Feature | Coefficient |
|---------|-------------|
| Jet mass | +0.219 |
| Jet width | +0.107 |
| pT dispersion | +0.603 |
| Multiplicity | **−1.392** |

The negative coefficient for multiplicity reflects that jets with more constituents are more likely to be gluon jets — consistent with QCD expectations.

### Confusion Matrix (threshold = 0.5)

|  | Predicted Gluon | Predicted Quark |
|---|---|---|
| **True Gluon** | 57,902 | 17,098 |
| **True Quark** | 16,623 | 58,377 |

---

## Key Findings

1. **Multiplicity is the strongest discriminator** — particle count carries the most separation power between quark and gluon jets, consistent with QCD color factor differences.

2. **pT dispersion provides complementary power** — reflecting differences in fragmentation hardness between the two jet types.

3. **Rest-frame analysis offers physical interpretability** — boosting to the center-of-mass frame removes kinematic correlations and highlights intrinsic geometric jet structure. Performance is comparable to lab-frame features, suggesting that most separation power originates from intrinsic radiation patterns rather than overall jet kinematics.

4. **A compact feature set is sufficient** — just four physically motivated observables achieve an AUC of 0.848 with a simple linear classifier.

---

## Usage

1. Clone or download this repository
2. Download the dataset from [Zenodo 3164691](https://zenodo.org/records/3164691) and place files in `data/`
3. Open the notebook:
   ```bash
   jupyter notebook ML4SCI_HEPSIM_GSoC2026_AvantikaPandey.ipynb
   ```
4. Run all cells in order

---

## References

- Komiske, P., Metodiev, E., & Thaler, J. (2019). *Pythia8 Quark and Gluon Jets* [Dataset]. Zenodo. https://zenodo.org/records/3164691
- Peskin, M. E., & Schroeder, D. V. (1995). *An Introduction to Quantum Field Theory*. Addison-Wesley.
- Larkoski, A. J., Moult, I., & Nachman, B. (2017). *Jet Substructure at the Large Hadron Collider*. Physics Reports.

---

## License

This project was developed as part of the **Google Summer of Code 2026** application for the **ML4SCI HEPSIM** organization.
