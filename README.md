# Supply Chain Network Risk Transmission — Research Dataset

Raw dataset for the project *Risk transmission along the supply-chain network of US listed firms:
network completion + causal machine learning + uncertainty quantification*.

Built **entirely from free public sources** (SEC EDGAR, Yahoo Finance, Kiel Institute) —
no subscription database required. Collected on **2026-09-19**. Sample: **S&P 1500**
(S&P 500 + MidCap 400 + SmallCap 600 = 1,506 firms; free equivalent of a Russell-3000 universe).

## Files

| Path | Grain | Scale | Source |
|---|---|---|---|
| `data/network/customer_disclosures.csv` | supplier × customer × year | **8,328 edges** (464 disclosing suppliers → 386 unique named customers, 2013–2026) | SEC EDGAR 10-K text (mandatory ≥10%-revenue customer disclosure) |
| `data/financials/altman_z.csv` | firm × fiscal year | **11,448 firm-years** (1,235 firms, FY2015–2027; complete Z: 6,040; distress events: 1,706) | SEC XBRL companyfacts API |
| `data/market/year_end_prices.csv` | ticker × year | **16,880 rows** (1,506 tickers, 2015–2026) | Yahoo Finance (adjusted close) |
| `data/events/kiel_trade_indicator.csv` | date × series | **5,611 daily rows**, 6 series (2019–2026) | Kiel Institute trade dashboard (trade.ifw-kiel.de/KTI) |
| `data/events/kiel/*.csv` | date | raw per-series files (freight rates, Red Sea / Panama Canal / Cape of Good Hope ship counts, congested ships, China port calls) | same as above |
| `data/universe_tickers.csv` | stock | 1,506 rows (`ticker, name, sector, in_sp500`) | Wikipedia constituent lists + GICS sector |

All CSVs are UTF-8-BOM encoded (Excel opens them directly). Every network edge carries the
verbatim 10-K excerpt and a `source_url` to the original SEC filing — fully auditable.

## Key fields

**Network edges** — `ticker` (supplier, the 10-K filer) → `customer_ticker` (customer,
resolved to a US ticker where possible); `pct_revenue` = edge weight (share of supplier
revenue); `disclosure_year` = fiscal year the share refers to; `customer_raw` = the name
string as written in the filing; `snippet` / `source_url` = evidence.

**Financials** — Altman Z components X1–X5, `Z`, `distress` (Z < 1.81), `year_end_price`,
`cik` for joining back to EDGAR.

**Merge-ready panel** — supplier-side firm-years with financials: 4,138; Z-complete: 2,413;
distress: 647. Edges where **both** sides have financial data: 6,543 (370 suppliers × 235
customers) — usable for contagion / spillover regressions without further collection.

## Quality notes (read before use)

- **Raw, uncleaned.** Ticker mapping removed all literal self-loops (customer = discloser)
  in this release; residual alias noise is limited to raw-name variants (`customer_raw`).
  ~19 tickers in the universe have no SEC ticker-CIK mapping (e.g. LEG, HLX, CWEN-A) and
  are absent from the financial panel.
- **Banks / insurers / REITs** are largely absent: no `AssetsCurrent` XBRL tagging and the
  Altman manufacturing model does not apply — out of scope by design.
- **Utilities / telecom / airlines** score pessimistic under Altman (leverage); use industry
  percentiles or robustness exclusions.
- COVID validation passed: airlines / cruises / hotels fall into distress exactly around 2020.
- `gdelt_supply_chain_events.csv` is **pending** (GDELT API rate-limited at collection time);
  the Kiel series already cover the 2023–24 Red Sea / Panama Canal disruptions.
- Yahoo Finance is unofficial — replace with LSEG Datastream / CRSP for the final paper.

## Provenance

Extracted with a checkpoint-resumable pipeline (EDGAR crawler + XBRL parser + bulk price
download + event fetchers, with a dual logging layer). To refresh or extend the sample:
re-run the fetchers (they resume from checkpoints), replace `data/`, and commit.
Collection logs available on request.
