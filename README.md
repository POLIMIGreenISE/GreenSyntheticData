# Balancing Privacy, Utility, and Sustainability: A Comparative Analysis of Synthetic Data Generation and Traditional Privacy-Preserving Techniques
This is the project of the master thesis of Rashmi Di Michino. 
It treats about the comparative analysis of two privacy preserving approaches: Synthetic Data Generation and Traditional Privacy-Presering Techniques along three fundamental criteria, Privacy, Utility, and Sustainability.

## Table of Contents
- [Installation](#installation)
- [Data Analysis](#data-analysis)
- [Synthetic Data](#synthetic-data)
- [Traditional Data](#traditional-data)
- [Comparison](#comparison)

## Installation
1. Clone the repository:
   ```bash
   git clone https://github.com/POLIMIGreenISE/GreenSyntheticData.git
   ```

2. Move into the NHANES project folder:
   ```bash
   cd NHANES_dataset
   ```

3. Create and activate a virtual environment.

   On Windows:
   ```bash
   python -m venv .venv
   .venv\Scripts\activate
   ```

   On macOS/Linux:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

4. Install the required Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```

5. Launch Jupyter Notebook:
   ```bash
   jupyter notebook
   ```

The `requirements.txt` file includes the Python dependencies required to run the notebooks in this folder:
- `data_analysis.ipynb`
- `traditional data/traditional_anonymization.ipynb`
- `synthetic data/synthetic_data_generation.ipynb`
- `comparison/comparison_NHANES.ipynb`

## Data Analysis
The Jupyter notebook Data Analysis performs data preprocessing and quality assessment on the NHANES (National Health and Nutrition Health Survey) 2013-2014 Age Prediction Subset dataset.

### Preprocessing
- Loads and cleans the dataset `NHANES_age_prediction.csv`: renames columns, maps categorical values (e.g., Gender: 1/2 → 0/1, Diabetes: ordinal to 0/1/2), handles missing values (removes one invalid Physical Activity record).
- Numerical variables: Age, BMI, Glucose Level, Oral Glucose Test, Insulin Level (no outliers removed).
- Categorical variables: Gender, age_group, Physical Activity, Diabetes.
- Saves cleaned dataset as `NHANES_1.csv`.

### Data Quality Assessment
Evaluates utility via classification on targets Diabetes (3-class/binary) and age_group (with/without Age, 3-class/binary). Uses DummyClassifier, LogisticRegression, and RandomForestClassifier with preprocessing pipelines. Metrics: Accuracy and Macro-F1.

Key results:
- Diabetes: Strong imbalance (96.5% non-diabetic); models fail to outperform dummy baseline due to limited discriminative power.
- age_group with Age: Near-perfect scores (e.g., Random Forest: 1.0 Accuracy/Macro-F1) due to deterministic relationship.
- age_group without Age: Realistic task (e.g., Logistic Regression: 0.69 Accuracy, 0.60 Macro-F1); models outperform dummy (0.84 Accuracy, 0.46 Macro-F1), with Logistic Regression showing better balance.
- Conclusion: Use binary Diabetes and exclude Age for future synthetic data generation to avoid trivial tasks. Saved this configuration as `NHANES_2.csv`

## Synthetic Data
The Jupyter notebook `synthetic_data_generation.ipynb` generates and evaluates synthetic data using the Synthetic Data Vault (SDV) library on the preprocessed NHANES_2.csv dataset.

### Generation Process
- Uses GaussianCopulaSynthesizer to model marginal distributions and covariance matrix from real data.
- Detects and validates metadata for single-table structure.
- Fits synthesizer on real data and samples an equivalent-sized synthetic dataset, saved as `synthetic_NHANES.csv`.

### Evaluation
- **SDV Built-in Metrics**: Diagnostic checks data validity and structure; quality report assesses column shapes (marginal distributions via KSComplement/TVComplement) and column pair trends (correlations/contingencies via CorrelationSimilarity/ContingencySimilarity). Results show high fidelity.
- **Custom MOSTLY AI Metrics**: 
    - **Accuracy Metrics**: Univariate accuracy for marginal distributions; bivariate accuracy for joint distributions.
    - **Similarity Metrics**: Centroid cosine similarity for mean representations; discriminator AUC using classifiers (logistic regression and random forest) to distinguish real from synthetic data.
    - **Distances Metrics**: Distance to Closest Record (DCR) to measure proximity of synthetic records to real data.

Key findings: Synthetic data preserves statistical properties well, with regularization effects on skewed distributions, balancing privacy and utility.


## Traditional Data
The notebook `traditional_anonymization.ipynb` evaluates classic record anonymization techniques on the same NHANES dataset variant used for synthetic data (`NHANES_2.csv`, binary `Diabetes_bin`, `Age` excluded).

### Setup
- quasi-identifiers: `age_group`, `Gender`, `Physical Activity`, `BMI`
- sensitive attributes: `Diabetes_bin`, `Glucose Level`, `Oral Glucose Test`, `Insulin Level`

### Privacy guarantees
- k-anonymity (`pycanon.anonymity`): initial `k=1`, then improved via BMI generalization to `k=5`, and via BMI suppression to `k=11`.
- l-diversity: computed per sensitive attribute, showing low diversity for binary `Diabetes_bin` in generalized data and better diversity after BMI suppression.
- t-closeness: measured for both releases, showing stronger closeness after BMI suppression.

### Re-identification simulation
- record linkage attack with attacker access to quasi-identifiers
- comparison of outputs:
  - generalized BMI release: smallest anonymous sets still size 5+, no unique matches
  - BMI suppressed release: larger sets, minimum 11 matches

### Utility/privacy trade-off metrics
- discernibility metric (DM) on equivalence classes
- degree of generalization (DoG)
- normalized certainty penalty (NCP)
- average equivalence class size
- result: BMI generalization gives moderate utility and weaker privacy; BMI suppression gives stronger privacy but higher information loss.

## Comparison
In the `comparison_NHANES.ipynb` notebook, the comparison between synthetic data and traditional anonymization is performed.

## Main goals
- Utility: classifier performance on `age_group` (original vs synthetic vs anonymized)
- Sustainability: energy/CO₂ footprint with CodeCarbon
- Privacy: formal privacy guarantees (k-anonymity, l-diversity, t-closeness), reidentification risk (record linkage, prosecutor risk), attribute disclosure risk

## Datasets
- `NHANES_2.csv`: preprocessed data with binary `Diabetes_bin`, no age column
- `synthetic_NHANES.csv`: generated by GaussianCopulaSynthesizer (SDV)
- `Traditionally_anon_NHANES_BMI.csv`: BMI binned
- `Traditionally_anon_NHANES_NO_BMI.csv`: BMI suppressed

## Utility evaluation
- classification task `age_group`
- model training:
  - original: train on real, test on real
  - synthetic: train on synthetic, test on real (TSTR)
  - anonymized (BMI/no BMI): consistent test set through record alignment
- models: DummyClassifier baseline, LogisticRegression (multinomial, class_weight='balanced'), RandomForest
- metrics: Accuracy, Macro-F1
- preprocessing: OneHotEncoder + StandardScaler

## Sustainability evaluation
- track with `codecarbon.OfflineEmissionsTracker`
- measure duration, energy_consumed, emissions, water, per-10k-record normalization
- compare:
  - synthetic generation
  - anonymization (BMI / NO BMI)

## Privacy evaluation
- released as-is:
  - synthetic with original QIs
  - anonymized with BMI binned
  - anonymized NO BMI
- aligned representation (synthetic with BMI binned / no BMI for direct comparison)
- metrics:
  - formal privacy guarantees (k-anonymity, l-diversity, t-closeness)
  - reidentification risk
    - record linkage attack on quasi-identifiers
    - prosecutor risk: 1/|equivalence_class|
  - attribute disclosure (maximum posterior probability, conditional entropy)

## Findings (summary)
- Utility: original highest, traditional anonymization preserves utility well, synthetic has larger drop (especially RF)
- Sustainability: synthetic generation has higher energy/emissions; anonymization is cheaper
- Privacy: traditional anonymization gives formal guarantees (k=5/11), synthetic as-is shows singleton-like risk; under aligned QIs synthetic behaves close to anonymized outcomes
