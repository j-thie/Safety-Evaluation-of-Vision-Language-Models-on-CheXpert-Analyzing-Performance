# Analysis Pipeline

All analysis commands are run from the repository root after inference JSON files have been generated.

The current scripts use hard-coded file lists. Edit the variables described below before running each script. The paths must point to prediction JSON files, not the original CheXpert annotation JSON, unless explicitly stated otherwise.

## Recommended output layout

```text
outputs/
├── predictions/
│   ├── medgemma/
│   └── ministral/
└── analysis/
    ├── metrics/
    ├── bootstrap/
    ├── invalid/
    ├── projection/
    └── uncertainty/
```

Most current scripts write into the working directory. Run them from the intended output directory or refactor them to accept an output directory.

## Core metrics


### `norm3_f1_acc.py`

Purpose: calculate three-choice accuracy, macro F1, and per-class F1 for categories across models and prompts.

Edit both lists under:

```python
pathways = {
    "MedGemma": [...],
    "Mistral": [...]
}
```

Run:

```bash
python norm3_f1_acc.py
```

Intended output:

```text
norm3_f1_acc.csv
```

Required fix: remove the second placeholder `raw_data` block, or replace it with:

```python
df = pd.read_csv("norm3_f1_acc.csv")
```

As uploaded, the script writes `norm3_f1_acc.csv` and then attempts to parse placeholder text as CSV.

## Bootstrap analyses

### `F1_bootstrapping.py`

Purpose: patient-level bootstrap confidence intervals for disease and projection F1, model summaries, and pairwise comparisons.

Edit:

```python
FILES = [
    "/absolute/path/to/prompt_a.json",
    "/absolute/path/to/prompt_b.json",
    ...
]
```

Run:

```bash
python F1_bootstrapping.py
```

Outputs:

```text
results_per_file_disease.csv
results_model_summary_disease.csv
results_pairwise_significance_disease.csv
results_per_file_projection.csv
results_model_summary_projection.csv
results_pairwise_significance_projection.csv
```

Required fixes:

1. `parse_name()` assumes exactly three underscore-separated filename components. Replace it with a regular-expression parser or explicit metadata.
2. `paired_bootstrap_diff()` must use the intersection of patient IDs present in both files.
3. Add `zero_division=0` to all `f1_score()` calls.
4. Confirm that every paired file contains the same patients and predictions in the same evaluation condition.



### `invalid_per_gender.py`

Purpose: count invalid responses by category and recorded sex.

Edit:

```python
MEDGEMMA_FILES = [...]
MISTRAL_FILES = [...]
```

Run:

```bash
python invalid_per_gender.py
```

Output:

```text
invalid_summary.csv
```

Required improvement: add rows for prompts with zero invalid responses. The current script creates rows only for categories containing at least one `INVALID`.

## Unknown and invalid response analyses

### `invlaid_calculations3.py`

Purpose: calculate cross-prompt output variability among cases where at least one prompt answer differs from the expected label.

Edit all `EXPERIMENTS` file lists.

Run:

```bash
python invlaid_calculations.py
python invlaid_calculations3.py
```

Current output: console only.

Important interpretation: these scripts do not restrict the analysis to parser value `INVALID`; they include any case with at least one incorrect answer. Rename the metric/documentation to **incorrect-case variability**, or change the filter to:

```python
has_invalid = any(answer == "INVALID" for answer in answers)
```

Required key fix: include the question identity in the comparison key. The current key `(patient_id, image_name, category)` can overwrite multiple questions from the same category. Prefer:

```python
key = (
    patient_id,
    image_name,
    pred["category"],
    pred["question"],
)
```

Correct the filenames:

```text
invalid_calculations.py
invalid_calculations_3choice.py
```

## Prompt uncertainty and consistency

### `entropy_cal_3.py`

Purpose: normalized entropy and invalid-response rate for three-choice outputs.

Edit `EXPERIMENTS` and confirm:

```python
SCENARIO_CLASSES = {
    "Normal": 4,
    "No Image Unknown": 4,
    "No Image Known": 4,
}
```

Four classes means `0`, `1`, `2`, and `INVALID`. Use three only if `INVALID` is excluded rather than treated as a class.

Run:

```bash
python entropy_cal_3.py
```

Current output: console only.

Required fixes:

- use common keys across all prompt files;
- replace `question_idx` with category/question identity;
- remove the duplicated full analysis loop;
- save results to a CSV.

###  `SD_Comparision_3.py`

Purpose: mean standard deviation of encoded prompt answers.

Edit all `EXPERIMENTS` file lists and run:

```bash
python SD_Comparision.py
python SD_Comparision_3.py
```

Current output: console only.

Required fixes:

- use common keys across prompt files;
- replace `question_idx` with category/question identity;
- document the numeric encoding, because standard deviation on categorical codes depends on the arbitrary class numbers;
- consider keeping entropy as the primary categorical disagreement measure.

Recommended filenames:

```text
prompt_sd_three_choice.py
```

## Projection analyses

### `count_zeros_for_frontal_lateral.py`

Purpose: count predicted zeros, expected zeros, invalid outputs, and total `Frontal_Lateral` questions.

Edit:

```python
FILES = [...]
```

Run:

```bash
python count_zeros_for_frontal_lateral.py
```

Current output: console only.

Recommended change: save one row per prompt to `frontal_lateral_zero_counts.csv`.



## Dependencies

The model requirements files do not currently include all analysis dependencies. Create a separate pinned file:

```text
analysis_requirements.txt
```

It must include exact tested versions of:

```text
numpy
pandas
scipy
scikit-learn
```

Do not copy arbitrary current versions. Export the versions from the environment used to produce the paper results.



