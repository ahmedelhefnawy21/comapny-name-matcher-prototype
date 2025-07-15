# Company Name Matcher

This prototype repository contains a **Jupyter Notebook-based pipeline** for standardising and matching company names across three datasets related to energy companies. The output is a well-documented, human-verifiable Excel workbook containing both exact and intelligent approximate matches.

The notebook is designed to merge and reconcile disparate datasets using both **fuzzy string matching** and **semantic similarity** based on sentence embeddings.

---

## Objectives

The goal is to:

1. **Standardise** noisy legal company names by cleaning and removing boilerplate elements (e.g. “Ltd”, “GmbH”, “Power Solutions”).
2. **Identify exact matches** of company names between datasets.
3. **Match remaining unmatched names** using fuzzy string similarity (multi-metric ensemble).
4. **Apply sentence embeddings** to detect semantically similar names.
5. **Assist in human review** with clear score thresholds and interactivity.
6. **Export results** to a multi-sheet Excel file.

---

## Input Files

| Filename            | Format | Description                                                              |
| ------------------- | ------ | ------------------------------------------------------------------------ |
| `wacc_all.csv`      | CSV    | Semicolon-delimited file with `Instrument` and WACC data.                |
| `names.csv`         | CSV    | Comma-delimited file linking `Instrument` to a company name.             |
| `plant_owners.xlsx` | Excel  | File containing raw names of renewable plant owners in its first column. |

---

## Matching Process Overview

The pipeline consists of **seven main stages**:

### 1. Load and Merge Input Data

* `wacc_all.csv` and `names.csv` are merged on the `Instrument` column.
* The resulting merged dataset is then compared to `plant_owners.xlsx`.

### 2. Clean and Standardise Names

* Applies a powerful regex-based function to remove:

  * Legal suffixes (e.g. GmbH, Ltd, SPA)
  * Industry descriptors (e.g. Group, Energy, Renewables)
  * Noise such as parentheses, quotes, punctuation
* Then applies `cleanco`'s `basename()` to remove remaining legal markers.
* Converts all names to uppercase and trims excess whitespace.

### 3. Exact Matching

* Matches cleaned names from the `plant_owners.xlsx` file to the merged dataset.
* Exact string match on `cleaned_name`.

### 4. Identify Generic Words

* Tokens appearing in >2% of names are flagged as generic.
* Used later to avoid false matches (e.g. "Green Power Renewables" vs "Global Power Renewables").

### 5. Fuzzy String Matching

* Uses the `name-matching` package to compare strings across 7 distance metrics:

  * Discounted Levenshtein
  * Editex
  * Ratcliff/Obershelp
  * FuzzyWuzzy (partial, token sort)
  * Weighted Jaccard
  * Double Metaphone
* A score boost of +30 is applied if one name is a subset of the other minus generic words.
* Top 5 candidates are selected for each unmatched owner.

### 6. Semantic Similarity Matching

* Encodes names using `sentence-transformers` with the `all-mpnet-base-v2` model.
* Computes cosine similarity.
* Matches are accepted if similarity ≥ 0.70.

### 7. Human Review & Decision

* For fuzzy and semantic results:

  * Matches with a score ≥ 0.85 are auto-approved.
  * Others are presented via `input()` for manual decision.
  * Decisions are remembered and reused.

---

## Output File

After running all steps, a multi-sheet Excel workbook is saved as `wacc_owners.xlsx`

It contains the following sheets:

| Sheet name             | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| `wacc_merged`          | Raw data from WACC and names.csv, cleaned                    |
| `wacc_owners`          | Exact matches between plant owners and WACC companies        |
| `wacc_owners_fuzzy`    | Fuzzy match results, with scores and approval flags          |
| `wacc_owners_semantic` | Semantic match results, with similarity scores and approvals |

---

## Configuration Parameters (in notebook)

| Parameter             | Default | Description                                            |
| --------------------- | ------- | ------------------------------------------------------ |
| `FUZZY_AUTO_APPROVE`  | 85      | Min score (out of 100) for auto-accept in fuzzy match  |
| `SEMANTIC_THRESHOLD`  | 0.70    | Min cosine similarity for semantic match               |
| `GENERIC_TOKEN_SHARE` | 0.02    | Words occurring in ≥2% of names are treated as generic |

You can adjust these values in the notebook cells before running the relevant sections.

---

## How to Run the Notebook

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Launch JupyterLab or Notebook

```bash
jupyter lab
```

### 3. Open and run

Open the file:

```
notebooks/matching.ipynb
```

Then click:

```
Run > Run All Cells
```

You will be prompted for manual review only when fuzzy or semantic scores fall below the auto-accept threshold.

---

## Review Step

During fuzzy/semantic review:

* You’ll be asked: `Approve (a), Reject (r), Skip (s)`
* Decisions are remembered and reused for repeated matches
* Scores below threshold require your attention; others are auto-approved

---

## Testing & Validation

⬜ Tests are planned to cover:

* Cleaning edge cases
* Scoring behavior of fuzzy matching
* Integrity of the exported workbook

---

## Troubleshooting

| Problem                          | Likely Cause                        | Solution                             |
| -------------------------------- | ----------------------------------- | ------------------------------------ |
| `KeyError: 'adjusted_score'`     | Score column not yet created        | Run score boosting cell first        |
| Notebook freezes on review step  | Running in headless/non-interactive | Skip review or disable interactivity |
| Semantic step is slow or crashes | OOM during SentenceTransformer      | Reduce batch size, or use CPU        |
| Matches seem too loose           | Thresholds are too low              | Raise fuzzy/semantic thresholds      |

---

## Credits & Acknowledgements

* [cleanco](https://github.com/microscopeIT/cleanco) — Legal suffix parser
* [sentence-transformers](https://www.sbert.net/) — Semantic model
* [name-matching](https://pypi.org/project/name-matching/) — Multi-metric fuzzy matcher
* [pandas](https://pandas.pydata.org/), [matplotlib](https://matplotlib.org/), [openpyxl](https://openpyxl.readthedocs.io/en/stable/)
