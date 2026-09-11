# Optimal World Basket

Public, read-only Streamlit dashboard and model-monitoring jobs for the
`PUBLIC-01` event-driven portfolio.

This repository deliberately does **not** contain the private optimizer,
publication UI, broker holdings, migration tools, or legacy weekly runner.
Portfolio versions are published from a separate operator environment and are
read here from the durable PostgreSQL ledger.

## What is included

- Current immutable target allocation and planning prices
- Historical estimated-net model performance
- World-equity benchmark comparison in INR
- 28-calendar-day statistical outlook
- Cost-aware fresh-cash and illustrative model-withdrawal calculations
- NSE/NYSE-aware model entry and review monitoring
- Append-only review evidence and optional owner alerts

The dashboard never places trades and never asks visitors to upload broker
reports. Model results are not the return of an investor's broker account.

## Streamlit Cloud deployment

1. Create a GitHub repository named `optimal-world-basket`.
2. Upload this package at the repository root, preserving directories.
3. Create a Streamlit Cloud app from `public_portfolio_performance.py`.
4. Add this Streamlit secret:

```toml
[public_basket]
database_url = "postgresql://USER:PASSWORD@HOST/DATABASE?sslmode=require"
```

Use a dedicated PostgreSQL role with `SELECT` access only for this Streamlit
secret. The public application code performs reads only, but database-level
permissions provide the durable security boundary. Do not add a publisher or
migration token to this deployment.

## GitHub Actions configuration

Create the `PRODUCTION` environment and add:

- Secret: `PUBLIC_BASKET_DATABASE_URL` (a writer role restricted to the trust
  and review jobs; this may differ from the Streamlit read-only credential)
- Variable: `PUBLIC_REVIEW_ENABLED` (`true` when monitoring is enabled)
- Variable: `PUBLIC_MODEL_BACKFILL_TRADING_DAYS` (`0` for a clean production release)
- Optional variable: `PUBLIC_REVIEW_ALERT_CHANNEL` (`none`, `telegram`, or `email`)
- Corresponding alert secrets only when an alert channel is enabled

The daily trust workflow updates NAV and forecasts. The model-review workflow
checks after supported NSE/NYSE opening and closing windows and retains a daily
weekend/holiday heartbeat.

## Safety boundaries

- PostgreSQL is the only durable record.
- Portfolio publications are immutable and basket-scoped.
- Public-data evidence is inspected before export.
- Missing market data, classification, tariffs, or calendars fail closed.
- Development backfill must be zero and the development ledger must be reset
  before the intended public release.

## Local verification

```text
python -m pip install -r requirements.txt
python -m pytest -q
```
