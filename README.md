# SVG Generation from Text Prompts — Submission Version

This repository contains the **submission version only** of a Kaggle notebook for generating compact SVG images from text prompts. This README documents **`svg_generator_submission_ver.ipynb`** and the accompanying **`submission.csv`** output. It intentionally does **not** describe the fixed/improved version. 

NOTE: THIS README IS AI GENERATED.

## Repository Files Covered Here

- `svg_generator_submission_ver.ipynb` — the notebook used for the submission pipeline
- `submission.csv` — the generated submission file

## Project Goal

The notebook builds a text-to-SVG generation pipeline for a Kaggle course competition. Given a natural-language prompt, the system produces a valid SVG constrained to:

- `width="256"`
- `height="256"`
- `viewBox="0 0 256 256"`
- under 16,000 characters
- safe SVG tags only

The workflow combines:

1. **SVG cleaning and validation** of training data
2. **Prompt retrieval** using TF-IDF similarity
3. **LoRA fine-tuning** of a Qwen causal language model
4. **Constrained generation with fallback logic** for more reliable submission output

## High-Level Pipeline

### 1) Environment and model discovery

The notebook is designed primarily for **Kaggle**. It searches for:

- competition data under `/kaggle/input/...`
- a usable Qwen model from several candidate model paths

It also sets seeds for reproducibility and defines training/generation hyperparameters in a `CONFIG` dictionary.

### 2) SVG sanitization and validation

Before training, the notebook cleans the training SVGs by:

- extracting the `<svg>...</svg>` region
- removing malformed or unsafe entries
- enforcing required SVG attributes
- checking allowed tags and disallowed attributes
- rejecting scripts, event handlers, and unsafe external references

If a generated SVG fails validation later, the pipeline falls back to safer alternatives.

### 3) Data preparation

The notebook loads:

- `train.csv`
- `test.csv`
- `sample_submission.csv`

Then it:

- normalizes prompts
- sanitizes SVGs
- filters invalid rows
- removes duplicate prompts
- computes SVG length statistics

### 4) Retrieval baseline

To improve robustness, the notebook creates a retrieval system using:

- word-level TF-IDF features
- character-level TF-IDF features
- cosine similarity (`linear_kernel`)

At inference time, if model generation is unusable, the pipeline retrieves the closest SVG from the cleaned training set using normalized prompt similarity plus lightweight scoring heuristics.

### 5) LoRA fine-tuning

The submission notebook fine-tunes a Qwen instruct model with PEFT/LoRA. Training examples are formatted as:

- **System:** SVG generation rules
- **User:** text prompt
- **Assistant:** target SVG

The notebook uses Hugging Face `Trainer` with a short training configuration suitable for Kaggle execution.

### 6) Batched inference and fallback logic

For each test prompt, the pipeline:

1. generates SVG text with the fine-tuned model
2. extracts the SVG block
3. sanitizes and validates it
4. if invalid, falls back to retrieved training SVG
5. if that also fails, falls back to a very simple safe SVG

This design prioritizes **submission validity** over visual complexity.

### 7) Submission export

Final results are written to:

- `submission.csv`

The notebook also runs local validation checks over the produced SVG strings before finishing.

## Main Libraries Used

- Python
- NumPy
- pandas
- PyTorch
- Hugging Face Transformers
- Hugging Face Datasets
- PEFT
- scikit-learn
- SciPy

## Important Configuration in the Submission Notebook

Some notable defaults in `svg_generator_submission_ver.ipynb`:

- max sequence length: `768`
- max train samples: `4000`
- max eval samples: `200`
- LoRA rank: `16`
- learning rate: `2e-4`
- epochs: `1`
- generation batch size: `8`
- retrieval top-k: `5`
- max new tokens: `128`

These settings reflect the **submission version** as stored in this repository.

## Expected Runtime Context

This notebook expects:

- a Kaggle-style filesystem layout
- competition CSV files present under `/kaggle/input`
- access to a compatible Qwen model
- GPU availability preferred for training/inference

Although some paths are searched dynamically, the notebook is clearly structured around running inside Kaggle.

## Output Format

The exported submission contains two columns:

- `id`
- `svg`

Each row stores a single SVG string corresponding to one test prompt.

## Notes About This README

- This README is **only for the submission version**.
- It does **not** document `svg_generator_fixed_ver.ipynb`.
- It reflects the code and artifacts currently present in this repository.

## Author

Tanvir Shahriar
