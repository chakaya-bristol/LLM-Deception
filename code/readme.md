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







