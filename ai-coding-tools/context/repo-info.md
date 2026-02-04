# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

metric-hub is Mozilla's central source of truth for metric definitions and metadata — a "semantic layer" between the data warehouse and analysis tools. TOML configuration files define metrics, data sources, dimensions, and segments for Mozilla products (Firefox Desktop, Fenix, Firefox iOS, Focus, Klar, etc.).

These definitions are consumed by:
- **Jetstream** — automated experiment analysis
- **OpMon** — operational monitoring for rollouts and product health
- **Looker** — auto-generated Explores with metrics as measures
- **BigQuery ETL** — data warehouse query generation

## Repository Structure

- `definitions/` — Core metric definitions (source of truth). Platform-specific TOML files (e.g., `firefox_desktop.toml`, `fenix.toml`).
- `jetstream/` — Experiment analysis configs. Individual experiment TOML files plus `defaults/`, `definitions/`, and `outcomes/` subdirectories.
- `opmon/` — Operational monitoring configs. Project TOML files plus `defaults/` and `definitions/`.
- `looker/` — Looker statistics definitions in `definitions/`.
- `lib/metric-config-parser/` — Python library that parses and validates all TOML configs. Published to PyPI as `mozilla-metric-config-parser`.
- `.script/` — Validation and documentation generation scripts (require GCP credentials for dry-run).
- `.docs/` — MkDocs-based documentation site.

## Config Precedence

Tool-specific configs (`jetstream/`, `opmon/`, `looker/`) override `definitions/` when used in their respective tool. In all other contexts, `definitions/` is the source of truth.

## Development Commands (metric-config-parser)

All commands run from `lib/metric-config-parser/`:

```bash
# Setup
python -m venv venv/
venv/bin/python -m pip install --upgrade pip
venv/bin/pip install --progress-bar off --upgrade -r requirements.txt

# Run unit tests (excludes integration tests)
venv/bin/pytest --ruff --ignore=metric_config_parser/tests/integration/

# Run integration tests only
venv/bin/pytest --ruff metric_config_parser/tests/integration/

# Run a single test file
venv/bin/pytest metric_config_parser/tests/test_config.py

# Run a single test
venv/bin/pytest metric_config_parser/tests/test_config.py::test_function_name

# Lint
venv/bin/ruff check metric_config_parser
venv/bin/ruff format --check metric_config_parser

# Type check
venv/bin/mypy metric_config_parser
```

## Linting Rules

Configured in `lib/metric-config-parser/pyproject.toml`. Ruff with line-length 100, target Python 3.11. Key rule sets: B, C4, E, F, I, PT, Q, RUF, SIM, TC, TID, UP, W. Ignored: E741, RUF005, RUF012, SIM105.

## CI/CD Behavior

- Changes to `definitions/` trigger dry-run SQL validation against BigQuery.
- Changes to `jetstream/` trigger jetstream container validation and may **automatically rerun experiment analysis** for live experiments and recently-completed experiments.
- Changes to `opmon/` trigger opmon container validation.
- Changes to `lib/metric-config-parser/` trigger pytest, ruff, and mypy.
- Add `[ci rerun-skip]` to a commit message to prevent automatic experiment reruns (recommended for `jetstream/defaults/` changes to avoid expensive reruns).

## TOML Config Format

Metrics are defined in TOML with sections like `[metrics.metric_name]` containing `select_expression`, `data_source`, `friendly_name`, `description`, `category`, and `type`. Select expressions use Jinja2 templates (e.g., `'{{agg_sum("active_hours_sum")}}'`). Data sources, segments, and dimensions follow analogous patterns.

## Architecture of metric-config-parser

The library in `lib/metric-config-parser/metric_config_parser/` uses `attrs` classes throughout:
- `config.py` — `ConfigCollection` loads configs from GitHub repos or local paths. Central orchestrator.
- `metric.py` — `MetricDefinition` for individual metrics.
- `data_source.py` — `DataSourceDefinition` for BigQuery table references.
- `analysis.py` — `AnalysisSpec` for Jetstream experiment analysis configuration.
- `monitoring.py` — `MonitoringSpec` for OpMon configuration.
- `experiment.py` — `Experiment` model and `Channel` enum.
- `outcome.py` — `OutcomeSpec` for experiment outcome groupings.
- `segment.py` — `SegmentDefinition` and `SegmentDataSourceDefinition`.
- `function.py` — `FunctionsSpec` for Jinja2 template functions (defined in `functions.toml`).
- `sql.py` — SQL generation from metric/data source definitions.
- `cli.py` — Click-based CLI entry point (`metric-config-parser` command).
