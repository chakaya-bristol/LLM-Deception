# Code
This folder contains the core code used to generate prompts for the different experimental conditions in the essay: **baseline**, **abstention**, and **retrieval-augmented**. Each function operates on a single row from the PubHealth dataset (typically a `pandas.Series`) and returns a text prompt to send to an LLM.

## Functions
### `baseline_prompt(row, with_confidence: bool = False) -> str`

**What it does:**  
Builds the **baseline** fact-checking prompt. The model is asked to decide whether the claim is true, false, a mixture, or unproven according to current scientific consensus.

- **Inputs:**
  - `row`: A PubHealth row with at least the `claim` field.
  - `with_confidence`: If `True`, the prompt also instructs the model to report a confidence score (e.g. “Confidence: 72%”).
- **Returns:**  
  A single string containing the full baseline prompt, including the claim text and label instructions and, the confidence instruction.

---

### `abstention_prompt(row, with_confidence: bool = False) -> str`

**What it does:**  
Builds the **abstention** condition prompt. It is similar to the baseline prompt but explicitly tells the model to answer **“I don't know”** if it is not reasonably sure.

- **Inputs:**
  - `row`: A PubHealth row with the `claim` field.
  - `with_confidence`: If `True`, the prompt also includes the confidence instruction.
- **Returns:**  
  A string with the abstention-style prompt, including:
  - The claim text.
  - Label instructions.
  - An explicit rule that the model should reply exactly “I don't know” when uncertain (plus confidence instructions).

---

### `make_evidence_snippets(row, max_chars_main: int = 800, max_chars_expl: int = 400) -> list[str]`

**What it does:**  
Extracts short text **snippets of evidence** from the PubHealth row to feed into the retrieval-augmented prompt.

- **Logic:**
  - If `main_text` is a non-empty string, it adds the first `max_chars_main` characters as one snippet.
  - If `explanation` is a non-empty string, it adds the first `max_chars_expl` characters as another snippet.
  - If neither is available, but `sources` is a non-empty string, it falls back to the first ~600 characters from `sources`.
- **Inputs:**
  - `row`: A PubHealth row with `main_text`, `explanation`, and/or `sources`.
  - `max_chars_main` (optional): Character limit for the `main_text` snippet.
  - `max_chars_expl` (optional): Character limit for the `explanation` snippet.
- **Returns:**  
  A **list of strings**, where each string is an evidence snippet to be shown to the model.

---

### `retrieval_augmented_prompt(row, with_confidence: bool = False) -> str`

**What it does:**  
Builds the **retrieval-augmented** prompt. It calls `make_evidence_snippets` to get supporting evidence, formats it into a numbered evidence block (`[1] ...`, `[2] ...`), and then asks the model to classify the claim **using only this evidence**.

- **Inputs:**
  - `row`: A PubHealth row containing the claim and associated text fields used as evidence.
  - `with_confidence`: If `True`, appends the confidence instruction at the end of the prompt.
- **Returns:**  
  A string containing:
  - A short system-style instruction (e.g. “Use only the passages below as evidence…”).
  - A numbered **Evidence** section with snippets.
  - The target claim.
  - Label instructions (and optionally the confidence instruction).

---

These functions are used to generate prompts for each condition over the sampled PubHealth claims, so that model behaviour can be compared across baseline, abstention, and retrieval-augmented setups.

---

### `extract_first_answer_block(text: str) -> str`

Cleans up raw model responses to keep only the **first structured answer block**.

- **What it does:**
  - If `text` is `None`, returns an empty string.
  - Splits the response into lines and looks for the first line starting with `Label:` (case-insensitive).
  - From that line onwards, keeps lines until:
    - It encounters a line starting with `## Your task` (case-insensitive), or
    - It has passed the `Confidence:` line.
  - If no `Label:` line is found, it simply returns the stripped original text.

- **Typical use:**
  To discard any extra prompts, instructions, or second answers that the model may append, leaving only:
  Label: ...
  Justification: ...
  Confidence: ...

  - **Returns:**
  A cleaned string with just the first `Label` / `Justification` / `Confidence` block (or the original stripped text if no `Label:` is found).

---

### `print_example(row, title: str | None = None) -> None`

Utility function for inspecting a single PubHealth example.

- **What it does:**
  - Prints an optional `title`.
  - Prints key fields from the given row (typically `claim`, `label`, and possibly explanation and model outputs if present).
  - Helps with qualitative inspection and debugging in the notebook.

- **Inputs:**
  - `row`: A single PubHealth row (e.g. `pandas.Series`).
  - `title` *(optional)*: Short descriptive title printed above the example.

- **Returns:**
  Nothing. It prints to standard output / notebook cell.

---

### `compute_metrics(df: pd.DataFrame) -> dict`

Computes the core performance and behaviour metrics across a set of model outputs.

- **Expected columns in `df`:**
  - `pred_label`: Model-predicted label.
  - `gold_label`: Ground-truth label from PubHealth.
  - `said_idk`: Boolean, `True` if the model answered “I don't know”.
  - `confidence` *(optional)*: Float in `[0, 1]` giving model confidence.
  - `used_citation` *(optional)*: Boolean flag for evidence use (mainly in the retrieval condition).

- **What it does:**

  Creates helper columns:

  - `is_correct`: `True` if `pred_label == gold_label`.
  - `answered`: `True` if `~said_idk`.

  Then computes:

  - `n`: Total number of examples (`len(df)`).
  - `coverage`: Proportion of examples answered (`answered` is `True`).
  - `accuracy_all`: Mean of `is_correct` over **all** examples (treats “I don't know” as wrong).
  - `accuracy_answered`: Mean of `is_correct` over **answered** examples only.
  - `hallucination_rate`: Error rate on answered examples (`1 − accuracy_answered`).
  - `abstention_rate`: Fraction of examples where `said_idk` is `True`.
  - `overconfidence`: Mean confidence on wrong answered examples (if `confidence` exists; otherwise `np.nan`).
  - `brier_score`: Brier score on answered examples with valid confidence values (or `np.nan` if not available).
  - `evidence_compliance`: Mean of `used_citation` over answered examples (or `np.nan` if the column is missing).

- **Returns:**

  A `dict` with keys:

  {
      "n",
      "coverage",
      "accuracy_all",
      "accuracy_answered",
      "hallucination_rate",
      "abstention_rate",
      "overconfidence",
      "brier_score",
      "evidence_compliance",
  }

---

### `compute_high_conf_error_metrics(df: pd.DataFrame, high_conf_threshold: float = 0.8) -> dict`

Focuses on high-confidence behaviour, such as how often the model is confidently wrong.

- **Expected columns in `df`:**
  - Same as for `compute_metrics`, plus a non-null `confidence` column in `[0, 1]`.

- **What it does:**
  - Filters to answered examples (`answered == True`) with non-null `confidence`.
  - Identifies **high-confidence** answers where `confidence >= high_conf_threshold`.
  - Computes:
    - `answered_with_conf`: Number of examples with a valid confidence score.
    - `high_conf_coverage`: Proportion of **all** examples that are high-confidence answers.
    - `high_conf_error_rate`: Among high-confidence answers, fraction that are incorrect.

- **Returns:**

  A `dict` summarising high-confidence behaviour, typically including:

  {
      "answered_with_conf": ...,
      "high_conf_coverage": ...,
      "high_conf_error_rate": ...,
  }

---

### `plot_metric_bar(summary_df: pd.DataFrame, metric: str, title: str | None = None, ylabel: str | None = None) -> None`

Creates a simple bar chart for a chosen metric across experimental conditions.

- **Typical input format:**
  - `summary_df`: Small `DataFrame` where each row corresponds to a condition (e.g. `baseline`, `abstention`, `retrieval`) and columns include metrics such as `accuracy_answered`, `hallucination_rate`, etc.  
    The condition names can either be in the index or in a dedicated column (e.g. `condition`).
  - `metric`: Name of the metric column to plot (e.g. `"accuracy_answered"`).

- **What it does (using `matplotlib`):**
  - Puts conditions on the x-axis.
  - Plots the values of `summary_df[metric]` as bar heights.
  - Optionally sets:
    - Plot title (`title`).
    - y-axis label (`ylabel`).

- **Inputs:**
  - `summary_df`: DataFrame of per-condition metrics (e.g. derived from `compute_metrics` and `compute_high_conf_error_metrics`).
  - `metric`: Name of the metric column to plot.
  - `title` *(optional)*: Plot title.
  - `ylabel` *(optional)*: Label for the y-axis.

- **Returns:**
  It produces a bar plot (e.g. inline in a notebook).






