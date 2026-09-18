# Hospital Readmission Analysis — Diabetic Patients

## Business Problem
Hospitals face financial penalties (CMS readmission penalty) for high 30-day readmission rates among diabetic patients. Hospital leadership needs to know which patient groups and clinical factors are driving readmissions, so interventions can be targeted instead of applied broadly.

## Dataset
**Diabetes 130-US hospitals for years 1999-2008** (Kaggle/UCI) — 101,766 patient encounters across 130 US hospitals, including demographics, diagnoses, lab procedures, medications, and readmission status.

## Business Questions
1. What's the overall readmission rate, and how does it vary by age group?
2. Which primary diagnoses have the highest readmission rates?
3. Does length of hospital stay correlate with readmission?
4. Do patients on more medications, or with more prior inpatient visits, get readmitted more?
5. Does admission type (emergency vs elective) affect readmission risk?

## Data Cleaning (Python / pandas)
- Replaced `?` placeholder values with proper nulls
- Dropped `weight`, `max_glu_serum`, `A1Cresult` — 83–97% missing, unusable
- Filled missing `medical_specialty`, `payer_code`, `race` with "Unknown" (moderate missingness, still useful as a category)
- Dropped rows with missing `diag_1/2/3` (<1.5% of data)
- Recoded `readmitted` (3-value: `<30`, `>30`, `NO`) into a binary target `readmitted_binary` (1 = readmitted within 30 days)
- Verified `encounter_id` had zero duplicates (each row = one unique hospital visit); `patient_nbr` duplicates (29,828) were expected, reflecting patients with multiple encounters
- **Result:** 100,244 clean rows, 47 columns, 0 missing values

## SQL Analysis (SQLite)
Loaded cleaned data into SQLite and queried:
```sql
-- Readmission rate by age group
SELECT age, COUNT(*) as total_patients,
       SUM(readmitted_binary) as readmitted_count,
       ROUND(100.0 * SUM(readmitted_binary) / COUNT(*), 2) as readmission_rate_pct
FROM encounters GROUP BY age ORDER BY age;

-- Top diagnoses by readmission rate
SELECT diag_1, COUNT(*) as total_patients,
       ROUND(100.0 * SUM(readmitted_binary) / COUNT(*), 2) as readmission_rate_pct
FROM encounters GROUP BY diag_1
HAVING COUNT(*) > 100
ORDER BY readmission_rate_pct DESC LIMIT 10;

-- Length of stay, medications, and prior visits vs readmission
SELECT readmitted_binary,
       ROUND(AVG(time_in_hospital), 2) as avg_length_of_stay,
       ROUND(AVG(num_medications), 2) as avg_medications,
       ROUND(AVG(number_inpatient), 2) as avg_prior_inpatient_visits
FROM encounters GROUP BY readmitted_binary;
```

## Key Findings
- **Overall 30-day readmission rate: 11.22%** (11,250 of 100,244 encounters)
- **Age is not a strong driver** — readmission rate stays fairly flat (10–14%) across most adult age groups
- **Diagnosis matters** — `V58` (aftercare/follow-up encounters) had the highest readmission rate (41.7%); diabetes-with-complications codes (250.6, 250.7) also ranked high (~18–19%), consistent with clinical expectation
- **Prior inpatient visits is the strongest predictor found** — readmitted patients averaged **1.22** prior inpatient visits vs **0.57** for non-readmitted patients (more than double)
- Length of stay (4.78 vs 4.37 days) and medication count (16.95 vs 16.01) showed only modest differences between readmitted and non-readmitted patients
- Admission type showed little variation in readmission rate (roughly 10–11.5% across Emergency, Urgent, Elective)

## Conclusion
Readmission risk in this dataset is driven less by age or how a patient was admitted, and much more by **clinical history** — patients with diabetes-related complications and a track record of prior inpatient visits are the strongest candidates for readmission-prevention programs (e.g. post-discharge follow-up calls, care coordination).

## Dashboard
Built in Power BI: KPI cards (readmission rate, total patients, avg length of stay), readmission rate by age group, top 10 diagnoses by readmission rate, and prior-inpatient-visits comparison.
*(Screenshot to be added)*

## Tools Used
Python (pandas), SQL (SQLite), Power BI

## Files
- `diabetic_data_clean.csv` — cleaned dataset
- `hospital_readmission_final.csv` — final dataset with labels, used in Power BI
