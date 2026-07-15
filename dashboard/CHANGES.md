# Dashboard Update Log
**Update:** 2026-05-04 (dispute-level denominator correction for Logic 3)

## Logic 3 denominator correction (2026-05-04)
- Rebuilt `PROV_FOCUS`, `PROV_NORMAL`, and `PLAN_FOCUS` in `analysis_outputs/index.html` from source PUF files using dispute-level aggregation.
- Provider and plan default metrics now use:
  - numerator = distinct disputes with `Default Decision = Yes`
  - denominator = distinct disputes
  - share = numerator / denominator
- Preserved the existing review-rule thresholds in the dashboard logic:
  - providers: `wins >= 10`, `disputes >= 50`, `share >= 60%`
  - plans: `disputes >= 50` (tiered in existing JS at 40/60/80%).
- Updated non-response narrative counts in Logic 3 hero/insight text to match the rebuilt dispute-level data.
- Current rebuilt result from available local Q1+Q2 OON Emergency/Non-Emergency + Air Ambulance PUF files:
  - `PROV_FOCUS` rows: 63
  - unique flagged providers shown by KPI: 47
  - `PROV_NORMAL` rows: 50
  - `PLAN_FOCUS` rows: 225

**Date:** 2026-04-27  
**Data source:** `idre.idre_silver.fee_schedule_joined_oon_emergency_nonemergency` (Q1+Q2 2025, 2,747,354 rows, 55 columns)  
**All changes sourced from new enriched silver tables** — replacing the old `oon_emergency_nonemergency_fee_join` (39 cols) with the new table (+16 Medicare benchmark columns).

---

## Logic3_plan_nonresponse_monitor.html

### 1. Insight box text updated
- Old: "In 1 out of every 5 disputes, health plans simply don't respond… 81 providers won more than 60%…"
- New: "In more than 1 out of every 4 disputes, health plans don't respond… 253 providers won more than 60%…"
- Rationale: New combined Q1+Q2 data shows overall default-win rate is 26.5% (not ~20%), and 253 unique providers flagged (not 81).

### 2. KPI — Flagged providers
- Old hardcoded: `81`
- New: Dynamically computed from PROV_FOCUS array → `253` unique provider names
- KPI label and description unchanged.

### 3. KPI — Default-win disputes
- Old hardcoded: `21,654`
- New: Dynamically computed → `728,006` (full Q1+Q2 combined)

### 4. KPI — Overall default-win rate
- New hardcoded: `26.5%` (validated: 728,006 / 2,747,354 = 26.5%)
- Previously not shown; added as third KPI.

### 5. PROV_FOCUS array replaced
- Old: 104 entries (single quarter, providers with ≥60% default-win share)
- New: 412 entries from `prov_focus.csv` (all flagged providers Q1+Q2, share ≥60%, min 50 disputes)
- New field in source: `medicare_flagged` (not displayed, included in data)

### 6. PROV_NORMAL array replaced
- Old: 40 entries
- New: 50 entries from `prov_normal.csv` (high-volume context providers below 60% threshold)
- Largest providers by dispute volume (Singleton Associates 168K disputes, Sonoran Radiology 45K, etc.)

### 7. PLAN_FOCUS array replaced
- Old: 69 entries (flagged plans)
- New: 150+ entries from `plan_all.csv` (all plans with 50+ disputes in either quarter)
- PLAN_NORMAL set to empty `[]` — all plan data now in PLAN_FOCUS

### 8. Five new geographic HTML sections added
All sections use existing CSS classes (`.card`, `.full-card`, `.data-table`, `.bar-list`, `.bar-item`) — no new CSS.

#### 8a. Geographic Overview bar chart (`#l3GeoList`)
- Data: `L3_GEO` array — 15 states, sorted by default-win volume
- TX leads with 359K default wins (31.1% rate); FL second at 78K (37.6% rate)
- Bar length = volume; label = state + rate%

#### 8b. Q1→Q2 State Trend table (`#l3StateTrendBody`)
- Data: `L3_STATE_TREND` array — 25 state rows
- Sorted by absolute change magnitude
- NE improved most (-36.8 pp); PA worsened most (+17.2 pp)

#### 8c. Corporate Network Footprint table (`#l3NetworkBody`)
- Data: `L3_NETWORK` array — 25 domain+state combinations
- Sorted by default-win rate
- ventrahealth.com and premierchoicebill.com appear across multiple states with near-100% rates
- scp-health.com FL: 12,961 disputes at 85.7% default-win rate

#### 8d. Place-of-Service by State table (`#l3PosBody`)
- Data: `L3_POS_STATE` array — 20 state+POS combinations
- Oregon Emergency Room (65.8%) and Outpatient Hospital (64.5%) are highest-risk combinations

#### 8e. Plan × State Risk Matrix table (`#l3PlanStateBody`)
- Data: `L3_PLAN_STATE` array — 25 plan+state combinations
- Seven plan+state pairs at 100% loss rate (marpaihealth.com NJ, optum.com CA, fridayhealthplans.com TX, etc.)
- ibx.com PA: 7,077 disputes at 85.6% loss rate

### 9. Five new JavaScript render functions added
- `renderL3Geo()`, `renderL3StateTrend()`, `renderL3Network()`, `renderL3PosState()`, `renderL3PlanState()`
- All called from `drawCharts()` at the end
- Geographic sections are not filtered by the quarter/service dropdowns (they show combined data)

### 10. Scatter plot axis scaling made adaptive
- `provXMax` and `planXMax` now computed dynamically from the filtered data instead of being hardcoded
- Prevents axis overflow with larger new dataset

---

## Logic4_high_ask_monitor.html

### 1. KPI_DATA object replaced
| Metric | Old (all) | New (all) |
|---|---|---|
| Total disputes (parseable) | 1,623,428 | 2,185,659 |
| Disputes ≥2× QPA | 1,447,571 | 1,936,114 |
| Disputes ≥4× QPA | 920,636 | 1,237,228 |
| Disputes ≥6× QPA | 636,017 | 842,565 |
| Providers ≥2× QPA | 13,097 | 16,199 |
| Providers ≥4× QPA | 9,830 | 12,347 |
| Providers ≥6× QPA (provs6x) | 8,323 | 10,453 |
| Avg ask (≥6× providers) | 70.8× | 70.8× (unchanged) |
| Max ask (single dispute) | 88,700× | 3,857,500× |

- Q1 values updated: total=918,947; gt6x=344,400; provs6x=6,465; max_ask=2,675,000
- Q2 values updated: total=1,266,712; gt6x=498,165; provs6x=8,274; max_ask=3,857,500

### 2. KPI card #3 (Max ask) description updated
- Old: "WALKER LAKE EMERGENCY GROUP PC asked for 88,700× the expected rate and won"
- New: "Highest ask on a single dispute seen in Q1–Q2 2025 (intraoperative monitoring sector)"
- Value updated to `3,857,500×`

### 3. VOL_DRIVERS array replaced
- Old: Unspecified entries
- New: 20 entries from `vol_drivers.csv` (Q1+Q2, providers with most ≥6× QPA disputes)
- Singleton Associates remains top by volume (92,951 Q2; 70,167 Q1 disputes at ~10× avg)
- NPPA SERVICES notable outlier: 6,006 disputes at avg 821× QPA

### 4. HIGH_ASK_WINNERS array replaced
- Old: Entries with max ~88,700× avg
- New: 20 entries from `high_ask_winners.csv`
- CUSHING NEUROMONITORING tops at 67,651× avg (24 disputes, 83.3% win rate)
- NEURO SAFETY, PLLC: 35,361× avg (97.8% win rate)

### 5. PAIRS array replaced
- Old: Entries with max ~88,700× pair average
- New: 24 entries from `pairs.csv`
- CUSHING NEUROMONITORING / bcbstx.com: 135,229× avg (100% win rate, 12 disputes)
- Sherman Oaks Hospital / multiplan.com: 43,668× avg (100% win rate)

### 6. TRENDS array replaced
- Old: Entries
- New: 15 entries from `trends.csv`
- DAPPER, JESSICA: Q1 avg 13.3× → Q2 avg 72,389× (+544,178% change)
- NARANG, ASHISH: Q1 13.1× → Q2 6,776× (+45,682%)

### 7. Insight box updated dynamically
- Now populated by `renderKpis()` using new KPI_DATA
- Shows: "10,453 providers asked for 6× or more… 842,565 disputes… avg 70.8× QPA… max 3,857,500× QPA"

### 8. Five new geographic HTML sections added
All use existing CSS classes — no new CSS.

#### 8a. Geographic Overview bar chart (`#l4GeoList`)
- Data: `L4_GEO` array — 15 states by disputes at ≥6× QPA
- TX dominates volume (442,719 disputes); OK highest avg multiple (357×); OH notable (263×)

#### 8b. Q1→Q2 State Trend table (`#l4StateTrendBody`)
- Data: `L4_STATE_TREND` array — 25 state rows sorted by absolute change
- NC: avg ask jumped from 22.8× to 1,223× (+5,265%) — flagged for investigation
- WI declined from 3,480× to 2,053× (‑41%)

#### 8c. Corporate Network Footprint table (`#l4NetworkBody`)
- Data: `L4_NETWORK` array — 25 domain+state combinations
- nmaiom.com MI: 6,018× avg (15 providers, 84.3% win rate)
- neuroendomke.com WI: 5,660× avg (75 providers, 95.8% win rate)
- halomd.com appears in 14 states with high avg asks and win rates

#### 8d. Place-of-Service by State table (`#l4PosBody`)
- Data: `L4_POS_STATE` array — 20 state+POS combinations
- WI Ambulatory Surgical Center: 6,966× avg, 95.4% win rate
- RI Inpatient Hospital: 5,010× avg, 88.6% win rate
- Inpatient hospital (POS 21) and ambulatory surgical centers (POS 24) dominate extreme asks

#### 8e. Medicare Flag Rate by State bar chart (`#l4MedicareList`)
- Data: `L4_STATE_MEDICARE` array — 23 states
- OH: 1.4% of disputes exceed 3× Medicare benchmark (highest)
- States with high "QPA below 50% Medicare" rates (IN 83.7%, SC 82.2%, WA 82.1%) flagged with ⚠ indicator
- Reveals structural undervaluation of QPA relative to Medicare in key states

### 9. Five new JavaScript render functions added
- `renderL4Geo()`, `renderL4StateTrend()`, `renderL4Network()`, `renderL4PosState()`, `renderL4StateMedicare()`
- All called from `renderAll()` which fires on page load

---

## Data Sources Reference

| CSV | Rows | Used in |
|---|---|---|
| `prov_focus.csv` | 412 | L3 PROV_FOCUS array |
| `prov_normal.csv` | 50 | L3 PROV_NORMAL array |
| `plan_all.csv` | 319 | L3 PLAN_FOCUS array |
| `vol_drivers.csv` | 20 | L4 VOL_DRIVERS array |
| `high_ask_winners.csv` | 20 | L4 HIGH_ASK_WINNERS array |
| `pairs.csv` | 24 | L4 PAIRS array |
| `trends.csv` | 15 | L4 TRENDS array |
| `l4_kpi.csv` | 3 | L4 KPI_DATA object |
| `l3_geo.csv` | 15 | L3 geographic overview |
| `l4_geo.csv` | 15 | L4 geographic overview |
| `l3_state_trend.csv` | 25 | L3 Q1→Q2 state trend |
| `l4_state_trend.csv` | 25 | L4 Q1→Q2 state trend |
| `l3_network_geo.csv` | 25 | L3 corporate network footprint |
| `l4_network_geo.csv` | 25 | L4 corporate network footprint |
| `l3_pos_state.csv` | 20 | L3 POS by state |
| `l4_pos_state.csv` | 20 | L4 POS by state |
| `l3_plan_state.csv` | 25 | L3 plan×state risk matrix |
| `l4_state_medicare.csv` | 23 | L4 Medicare flag rate by state |
| `high_ask_medicare.csv` | 25 | L4 HIGH_ASK_WINNERS + PAIRS med field (provider-level avg x-Medicare) |
| `l4_medicare_kpi.csv` | 3 | L4 KPI_DATA Medicare threshold fields; Medicare KPI cards; Medicare threshold strip |
| `dual_flagged.csv` | 20 | L4 DUAL_FLAGGED array (new dual-flagged providers section) |

---

## Logic4 Medicare Integration (Pass 2) — 2026-04-27

### KPI Strip
- Grid expanded from 4 to 6 columns.
- Added **Providers >=3x Medicare** KPI card (619 overall; green color).
- Added **Disputes >=3x Medicare** KPI card (8,053 overall; green color).
- Source: `l4_medicare_kpi.csv`.

### Threshold Strip
- Section label updated to: "Disputes above provider ask thresholds — QPA and Medicare benchmarks".
- Added `>= 2x Medicare` threshold tile (14,675 disputes / 859 providers overall).
- Added `>= 3x Medicare` threshold tile (8,053 disputes / 619 providers overall).
- Medicare threshold values displayed in green to distinguish from QPA (red).

### HIGH_ASK_WINNERS — bar to table
- Right card in the two-column row converted from a bar chart to a table.
- New columns: `#`, Provider, Q, Avg xQPA, Avg xMedicare, Win %, Disputes.
- `med` field added to each row from `high_ask_medicare.csv` (joined by provider name + quarter).
- Providers without a Medicare rate match show `N/A` (no match for: Dr. Arvind Ahuja MD q1, KARLI GOLDSTEIN q1/q2, WALKER LAKE q2, PHYSIOLOGIC ASSESSMENT q1, RACHAEL HAVERLAND q2).
- Color coding: >=100x Medicare = red, >=10x = orange, >=3x = amber.

### PAIRS Table
- Added `Avg xMedicare` column (8 columns total).
- `med` field added to each pair using provider-level proxy from `high_ask_medicare.csv` (joined by provider name + quarter).
- Pairs with no Medicare match show `N/A`.

### DUAL_FLAGGED Section (new)
- New JS array `DUAL_FLAGGED` (20 rows from `dual_flagged.csv`).
- New HTML section: "Dual-flagged providers — extreme on both QPA and Medicare benchmarks".
- Table columns: #, Provider, Quarter, Disputes, Avg xQPA, Avg xMedicare, Win Rate.
- Placed after the TRENDS section, before geographic analysis.
- Notable entries: NPPA SERVICES (17,201x Medicare, 89% win rate), AMERICAN INTRAOPERATIVE MONITORING (17,096x Medicare).

### Medicare Flag Rate by State (restored)
- HTML section restored using existing `L4_STATE_MEDICARE` array and `renderL4StateMedicare()` function.
- Placed as the fourth (final) geographic section.
- QPA-below-50%-of-Medicare annotation changed from emoji to plain text: `(QPA<50% Mcare: X%)`.

### KPI_DATA
- Added Medicare fields to all three period objects (all, q1, q2):
  - `total_with_medicare`, `gt2x_medicare`, `gt3x_medicare`, `provs_gt2x_medicare`, `provs_gt3x_medicare`.
- Source: `l4_medicare_kpi.csv`.

### Insight Box
- Dynamic render updated to mention Medicare threshold alongside QPA.

### VOL_DRIVERS note
- Updated to reference NPPA Services' dual-flagged status (821x QPA and >15,000x Medicare).

---

## Files Modified

| File | Type of Change |
|---|---|
| `Logic3_plan_nonresponse_monitor.html` | Data arrays replaced; insight text updated; 5 new geographic sections added |
| `Logic4_high_ask_monitor.html` | Data arrays replaced; KPI_DATA replaced; insight text updated; 5 new geographic sections added; Medicare KPI cards; Medicare threshold strip; HIGH_ASK_WINNERS converted to dual-benchmark table; PAIRS xMedicare column added; DUAL_FLAGGED section added; Medicare state chart restored |
| `CHANGES.md` | Created (this file) |
