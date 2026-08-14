# Working, but Uncovered

**An R-first, reproducible demonstration of how IPUMS ACS microdata can be turned into statistically responsible Texas health-coverage estimates, GIS deliverables, and Flourish-ready public visualizations.**

> **Data status:** This repository ships with **synthetic microdata and synthetic test geometry only** so that the complete workflow can be reviewed without redistributing restricted IPUMS records. Nothing in `demo/` is a real estimate. The production path requests Texas-only IPUMS extracts and downloads authoritative 2020-vintage PUMA geography.

## Why this project exists

The project is deliberately shaped like a small research engagement rather than a one-off notebook. It demonstrates the ability to:

- define and preserve IPUMS extract specifications in code;
- work in R with ACS person weights and all 80 replicate weights;
- derive disability from the six contemporary ACS difficulty measures and test IPUMS source-code direction explicitly;
- produce statewide and PUMA-level estimates with uncertainty and reliability flags;
- join estimates to Census geography and export ArcGIS/QGIS-compatible files;
- prepare tidy, documented tables and GeoJSON for Flourish;
- use modular functions, dependency-aware pipelines, tests, CI, version control, and research documentation;
- protect restricted microdata while making the analytical workflow reproducible.

The substantive proof of concept asks:

> Among Texans ages 19–64, where are health-insurance gaps concentrated, and how do those gaps differ by employment, family income, race and ethnicity, citizenship, disability, age, and geography?

## What is included

| Layer | Implementation |
|---|---|
| Data acquisition | `ipumsr::define_extract_micro()` with Texas case selection, saved JSON definitions, API submission, wait, and download workflow |
| Analysis | 2024 ACS one-year statewide validation plus 2020–2024 ACS five-year PUMA analysis |
| Survey statistics | `survey::svrepdesign(type = "ACS")`, `PERWT`, `REPWTP1`–`REPWTP80`, 90% confidence intervals |
| GIS | `sf`, 2020-vintage PUMAs, validated joins, GeoPackage, simplified GeoJSON, zipped shapefile fallback |
| Visualization handoff | Flourish-ready CSVs, custom GeoJSON, tooltip fields, field-binding instructions |
| Reproducibility | `targets`, `renv` bootstrap, Quarto report, synthetic fixtures, `testthat`, GitHub Actions |
| Governance | no raw IPUMS data in Git; extract definitions, metadata, methods, decisions, and aggregated outputs only |

## Repository tour

```text
.
├── _targets.R                     # dependency graph and pipeline entry point
├── R/                             # small, testable analytical modules
├── scripts/                       # bootstrap, extract, run, render, and validation commands
├── data-synthetic/                # clearly marked synthetic fixtures
├── data-raw/                      # ignored location for licensed IPUMS downloads
├── docs/                          # methods, decisions, data dictionary, GIS and Flourish handoffs
├── report/                        # Quarto analytical report
├── tests/testthat/                # unit and integration tests
├── demo/                          # precomputed synthetic outputs and visual previews
├── dev/                           # deterministic fixture generator and repository validator
├── .python-version / requirements-dev.txt  # pinned Python review-artifact toolchain
└── .github/workflows/ci.yml       # Python reproducibility plus R pipeline and report checks
```

## Fastest review path

A reviewer can understand the work without running anything:

1. Read [`docs/PORTFOLIO_WALKTHROUGH.md`](docs/PORTFOLIO_WALKTHROUGH.md).
2. Inspect [`R/ipums_extract.R`](R/ipums_extract.R) and [`R/survey_design.R`](R/survey_design.R).
3. Inspect [`R/estimate_uninsurance.R`](R/estimate_uninsurance.R), [`R/geography.R`](R/geography.R), and [`R/export_flourish.R`](R/export_flourish.R).
4. Open [`demo/index.html`](demo/index.html), then inspect the upload-ready files in [`demo/flourish/`](demo/flourish/) and [`demo/gis/`](demo/gis/).
5. Review [`docs/METHODS.md`](docs/METHODS.md), [`docs/PROJECT_HANDOFF.md`](docs/PROJECT_HANDOFF.md), and the CI workflow.

## Run the synthetic demonstration

Requirements: R 4.4 or newer, a current Quarto installation, and system libraries required by `sf`.

```bash
Rscript scripts/bootstrap.R
Rscript scripts/run_pipeline.R
Rscript scripts/validate_outputs.R
Rscript scripts/render_report.R
```

An isolated, fully pinned Python development environment can independently regenerate and validate the public review bundle without R:

```bash
python3.13 -m venv .venv
.venv/bin/python -m pip install --requirement requirements-dev.txt
.venv/bin/python dev/check_fixture_reproducibility.py
.venv/bin/python dev/validate_repository.py --check
```

`check_fixture_reproducibility.py` regenerates every committed file under `data-synthetic/` and `demo/` twice and compares SHA-256 hashes. The validator checks schemas, joins, GIS formats, synthetic-data disclosures, deterministic GeoPackage and shapefile metadata, documentation links, credential hygiene, and the committed validation reports. Run `.venv/bin/python dev/validate_repository.py` without `--check` only when intentionally refreshing those reports.

The default `config.yml` uses `project.data_mode: synthetic`. No account, API key, or restricted data are needed.

## Run with real IPUMS data

1. Register for IPUMS USA and obtain an API key.
2. Copy `.Renviron.example` to `.Renviron` and add the key, or set `IPUMS_API_KEY` in your shell.
3. Review the variable list and universe in [`docs/IPUMS_SETUP.md`](docs/IPUMS_SETUP.md).
4. Define, submit, wait for, and download both extracts:

```bash
Rscript scripts/request_ipums_extracts.R
```

5. Change `project.data_mode` in `config.yml` from `synthetic` to `ipums`, or run:

```bash
DATA_MODE=ipums Rscript scripts/run_pipeline.R
DATA_MODE=ipums Rscript scripts/validate_outputs.R
Rscript scripts/render_report.R
```

The extract definition requests:

- **`us2024a`** for the statewide 2024 one-year estimate;
- **`us2024c`** for the 2020–2024 five-year PUMA analysis;
- Texas records only through a `STATEFIP == 48` IPUMS case selection;
- `PERWT` and the `REPWTP` group, which expands to all 80 person replicate weights.

Raw downloads remain under `data-raw/ipums/` and are excluded from version control.

## Principal outputs

After a successful run, `outputs/` contains:

```text
outputs/
├── flourish/
│   ├── statewide_summary.csv
│   ├── puma_map_data.csv
│   ├── puma_rankings.csv
│   ├── subgroup_estimates.csv
│   ├── puma_regions.geojson
│   └── manifest.json
├── gis/
│   ├── tx_puma_uninsurance.gpkg
│   ├── tx_puma_uninsurance.geojson
│   ├── tx_puma_uninsurance_shapefile.zip
│   └── field_dictionary.csv
├── tables/
│   ├── statewide_one_year.csv
│   ├── statewide_five_year.csv
│   ├── puma_five_year.csv
│   ├── subgroup_five_year.csv
│   └── benchmark_reconciliation.csv
└── validation/
    ├── output_schema_check.csv
    └── run_metadata.json
```

## Important analytical boundaries

- The one-year file is used for the statewide 2024 comparison; the pooled five-year file is used for PUMA estimates.
- PUMAs are not counties or census tracts. The project does not manufacture smaller-area estimates from public-use microdata.
- Replicate weights are used for variance estimation, not as 80 independent parameter estimates.
- Reliability thresholds in this demonstration are transparent project review rules, not official Census Bureau suppression standards.
- The benchmark comparison is a reconciliation exercise. A difference can reflect universe, definition, sample, or rounding choices and is not automatically an error.

## Review and handoff

- [`docs/PROJECT_HANDOFF.md`](docs/PROJECT_HANDOFF.md) distinguishes what is complete from what requires an R/IPUMS-enabled environment.
- [`docs/GIT_WORKFLOW.md`](docs/GIT_WORKFLOW.md) documents the intended branch, review, and release practice.
- [`validation/`](validation/) contains machine-readable repository checks produced by the development validator.

## Environment locking

This starter repository does not contain a fabricated `renv.lock`. `scripts/bootstrap.R` installs the discovered dependencies and creates a real lockfile on the first R-enabled run. That generated `renv.lock` and `renv/activate.R` should be reviewed and committed immediately afterward.

## License and data terms

Code and original documentation are available under the MIT License. IPUMS microdata are governed by IPUMS terms of use and are not redistributed here. Census TIGER/Line geography is a U.S. government data product. See [`docs/SOURCES_AND_CITATION.md`](docs/SOURCES_AND_CITATION.md).
