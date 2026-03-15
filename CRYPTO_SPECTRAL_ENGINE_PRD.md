# Crypto Market Spectral Decomposition Engine

## 1) One-Page Product Requirements Document (PRD)

### Product Vision
Build a production-oriented analytics engine that discovers hidden cyclic behavior and market regimes in cryptocurrency markets using spectral decomposition, then exposes interpretable signals for research, discretionary trading, and systematic strategy development.

### Problem Statement
Crypto markets are noisy, non-stationary, and regime-switching. Most retail analytics tools are trend-only and fail to separate cyclical structure from noise. Teams need a robust way to:
- identify dominant periodicities,
- quantify when cycles are stable vs decaying,
- detect regime shifts earlier than price-only indicators,
- and transform this into actionable, testable signals.

### Target Users
- **Quant researchers**: feature generation for alpha models.
- **Systematic traders**: cycle- and regime-aware strategy filters.
- **Discretionary traders/analysts**: interpretable dashboards and alerts.
- **Risk teams**: volatility regime detection and exposure gating.

### Core User Stories
1. As a quant, I can ingest OHLCV and derive rolling spectral features per asset/timeframe.
2. As a trader, I can view dominant cycle length, phase, and confidence in near real time.
3. As a PM, I can see regime labels (trend, mean-reversion, high-volatility, transition) and change points.
4. As a developer, I can access features/signals via API and backtest them quickly.

### Scope
#### In scope (MVP)
- Ingestion of historical and streaming OHLCV for top exchanges.
- Spectral decomposition pipeline (FFT + wavelet + SSA optional in v1).
- Rolling feature extraction: band power, spectral entropy, dominant frequency, phase coherence.
- Regime classification (HMM or clustering baseline).
- Alerting for regime transition and cycle-strength breakdown.
- Lightweight dashboard and REST API.

#### Out of scope (MVP)
- Automated trade execution.
- On-chain analytics.
- LLM-based strategy generation.

### Functional Requirements
- Multi-asset, multi-timeframe support (e.g., 5m, 1h, 4h, 1d).
- Configurable decomposition window length and overlap.
- Feature store for time-indexed derived signals.
- Regime engine with confidence scores and transition probabilities.
- API endpoints for latest signals + historical regime timeline.
- Dashboard: spectral heatmap, dominant cycle timeline, regime strip, alert log.

### Non-Functional Requirements
- Latency target: < 2 minutes from candle close to refreshed features (MVP).
- Reliability: > 99% scheduled job success.
- Reproducibility: deterministic pipeline configs and versioned datasets.
- Observability: structured logs, metrics, and data quality checks.

### Success Metrics
- **Signal quality**: statistically significant predictive lift vs baseline (e.g., Sharpe uplift in backtests).
- **Regime accuracy proxy**: regime labels correlate with realized volatility/trend diagnostics.
- **Operational**: < 1% missing-feature intervals per asset/timeframe.
- **User adoption**: weekly active users of dashboard/API consumers.

### Risks & Mitigations
- **Non-stationarity** -> use rolling windows + adaptive thresholds.
- **Overfitting** -> walk-forward validation, cross-asset holdouts.
- **Data gaps/outliers** -> robust cleaning and anomaly detection.
- **Interpretability concerns** -> expose feature attribution and confidence.

### MVP Exit Criteria
- End-to-end pipeline runs daily + intraday for selected assets.
- Regime labels and spectral indicators available through API and dashboard.
- Backtesting notebook demonstrates value for at least one use case (e.g., regime filter improves drawdown profile).

---

## 2) Recommended Repository Structure

```text
crypto-spectral-engine/
├─ README.md
├─ pyproject.toml
├─ requirements.txt
├─ .env.example
├─ configs/
│  ├─ data_sources.yaml
│  ├─ features.yaml
│  ├─ regimes.yaml
│  └─ alerts.yaml
├─ data/
│  ├─ raw/                 # optional local dev only
│  ├─ processed/
│  └─ artifacts/
├─ docs/
│  ├─ PRD.md
│  ├─ architecture.md
│  ├─ api_contract.md
│  └─ runbooks/
├─ src/
│  └─ spectral_engine/
│     ├─ ingestion/
│     │  ├─ exchange_clients.py
│     │  ├─ stream_consumer.py
│     │  └─ validators.py
│     ├─ preprocessing/
│     │  ├─ cleaning.py
│     │  ├─ transforms.py
│     │  └─ stationarity.py
│     ├─ spectral/
│     │  ├─ fft.py
│     │  ├─ wavelet.py
│     │  ├─ ssa.py
│     │  └─ rolling_spectrum.py
│     ├─ features/
│     │  ├─ feature_builder.py
│     │  ├─ spectral_features.py
│     │  └─ feature_store.py
│     ├─ regimes/
│     │  ├─ hmm.py
│     │  ├─ clustering.py
│     │  ├─ changepoint.py
│     │  └─ regime_service.py
│     ├─ signals/
│     │  ├─ signal_rules.py
│     │  └─ confidence.py
│     ├─ api/
│     │  ├─ main.py
│     │  ├─ routes_signals.py
│     │  └─ schemas.py
│     ├─ dashboard/
│     │  ├─ app.py
│     │  └─ components/
│     ├─ orchestration/
│     │  ├─ jobs.py
│     │  └─ schedules.py
│     └─ utils/
│        ├─ logging.py
│        └─ metrics.py
├─ notebooks/
│  ├─ 01_eda.ipynb
│  ├─ 02_spectral_validation.ipynb
│  └─ 03_regime_backtest.ipynb
├─ tests/
│  ├─ unit/
│  ├─ integration/
│  └─ smoke/
├─ scripts/
│  ├─ run_backfill.py
│  ├─ run_intraday.py
│  └─ seed_demo_data.py
└─ docker/
   ├─ Dockerfile.api
   ├─ Dockerfile.worker
   └─ docker-compose.yml
```

---

## 3) Concrete Implementation Plan

### Tech Stack (pragmatic MVP)
- **Language**: Python 3.11+
- **Core libs**: numpy, scipy, pandas, pywavelets, statsmodels, scikit-learn, hmmlearn
- **API**: FastAPI + Pydantic
- **Storage**: PostgreSQL (metadata/features) + parquet/object storage for large time series
- **Orchestration**: Prefect (or Airflow) + cron fallback
- **Dashboard**: Streamlit (fast MVP) or lightweight React frontend
- **Observability**: Prometheus/Grafana + structured logs

### Milestones & Timeline (10 weeks)

#### Phase 0 (Week 1): Foundations
- Finalize data contracts and feature definitions.
- Set up repo, environments, CI, linting, and test harness.
- Define target assets (e.g., BTC, ETH, SOL) and timeframes.

**Deliverables**: project skeleton, CI green, config templates.

#### Phase 1 (Weeks 2-3): Data + Preprocessing
- Build historical downloader + incremental ingestion.
- Implement validation (missing candles, duplicate timestamps, spikes).
- Implement detrending/normalization/stationarity utilities.

**Deliverables**: reliable raw-to-processed pipeline and quality report.

#### Phase 2 (Weeks 4-5): Spectral Core
- Implement FFT-based rolling spectrum with overlap windows.
- Add wavelet decomposition and time-frequency heatmaps.
- Compute cycle metrics: dominant period, band power ratios, spectral entropy, phase.

**Deliverables**: spectral feature tables and validation notebooks.

#### Phase 3 (Weeks 6-7): Regime Engine
- Baseline model: clustering on spectral + volatility/trend features.
- Advanced model: HMM with transition probabilities.
- Add change-point detection for regime boundary confidence.

**Deliverables**: regime labels + confidence + transition matrix endpoint.

#### Phase 4 (Weeks 8-9): Signals + API + Dashboard
- Define actionable signal rules (cycle alignment, regime filters).
- Build FastAPI endpoints for latest/historical outputs.
- Build dashboard views: spectrum heatmap, regime timeline, alert feed.

**Deliverables**: usable analyst interface and developer API.

#### Phase 5 (Week 10): Backtesting + Hardening
- Walk-forward backtests for key hypotheses.
- Reliability tuning, retries, alerting, and runbooks.
- Package MVP release + deployment docs.

**Deliverables**: release candidate with performance report.

### Detailed Work Breakdown (by stream)
1. **Data stream**: connectors, schema checks, lag monitoring.
2. **Model stream**: decomposition modules, feature registry, regime models.
3. **Product stream**: API contracts, dashboard UX, alert policies.
4. **Platform stream**: CI/CD, containerization, metrics, incident runbooks.

### Validation Strategy
- Unit tests for transforms and spectral functions.
- Integration tests from ingest -> features -> regimes.
- Statistical validation:
  - feature stability by regime,
  - out-of-sample significance,
  - turnover/slippage-aware backtests.

### Initial KPIs for Go/No-Go
- At least one regime-aware strategy improves max drawdown by >= 15% vs baseline.
- Feature freshness SLA > 98%.
- False regime-flip alert rate < 10% on evaluation window.

### Phase-2 Enhancements (post-MVP)
- Multi-exchange microstructure features (funding, OI, liquidations).
- Bayesian spectral models / multitaper methods.
- Online learning for adaptive regime thresholds.
- Portfolio-level regime aggregation and allocator hooks.
