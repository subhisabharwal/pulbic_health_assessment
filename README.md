# pulbic_health_assessment
# 🏥 Assessment of Public Satisfaction with Healthcare Across Indian States


[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![PyMC](https://img.shields.io/badge/PyMC-5.x-orange.svg)](https://www.pymc.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

##  Overview

This project analyses public satisfaction with healthcare services across Indian states using data from the **Citizens Survey 2022–23** (Harvard Dataverse, `doi:10.7910/DVN/M1DFBO`), a nationally representative sample of nearly **50,000 households**.

A core challenge is the large disparity in state-level sample sizes — some states have thousands of observations while others have fewer than 120 — which makes simple mean comparisons unreliable. We address this by combining:

- **Classical statistical analysis** (ANOVA, standard errors, group comparisons)
- **Linear Mixed-Effects Modelling** (partial pooling via LMM)
- **Bayesian Hierarchical Modelling** (using PyMC for robust, shrinkage-adjusted estimates)


##  Research Questions

- How do satisfaction scores differ across Indian states after accounting for sampling uncertainty?
- Does health insurance coverage systematically modify satisfaction levels, and does this vary by region?
- What is the advantage of Bayesian shrinkage for data-sparse states compared to classical methods?
- Are there identifiable clusters of states with similar satisfaction and utilisation profiles?
- Which dimensions of healthcare experience (waiting time, cost, staff behaviour, etc.) drive overall satisfaction?
- Does rural vs. urban residence influence satisfaction differently?

---

##  Repository Structure

```
.
├── classical approach.ipynb           # Main analysis notebook (preprocessing + EDA)
├── inference using LMM.ipynb          # Hierarchical Bayesian model notebook (fixed & cleaned)
├── converted_file.csv                 # Processed dataset (from Harvard Dataverse)
├── infernece using baesian appoach    # Full project report
├── README.md

```

---

##  Dataset

**Source:** Citizens Survey 2022–23 — Harvard Dataverse
**Link:** [doi:10.7910/DVN/M1DFBO](https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/M1DFBO)

| Category | Key Variables |
|---|---|
| Demographics | Age, gender, education, occupation, religion, social category |
| Geographic | State, district, rural/urban |
| Health Status | Physical health, mental health |
| Outpatient Care | Facility type, waiting time, cost, doctor behaviour |
| Inpatient Care | Hospital type, duration, expenditure, staff behaviour |
| Insurance | Status, type, premium, coverage adequacy |
| Schemes | Ayushman Bharat enrolment and utilisation |
| Outcomes | Satisfaction (OP, IP), UHC index score |

---

##  Data Preprocessing

1. **Initial inspection** — shape, dtypes, descriptive statistics
2. **Missing value assessment** — ranked columns by missingness
3. **Drop "Others/Specify" columns** — removed sparse open-text fields (`'oth'` in column name)
4. **Threshold-based column filtering** — dropped columns with >80% missing values
5. **Feature selection** — curated subset across demographics, insurance, satisfaction, cost, and access domains
6. **Satisfaction encoding** — Yes→1, No→0, CNS/DNA→NaN; `b6v` encoded on a 3-point scale (0, 0.5, 1)
7. **Composite satisfaction score** — row-wise mean across all satisfaction variables (`overall_sat`)
8. **Insurance re-encoding** — binary: Have Insurance→1, No→0
9. **Provider type derivation** — Public vs. Private based on `b1_most`
10. **Log transformation** — applied `log1p` to out-of-pocket expenditure to reduce skew
11. **Geographic region assignment** — North, South, Northeast, Central/Other
12. **State-level aggregation** — for clustering analysis
13. **Feature scaling** — `StandardScaler` applied before K-Means clustering

---

##  Methods

### Classical Approach
- State-wise mean satisfaction with standard errors
- One-way **ANOVA** across states: `F(28, 41774) = 127.74, p < 0.001`
- Subgroup comparisons by region, insurance status, gender, age, and rural/urban

### Linear Mixed-Effects Model (LMM)
- Fixed effects: insurance status, residence, region
- Random intercept per state for partial pooling
- Estimated via maximum likelihood

### Bayesian Hierarchical Logistic Model
- Binary outcome: satisfied (1) vs. not satisfied (0)
- State-level random effects with Half-Normal hyperprior on variance
- Fixed effects: insurance, provider type, rural/urban, cost, wait time
- Weakly informative priors: `β ~ N(0, 2)`, `α ~ N(0, 5)`, `σ_state ~ HalfNormal(0, 1)`
- MCMC sampling via **PyMC** (1000 samples, 1000 tuning steps, `target_accept=0.9`)
- Diagnostics: trace plots, posterior distributions, posterior predictive checks (ArviZ)

---

##  Key Findings

- **Overall satisfaction is high** across states (0.85–0.97 on classical estimates), but tightly clustered
- **ANOVA confirms significant state-level differences** though between-state variation is modest relative to within-state variation
- **Bayesian shrinkage** pulls extreme estimates (especially from small states) toward the global mean, improving reliability without erasing genuine variation
- **North region** shows the highest Bayesian-adjusted satisfaction; **South and Central/Other** fall below the overall mean
- **Insurance status** shows no strong observable effect on satisfaction in classical comparisons
- **Rural respondents** show slightly more concentrated high satisfaction vs. urban respondents
- **Cluster analysis** identifies 3 state clusters: low utilisation/high satisfaction (majority), high utilisation/high satisfaction (Punjab, Haryana), and moderate utilisation/variable satisfaction (Sikkim, Mizoram)

---

## ⚙️ Installation & Setup

```bash
# Clone the repository
git clone https://github.com/subhisabharwal/pulbic_health_assessment.git
cd pulbic_health_assessment

# Create a virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Requirements

```
pandas
numpy
matplotlib
seaborn
scipy
statsmodels
scikit-learn
pymc>=5.0
arviz
jupyter
```

---

##  Running the Notebooks

```bash
jupyter notebook
```

>  The Bayesian model (`infernce using bayesian approach.ipynb`) requires PyMC and can take **15–30 minutes** to run depending on hardware. A GPU is not required but more RAM is beneficial.

---

##  Limitations

- **Cross-sectional data** — causal claims about scheme effects cannot be established
- **Subjective satisfaction** — self-reported measures may reflect expectations, not objective quality; social desirability bias may inflate scores
- **Sample imbalance** — states with n < 120 remain approximate despite hierarchical modelling
- **Survey weights not applied** — due to missing design variables
- **Unobserved confounding** — political and institutional factors are not captured

---

##  Future Work

- Link with NFHS/DLHS for quasi-experimental (diff-in-diff) causal analysis
- Extend Bayesian model to district- and region-level random effects with spatial CAR priors
- Apply factor analysis for weighted satisfaction scoring
- NLP on open-ended responses for qualitative drivers
- Replicate on future survey waves for temporal validation

---

##  References

1. Citizens Survey 2022–23 — [Nature Scientific Data](https://www.nature.com/articles/s41597-026-06775-6)
2. The Lancet Commission on India Citizen Health — [thelancet.com](https://www.thelancet.com/commissions-do/India-Citizen-Health)
3. Bayesian hierarchical methods — [Springer](https://link.springer.com/article/10.1007/s10479-023-05437-9)
4. Small-area estimation — [SAGE Journals](https://journals.sagepub.com/doi/10.1177/09622802241293776)
5. PyMC modelling — [Springer](https://link.springer.com/article/10.3758/s13428-023-02204-3)

---

##  License

This project is submitted as part of DS3006 Advanced Statistics coursework. All data is publicly available via Harvard Dataverse under its original licence terms.
