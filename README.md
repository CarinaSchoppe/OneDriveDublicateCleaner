# OneDrive Duplicate Cleaner

A safety-first Jupyter workflow for cleaning large OneDrive folders.

Primary notebook: `onedrive_duplicate_cleaner.ipynb`

## Features

- Exact duplicate detection (size + SHA-256)
- Near-duplicate image detection via dHash
- AI visual similarity grouping via CLIP
- Optional exhaustive AI comparison mode (all image pairs)
- AI-assisted semantic image renaming (free local open-source model inference)
- Optional AI-assisted semantic video renaming from sampled video frames
- AI quarantine move (keep one, move similar images)
- Two final CSV reports (concise output)

## How Good Is The AI Similarity Detection?

It is strong for practical cleanup, but no model is perfect.

To maximize recall:

- set `AI_COMPARE_ALL_PAIRS = True` (every image vs every image)
- start with `AI_SIMILARITY_PERCENT = 92.0`
- keep `AI_VERBOSE = True`

Tradeoff:

- exhaustive mode can be much slower on large libraries (`O(n^2)` comparisons)

## AI Semantic Rename (Images + Optional Videos)

The rename phase can add AI labels to timestamp-based filenames.

Example:

- from: `PXL_20240123_123456.jpg`
- to: `2024-01-23_12-34-56_DogChildrenBeach.jpg`

Video example (frame-sampled content description):

- from: `VID_20240123_123456.mp4`
- to: `2024-01-23_12-34-56_A_person_walking_on_a_beach_at_sunset.mp4`

Rules:

- uses local inference with Hugging Face models (download once, then run locally)
- default primary model: `Salesforce/blip-image-captioning-large`
- fallback model: `Salesforce/blip-image-captioning-base`
- optional video labels sample multiple frames and aggregate captions
- `AI_RENAME_MAX_WORDS > 0`: CamelCase label with max words
- `AI_RENAME_MAX_WORDS = 0`: full mini-sentence label (underscore style)
- filename collisions are auto-resolved with `_1`, `_2`, ...

## Safety Model

Destructive actions are blocked unless all gates are open.

Execution gate requires all:
1. `DRY_RUN = False`
2. `CONFIRM_EXECUTION = True`
3. `CONFIRM_TEXT = "YES_DELETE_AND_RENAME"`

And action flags:
- delete duplicates: `DELETE_DUPLICATES = True`
- rename images: `RENAME_IMAGES = True`
- move AI groups to quarantine: `MOVE_AI_GROUPS_TO_QUARANTINE = True`

## Requirements

- Python 3.10+
- Jupyter Notebook / JupyterLab

Install base:
```bash
pip install pillow
```

Install AI stack:
```bash
pip install torch transformers tqdm ipywidgets
```

For optional video content renaming:

```bash
pip install opencv-python
```

Optional (better Hugging Face rate limits):

- set `HF_TOKEN`

## Recommended Starting Profile

For high recall and strong rename quality:

- `AI_COMPARE_ALL_PAIRS = True`
- `AI_SIMILARITY_PERCENT = 92.0`
- `AI_VERBOSE = True`
- `RENAME_IMAGES = True`
- `RENAME_WITH_AI_LABEL = True`
- `AI_RENAME_MODEL_NAME = "Salesforce/blip-image-captioning-large"`
- `AI_RENAME_MODEL_FALLBACK = "Salesforce/blip-image-captioning-base"`
- `AI_RENAME_MAX_WORDS = 8` (or `0` for full mini-sentence)
- `RENAME_ALL_IMAGES_WITH_AI_LABEL = False` (set `True` to rename all images)

## Quick Start

1. Open `onedrive_duplicate_cleaner.ipynb`
2. Set `ROOT_DIR`
3. Run preview phases:
   - Phase 1 (duplicates)
   - Phase 1B (near-duplicates)
   - Phase 1C (AI visual grouping)
   - Phase 1D (AI quarantine plan)
   - Phase 3 (rename preview)
4. Run Phase 5 to generate the two final reports
5. If output is correct, enable execution flags and run live phases

## CLI Setup Wizard

Use the interactive CLI to configure settings without editing notebook code manually.

Show all settings, explanations, and defaults:

```bash
python duplicate_cleaner_cli.py show
```

Run guided setup (yes/no + numeric/text prompts, with defaults):

```bash
python duplicate_cleaner_cli.py wizard
```

Write config to a custom path:

```bash
python duplicate_cleaner_cli.py wizard --config my_config.json
```

Default output file:

- `duplicate_cleaner_cli_config.json`

Notebook integration:

- The configuration cell auto-loads `duplicate_cleaner_cli_config.json` if it exists.
- Values from `settings` override notebook defaults.
- `AI_QUARANTINE_DIR` is re-derived from `ROOT_DIR` if not explicitly set in config.

## Key Configuration

### Core

- `ROOT_DIR`
- `DRY_RUN`
- `SHOW_PREVIEW_DETAILS`
- `EXCLUDE_DIR_NAMES`

### Near-duplicates
- `FIND_SIMILAR_IMAGES`
- `SIMILAR_HASH_THRESHOLD`
- `SIMILAR_NEIGHBOR_WINDOW`
- `SIMILAR_MIN_FILE_SIZE_BYTES`

### AI visual grouping
- `FIND_AI_VISUAL_GROUPS`
- `AI_MODEL_NAME`
- `AI_SIMILARITY_THRESHOLD` (0..1 fallback)
- `AI_SIMILARITY_PERCENT` (preferred)
- `AI_NEIGHBOR_WINDOW`
- `AI_COMPARE_ALL_PAIRS` (`True` = exhaustive)
- `AI_BATCH_SIZE`
- `AI_VERBOSE`
- `AI_VERBOSE_PAIR_LOG_LIMIT`

### AI semantic rename
- `RENAME_IMAGES`
- `RENAME_WITH_AI_LABEL`
- `AI_RENAME_MODEL_NAME`
- `AI_RENAME_MODEL_FALLBACK`
- `AI_RENAME_MAX_WORDS` (`0` = full mini-sentence)
- `AI_RENAME_MAX_CHARS`
- `AI_ENRICH_DATE_ONLY_NAMES`
- `RENAME_ALL_IMAGES_WITH_AI_LABEL`
- `RENAME_VIDEOS`
- `RENAME_WITH_AI_VIDEO_LABEL`
- `RENAME_ALL_VIDEOS_WITH_AI_LABEL`
- `AI_VIDEO_SAMPLE_FRAMES`
- `AI_VIDEO_MIN_FRAME_INTERVAL_SECONDS`
- `AI_VIDEO_FRAME_MAX_DIM`
- `RENAME_VERBOSE_SUGGESTIONS`
- `PREVIEW_RENAME_SUGGESTIONS_WHEN_RENAME_DISABLED`

## In-Notebook Documentation Helpers

The notebook includes explicit documentation and sanity-check helpers:

- `show_settings_reference()`
- `show_method_reference()`
- `validate_configuration()`

Use these after the helper cell is executed to inspect every setting/method with examples.

## Final Reports (Only 2 Files)

Written to:
- `ROOT_DIR/_duplicate_cleaner_reports`

Files:

1. `rename_and_similarity_report_<timestamp>.csv`
   - rename-required entries
   - similar-image entries with `%` similarity
2. `duplicates_keep_delete_report_<timestamp>.csv`
   - duplicate group
   - keep file
   - delete file list (`x | y | z`)

## Troubleshooting

### `IProgress not found`
```bash
pip install -U ipywidgets tqdm
```

### Hugging Face unauthenticated warning

- optional only; set `HF_TOKEN` for better limits/speed

### AI too strict / too loose

- stricter: increase `AI_SIMILARITY_PERCENT`
- broader: decrease `AI_SIMILARITY_PERCENT`
- exhaustive: set `AI_COMPARE_ALL_PAIRS = True`

## Notes

- Use preview-first workflow for safety.
- For important data, keep backups before live execution.

## Preview-only logging mode

You can keep renaming disabled and still get full AI rename suggestions in logs.

Set:

- `RENAME_IMAGES = False`
- `PREVIEW_RENAME_SUGGESTIONS_WHEN_RENAME_DISABLED = True`
- `RENAME_VERBOSE_SUGGESTIONS = True`

Then Phase 3 logs lines like:

- source image path
- current filename
- proposed new AI-based filename
- proposed target path
- optional raw AI caption

No file is renamed until `RENAME_IMAGES=True` and execution gates are satisfied.
