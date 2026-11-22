# 🏃 Physical Activity vs. Income (BRFSS 2019)

This project uses CDC survey data to test whether how much people exercise helps predict their income, beyond what age, education, employment, marital status, and health already tell us. The workflow covers cleaning coded survey data in Python, exploring it, training classification models, comparing models with and without the activity columns, and exporting the test-set results for a Power BI report.

## 📖 Table of Contents
- [Disclaimer](#-disclaimer)
- [Skills Demonstrated](#-skills-demonstrated)
- [Applications Used](#-applications-used)
- [Data Source](#-data-source)
- [Process of Project](#-process-of-project)
  - [1. Data Cleaning](#1--data-cleaning-python)
  - [2. Exploration](#2--exploration-python)
  - [3. Modeling](#3--modeling-python)
  - [4. Does Activity Add Anything?](#4--does-activity-add-anything)
  - [5. Power BI Export and Dashboard](#5--power-bi-export-and-dashboard)
- [Findings](#-findings)
- [Limitations](#-limitations)
- [How to Reproduce](#-how-to-reproduce)
- [File Structure](#-file-structure)

## 🚨 DISCLAIMER
This project uses a third-party data source, the CDC [Behavioral Risk Factor Surveillance System (BRFSS)](https://www.cdc.gov/brfss/annual_data/annual_2019.html). The data is self-reported survey data and may be inaccurate or incomplete. This project is solely intended to demonstrate skills in Python, classification modeling, and Power BI. It is not affiliated with or endorsed by the CDC, and it does not give medical or financial advice.

## 🔧 Skills Demonstrated
- Cleaning coded survey data (pandas, numpy)
- Classification modeling (Gradient Boosting, Random Forest) with a train, validation, and test split
- Model evaluation (accuracy, macro F1, ROC-AUC, confusion matrix) compared against a baseline
- Testing what a feature group adds by comparing models with and without it, overall and by age group
- Preparing model results for a BI tool (Power BI)

## 💻 Applications Used
- **Python** (pandas, numpy, matplotlib, seaborn, scikit-learn 1.4 or newer)
- **Jupyter Notebook**
- **Power BI Desktop** for displaying the model results

## 📊 Data Source
[CDC BRFSS 2019 annual data](https://www.cdc.gov/brfss/annual_data/annual_2019.html)
A custom extract with 418,268 respondents and 29 columns, saved as `raw_data/brfss_2019_custom.csv`. Income is a household income bracket (8 brackets). After cleaning, 307,215 respondents remain.

## 🧩 Process of Project

### 1. 🧹 Data Cleaning (Python)
- Dropped `BIRTHSEX` (84% missing), `EXEROFT2` and `EXERHMM2` (over 50% missing), `_AGEG5YR` (duplicates `_AGE80`), and the exercise-type codes `EXRACT11` and `EXRACT21`. `_RACE` was not used as a predictor.
- Kept respondents with a valid income bracket, a yes/no answer on exercise, and complete activity data (418,268 down to 307,297), then removed 82 more for impossible weekly activity totals.
- Turned refused and "don't know" answers into missing values, and recoded "none" answers (for example, 88 for children) to 0.
- Gave people who report no exercise 0 minutes.
- Left remaining missing values for the models to handle: Gradient Boosting accepts them directly and the Random Forest fills them with training-set medians.

| Field | How It Is Built |
|---|---|
| Activity Minutes | `PA2MIN_` minus `PA2VIGM_` (`PA2MIN_` counts vigorous minutes twice) |
| Weekly Frequency Fields | BRFSS frequency codes converted to times per week |
| BMI | `_BMI5` divided by 100 |
| 3-Bracket Income | Under $25k, $25k to $75k, $75k+ |
| `INCOME_50K` | 1 if household income is $50k or more |
| `INCOME_75K` | 1 if household income is $75k or more |

### 2. 🔎 Exploration (Python)
- Charted the income distribution, the activity distribution, and activity by income bracket.
- Built a summary table of activity by income bracket and a correlation heatmap.

### 3. 🤖 Modeling (Python)
- Split respondents 60% training, 20% validation, and 20% test, keeping income brackets in the same proportions in each part. Rows are split by position because `SEQNO` repeats across states.
- Fit Gradient Boosting (main model) and Random Forest on training rows and compared them on validation rows. The test rows were used once, for the final model.
- Tried four targets: 8 brackets, 3 brackets, and yes/no splits at $50k and $75k.

| Target | Always Guess the Larger Group | Gradient Boosting |
|---|---|---|
| 3 brackets | about 40% | 63.2% (validation) |
| $50k+ yes/no | 51.4% | 77.3% (test) |

The exact bracket is hard to predict because neighboring brackets look almost alike to the model. The yes/no question is much easier.

**Final model (test rows):** Gradient Boosting predicting $50k or more reached 77.3% accuracy and a ROC-AUC of 0.858. AUC runs from 0.5 (a coin flip) to 1.0 (perfect).

### 4. 🔬 Does Activity Add Anything?
For each comparison, one model uses a base set of features and a second model adds the five activity columns. Both use the same settings, training rows, and validation rows.

| Base Features | AUC Without Activity | AUC With Activity | Gain |
|---|---|---|---|
| Age + Sex | 0.620 | 0.693 | +0.073 |
| Demographics | 0.850 | 0.854 | +0.004 |
| Demographics + Health | 0.857 | 0.860 | +0.002 |
| Under 65, Demographics + Health | 0.862 | 0.864 | +0.002 |

The same comparison inside each age group (18-34, 35-49, 50-64, 65-80) gained between about +0.001 and +0.003.

### 5. 📈 Power BI Export and Dashboard
- The last notebook cell saves the test-set results as CSVs in a `powerbi` folder: actual vs. predicted income with the predicted probability, ROC curve points, headline metrics, and the with/without activity comparison.
- Planned report visuals: metric cards, a confusion matrix, accuracy by age group, education, and employment, a predicted probability histogram, an ROC curve, and a bar chart comparing models with and without activity.

*Dashboard in progress. Add screenshots here when it is finished.*

## 🔍 Findings

**1. Activity Helps Predict Income Only When Little Else Is Known**

With only age and sex in the model, adding the activity columns raises AUC from 0.620 to 0.693. Active people are more likely to be in higher-income households.

**2. Once Education, Employment, and Marital Status Are Known, Activity Adds Almost Nothing**

With demographics and health in the model, adding activity changes AUC from 0.857 to 0.860. A change of about 0.002 would not noticeably change who is classified correctly, so activity tells the model very little that education, employment, and marital status do not already tell it.

**3. The Result Holds for Working-Age Adults and Across Age Groups**

Limiting the data to people under 65 gives the same small gain (+0.002), so retirees are not driving the result. The gain stays between about +0.001 and +0.003 in every age group.

**4. Median Activity Rises With Income, but Average Activity Levels Off**

Median weekly activity climbs from 28 minutes in the lowest income bracket to 225 minutes in the highest. Average activity levels off above $35k at about 430 minutes, and the lowest brackets have a much higher average than median (305 vs. 28). This suggests many people with little or no activity alongside a few with very high totals.

**5. Education Is a Stronger Partner of Income Than Activity**

In the correlation heatmap, education relates to income much more strongly than activity does (roughly 0.44 vs. 0.18).

**6. Exact Income Brackets Are Hard to Predict, but Yes/No Splits Are Not**

On three brackets the best model reached about 63% against about 40% for guessing. On the $50k yes/no question it reached 77% against 51%, with a balanced score across both groups.

## 📝 Limitations
- Income is household income in brackets, and the top bracket ($75k+) holds 35% of respondents.
- Results use the unweighted sample, so they describe these respondents, not the US population.
- This is an association, not a cause. Activity is self-reported.
- Race was not used as a predictor.
- Gains of about 0.002 AUC are small enough to move from run to run, and confidence ranges were not calculated.

## 🚀 How to Reproduce
1. Clone this repo
2. Install Python 3 and the packages: `pandas`, `numpy`, `matplotlib`, `seaborn`, and `scikit-learn` (1.4 or newer)
3. Keep the folder layout below. The notebook reads `../raw_data/brfss_2019_custom.csv`
4. Open [activity_v_income.ipynb](notebooks/activity_v_income.ipynb) and run all cells
5. The last cell writes the Power BI files to the `powerbi` folder
6. Connect Power BI Desktop to those CSV files (Get Data, then Text/CSV)

## 📁 File Structure

```
├── notebooks/
│   ├── activity_v_income.ipynb   # Cleaning, exploration, modeling, comparison, and Power BI export
├── raw_data/
│   ├── brfss_2019_custom.csv     # BRFSS 2019 extract used in the notebook
├── powerbi/
│   ├── test_predictions.csv      # Test-set actual vs. predicted income and probabilities
│   ├── roc_curve.csv             # ROC curve points
│   ├── model_metrics.csv         # Accuracy, macro F1, ROC-AUC, and baseline
│   ├── activity_comparison.csv   # AUC with vs. without activity
└── README.md
```
