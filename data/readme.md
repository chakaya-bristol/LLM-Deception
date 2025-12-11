# Data files 
This project uses the **PubHealth** dataset, a fact-checking dataset created specifically for the public health domain. The dataset was introduced by Kotonya & Toni (2020).

PubHealth contains approximately **11,000 health-related claims**, each paired with journalist-crafted, gold-standard explanations. 

## Labels
Each instance in the dataset is annotated with one of four labels:

- `true`
- `false`
- `unproven`
- `mixture`

These labels indicate the veracity of the claim. Each instance also includes an **explanation** field, which provides a justification for the label and is treated as the gold-standard explanation.

## Columns
The dataset consists of text data only, with the following columns:

- `claim_ID`
- `claim`
- `claim_url`
- `main_text`
- `label`
- `explanation` (explanation for the label)
- `subject_tags`
- `date_published`
- `fact_checkers_authors`
- `evidence_sources`

## File format and splits
The data is provided as **TSV (tab-separated values)** files and is split into standard train/test/development sets:

- `train.tsv` – training split  
- `dev.tsv` – development/validation split  
- `test.tsv` – test split

## Source
Kotonya, N., & Toni, F. (2020). *Explainable Automated Fact-Checking for Public Health Claims.*

Please refer to the original dataset and paper for full details and citation in academic work.


