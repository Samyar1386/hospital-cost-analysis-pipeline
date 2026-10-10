# Hospital Cost Analysis: From Raw Data to an Automated Report

An end-to-end analytics project: data cleaning in Python, an interactive Excel dashboard, and an n8n workflow that turns a set of pre-defined KPIs into an emailed report written by an LLM, with a validation step that checks every number before the email is sent.

## Business Question

> Do hospitals differ in cost per day of stay?

Cost per Day = Billing Amount / Length of Stay, where Length of Stay = Discharge Date - Date of Admission.

## Dataset

A practice dataset of 55,500 patient records across 10 US hospitals (17 columns: demographics, condition, admission type, insurance provider, billing amount, test results, dates).

> The raw data is not included in this repository.It is publicly available at:
https://t.me/Dadehkar/20

## Pipeline

```
Raw data  ->  Python cleaning & KPI analysis  ->  Excel dashboard
                                                       |
                                              KPI_Export sheet
                                                       |
   Form upload -> n8n -> LLM writes report -> number validation -> Email
```

## 1. Cleaning and Analysis (Python)

- No missing values and no duplicate rows were found.
- 108 rows (0.19%) had a negative Billing Amount. Their typical size (median about -369) is roughly 70 times smaller than normal bills (median about 25,574), so they look like adjustments or refunds rather than sign errors. I kept them as a separate set instead of using `abs()` or deleting them, and excluded them from the main analysis (55,392 rows).
- Length of Stay had no zero or negative values.
- Cost per Day is heavily skewed (mean 3,393 vs median 1,651), because very short stays with large bills inflate the mean. Hospitals were therefore compared with standard errors, not raw means.
- Each group was tested with a z-score based on its standard error: Hospital, Medical Condition, Admission Type, Insurance Provider, Gender, Blood Type, Test Results, plus a proportion test for abnormal results by hospital and Age correlations.

## 2. Excel Dashboard

KPI cards (mean and median cost per day, total patients, mean length of stay, abnormal-result rate, adjustment records), cost-per-day charts with error bars by hospital and by condition, admission-type distribution, abnormal rate by hospital, and a PivotTable with slicers.

## 3. Automated Report (n8n + LLM)

1. Form Trigger: upload the workbook.
2. Extract from File: read the `KPI_Export` sheet (one row of KPIs).
3. Edit Fields: attach the fixed analytical question.
4. LLM (OpenRouter, GPT_4.1_Mini , temperature 0): writes the report.
5. Code node: checks that every KPI number appears in the report and that no number outside the KPIs appears.
6. IF: valid report -> email; failed validation -> alert email with the extra/missing numbers.

**Design decision:** the question and KPIs are defined by me in advance. The LLM only writes the text; it does not choose what matters or calculate anything.

## Key Finding

No hospital differs significantly from the overall average in cost per day. The largest deviation (Cleveland Clinic, |z| = 2.06) is below the Bonferroni-corrected threshold of 2.81 for 10 simultaneous tests. No other variable tested showed a significant relationship with cost either.

## Limitations

- The data is a practice dataset; the lack of any relationship between cost and other variables is consistent with synthetic data. These results should not be read as statements about the real hospitals named in it.
- The validation step checks numbers, not wording. The model could still describe the findings in a misleading way.
- Tests were run per variable, not in a joint model.

## How to Run

1. Run the notebook in `notebooks/` to reproduce the cleaning and KPIs.
2. Open the workbook in `excel/` (the `KPI_Export` sheet feeds the workflow).
3. Start n8n (Docker), import `n8n/workflow.json`, and create your own credentials (OpenRouter API key, SMTP).
4. Open the form URL, upload the workbook, and submit.

## Tools

Python (pandas, numpy), Excel, n8n (Docker), OpenRouter API.