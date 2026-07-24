# MF Scheme Data

Auto-published TER, AUM, and ISIN data for the full Indian mutual fund
universe (all AMFI scheme codes — every plan and option, not just Direct
Growth).

- 🔗 **Raw data (JSON)**: [`exports/mf_ter_aum_latest.json`](https://github.com/LessOneLabs/mf-scheme-data/blob/main/exports/mf_ter_aum_latest.json) — dict keyed by AMFI scheme code
- 📄 **Raw data (CSV)**: [`exports/mf_ter_aum_latest.csv`](https://github.com/LessOneLabs/mf-scheme-data/blob/main/exports/mf_ter_aum_latest.csv)

This repo is auto-updated weekly by a private companion repo that fetches
scheme, TER, and AUM data directly from AMFI's own published endpoints,
matches TER's NSDL-keyed records to AMFI scheme codes, and publishes the
merged result here. The matching/mapping logic itself is not published —
only its output. This data feeds [MF Quant Analyzer](https://github.com/LessOneLabs/mf-quant-analyzer-data)'s
Efficiency scoring.

## Fields

| Field | Meaning |
|---|---|
| (dict key) | AMFI scheme code |
| `name` | Scheme name (this specific plan/option) |
| `plan` | `direct` / `regular` / `unspecified` |
| `isin_g` | ISIN (Growth), if assigned |
| `isin_r` | ISIN (Reinvestment), if assigned |
| `aum` | Average AUM, absolute rupees (most recent AMFI-published period) |
| `ter` | Total Expense Ratio, percent |
| `ter_date` | Date the TER figure is as-of |
| `nsdl` | The matched NSDL scheme code (TER source), for traceability |

A missing field means AMFI hasn't published that data for this scheme, or
it couldn't be confidently matched — never a fabricated value.

## Disclaimer

This data is for informational and educational purposes only and does
not constitute investment advice. Mutual fund investments are subject to
market risks. Read all scheme-related documents carefully before
investing. Past performance is not indicative of future returns. This
project has no affiliation with any AMC, SEBI, or AMFI.

## License

Code in this repo (any scripts, if present) is MIT licensed — see
[`LICENSE`](https://github.com/LessOneLabs/mf-scheme-data/blob/main/LICENSE).
This does not extend any license over the underlying scheme data itself,
which is sourced from AMFI.
