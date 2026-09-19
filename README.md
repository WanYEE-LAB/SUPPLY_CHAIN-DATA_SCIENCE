# Supply Chain Network Risk Transmission — Research Dataset

Raw dataset for the project *Risk transmission along the supply-chain network of US listed firms:
network completion + causal machine learning + uncertainty quantification*.

Built **entirely from free public sources** — no subscription database required.
Collected on **2026-09-19**. Sample: **S&P 1500**
(S&P 500 + MidCap 400 + SmallCap 600 = 1,506 firms; free equivalent of a Russell-3000 universe).

## Files

| Path | Grain | Scale | Source |
|---|---|---|---|
| `data/network/customer_disclosures.csv` | supplier × customer × year | **8,328 edges** (464 disclosing suppliers → 386 unique named customers, 2013–2026) | SEC EDGAR 10-K text (mandatory ≥10%-revenue customer disclosure) |
| `data/financials/altman_z.csv` | firm × fiscal year | **11,448 firm-years** (1,235 firms, FY2015–2027; complete Z: 6,040; distress events: 1,706) | SEC XBRL companyfacts API |
| `data/financials/liabilities_supplement.csv` | firm × fiscal year | **4,420 firm-years** (442 firms incl. 254/258 universe Financials; banking-tag set) | SEC XBRL companyfacts API |
| `data/market/daily/YYYY.csv` | ticker × trading day | **4,408,770 daily rows**, 2015–2026, raw OHLCV (1,509 tickers; 4 without data) | Yahoo Finance (unofficial API) |
| `data/market/year_end_prices.csv` | ticker × year | **16,880 rows** (2015–2026) | Yahoo Finance (adjusted close) |
| `data/events/kiel_trade_indicator.csv` | date × series | **5,611 daily rows**, 6 series (2019–2026) | Kiel Institute trade dashboard (trade.ifw-kiel.de/KTI) |
| `data/events/kiel/*.csv` | date | raw per-series files (freight rates, Red Sea / Panama Canal / Cape of Good Hope ship counts, congested ships, China port calls) | same as above |
| `data/universe_tickers.csv` | stock | 1,506 rows (`ticker, name, sector, in_sp500`) | Wikipedia constituent lists + GICS sector |

All CSVs are UTF-8-BOM encoded (Excel opens them directly). Every network edge carries the
verbatim 10-K excerpt and a `source_url` to the original SEC filing — fully auditable.

## Data sources & authority

- **SEC EDGAR companyfacts API** (`data.sec.gov/api/xbrl/companyfacts`) — the official SEC
  structured-database endpoint for every US listed filer's XBRL submissions (10-K/10-Q/20-F).
  It is the primary source from which Compustat itself ingests; using it directly is free,
  key-complete, and carries the same legal-filing authority.
- **Yahoo Finance** (via `yfinance`) — the largest free market-data portal; widely validated
  against CRSP daily data in the academic literature and standard for working papers.
  Unofficial, though: replace with CRSP / LSEG Datastream for the final submission
  (downstream CAR / Merton-DD code needs no change).
- **Kiel Institute Trade Indicator** — the standard free daily series for 2023–24 trade
  disruptions (Red Sea, Panama Canal, freight rates), used in recent supply-chain research.

## Key fields

**Network edges** — `ticker` (supplier, the 10-K filer) → `customer_ticker` (customer,
resolved to a US ticker where possible); `pct_revenue` = edge weight (share of supplier
revenue); `disclosure_year` = fiscal year the share refers to; `customer_raw` = the name
string as written in the filing; `snippet` / `source_url` = evidence.

**Financials** — Altman Z components X1–X5, `Z`, `distress` (Z < 1.81), `year_end_price`,
`cik` for joining back to EDGAR. `liabilities_supplement.csv` carries the raw banking tag set
(`total_liab, deposits, equity, …`) for Merton Distance-to-Default; `deposits` is naturally
empty for insurers/brokers.

**Daily prices** — raw unadjusted OHLC + `adj_close` (dividend/split-adjusted) + volume.
Use `adj_close` for returns; the unadjusted OHLC is preserved untouched.

**Merge-ready panel** — supplier-side firm-years with financials: 4,138; Z-complete: 2,413;
distress: 647. Edges where **both** sides have financial data: 6,543 (370 suppliers × 235
customers). Financial-sector firms are covered by `liabilities_supplement.csv` +
daily prices (Merton DD), enabling the credit-channel mechanism test.

## Quality notes (read before use)

- **Raw, uncleaned.** Ticker mapping removed all literal self-loops (customer = discloser)
  in this release; residual alias noise is limited to raw-name variants (`customer_raw`).
  ~19 tickers have no SEC ticker-CIK mapping (e.g. LEG, HLX, CWEN-A) and are absent from
  the financial panels.
- **Banks / insurers** are absent from `altman_z.csv` by design (no `AssetsCurrent` XBRL
  tagging; the Altman manufacturing model does not apply). Use `liabilities_supplement.csv`
  with Merton DD instead — this also serves as an all-industry robustness distress measure.
- **Utilities / telecom / airlines** score pessimistic under Altman (leverage); use industry
  percentiles or robustness exclusions.
- COVID validation passed: airlines / cruises / hotels fall into distress exactly around 2020.
- `gdelt_supply_chain_events.csv` is **pending** (GDELT API rate-limited at collection time);
  the Kiel series already cover the 2023–24 Red Sea / Panama Canal disruptions.
- 4 tickers lack daily prices (CWEN-A, GIVE, SBE, WELA); delisted names may have truncated
  histories depending on Yahoo coverage.

## Provenance

Extracted with a checkpoint-resumable pipeline (EDGAR crawler + XBRL parser + bulk daily/year-end
price download + event fetchers, with a dual logging layer). To refresh or extend the sample:
re-run the fetchers (they resume from checkpoints), replace `data/`, and commit.
Collection logs available on request.
