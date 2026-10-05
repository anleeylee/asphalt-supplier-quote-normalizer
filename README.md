# Asphalt Supplier Quote Normalizer

**Convert supplier material quotes into a normalized, evidence-backed price dataset** — material-only, delivered, installed and public-bid prices kept strictly separate.

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#usage)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**English** | [简体中文](./README.zh-CN.md)

---

## What it solves

Supplier price sheets use inconsistent units, scopes and validity windows — a "per ton" price from one supplier may mean something completely different from another's. This tool **reads supplier quotes and extracts** supplier, material, mix, unit, unit price, minimum load, surcharges, effective/expiration dates and region into one comparable dataset.

It **hard-separates material-only, delivered, installed and public-bid prices** — never merging them — and maps to a normalized category only with evidence. Unclear units or scopes are flagged instead of guessed.

## Features

- **Supplier quote extraction** — supplier, material, mix, unit, unit price, min load, surcharges, validity dates, region
- **Strict price-type separation** — material-only / delivered / installed / public-bid never merged
- **Evidence-based categorization** — normalized categories only when supported
- **Validity windows** — effective/expiration dates preserved for every price
- **Flagged ambiguities** — unclear units or scopes → review queue

## Install

```bash
pip install -r requirements.txt
```

## Usage

```bash
python -m scripts.s07_supplier_quote_normalizer --project ./project --input ./fixtures/suppliers --region "City Rock"
```

Run with `--help` for all options. Every script in this family shares the same CLI convention: `--project <path> --input <path> --output <path> --format json|csv|md|xlsx|pdf --config <path> --verbose --dry-run`.

## Outputs

- `supplier_prices.json` — normalized price items with basis and validity window
- `supplier_prices.csv`
- `supplier_quote_summary.md`

## How it works

- **Standard lifecycle** — `DISCOVER → INGEST → EXTRACT → NORMALIZE → VALIDATE → OUTPUT → AUDIT`
- **Evidence state machine** — every price moves `EXTRACTED → NORMALIZED → VALIDATED → VERIFIED`; stopping earlier is valid
- **Never merge price types** — a material-only price is never compared against a delivered price

## Quality gates

- deterministic schema validation on every record;
- golden sample fixtures with exact expected values (`tests/`);
- review queue: low-confidence values, ambiguous scales and conflicts land in `review_queue.json` / summary — never a silent fix;
- audit log: every run writes `run_id`, timings, input hashes, engine version and outputs to `<project>/audit/`.

## Testing

```bash
python -m pytest -q
```

## Security & privacy

Local files stay local by default. API keys live in environment variables (`ASPHALTCOSTS_API_KEY`, `ADI_AI_API_KEY`) — never in source code. Derived files are written to `working/` or `output/`; source files are never modified.

## License

MIT — see [LICENSE](LICENSE). Part of the [Asphalt Desktop Intelligence](https://github.com/anleeylee/asphalt-desktop-intelligence) toolkit.

## The calculation engine

[**AsphaltCosts.com**](https://asphaltcosts.com/) is the deterministic calculation layer: area → compacted volume → net tons → order tons (allowance applied once) → truckloads → material cost, with sourced planning defaults (145 lb/ft³ FHWA density, editable allowance and truck capacity). This tool feeds measured and validated inputs into that engine (or its labeled local mirror, `asphaltcosts-web-engine/1.0-mirror`) and never re-implements the formulas.
