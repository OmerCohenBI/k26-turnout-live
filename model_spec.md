# Model specification — K26 live dashboard (27/10/2026)

All code that produces the numbers is in this folder (`code_*.py`). Raw inputs are the CEC files in this folder.
Notation: b = ballot box, c = settlement (CBS code), s = stratum, f = party family, e = eligible voters (בזב),
v = voters (מצביעים), "22" = the 25th Knesset election, 1 Nov 2022.

## 1. Data sources
| input | file here | origin |
|---|---|---|
| 2022 results per ballot / settlement | `k25_expb_cec.csv`, `k25_expc_cec.csv` | https://media25.bechirot.gov.il/files/expb.csv, expc.csv (UTF-8 copy) |
| 2019–2021 results | `k21…k24_expb_cec.csv / _expc_cec.csv` | mediaNN.bechirot.gov.il (source encoding cp1255, converted) |
| 2026 results (from ~22:00) | `k26_expb.csv`, `k26_expc.csv` | https://media26.bechirot.gov.il/files/ (polled every 30 s) |
| 2026 election-day turnout per ballot | `k26_turnout_kalpi.json` (raw), `k26_turnout_by_ballot.csv` (clean), `k26_turnout_by_ballot_rounds.csv` (all rounds) | CEC turnout map https://mapvotes.bechirot.gov.il/ (`/data/votes/latest/kalpi_data.json`), relayed every minute |
| 2026 turnout per settlement | `k26_turnout_municipal.json` (raw), `k26_turnout_by_settlement.csv` | same map (`municipal_data.json`) |
| national hourly turnout K21–K25 | `turnout_hours.json` | CEC "אחוז ההצבעה המצטבר בחתך שעות" (K21–K24), press quoting the CEC (K25) |
| lists, letters, poll average | `k26_parties.json` | ynet / mako / handled.team; 8 polls 21–23/10 |
| list continuity 2022→2026 | `families.json` | decided by the dashboard owner |

## 2. How the CEC turnout numbers are defined (verified on the data)
- Ballot: `percent = מצביעים / בזב` of that box, measured at `updated` (HH:MM). Each box reports at its own time.
- Settlement (`municipal_data.json`): **sum over the boxes that have reported only** — `בזב` and `מצביעים` are the sums of the
  reporting boxes, `kalpis` = number of reporting boxes, `updated` = the latest box time. Verified: 554/554 settlements match
  the box sums exactly. It is therefore NOT out of all eligible voters of the settlement.
- National (`total` in the same files): the CEC's official national reading (e.g. 16.7% = 1,223,286 / 7,331,981 = all eligible
  voters, as of 10:00, published 11:09). It is a separate CEC figure, **not** the sum of the box feed (the 8,220 boxes in the
  first round summed to 488,480 voters of 5,116,808 eligible, measured mostly 09:00–09:30).
- Double envelopes (soldiers, diplomats, prisoners, hospitals…) are not in any election-day turnout figure; they are
  settlement 9999 in the results files and are counted last.

## 3. Matching 2026 boxes to 2022
key = `city_code:ballot` (ballot like `12` or `12.1`). Match order: exact key → same base number (before the dot, aggregated) →
the whole settlement. A box is **comparable** when the match is exact AND 0.75 ≤ e26/e22 ≤ 1.35 AND e26 ≥ 100; otherwise it is
reported but excluded from box-level comparisons and outlier lists.

## 4. Turnout vs 2022 (no polls, no intraday curve)
Boxes measured in the same half-hour window w are compared with each other, because the time of measurement dominates raw %.
For a comparable box b in window w:
  expected_b = e_b × T_w × t22_b / T22_w,   T_w = Σv/Σe of the comparable boxes in w now,  T22_w = Σ(e·t22)/Σe of the same boxes
  index_b = v_b / expected_b   (1.00 = same relative turnout as in 2022; `vs_2022_pct` = (index−1)×100)
Settlement: same formula using every reporting box and the settlement's own 2022 turnout t22_c (so renumbered boxes count):
  index_c = Σ v_b / Σ e_b·T'_w·t22_c/T22'_w ;  z = (actual − expected) / (2·√expected)   (×2 = allowance for over-dispersion)
Groups (strata) use comparable boxes only. Caveat: 2022 has no per-box hourly data, so a morning pace is compared with the
2022 *final* relative turnout — communities that vote early look high in the morning.

## 5. National final-turnout estimate
as_of = the even hour of the CEC national reading (publication time − 25 min, rounded down to 10/12/14/16/18/20).
final_est = now × mean_k( final_k / at_hour_k ) over K21–K25; range = now × min/max of those ratios; capped at 92%.

## 6. Strata (from each box's 2022 vote)
arab: Arab lists ≥ 50% · haredi: Shas+UTJ ≥ 50% · rzp: Religious Zionism (2022) ≥ 30% · right: Netanyahu bloc ≥ 55% ·
centre: 38–55% · centre_left: < 38% · envelopes: settlement 9999.

## 7. Turnout-only forecast (default before the count; table "התחזית לפי ההצבעה היום")
Units u = settlement × stratum (2022 composition). Projected final turnout of a reporting box:
  tf_b = min(0.97, t22_b × (final_est / 0.7063) × index_b);  tf_u = Σ e_b·tf_b / Σ e_b over reporting boxes of u
  (units with no reporting box: t22_u × final_est/0.7063)
Votes of family f: V_f = Σ_u e22_u × growth × tf_u × (valid/voted)22_u × share22_{u,f},  growth = 7,331,981 / 6,788,804.
Seats: Bader–Ofer (D'Hondt) on families, 3.25% threshold of total valid, no surplus agreements.
"2022 if together" = same with tf_u = t22_u. Difference = effect of today's turnout only.
Families (families.json): פה+כן(2022) → רק+דרך+כן · ט → ט+ב+ני · אמת+מרצ → אמת · ום+ד → ודם · others 1:1 · rest → אחרות.
Split inside a family (e.g. רק vs דרך) is NOT identifiable from 2022 data; where a per-list number is shown it uses the poll mix.

## 8. Forecast during the count (after ~22:00)
For every counted box: stratum and 2022 composition via §3. Per stratum s and family f:
  swing_{s,f} = (Σ votes_f now − Σ share22_{b,f}·valid_b) / Σ valid_b over counted boxes; turnout ratio and valid/voted likewise;
  shrunk toward the national values with weight n/(n+25) (n = counted boxes in s).
Uncounted units: votes = e × growth × turnout (from §7 if available) × valid ratio × (share22_{u,f} + swing_{s,f}), clipped ≥0,
renormalised; inside a family split by the counted mix of s (shrunk to the poll mix when few boxes). The uncounted part is
rescaled so counted + projected voters = final_est × eligible. Seats: Bader–Ofer per list. Range: 40 bootstrap resamples of
counted boxes (10–90%), widened by round(5×(1−f)) seats per bloc / round(2×(1−f)) per list, f = counted share of eligible.
Backtest (K24→K25, small boxes first): seat error 16 at 5% counted (raw count: 22), 5 at 25% (raw: 11).

## 9. Known limitations
No surplus agreements · family split needs polls before the count · morning turnout pattern ≠ final pattern · CEC box feed
lags ~30 min · box renumbering since 2022 limits box-level matching to ~43% of boxes (settlement level uses all).
