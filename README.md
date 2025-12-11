# LLM-Deception

This repository contains the code and data setup for a case study on deception-like behaviour in large language models (LLMs) in a public health fact-checking setting.

The project evaluates how simple prompt changes and access to evidence affect an LLM’s tendency to:

- give correct vs misleading answers,
- admit uncertainty (“I don’t know”),
- express confidence, and
- ground its answers in cited evidence.

---

## Project overview

The study uses health-related claims from the **PubHealth** dataset to simulate a fact-checking assistant. For each claim, the model is asked to decide whether it is **true**, **false**, **unproven**, or a **mixture** according to current scientific consensus.

Three prompting conditions are compared:

1. **Baseline** – the model must always pick a label.
2. **Abstention** – the model can answer *“I don’t know”* if unsure.
3. **Retrieval-augmented** – the model is given gold evidence snippets from PubHealth and asked to answer using only this evidence.

Model outputs are then analysed for accuracy, abstention, confidence calibration, and “deception-like” patterns (confident, fluent answers that are not supported by evidence).

---

## Dataset

The data comes from the **PubHealth** dataset introduced by Kotonya & Toni (2020), which contains around 11,000 fact-checked public health claims with:

- a veracity label (`true`, `false`, `mixture`, `unproven`), and  
- a gold-standard, journalist-written explanation for each claim.

In this project, the TSV files are used to:

- sample claims for each experiment condition,
- build prompts for the LLM, and
- compare model predictions with the gold labels and explanations.

See `data/README.md` for more detail on the dataset and file layout.

---

## Repository structure

- `data/`  
  Description of the PubHealth dataset and expected TSV files.  
  See `data/README.md`.

- `code/`  
  Notebooks and helper functions for prompt generation, model evaluation, metrics, and plots.  
  See `code/README.md`.

- `README.md` (this file)  
  High-level overview of the project.

---

## Citation

If you use the dataset, please cite the original PubHealth paper:

> Kotonya, N., & Toni, F. (2020). Explainable Automated Fact-Checking for Public Health Claims.
