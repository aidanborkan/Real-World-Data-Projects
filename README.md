Mini Project AB — Comparing OMOP and MIMIC Prescription Data

Overview

This project explores how medication exposure data differ between two widely used clinical informatics frameworks:

OMOP (Common Data Model): an observational research standard used to harmonize electronic health record (EHR) and claims data.

MIMIC-III: a publicly available critical-care database from Beth Israel Deaconess Medical Center containing de-identified ICU EHR data.

By analyzing prescription counts, source codes, and drug type classifications, the study evaluates how medication events are structured and recorded across these two environments — a core issue in real-world data interoperability and reproducibility.

Data and Methods

The analysis was written in R Markdown and rendered as Mini_Project_AB.html.
Key steps and tools include:

| Step                    | Purpose                                                              | Tools / Packages                          |
| ----------------------- | -------------------------------------------------------------------- | ----------------------------------------- |
| Database connection     | Connect to OMOP-formatted MIMIC data (`mimic3_demo_omop`)            | `DBI`, `dplyr`, `dbplyr`                  |
| Summary statistics      | Compare yearly prescription volumes in OMOP vs MIMIC                 | `dplyr`, `summary()`                      |
| Log-transformed metrics | Normalize skewed prescription counts for comparison                  | base R `log1p()`                          |
| Visualization           | Overlay temporal trends of prescriptions per year                    | `ggplot2`, `theme_minimal()`              |
| Source inspection       | Identify unique `drug_source_value` codes and `DRUG_TYPE` categories | `distinct()`, `tibble`, SQL-style joins   |
| Concept mapping         | Join `drug_exposure` to `concept` tables to derive `drug_type_name`  | `left_join()` on `concept_id`             |
| Categorical counts      | Summarize and visualize distribution of drug types                   | `group_by()`, `summarize()`, `geom_bar()` |

Findings

Higher prescription counts in MIMIC: all summary statistics (min, median, mean, max) were larger for MIMIC than OMOP, suggesting more granular or inclusive drug exposure capture

Mini_Project_AB

.

Normalization effects: after log transformation, inter-model differences decreased, implying differing data-scale conventions rather than true volume differences

Mini_Project_AB

.

Source diversity: OMOP showed 1,100 + unique drug_source_value entries, while MIMIC exhibited fewer but broader DRUG_TYPE categories (e.g., BASE, MAIN, ADDITIVE), highlighting schema design contrasts

Mini_Project_AB

.

Vocabulary richness: qualitative inspection of drug names revealed MIMIC’s coverage of compounded or ophthalmic treatments absent from OMOP, possibly due to differing data pipelines

Mini_Project_AB

.

Clinical Informatics Relevance

This comparison reflects practical challenges in ETL and standardization of medication data across systems.
It demonstrates how:

Prescription event representation depends on vocabulary mappings and concept IDs.

Schema transformation (OMOP conversion) can affect medication counts and source diversity.

Log-based normalization and metadata exploration aid cross-model quality checks.

Such work is directly applicable to data governance, model validation, and interoperability assessments in clinical informatics pipelines.

Deliverables

Mini_Project_AB.html — Rendered R Markdown report with code, output, and figures.

Mini_Project_AB.Rmd (if present) — Source file for reproducibility.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Mini Project 2 — Hypertension Phenotyping Using MIMIC and BigQuery
Overview

This project develops and evaluates a method to identify hypertensive patients by combining structured EHR data from MIMIC-III with a manually curated gold standard hosted on Google BigQuery. It demonstrates how prescription data, billing codes, and charted blood pressure measurements can be integrated to define clinical phenotypes — a core task in clinical informatics.

Clinical Context:

Hypertension is defined clinically as:

Systolic BP ≥ 140 mmHg on ≥ 2 occasions
&
Diastolic BP ≥ 90 mmHg on ≥ 2 occasions

Accurate hypertension phenotyping is crucial for both clinical research and population health analytics, as it determines cohort inclusion, prevalence estimates, and treatment-outcome relationships.

Data Sources and Methods:

| Component                                                    | Purpose                                               | Tools / Packages                      |
| ------------------------------------------------------------ | ----------------------------------------------------- | ------------------------------------- |
| **Google BigQuery (course3_data.hypertension_goldstandard)** | Manually validated gold-standard dataset              | `bigrquery`, `DBI`                    |
| **MIMIC-III CHARTEVENTS / LABEVENTS**                        | Extract vital signs and lab values                    | `dbplyr`, `tidyverse`, `dplyr`, `DBI` |
| **Antihypertensive Medication Table**                        | Identify hypertension-related prescriptions           | `distinct()`, `select()`              |
| **Predictive Evaluation**                                    | Compare prescription-based detection to gold standard | Custom R functions using `caret`      |
| **Visualization and Validation**                             | Explore distribution and diagnostic performance       | `ggplot2`, `summary()`, `table()`     |

Key implementation steps:

Connected to Google BigQuery using OAuth for authenticated access.

Queried the hypertension_goldstandard table to retrieve labeled subjects.

Extracted and summarized systolic and diastolic blood pressure ITEMIDs from CHARTEVENTS based on validated mappings.

Merged prescription data from D_ANTIHYPERTENSIVES to test whether drug exposure correlates with true hypertensive status.

Computed sensitivity, specificity, PPV, and NPV for each medication (e.g., atenolol, clonidine, captopril).

Findings

Atenolol and Clonidine emerged as the most specific predictors of hypertension (specificity = 1.0), though based on small sample sizes.

Low NPV values across medications indicated that absence of a prescription is a poor indicator of non-hypertension.

The combined approach (medications + BP measurements) yielded a more accurate phenotype than either source alone.

Approximately 63 of 99 patients were labeled hypertensive, confirming consistency between MIMIC-derived features and the gold-standard dataset

AB_mini_project_2

.

Clinical Informatics Relevance

This project illustrates how multi-source EHR integration (billing codes, vital signs, and prescriptions) can improve algorithmic phenotyping for chronic diseases.
It models the practical workflow of:

Accessing federated clinical data (via BigQuery)

Using ETL logic to join structured tables

Performing model evaluation to validate operational definitions of disease

These methods align with best practices in clinical data warehousing, phenotype validation, and RWD curation — foundational to informatics-driven quality improvement and population health research.

Deliverables

AB_mini_project_2.html: Rendered R Markdown report

Source code (R Markdown): Demonstrates reproducible analytics using bigrquery, dplyr, and caret

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Mini Project 3 — Diabetes Complications Detection from Clinical Notes
Overview

This project applies natural language processing (NLP) methods to identify diabetes-related complications — neuropathy, retinopathy, and nephropathy — within unstructured clinical text (history and physical, operative notes, discharge summaries).
The dataset was hosted on Google BigQuery (course4_data.diabetes_notes), and all analysis was conducted using R and bigrquery for federated querying.

Clinical Context

Patients with diabetes often develop microvascular complications that may appear in narrative sections of clinical documentation rather than structured EHR fields. Extracting these complications supports:

Automated cohort identification

Population-level complication surveillance

Improved phenotype completeness for data warehouses

Methods:

| Step                           | Description                                                                                 | Tools / Packages             |
| ------------------------------ | ------------------------------------------------------------------------------------------- | ---------------------------- |
| **Database Connection**        | Connected to Google BigQuery’s *learnclinicaldatascience* project                           | `bigrquery`, `DBI`           |
| **Data Retrieval**             | Pulled the `diabetes_notes` table containing clinical note text and metadata                | `tbl()`, `collect()`         |
| **Condition Identification**   | Filtered text sections containing target complication terms                                 | `dplyr`, `stringr`           |
| **Structured Output Creation** | Built a tibble of `NOTE_ID`, `NOTE_TYPE`, and `CONDITIONS_TEXT` for manual/automated review | `tidyverse`, `DT`            |
| **Sampling & Visualization**   | Displayed representative note excerpts using R’s interactive DataTables                     | `DT`, `magrittr`             |
| **Performance Evaluation**     | Calculated precision, recall, and F1 score for each condition vs manual reference           | `caret`, `confusionMatrix()` |

Results:

| Condition       | Precision | Recall | F1 Score | Interpretation                                                                               |
| --------------- | --------- | ------ | -------- | -------------------------------------------------------------------------------------------- |
| **Neuropathy**  | 68.4%     | 86.7%  | 76.5%    | Strong recall, moderate precision — model captures most true cases but over-identifies some. |
| **Retinopathy** | 20%       | 33.3%  | 25%      | Poor detection — suggests limited keyword coverage or contextual misclassification.          |
| **Nephropathy** | 75%       | 60%    | 66.7%    | Balanced performance — acceptable accuracy for initial rule-based extraction.                |


Total identified cases: 28 out of ~150 analyzed notes contained explicit complication mentions

.

High recall but low precision patterns imply that many false positives arose from negations (e.g., “no evidence of neuropathy”) or irrelevant contexts — a common issue in unstructured note mining.

Discussion

This project demonstrates the transition from structured to unstructured data analytics in clinical informatics.
Key insights:

Many EHR data fields omit narrative details critical to phenotype completeness.

Text-mining pipelines can recover valuable information about complications not coded elsewhere.

The next step in such workflows is to add negation detection and context analysis (e.g., via regex or NLP libraries like tidytext or quanteda).

Clinical Informatics Relevance

This work reflects core informatics competencies:

Data integration: Extracting from BigQuery (federated data) and transforming into structured tibbles.

Phenotyping: Operationalizing definitions for diabetic complications.

Evaluation: Quantifying model performance (precision, recall, F1) as part of data quality assurance.

Governance: Highlighting the limits of text-only detection and the need for hybrid approaches (text + structured EHR fields).

Deliverables:

AB_mini_prj_3.html — Rendered report with code, tables, and confusion-matrix results.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Mini Project 4 — Mortality Prediction in ICU Patients Using MIMIC-III
Overview

This project develops a predictive model for in-hospital mortality among ICU patients, using structured data from the MIMIC-III clinical database.
It illustrates the full data-science workflow common in clinical informatics — from cohort definition and feature selection to model evaluation and interpretation — while emphasizing the challenges of missing data, cohort bias, and model generalizability.

Clinical Context

Predicting patient outcomes such as ICU mortality supports:

Early risk stratification and allocation of clinical resources

Quality improvement through identification of modifiable risk factors

Decision support and benchmarking for data-driven clinical care

MIMIC-III provides rich ICU data (vitals, admissions, outcomes) to simulate a real-world predictive modeling task within a governed research environment.

Methods

| Step                    | Description                                                                                                                  | Tools / Packages            |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| **Database connection** | Queried MIMIC-III demo dataset through Google BigQuery                                                                       | `bigrquery`, `DBI`          |
| **Table ingestion**     | Imported key tables: `PATIENTS`, `ADMISSIONS`, `ICUSTAYS`, `DIAGNOSES_ICD`, and `CHARTEVENTS`                                | `dplyr`, `dbReadTable()`    |
| **Cohort definition**   | Filtered ICU admissions where `HOSPITAL_EXPIRE_FLAG == 1` (in-hospital death)                                                | `filter()`, `mutate()`      |
| **Feature selection**   | Chose vital-sign predictors — *mean heart rate, systolic blood pressure, respiratory rate* — from the first 24 h of ICU stay | `group_by()`, `summarise()` |
| **Model development**   | Built and trained a logistic regression model to classify survival status                                                    | base R `glm()`              |
| **Model validation**    | Assessed discrimination using training/test split and **AUC (Area Under the Curve)** metrics                                 | `caret`, ROC curves         |
| **Data auditing**       | Checked for missing (`NA`), infinite (`Inf`), and inconsistent values; excluded incomplete records                           | `is.na()`, `filter()`       |

Results

The model achieved an AUC of ~0.70 on the training set and ~0.49 on the test set, suggesting moderate internal discrimination but poor generalizability

AB_mini_proj_4

.

Extensive missing data and limited predictor diversity (few vital signs) reduced model accuracy.

Excluding patients with incomplete data may have introduced selection bias, limiting representativeness of the ICU population.

Future iterations should integrate lab values, length of stay, and medication data to enhance predictive power.

Discussion

The project demonstrates both the feasibility and pitfalls of predictive modeling with EHR data:

Data quality directly affects model reliability — missing or miscoded fields propagate bias.

Cohort definition (ICU deaths only) narrows clinical scope, potentially excluding post-discharge mortality or hospice transitions.

Evaluating socio-demographic variables (race, insurance, ethnicity) can uncover health disparities and algorithmic bias

AB_mini_proj_4

.

Clinical Informatics Relevance

This work exemplifies how an informatics specialist:

Extracts structured EHR data from a relational database (SQL-like BigQuery).

Designs an analytic cohort following reproducible data-governance steps.

Applies supervised learning for outcome prediction within ethical and methodological constraints.

Performs model audit and interprets metrics for decision-support reliability.

Deliverables

AB_mini_proj_4.html: Rendered R Markdown report

R source code: full pipeline from data extraction to AUC evaluation

