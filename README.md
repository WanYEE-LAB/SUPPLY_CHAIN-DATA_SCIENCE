# Supply Chain Network Risk Transmission — Research Dataset

Raw dataset for the project *Risk transmission along the supply-chain network of US listed firms:
network completion + causal machine learning + uncertainty quantification*.

Built **entirely from free public sources** (SEC EDGAR, Yahoo Finance, Kiel Institute) —
no subscription database required. Collected on **2026-09-19**. Sample: S&P 500 constituents (503 firms).

## Files

| Path | Grain | Scale | Source |
|---|---|---|---|
| `data/network/customer_disclosures.csv` | supplier × customer × year | **2,665 edges** (153 disclosers × 163 customers, 2013–2026) | SEC EDGAR 10-K text (mandatory ≥10%-revenue customer disclosure) |
| `data/financials/altman_z.csv` | firm × fiscal year | **4,038 rows** (415 firms, FY2015–2027; complete Z: 2,181) | SEC XBRL companyfacts API |
| `data/market/year_end_prices.csv` | ticker × year | **5,840 rows** (2015–2026) | Yahoo Finance (adjusted close) |
| `data/events/kiel_trade_indicator.csv` | date × series | **5,611 daily rows**, 6 series (2019–2026) | Kiel Institute trade dashboard (trade.ifw-kiel.de/KTI) |
| `data/events/kiel/*.csv` | date | raw per-series files (freight rates, Red Sea / Panama Canal / Cape of Good Hope ship counts, congested ships, China port calls) | same as above |
| `data/sp500_tickers.csv` | stock | 503 rows | Wikipedia constituents + GICS sector |

All CSVs are UTF-8-BOM encoded (Excel opens them directly). Every network edge carries the
verbatim 10-K excerpt and a `source_url` to the original SEC filing — fully auditable.

## Key fields

**Network edges** — `ticker` (supplier, the 10-K filer) → `customer_ticker` (customer);
`pct_revenue` = edge weight; `disclosure_year` = fiscal year the share refers to;
`snippet` / `source_url` = evidence.

**Financials** — Altman Z components X1–X5, `Z`, `distress` (Z < 1.81), `year_end_price`.

## Quality notes (read before use)

- **Raw, uncleaned.** Known systematic noise: self-name alias contamination (~9%), e.g.
  SNA (Snap-on) → SNAP. Cleaning rule: drop edges whose normalized customer alias is a substring
  of the discloser's firm name. A cleaned release will follow.
- **Banks/insurers/REITs (88 firms)** are absent: no `AssetsCurrent` XBRL tagging and the Altman
  manufacturing model does not apply — out of scope by design.
- **Utilities/telecom/airlines** score pessimistic under Altman (leverage); use industry
  percentiles or robustness exclusions.
- COVID validation passed: airlines/cruises/hotels fall into distress exactly around 2020.
- `gdelt_supply_chain_events.csv` is **pending** (GDELT API rate-limited at collection time);
  the Kiel series already cover the 2023–24 Red Sea / Panama Canal disruptions.
- Yahoo Finance is unofficial — replace with LSEG Datastream / CRSP for the final paper.

## Provenance

Extracted with a checkpoint-resumable pipeline (EDGAR crawler + XBRL parser + bulk price
download + event fetchers, with a dual logging layer). Collection logs available on request.
