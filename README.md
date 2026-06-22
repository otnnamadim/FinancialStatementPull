# FinancialStatementPull

A lightweight Python toolkit for pulling and parsing financial statement data directly from the SEC's [XBRL Company Facts API](https://www.sec.gov/search-filings/edgar-application-programming-interfaces) — built by a CPA, for analyzing real company filings programmatically.

> **Disclaimer:** The companies referenced in this repository's watchlist are not specific investment recommendations. This project exists to illustrate the functionality of the SEC's Company Facts API in extracting financial information to conduct financial statement analysis.

## Why this project exists

Every public company filing with the SEC tags its financial statement data using XBRL (eXtensible Business Reporting Language) — a structured format that, in theory, makes financial data machine-readable across all 10-Ks and 10-Qs. In practice, working with this data requires understanding both the API and the accounting concepts behind it.

This project pulls that data directly from SEC's public API and flattens it into clean, analysis-ready `pandas` DataFrames — with a particular focus on a problem most financial data tools ignore: **foreign private issuers (FPIs) don't report under the same accounting standard as domestic filers.** US-based companies file under US GAAP; many FPIs file under IFRS, with different XBRL taxonomies and tagging conventions entirely. A tool that blindly assumes every company uses the `us-gaap` taxonomy will silently fail — or worse, return misleading results — for a meaningful share of SEC filers.

## What it does

- **Pulls raw XBRL company facts** for any public company by CIK (Central Index Key) directly from the SEC's API
- **Flattens nested JSON** into tidy DataFrames for any US GAAP financial statement line item (revenue, assets, net income, etc.)
- **Lists all available GAAP concepts** a company has reported, so you can discover what's available before querying
- **Distinguishes domestic filers from foreign private issuers** in its sample watchlist, surfacing the accounting standard gap directly

## Project structure

```
FinancialStatementPull/
├── financial_pull/
│   ├── __init__.py        # Public API
│   ├── sec_client.py       # HTTP layer — talks to the SEC API
│   ├── parsers.py          # Pure functions — JSON to DataFrame, no network calls
│   └── watchlist.py        # Sample company watchlist (domestic + FPI)
├── tests/
│   ├── conftest.py
│   ├── fixtures/           # Trimmed, realistic SEC API responses for testing
│   ├── test_sec_client.py  # Mocked HTTP tests
│   ├── test_parsers.py     # Parsing logic tests
│   └── test_watchlist.py   # Data integrity tests
├── requirements.txt
├── pytest.ini
└── README.md
```

The package separates **I/O** (`sec_client.py`) from **parsing logic** (`parsers.py`) deliberately — this means the parsing functions can be fully unit tested against fixture data without making a single live network call. See [Testing](#testing) below.

## Installation

```bash
git clone https://github.com/otnnamadim/FinancialStatementPull.git
cd FinancialStatementPull
pip install -r requirements.txt
```

## Usage

```python
from financial_pull import get_company_facts, extract_fsli_to_dataframe, list_available_gaap_metrics

# The SEC requires a descriptive User-Agent identifying the requester.
# See: https://www.sec.gov/os/accessing-edgar-data
USER_AGENT = "Your Name your.email@example.com"

# Pull raw company facts for Flex Ltd (CIK 866374)
facts = get_company_facts("0000866374", USER_AGENT)

# See what GAAP concepts this company has reported
available_metrics = list_available_gaap_metrics(facts)
print(available_metrics.head())

# Pull a specific line item across all filed periods
revenue_df = extract_fsli_to_dataframe(facts, "Revenues")
print(revenue_df)
```

```python
from financial_pull import afrotech_investment_watchlist

watchlist = afrotech_investment_watchlist()
print(watchlist)
#    category                           company_name ticker         cik
# 0  domestic                               Flex Ltd   FLEX  0000866374
# 1  domestic                                Vertiv     VRT  0001674101
# ...
# 4       fpi  FoxCon - Hon Hai Precision Industry Co  HNHPF  0001611906
# 5       fpi                   Taiwan Semiconductor    TSM  0001046179
```

## A note on foreign private issuers (FPIs)

Several companies in the sample watchlist — Taiwan Semiconductor, GlobalFoundries, STMicroelectronics, Infineon, and Hon Hai (Foxconn) — are foreign private issuers. FPIs typically:

- File annual reports on **Form 20-F** rather than Form 10-K
- Report financial statements under **IFRS** (International Financial Reporting Standards) rather than **US GAAP**
- Tag their XBRL data under the `ifrs-full` taxonomy rather than `us-gaap` — meaning functions like `extract_fsli_to_dataframe`, which currently query the `us-gaap` taxonomy, will correctly return an empty result for these companies rather than silently misreporting data

Building out IFRS-to-GAAP concept reconciliation (mapping `ifrs-full:Revenue` to `us-gaap:Revenues`, for example, and flagging where the underlying recognition principles genuinely differ) is the next planned phase of this project.

## Testing

The test suite mocks all network calls and runs entirely offline against fixture data trimmed from real SEC API responses.

```bash
pip install -r requirements.txt
pytest
```

```
20 passed in 0.87s
```

Test coverage includes:
- **Parsing logic** against both a US GAAP domestic filer fixture and an IFRS-reporting FPI fixture (confirming the graceful empty-result behavior described above)
- **HTTP client behavior** — CIK padding, header construction, error handling — with `requests.get` fully mocked
- **Watchlist data integrity** — CIK format validation, duplicate detection, category validation

## Roadmap

- [ ] IFRS-to-GAAP concept mapping layer for FPI comparability
- [ ] Ratio and trend analysis module (liquidity, leverage, profitability) across the watchlist
- [ ] CLI for ad-hoc company lookups
- [ ] Rate limiting / response caching for SEC's fair access policy

## About

Built by [Ozoemena Nnamadim](https://github.com/otnnamadim), CPA — combining a Big 4 audit and public sector finance background with hands-on data engineering to make SEC filing data more accessible for financial analysis.

## License

This project is provided for educational and illustrative purposes. See disclaimer above.
