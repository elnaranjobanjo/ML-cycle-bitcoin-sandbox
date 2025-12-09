Install dependencies and run every workflow command through the Taskfile (`task <name>`).

# Bitcoin ML Lifecycle Sandbox

This sandbox ingests raw Bitcoin trading data, persists it to DuckDB, trains forecasting models, and manages the full ML lifecycle with MLflow so we can upgrade and deploy better models quickly.

## Quick Start

```bash
# show available tasks
task -l

# sync Python dependencies
task sync

# pull fresh BTC/USDT candles, store them in DuckDB, and build reports
task ingest
```

All other workflows (`track`, `register`, `api`, `mlflow-ui`, `style`) are also exposed as Taskfile entries so the project can be installed and executed in a consistent, reproducible way.

## Data Lifecycle

- **Ingestion**: `task ingest` fetches BTC/USDT minute candles from Binance (configurable via `config/bitcoin_ingest.json`) and loads them into `feature_store/bitcoin.duckdb` as typed OHLCV rows.
- **Storage & access**: DuckDB acts as the feature store so analysts and training jobs can stream historical candles without a heavy database dependency. Convenience helpers in `src/data_ingestion_service` load typed entities or provide row counts for monitoring tasks.
- **Reporting**: Every ingest run renders PDF diagnostics to `reports/ingestion/<timestamp>/report.pdf`, providing quick sanity checks on volume, price swings, and engineered features.

## Modeling and MLflow Operations

- **Training**: `task track` executes `main.py track`, runs model training jobs under MLflow, and logs metrics, parameters, and artifacts to the local `mlruns` directory (or the URIs defined in `.env`).
- **Model upgrades**: `task register` promotes a specific MLflow run into the registry so the best-performing Bitcoin model receives an alias (e.g., `production`). This enables repeatable upgrades instead of ad-hoc file copies.
- **Experiment management**: `task mlflow-ui` launches the MLflow UI for browsing experiments, comparing runs, and validating that new training sessions outperform previous baselines before promotion.

## Deployment

- `task api` starts the FastAPI service in `src/api`, automatically loading the DuckDB-backed features and the MLflow model referenced by `MODEL_URI` (defaults to `models:/bitcoin-model@production`).
- The service exposes `GET /health` for readiness checks and `POST /predict` for inference payloads shaped as `{"features": [...]}`. Update the `MODEL_URI` in your shell or `.env` to swap between staging/production models.

## Project Layout

- `src/data_ingestion_service/`: Binance client, DuckDB writers/readers, ingest CLI entry point.
- `src/reporting/`: PDF report builders, plotting helpers, and filesystem layout utilities.
- `src/ml/`: Model training code plus feature engineering utilities shared across experiments.
- `src/MLOps/`: MLflow orchestration scripts for tracking, registering, and deploying runs.
- `main.py`: Task entry points (`ingest`, `track`, `register`) wired up for Taskfile automation.

By storing Bitcoin trading data in DuckDB, iterating on models, and managing registries/deployments via MLflow, the project demonstrates a compact yet complete ML lifecycle optimized for experimentation.
