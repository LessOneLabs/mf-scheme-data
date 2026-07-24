# Data Reference

Everything a consuming application (or AI agent) needs to fetch and use
this dataset correctly. For the "what/why/how" narrative, see the [main
page](https://lessonelabs.github.io/mf-scheme-data/); this file is the
terse, structured reference.

## Fetching it

```python
import json, urllib.request

url = "https://raw.githubusercontent.com/LessOneLabs/mf-scheme-data/main/exports/mf_ter_aum_latest.json"
data = json.loads(urllib.request.urlopen(url).read())

fund = data["funds"].get("118632")
if fund:
    print(fund.get("name"), fund.get("ter"), fund.get("aum"))
```

```javascript
const res = await fetch("https://raw.githubusercontent.com/LessOneLabs/mf-scheme-data/main/exports/mf_ter_aum_latest.json");
const data = await res.json();
const fund = data.funds["118632"];
if (fund) console.log(fund.name, fund.ter, fund.aum);
```

CSV is at the same path with `.csv` instead of `.json` — use whichever
suits your tooling; both cover the same fields.

GitHub Pages mirror (same content, served over `https://lessonelabs.github.io/`):
```
https://lessonelabs.github.io/mf-scheme-data/exports/mf_ter_aum_latest.json
https://lessonelabs.github.io/mf-scheme-data/exports/mf_ter_aum_latest.csv
```

## Schema

### JSON — top level
```json
{
  "generated_at": "2026-07-24T08:25:07.589045",
  "ter_month": "07-2026",
  "aum_fy_id": 1,
  "aum_period_id": 1,
  "funds": { "<scheme_code>": { ... }, ... }
}
```

### JSON — per-fund record (`funds["<scheme_code>"]`)

| Field | Type | Always present? | Meaning |
|---|---|---|---|
| `name` | string | Yes | Scheme name for this specific plan/option |
| `plan` | string | Yes | `"direct"` / `"regular"` / `"unspecified"` |
| `isin_g` | string | No | ISIN (Growth) |
| `isin_r` | string | No | ISIN (Reinvestment) |
| `aum` | integer | No | Average AUM, **absolute rupees** (not lakhs, not crore) |
| `ter` | number | No | Total Expense Ratio, percent (e.g. `0.87` means 0.87%) |
| `ter_date` | string | No | Date TER is as-of, `YYYY-MM-DD` |
| `nsdl` | string | No | The matched NSDL-format scheme identifier this TER value came from |

**The scheme code itself is the dict key, not a field inside the
record** — don't look for an `id`/`code` field inside each record; index
into `data["funds"]` directly.

### CSV columns
```
scheme_code,scheme_name,isin_growth,isin_reinvestment,plan,aum,ter_pct,ter_date,nsdl_scheme_code
```
Same data as the JSON, one row per scheme code, blank cells where the
JSON would omit the field.

## Critical: missing fields are absent, not null or zero

If `ter` isn't in a record, **do not treat that as TER = 0**. It means
no TER value is currently attached to this specific scheme code — for
example because it hasn't been published for this period, or it's a
plan/option variant that TER data doesn't distinguish. Same for `aum`,
`isin_g`, `isin_r`. Always check for key presence:

```python
ter = fund.get("ter")   # None if absent - handle explicitly
if ter is not None:
    ...
```

A record with **no** `ter` and **no** `aum` at all is still a real,
valid scheme — it's in the dataset because it exists in the reference
scheme master, just without cost/size data currently attached.

## Coverage expectations

- Universe: all scheme codes across every plan and option (Direct,
  Regular, Growth, IDCW, and others) — not filtered to Direct Growth.
- Not every scheme has `ter` and/or `aum` populated. This is expected,
  not a data quality bug — see the FAQ on the main page for why.
- Coverage is very high but deliberately not 100% — ambiguous or
  ungrouped edge cases are left without a value rather than guessed.

## Update cadence & versioning

- Refreshed weekly. `generated_at` in the JSON tells you exactly when.
- **No versioning guarantee on the schema** — this is a young project.
  If you're building something that depends on the exact field set,
  check `generated_at` changes and re-validate the shape occasionally
  rather than assuming permanent stability. If the schema does change in
  a breaking way, it will be noted in this repo's commit history (no
  separate changelog is currently maintained).
- Only the `_latest` files exist — there is no historical/versioned
  archive published here. If you need point-in-time history, you'll
  need to snapshot the file yourself on your own schedule.

## Suggested use cases

- Joining a fund's cost (TER) and size (AUM) to its identity (scheme
  code, ISIN) without building your own compilation/matching pipeline.
- Screening or ranking funds by expense ratio within a category.
- Cross-referencing ISIN to scheme code for portfolio-holding lookups.

## What this is not

- Not investment advice, and not a recommendation engine — it's raw
  reference data.
- Not a NAV/price feed — no daily price data is included here.
- Not guaranteed real-time — weekly refresh means data can be up to
  ~7 days stale relative to the latest disclosure.
