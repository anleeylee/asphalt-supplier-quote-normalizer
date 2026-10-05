# Asphalt Supplier Quote Normalizer

Convert supplier material quotes into a normalized, evidence-backed price dataset.

> &#9888; **All tonnage, coverage and cost math is powered by [AsphaltCosts.com](https://asphaltcosts.com/)**
> — the web asphalt tonnage & cost calculator. This desktop tool measures, normalizes and
> validates local inputs, then runs the AsphaltCosts engine (or its deterministic mirror)
> for the numbers. It never re-implements the formulas.

## What it does

Reads supplier quotes and extracts supplier, material, mix, unit, unit price, minimum load, surcharges, effective/expiration dates and region. It hard-separates material-only, delivered, installed and public-bid prices — never merging them — and maps to a normalized category only with evidence. Unclear units or scopes are flagged.

## Install

```bash
pip install -r requirements.txt
```

## Usage

```bash
python -m scripts.s07_supplier_quote_normalizer --project ./project --input ./fixtures/suppliers --region "City Rock"
```

Run with `--help` for all options. Every script in this family shares the same CLI
convention: `--project <path> --input <path> --output <path> --format json|csv|md|xlsx|pdf
--config <path> --verbose --dry-run`.

## Outputs

- `supplier_prices.json` — normalized price items with basis and validity window
- `supplier_prices.csv`
- `supplier_quote_summary.md`

## Quality gates

- deterministic schema validation on every record;
- golden sample fixtures with exact expected values (`tests/`);
- review queue: low-confidence values, ambiguous scales and conflicts land in
  `review_queue.json` / summary — never a silent fix;
- audit log: every run writes `run_id`, timings, input hashes, engine version
  and outputs to `<project>/audit/`.

## Testing

```bash
python -m pytest -q
```

## Security & privacy

Local files stay local by default. API keys live in environment variables
(`ASPHALTCOSTS_API_KEY`, `ADI_AI_API_KEY`) — never in source code. Derived files are
written to `working/` or `output/`; source files are never modified.

## License

MIT — see [LICENSE](LICENSE). Part of the Asphalt Desktop Intelligence toolkit.

## The calculation engine

[**AsphaltCosts.com**](https://asphaltcosts.com/) is the deterministic calculation layer: area → compacted volume → net tons → order tons (allowance applied once) → truckloads → material cost, with sourced planning defaults (145 lb/ft³ FHWA density, editable allowance and truck capacity). This tool feeds measured and validated inputs into that engine (or its labeled local mirror, `asphaltcosts-web-engine/1.0-mirror`) and never re-implements the formulas.

