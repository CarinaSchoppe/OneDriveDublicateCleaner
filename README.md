# OneDrive Duplicate Cleaner

A practical, safety-first Jupyter workflow to clean up large OneDrive folders.

This project helps you:
- detect exact duplicate files,
- detect near-duplicate images,
- detect AI-based visually similar image bursts,
- quarantine AI-grouped duplicates instead of deleting them,
- rename cryptic image filenames to readable timestamp names,
- export detailed CSV reports for every plan and execution step.

Primary notebook: `onedrive_duplicate_cleaner.ipynb`

## What It Does

### 1) Exact duplicate detection
- Scans files recursively under `ROOT_DIR`.
- Groups candidates by file size, then confirms duplicates with SHA-256.
- Keeps one file per group and plans deletion for the rest.

### 1B) Near-duplicate image detection (dHash)
- Computes 64-bit perceptual hashes.
- Compares only nearby files in time and within the same resolution bucket for speed.
- Flags similar images without deleting them.

### 1C) AI visual grouping (CLIP)
- Uses `openai/clip-vit-base-patch32` (or a custom CLIP model name).
- Finds visually similar burst/group-photo variants with small motion changes.
- Supports threshold as either:
  - float in `[0..1]` via `AI_SIMILARITY_THRESHOLD`, or
  - percent via `AI_SIMILARITY_PERCENT` (e.g. `93.0`).
- Includes verbose scan/batch/pair logs.

### 1D) AI quarantine move
- Builds a move plan from AI groups (keep one, move the rest).
- Moves files to quarantine instead of deleting:
  - `_duplicate_cleaner_quarantine/ai_visual`

### 2) Duplicate deletion (optional)
- Executes deletion plan only when safety conditions are explicitly met.

### 3) Cryptic image rename (optional)
- Renames cryptic image filenames into timestamp-based names.
- Always previews first, then optionally executes.

### 5 + 6B) Reporting
- Writes CSV reports for plans and executions into:
  - `_duplicate_cleaner_reports`

## Safety Model

Destructive actions are blocked unless all safety switches are set correctly.

Execution gate requires:
1. `DRY_RUN = False`
2. `CONFIRM_EXECUTION = True`
3. `CONFIRM_TEXT = "JA_LOESCHEN_UND_UMBENENNEN"`

Plus action-specific flags:
- delete duplicates: `DELETE_DUPLICATES = True`
- rename images: `RENAME_IMAGES = True`
- move AI groups to quarantine: `MOVE_AI_GROUPS_TO_QUARANTINE = True`

Recommended workflow:
1. Keep `DRY_RUN=True` and run preview phases first.
2. Review on-screen output and CSV reports.
3. Enable only the action(s) you actually want to execute.

## Requirements

- Python 3.10+
- Jupyter Notebook / JupyterLab
- OS: tested in Windows/OneDrive-style setups

Install base dependencies:
```bash
pip install pillow
```

Install AI dependencies (for Phase 1C):
```bash
pip install torch transformers tqdm ipywidgets
```

Notes:
- First AI run downloads CLIP model weights from Hugging Face.
- Without `HF_TOKEN`, downloads still work but may be slower/rate-limited.

## Quick Start

1. Open `onedrive_duplicate_cleaner.ipynb`.
2. In the configuration cell, set at least:
   - `ROOT_DIR`
   - `DRY_RUN`
3. Run preview phases in this order:
   - Phase 1
   - Phase 1B
   - Phase 1C
   - Phase 1D
   - Phase 3
4. Write reports:
   - Phase 5
   - Phase 6B
5. If everything looks correct, enable execution flags and run the required execution phases.

## Key Configuration

### Core
- `ROOT_DIR`: root folder to scan (default: `Path.home() / "OneDrive"`)
- `DRY_RUN`: global safety mode
- `SHOW_PREVIEW_DETAILS`: print detailed previews
- `EXCLUDE_DIR_NAMES`: folders ignored during recursion
- `INCLUDE_EXTENSIONS`: optional exact-duplicate filter by extension

### Near-duplicates (1B)
- `FIND_SIMILAR_IMAGES`
- `SIMILAR_HASH_THRESHOLD` (0..64, lower is stricter)
- `SIMILAR_NEIGHBOR_WINDOW`
- `SIMILAR_MIN_FILE_SIZE_BYTES`

### AI visual groups (1C)
- `FIND_AI_VISUAL_GROUPS`
- `AI_MODEL_NAME`
- `AI_SIMILARITY_THRESHOLD` (0..1)
- `AI_SIMILARITY_PERCENT` (e.g. `93.0`, overrides threshold)
- `AI_NEIGHBOR_WINDOW`
- `AI_MIN_FILE_SIZE_BYTES`
- `AI_BATCH_SIZE`
- `AI_VERBOSE`
- `AI_VERBOSE_PAIR_LOG_LIMIT`

### Quarantine (1D)
- `MOVE_AI_GROUPS_TO_QUARANTINE`
- `AI_QUARANTINE_DIR`

### Execution controls
- `DELETE_DUPLICATES`
- `RENAME_IMAGES`
- `CONFIRM_EXECUTION`
- `CONFIRM_TEXT`

## Generated Reports

Reports are timestamped and written to:
- `ROOT_DIR/_duplicate_cleaner_reports`

### Phase 5
- `duplicates_plan_YYYYMMDD_HHMMSS.csv`
- `renames_plan_YYYYMMDD_HHMMSS.csv`
- `similar_images_plan_YYYYMMDD_HHMMSS.csv`
- `similar_images_pairs_YYYYMMDD_HHMMSS.csv`
- `duplicates_executed_YYYYMMDD_HHMMSS.csv`
- `renames_executed_YYYYMMDD_HHMMSS.csv`

### Phase 6B
- `ai_visual_groups_plan_YYYYMMDD_HHMMSS.csv`
- `ai_visual_groups_pairs_YYYYMMDD_HHMMSS.csv`
- `ai_quarantine_plan_YYYYMMDD_HHMMSS.csv`
- `ai_quarantine_executed_YYYYMMDD_HHMMSS.csv`

## Project Layout

- `onedrive_duplicate_cleaner.ipynb` - main workflow notebook
- `models/` - optional local model assets (if used)

## Troubleshooting

### AI phase loads but finds nothing
- Lower strictness:
  - reduce `AI_SIMILARITY_THRESHOLD` (or `AI_SIMILARITY_PERCENT`)
  - increase `AI_NEIGHBOR_WINDOW`

### Too many false positives
- Increase strictness:
  - raise `AI_SIMILARITY_THRESHOLD` / `AI_SIMILARITY_PERCENT`
  - reduce `AI_NEIGHBOR_WINDOW`

### iProgress warning in notebook
- Install/update widgets:
```bash
pip install -U ipywidgets
```

### Pillow missing
- Install:
```bash
pip install pillow
```

## Important Notes

- This workflow is intentionally conservative and preview-first.
- AI quarantine is safer than direct deletion for uncertain visual matches.
- Always keep backups for critical folders before live execution.
