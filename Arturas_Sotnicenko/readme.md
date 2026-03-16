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
- `macro_indicator_notebook_helpers.py`: shared notebook context builders and display helpers for multi-country notebook sections.
- `macro_indicator_adapters.py` / `macro_indicator_constants.py`: source adapters, templates, and mapping registries.

## Baltic Output Layout

The Baltic batch layout is reserved as a separate additive output root and does not replace the existing Lithuania default output directory.

- Baltic output root:
  - `bs2026_student_projects/Arturas_Sotnicenko/outputs_baltics`
- Per-country directories:
  - `bs2026_student_projects/Arturas_Sotnicenko/outputs_baltics/LT`
  - `bs2026_student_projects/Arturas_Sotnicenko/outputs_baltics/LV`
  - `bs2026_student_projects/Arturas_Sotnicenko/outputs_baltics/EE`
- Root-level batch summary artifacts:
  - `bs2026_student_projects/Arturas_Sotnicenko/outputs_baltics/country_run_summary.csv`
  - `bs2026_student_projects/Arturas_Sotnicenko/outputs_baltics/country_acceptance_summary.csv`

This path convention is the contract for upcoming batch-runner work.

## Baltic Batch API

The batch orchestration entrypoint is:

```python
from macro_indicator_pipeline import run_country_batch

batch = run_country_batch(
    country_codes=("LT", "LV", "EE"),
    outdir_root="bs2026_student_projects/Arturas_Sotnicenko/outputs_baltics",
)
```

The batch result exposes:
- `country_results`: per-country pipeline results keyed by country code
- `country_output_dirs`: resolved output directory per country
- `country_run_summary`: one summary row per country
- `country_acceptance_summary`: acceptance checks stacked across countries

Calling `run_country_batch(...)` also persists the batch-level summary artifacts to the Baltic output root:
- `country_run_summary.csv`
- `country_acceptance_summary.csv`

Batch reuse remains isolated per country output directory, and cached reuse is validated against the run metadata contract, including `country_code` and `chart_profile`.

## Baltic Notebook Helpers

The shared notebook helper path for the Baltic notebook is:

```python
from macro_indicator_pipeline import render_country_notebook_section

country_context = render_country_notebook_section(
    country_code="LT",
    country_results=batch["country_results"]["LT"],
    country_output_dir=batch["country_output_dirs"]["LT"],
)
```

The notebook helper surface also exposes:
- `DEFAULT_NOTEBOOK_INDICATOR_GROUPS`
- `build_country_notebook_context(...)`
- `display_country_diagnostics(...)`
- `render_country_indicator_groups(...)`
- `display_country_zscore_visuals(...)`

These helpers render from the already-produced batch result tables and chart manifests; they do not trigger a second data collection pass.

## Baltic Notebooks

The Baltic notebooks live at:
- `bs2026_student_projects/Arturas_Sotnicenko/Macroeconomic_timeseries_baltics.ipynb`
- `bs2026_student_projects/Arturas_Sotnicenko/Macroeconomic_timeseries_baltics_normalized.ipynb`

`Macroeconomic_timeseries_baltics.ipynb` runs one Baltic batch with `chart_profile="harmonized_overlay"` and renders separate Lithuania, Latvia, and Estonia sections through `render_country_notebook_section(...)`.

`Macroeconomic_timeseries_baltics_normalized.ipynb` reuses the same Baltic batch outputs but renders only the normalized z-score visuals for each country through `build_country_notebook_context(...)` and `display_country_zscore_visuals(...)`.
