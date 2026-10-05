# Asphalt Quote Comparator

**Normalize contractor asphalt bids into a comparable table** — $/SF, $/ton, implied thickness and scope flags — with full evidence and no ranking.

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#usage)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**English** | [简体中文](./README.zh-CN.md)

---

## What it solves

Contractor bids arrive in a dozen different formats — different units, different scope line items, some with tonnage, some without — so comparing them fairly is a spreadsheet nightmare. This tool parses one or more **quote texts / PDFs / spreadsheets into a comparable dataset**: area, tons, thickness, scope line items, unit prices, lump sums and grand total, with material / delivery / labor / equipment / removal kept separate and original wording preserved.

Deterministic derived metrics ($/SF, $/ton, implied thickness) are computed from stated quantities only. Flags call out plan-quantity mismatch, spec-thickness mismatch, missing scope, missing tonnage and missing totals. **It never picks a "best" contractor** — the decision stays with the homeowner or project manager.

## Features

- **Multi-format quote parsing** — PDF / text / spreadsheet contractor bids
- **Apples-to-apples normalization** — area, tons, thickness, scope line items, unit prices, lump sums
- **Separated cost categories** — material / delivery / labor / equipment / removal never merged
- **Deterministic derived metrics** — $/SF, $/ton, implied thickness from stated quantities only
- **Critical flags** — plan-quantity mismatch, spec-thickness mismatch, missing scope / tonnage / totals
- **No ranking** — evidence and comparison, decision left to the user

## Install

```bash
pip install -r requirements.txt
```

## Usage

```bash
python -m scripts.s05_quote_comparator --project ./project --input ./fixtures/quotes
```

Run with `--help` for all options. Every script in this family shares the same CLI convention: `--project <path> --input <path> --output <path> --format json|csv|md|xlsx|pdf --config <path> --verbose --dry-run`.

## Outputs

- `quote_comparison.json` — normalized quotes + derived metrics + flags
- `quote_comparison.xlsx` — side-by-side comparison sheet
- `quote_flags.md` — critical flags in human-readable form

## How it works

- **Standard lifecycle** — `DISCOVER → INGEST → EXTRACT → NORMALIZE → VALIDATE → CALCULATE/CLASSIFY → OUTPUT → AUDIT`
- **Evidence state machine** — every quoted value moves `EXTRACTED → NORMALIZED → VALIDATED → VERIFIED`; stopping earlier is valid
- **Original wording preserved** — normalization never loses what the contractor actually wrote

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
