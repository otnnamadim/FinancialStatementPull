# FinancialStatementPull

This is a Python coding project developed for extracting and parsing financial statement data directly from the SEC's EDGAR Database via the Company Facts API. 

> **Disclaimer:** The companies referenced in this repository's watchlist are not specific investment recommendations. This project exists to illustrate the functionality of the SEC's Company Facts API in extracting financial information to conduct financial statement analysis.

## Why this project exists
Every public company filing with the SEC tags its financial statement data via XBRL. I've previously worked on IFRS and US GAAP SEC financial reporting projects as a CPA where I've prepared disclosures and tagged XBRL data for submission to EDGAR; however, this project retrieves the 10-K and 10-Q data from the investing public side. 

This project pulls that data directly from SEC's public API and flattens it into clean, analysis-ready `pandas` DataFrames — with a particular focus on distinguishing foreign private issuers (FPIs) from domestic filers. US-based companies, domestic filers, issue their financial statements under US GAAP, while many FPIs file under IFRS, with different XBRL taxonomies and tagging conventions entirely. 

## How it works?
- **Pulls raw XBRL company facts** for any public company by referencing the company's CIK (Central Index Key) via the Company Facts API
- **Flattens nested JSON Company Facts** into tidy DataFrames for all tagged financial statement line items
- **Lists all available GAAP concepts** a company has reported to explore the available financial statement line before querying
- **Distinguishes domestic filers from foreign private issuers** in its sample watchlist highlighting differences in accounting standards for each company
