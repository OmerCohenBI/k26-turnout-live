# בחירות לכנסת ה-26 (27/10/2026) — אחוזי הצבעה ותוצאות לפי קלפי

נתונים ציבוריים של ועדת הבחירות המרכזית, בקבצים נקיים, **מתעדכנים אוטומטית בכל סבב שהוועדה מפרסמת**
(אחוזי הצבעה במהלך היום; תוצאות לפי קלפי מ-22:00). לבדוק אם השתנה משהו: `status.json`.

Public data of Israel's Central Elections Committee (CEC), mirrored automatically after every CEC round.
To see whether anything changed, read **status.json** (round time, last update, row counts, sha256 per file).
Raw download pattern: `https://raw.githubusercontent.com/OmerCohenBI/k26-turnout-live/main/<file>` (GitHub's raw cache can lag up to ~5 minutes).

## Files
| file | content |
|---|---|
| status.json | last CEC round, update time (UTC and Israel), per-file sha256 / rows |
| k26_turnout_by_ballot.csv | election-day turnout per ballot box, latest round: city_code, city_name, ballot, neighborhood_id, eligible, voters, turnout_pct, measured_at, cec_round, eligible_2022, turnout_2022_final_pct, vs_2022_pct, comparable_to_2022, group_2022 |
| k26_turnout_by_ballot_rounds.csv | the same per ballot for every round published today (time series) |
| k26_turnout_by_settlement.csv | per settlement, latest round (reporting boxes only, as the CEC computes it) + 2022 comparison |
| k26_turnout_kalpi.json, k26_turnout_municipal.json | the raw CEC turnout-map feed (per box / per settlement) |
| turnout_hours.json | national cumulative turnout by hour, 2019–2022 elections (+ today's official readings) |
| k26_expb.csv, k26_expc.csv | CEC 2026 results per box / per settlement (header only until counting starts) |
| k25_expb_cec.csv, k25_expc_cec.csv | CEC 2022 results per box / per settlement (for cross-checking) |
| k26_parties.json, families.json | ballot letters → lists; which 2026 list continues which 2022 list(s) |
| model_spec.md | every definition and formula (turnout definitions, matching to 2022, vs_2022 index, forecasts) |

## Definitions (short; full detail in model_spec.md)
- Box turnout = voters / eligible of that box at `measured_at`; boxes report at different times.
- Settlement turnout = sum over **reporting boxes only** (exactly the CEC's settlement figure), not all eligible voters.
- National = the CEC's official reading at an even hour, over all 7,331,981 eligible voters.
- vs_2022_pct: the box compared with the boxes measured in the same half hour, relative to where it stood in 2022
  (100 × (actual / expected − 1)); blank when the box changed since 2022 (renumbered or eligible changed > 25%).
- Column names never change between rounds; any change will be listed here with a date.

## Change log
- 2026-10-27: first version.
