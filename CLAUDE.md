# CLAUDE.md

This file provides guidance for Claude Code when working with the EMHASS codebase.

## Project Overview

EMHASS (Energy Management for Home Assistant) is a Python-based optimization tool for residential energy management. It uses Linear Programming to optimize home energy usage considering electricity prices, solar PV generation, and battery storage.

## Project Structure

```
src/emhass/          # Main package source
  command_line.py    # CLI entry point and main orchestration
  optimization.py    # Linear programming optimization engine
  forecast.py        # PV, load, and price forecasting
  retrieve_hass.py   # Home Assistant data retrieval
  utils.py           # Utility functions and configuration handling
  web_server.py      # Quart-based web server (REST API)
  machine_learning_forecaster.py  # ML-based load forecasting
  machine_learning_regressor.py   # ML regression utilities
  connection_manager.py           # Connection management
  websocket_client.py             # WebSocket client
  data/              # Static data files (PV modules, inverters, config defaults)
  templates/         # Jinja2 HTML templates
  static/            # Static web assets
tests/               # Test suite (pytest)
scripts/             # Development and analysis scripts
docs/                # Sphinx documentation
```

## Development Setup

**Python Version:** 3.12 (supports 3.10-3.12)

```bash
# Install with uv (recommended)
uv sync

# Or with pip in virtual environment
python -m venv .venv
source .venv/bin/activate
pip install -e ".[test,dev]"
```

## Common Commands

```bash
# Run tests
pytest tests/

# Run tests with coverage
coverage run -m pytest tests/ && coverage report

# Lint with ruff
ruff check src/ tests/
ruff format src/ tests/

# Build Docker image
docker build -t emhass-local .

# Run web server locally
emhass --action 'dayahead-optim' --config config.json
```

## Key Architecture

### Optimization Flow
1. `command_line.py` - Entry point, parses CLI args and runtime params
2. `retrieve_hass.py` - Fetches data from Home Assistant
3. `forecast.py` - Generates PV, load, and price forecasts
4. `optimization.py` - Runs LP optimization using PuLP/HiGHS
5. `web_server.py` - Exposes REST API for Home Assistant integration

### Configuration
- `config.json` - User configuration (stored in /share for Docker)
- `secrets_emhass.yaml` - Sensitive data (HA URL, API token, location)
- `src/emhass/data/config_defaults.json` - Default configuration values

### Web Server Endpoints (Quart async)
- `POST /action/dayahead-optim` - Day-ahead optimization
- `POST /action/naive-mpc-optim` - MPC optimization
- `POST /action/perfect-optim` - Perfect optimization (historical)
- `POST /action/publish-data` - Publish results to Home Assistant
- `POST /action/forecast-model-fit` - Train ML forecaster
- `POST /action/forecast-model-predict` - ML forecast prediction

## Testing

Tests use pytest with fixtures in individual test files. Key test files:
- `test_optimization.py` - Optimization engine tests
- `test_forecast.py` - Forecasting tests
- `test_web_server.py` - Web API tests
- `test_utils.py` - Utility function tests

## Code Style

- **Linting:** ruff (see `[tool.ruff]` in pyproject.toml)
- **Line length:** 100 characters
- **Target Python:** 3.11+
- Uses type hints throughout

## Dependencies

Key dependencies:
- `pulp` + `highspy` - Linear programming optimization
- `pandas` + `numpy` - Data manipulation
- `pvlib` - Solar PV modeling
- `skforecast` - Time series forecasting
- `quart` - Async web framework
- `plotly` - Visualization
