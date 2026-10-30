# Halloween and Crime in Montréal

**INSY 674 Final Project**

**McGill University**

**Fall 2026**

**Core Question:**

> **Does crime rise around Halloween in Montréal? If so, by how much, which types of crime drive it, and where does it concentrate? How can police anticipate demand and allocate patrol resources more effectively during this period?**

The analysis and models are in `Model_Building.ipynb` (Python, Google Colab). It follows a standard structure from problem framing to a final model evaluated on held-out years. It then uses the four model families to answer the Halloween question and forecasts Halloween 2026 by police district.

## Team

| Name | GitHub ID |
| :---- | :---- |
| Astrid Philippet | [astridphilippet](https://github.com/astridphilippet) |
| Jose Argueta | [josem-arguetad](https://github.com/josem-arguetad) |
| Mia Ghrayeb | [miaghrayeb](https://github.com/miaghrayeb) |
| Orlane Donkpegan | [orlanedonkpegan](https://github.com/orlanedonkpegan) |
| Anto Abousefian | [AntoA301](https://github.com/AntoA301) |
| Katerina Avvakumova | [katerinaavvakumova](https://github.com/katerinaavvakumova) |
| Taha Maqbool | [maqbooltaha](https://github.com/maqbooltaha) |

## Key results

### 1\. Predicting high-crime days

A high-crime day has more reported crimes than the 75th percentile of the training years (more than 96 crimes). All models are compared with AUC, or the probability that a model ranks a random high-crime day above a random normal day (0.5 \= coin flip, 1.0 \= perfect).

|  | Cross-validation AUC (2019–2023) | Test AUC 2024 | Test AUC 2025 |
| :---- | :---- | :---- | :---- |
| **Final model:** rank-averaged ensemble of tuned logistic regression, Bayesian Negative Binomial and neural network | **0.814 ± 0.064** | **0.664** | **0.674** |
| Baseline (seasonal naïve, essentially "busy if the same period last year was busy") | 0.688 ± 0.045 | 0.649 | 0.648 |

&nbsp;

- Calendar structure (day of week, season, holidays) carries most of the predictable signal. Adding past crime levels (lag features) lowered AUC for every model, so the final model uses calendar features only.  
- The final model beats the baseline in both test years, but by less than cross-validation suggested (test AUC is about 0.14 lower). It should be considered a risk-ranking aid, not an automated deployment rule.

### 2\. Halloween (Oct 30 – Nov 2\)

Key takeaway: Halloween changes the mix of crime, not the total.

| Crime type | Bayesian Negative Binomial: change in expected daily crimes (94% credible interval) | Logistic regression: odds ratio for share of crime (95% CI) |
| :---- | :---- | :---- |
| **Robbery** | **\+25.1%** (+6.5% to \+44.9%) | **1.23** (1.07–1.42) |
| **Mischief** | **\+21.5%** (+8.5% to \+35.6%) | **1.22** (1.12–1.33) |
| Theft from vehicle | −7.6% (−16.6% to \+2.5%) | **0.84** (0.78–0.92) |
| Break-in | −1.8% (−12.3% to \+9.9%) | 0.92 (0.85–1.00) |
| Vehicle theft | −4.8% (−19.1% to \+12.6%) | 1.02 (0.93–1.13) |
| **Total crime** | \+1.6% (−5.0% to \+8.7%): no clear change | — |

&nbsp;

- Total crime sees no consistent increase. The Bayesian, random-forest and neural-network analyses all agree on this.  
- The Halloween mischief effect does not differ significantly between police districts (likelihood-ratio test, p \= 0.067), so it is treated as citywide.  
- For Halloween 2026, the forecasted highest-risk districts are **21, 38, 20, 26 and 15**. Their average probability of at least one mischief report per day is 61–83%. About 12–14 districts are expected to have mischief each day of the window.

A key takeaway for police would be to focus Halloween prevention on mischief and robbery across the city, rather than adding patrols for crime in general.

## Data

| Source | City of Montréal open data, *Actes criminels* (SPVM), also published on [Données Québec](https://www.donneesquebec.ca/recherche/dataset/vmtl-actes-criminels) |
| :---- | :---- |
| **File used** | `actes-criminels.xlsx` (original French file; translated to English inside the notebook) |
| **Downloaded** | Oct 22, 2026 |
| **Rows loaded** | 363,024 reported crimes, Jan 1, 2015 – Oct 19, 2026 |
| **Columns** | crime category (6 types), report date, shift (day / evening / night), police district, coordinates |
| **Modelling period** | 2015–2023 for development and 2024–2025 for testing. 2026 is used only for the Halloween 2026 forecast. |

&nbsp;

Important to note that dates are report dates, not the dates the crimes happened. About 17% of rows have no coordinates because the city removes them for privacy. Those rows are kept, because they still count as crimes.

## Notebook structure (mapped to problem solving checklist)

| Notebook section | Checklist step | What it does |
| :---- | :---- | :---- |
| 0 | **Frame the Problem and Look at the Big Picture** | Objective, users, current workaround, framing (supervised, binary classification, offline), choice of AUC |
| 1–2 | **Setup** | Imports, random seed, `FAST` flag; loads the data and translates it from French |
| 3 | **Explore the Data** | Attribute profile, target distribution, outliers, trends, calendar patterns, Halloween first look, correlations, extra data that would help, summary of learnings |
| 4 | **Prepare the Data** | Cleaning, daily counts, calendar and lag features, train/test separation, sliding-window folds, leak-free preprocessing pipeline |
| 5 | **Baseline** | Seasonal-naive benchmark that every model must beat |
| 6 | **Shortlist Promising Models** | Logistic regression, Bayesian Negative Binomial (PyMC), random forest and neural network with default settings. 5-fold sliding-window CV (mean ± std AUC), permutation importance, error analysis, three feature iterations, shortlist of three |
| 7 | **Fine-Tune the System** | Random search with cross-validation; the scaler choice is treated as a hyperparameter |
| 8 | **Fine-Tune (ensemble)** | Rank-averaged ensemble, choice of training window (3-year vs. expanding) |
| 9 | **Fine-Tune (test set)** | Final model evaluated on 2024–2025 against the baseline |
| 10 | **Halloween findings** | Each model answers part of the question: odds (logistic), % change with uncertainty (Bayesian), actual vs. expected (random forest, neural network), districts |
| 11 | **Halloween 2026 forecast** | District-level mischief probabilities for Oct 30 – Nov 2, 2026 |
| 12 | **Save results** | Writes 13 result tables as CSV to `results/` folder |
| Final cell | **Present Your Solution** | Final model, performance vs. baseline, what worked and what didn't, assumptions and limitations |

### Safeguards against leakage

1. Each year from 2019 to 2023 is scored by a model trained only on earlier years. The data is never shuffled.  
2. Features built from past crime counts use only data at least 29 days old, which matches planning about a month ahead.  
3. Test years are held out from training all together.

## How to run

### Option A: Google Colab (as the notebook is set up)

1. Create a folder named `INSY674 Group Project` in your Google Drive ("My Drive") and put `actes-criminels.xlsx` in it. If the folder was shared with you, add a shortcut to it in My Drive first (right-click → *Organize* → *Add shortcut*).  
2. Open `Model_Building.ipynb` in Colab.  
3. In the **Setup** cell, choose the run mode:  
   - `FAST = True`: quick trial run with fewer search iterations and posterior draws. Use this to check that everything runs. The numbers will differ from the final results.  
   - `FAST = False`: full run that produces the results reported above.  
4. **Runtime → Run all**, and allow Drive access when asked.

PyMC, ArviZ, statsmodels and scikit-learn come pre-installed on Colab. The notebook installs `holidays` itself.

### Option B: Run locally

```sh
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
pip install notebook                    # only if you don't use VS Code / another Jupyter app
jupyter notebook Model_Building.ipynb
```

&nbsp;

The setup cell is written for Colab. To run it locally, edit the setup cell:

- delete the `from google.colab import drive` and `drive.mount(...)` lines;  
- delete `!pip -q install holidays` (requirements.txt already installs it);  
- set `DATA_DIR` to the folder that contains `actes-criminels.xlsx`, ending with a `/`.

### Runtime

The Bayesian model is fitted with MCMC sampling, many times over: cross-validation, tuning, and once for each crime type. It accounts for most of the runtime.

- `FAST = True`: about 15 minutes (measured on a 2-core CPU).  
- `FAST = False`: considerably longer, so allow at least an hour on a standard Colab CPU runtime.

### Reproducibility

- One random seed (`SEED = 301`) is set at the top and passed to every model. MCMC sampling and multi-core training can still cause very small differences between machines.  
- `requirements.txt` pins the library versions. To record the exact versions of a Colab session, run `!pip freeze | grep -iE "numpy|pandas|scipy|scikit-learn|statsmodels|matplotlib|pymc|arviz|holidays|openpyxl"`.

## Outputs

Section 12 saves these tables as CSV files to `results/`:

| File | Contents |
| :---- | :---- |
| `attribute_profile.csv` | Profile of every column (type, % missing, distribution, noise, usefulness) |
| `iterations.csv` | CV AUC of each model for each feature iteration |
| `shortlist.csv` | Every possible shortlist of three models, with ensemble AUC and error correlation |
| `tuning.csv` | Default vs. tuned CV AUC |
| `final_comparison.csv` | Tuned models, ensemble and baseline |
| `training_window.csv` | 3-year vs. expanding training window |
| `final_test.csv` | Final model vs. baseline on 2024 and 2025 |
| `logistic_type_effects.csv` | Halloween odds ratios by crime type |
| `logistic_district_effects.csv` | Halloween mischief odds ratios by district |
| `logistic_district_backtest.csv` | District-day model performance with and without the Halloween feature |
| `bayesian_by_type.csv` | Bayesian Halloween effects by crime type and time of day |
| `rf_actual_vs_expected.csv` | Random forest: actual vs. expected crimes around Halloween |
| `forecast_2026.csv` | Halloween 2026 mischief probabilities by district |

## Assumptions and limitations

The full discussion, with impact and mitigation for each point, is in the notebook's final cell. In short:

1. **Report date ≠ occurrence date**: Incidents from Halloween night may be reported on Nov 1 or later, which is why the window is Oct 30 – Nov 2\.  
2. **No context data:** weather, events, school schedules and daylight are not in the dataset.  
3. **Police deployment is unobserved:** Results are associations, not causal effects.  
4. **Only six crime categories:** Assaults and disturbances, for example, are not included.  
5. **District counts are not population-adjusted**: Some districts are more populous than others, and some merged or closed during the period. District-level analyses use only the districts present in every year.  
6. **Performance drops on recent years:** (CV AUC 0.814 vs. test AUC 0.664–0.674), so the model needs periodic recalibration.