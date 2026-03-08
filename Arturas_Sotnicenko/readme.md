# Arturas Sotnicenko - Data Analytics Workspace

This folder contains a minimal data analytics setup with a devcontainer, dependencies, and a starter notebook.

## Quick start

1. Open this folder in VS Code: `Arturas_Sotnicenko`.
2. Reopen in container when prompted.
3. Open the starter notebook and run the cells.

## Contents

- `.devcontainer/devcontainer.json` - Devcontainer configuration.
- `requirements.txt` - Python dependencies.
- `notebooks/starter.ipynb` - Starter analysis notebook.
- `macro_indicator_pipeline.py` (project root) - Builds indicator index + source coverage tables and flags missing dataset codes/keys.

## Indicator Coverage Pipeline

Build indicator/source coverage files:

```bash
python macro_indicator_pipeline.py \
  --country-code LT \
  --outdir bs2026_student_projects/Arturas_Sotnicenko/outputs

# Shareable overlay report is written to:
# bs2026_student_projects/Arturas_Sotnicenko/outputs/chart_share_report.html
# Share metadata is written to:
# bs2026_student_projects/Arturas_Sotnicenko/outputs/share_manifest.csv
```

Notebook usage:

```python
from macro_indicator_pipeline import run, stream_resolved_sources

results = run(country_code="LT", outdir="bs2026_student_projects/Arturas_Sotnicenko/outputs")
display(results["indicator_index"])
display(results["coverage"].head(20))
display(results["missing_queue"].head(20))

# Replacement for notebook cell 6 (resolve + stream currently resolved endpoints):
stream = stream_resolved_sources(country_code="LT", coverage=results["coverage"])
display(stream["endpoint_status_df"])
# Example loaded dataset:
first_key = next(iter(stream["normalized_sources"]))
display(stream["normalized_sources"][first_key].head())
```

## Default Chart Surface Contract

The current Lithuania baseline for `run(country_code="LT", include_analysis=True)` is the future `full` chart profile contract for the Baltic refactor.

- Per-indicator chart outputs currently include:
  - `raw`
  - `raw_harmonized`
  - `overlay_index`
- Z-score chart outputs currently include:
  - `zscore_heatmap`
  - `zscore_timelines`
- Share/report outputs currently include:
  - `chart_share_report.html`
  - `share_manifest.csv`

This baseline must remain the default behavior until the chart-profile seam is introduced. Any later compact Baltic view must be additive and optional, not a change to the existing Lithuania default output surface.

## Internal Module Layout

The current pipeline is organized so notebook-facing imports stay stable while internal responsibilities are split by concern:

- `macro_indicator_pipeline.py`: public compatibility facade for notebooks and scripts.
- `macro_indicator_country_profiles.py`: country metadata, ISO lookups, and country-specific harmonization hooks.
- `macro_indicator_helpers.py`: URL resolution, parsing helpers, and source normalization utilities.
- `macro_indicator_core.py`: orchestration, harmonization, chart generation, and cache validation.
- `macro_indicator_adapters.py` / `macro_indicator_constants.py`: source adapters, templates, and mapping registries.
