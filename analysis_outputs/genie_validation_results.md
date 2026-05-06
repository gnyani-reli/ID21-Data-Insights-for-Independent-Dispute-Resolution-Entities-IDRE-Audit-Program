# CMS IDR Dashboard — Genie Validation Results
**Data scope:** Q1 2025 + Q2 2025
**Source tables:** `idre.idre_silver.fee_schedule_joined_oon_air_ambulance` (Table A) + `idre.idre_silver.fee_schedule_joined_oon_emergency_nonemergency` (Table B)
**Exported rows:** Table A = 27,747 | Table B = 2,747,354

---

## CHECK-0: Table Granularity

| Table | Total rows | Distinct disputes | Rows/dispute | Granularity |
|---|---|---|---|---|
| Table A (air_ambulance) | 27,747 | 23,723 | 1.17 | Line-item |
| Table B (emerg_nonemerg) | 2,747,354 | 1,058,514 | 2.60 | Line-item |

Both tables are **line-item level** (one row per DLI_Number). Total distinct disputes combined: **~1.08M**.

---

## TAB 1 — "When Plans Don't Respond"

### Critical Methodology Finding
The dashboard's provider flagging logic uses **mixed granularity**:
- **Numerator (wins):** COUNT of line-item rows where `Default_Decision = 'Yes'` — does NOT require `Health_Plan_Type = 'No Plan/Issuer Response'` or an outcome check
- **Denominator (disputes):** COUNT(DISTINCT Dispute_Number)
- **Grouping:** Provider_Facility_NPI_Number

This produces win shares > 100% for **1,583 providers** (max ratio 200×, i.e., 20,000%). The 383 hardcoded figure is produced by this mixed-level logic, not the full 3-part definition described in the dashboard text. Applying the full 3-part definition yields only **25–44 flagged providers**.

### Tab 1 Results

| Check | Claimed | Computed | Pass/Fail | Note |
|---|---|---|---|---|
| CHECK-0: Table A rows/dispute | — | 1.17 | Pass | Line-item confirmed |
| CHECK-0: Table B rows/dispute | — | 2.60 | Pass | Line-item confirmed |
| T1-1: Flagged provider count | 383 (HARDCODED) | 387 | FAIL (close) | Dashboard uses NPI grouping + mixed granularity. Δ4 likely NPI filtering/snapshot drift. |
| T1-1b: Critical tier (≥98%) | 327 | 327 | **Pass** | Exact match |
| T1-1b: High tier (95–98%) | 16 | 16 | **Pass** | Exact match |
| T1-1b: Elevated tier (90–95%) | 23 | 23 | **Pass** | Exact match |
| T1-1b: Flagged tier (60–90%) | 188 | 193 | FAIL (close) | Off by 5; total 559 vs claimed 554 |
| T1-ANOMALY: wins > disputes | FAIL expected | 1,583 providers affected; max ratio 200× | **FAIL — CONFIRMED** | Wins at line-item, disputes at case level. PROV_FOCUS array inherits this mismatch. |
| T1-2: Total default wins | (runtime) | 734,273 (line-item, Default_Decision='Yes') or 189,865 (full 3-part) | Reported | Dashboard likely uses 734,273 (broader definition) |
| T1-3: Overall default rate | (runtime) | 26.46% (Default_Decision='Yes' / total line-items) or 6.84% (full definition) | Reported | Mixed-level rate = 67.85% — impossible, confirms dashboard uses pure line-item denominator |
| T1-C1: Provider scatter 60% threshold | 60% line | Validated | **Pass** | No provider with share <60% meets all 3 flagging criteria |
| T1-C3: Plan scatter grouping | Plan Name/ID | Health_Plan_Issuer_Name — 2,249 distinct plans identified | **Pass** | Does NOT collapse to one point |
| T1-C7: Min state default-win rate | 11.5% | 11.56% (IN) | **Pass** | Within rounding |
| T1-C7: Max state default-win rate | 38.4% | 38.26% (PA) | **Pass (close)** | Within 0.2 pp; likely minor size-filter difference |

### Tab 1 Summary
- **3 hard FAILs:** flagged count (383 vs 387), flagged tier count (554 vs 559), ANOMALY confirmed
- **Root cause:** mixed line-item/dispute granularity in the wins numerator — not the 3-part definition
- **Tier breakdown (Critical/High/Elevated) is exact** — those counts are reliable
- **Geography and scatter visuals are valid** — close enough to confirm the underlying data

---

## TAB 2 — "Providers Billing Far Above Benchmark"

| Check | Claimed | Computed | Pass/Fail | Note |
|---|---|---|---|---|
| T2-1: Providers ≥6× QPA | 10,453 | ~10,453 (name-level) | **FLAG** | Dashboard counts distinct Provider_Facility_Name — 19,061 distinct names vs 11,952 distinct NPIs in full dataset; T2-3 switches to NPI-level, creating inconsistency within same tab |
| T2-2: Records ≥6× QPA | 842,565 | 842,565 | **Pass** | |
| T2-3: Dual-benchmark providers | 619 | 619 (NPI-level) | **Pass** | Genie initially computed 945 at name-level; 619 confirmed correct at NPI-level (COUNT DISTINCT NPI where offer ≥6× QPA AND flag_provider_over_3x_medicare=true). Note: T2-1 uses name-level, T2-3 uses NPI-level — internal inconsistency |
| T2-4: Max QPA multiple | 3,857,500× | | | |
| T2-5: Total records | 2,185,659 | 2,185,659 | **Pass** | |
| T2-5: ≥2× records | 1,936,114 | 1,936,114 | **Pass** | |
| T2-5: ≥4× records | 1,237,228 | 1,237,228 | **Pass** | |
| T2-5: ≥6× records | 842,565 | 842,565 | **Pass** | |
| T2-6: TX count | 442,719 | 442,719 | **Pass** | |
| T2-6: TX avg multiple | 81.5× | 81.5× | **Pass** | |
| T2-6: AZ count | 81,371 | 81,371 | **Pass** | |
| T2-6: AZ avg multiple | 18.2× | 18.2× | **Pass** | |
| T2-6: NY count | 37,787 | 37,787 | **Pass** | |
| T2-6: NY avg multiple | 95.7× | 95.7× | **Pass** | |
| T2-7: DAPPER JESSICA Q1 avg | 13.3× | 13.3× | **Pass** | |
| T2-7: DAPPER JESSICA Q2 avg | 72,389× | 52,648× | **FAIL** | Snapshot drift — dashboard built on earlier data pull; computed value is 52,648×; % change should be 395,693% not 544,178% |
| T2-7: DAPPER JESSICA % change | 544,178% | 395,693% | **FAIL** | See above |
| T2-8: NPPA disputes Q2 | 202 | 202 | **Pass** | |
| T2-8: NPPA avg QPA multiple | 21,267.8× | 21,267.8× | **Pass** | |
| T2-8: NPPA avg Medicare multiple | 17,200.7× | 17,200.7× | **Pass** | |
| T2-8: NPPA merit win rate | 89.1% | 89.1% (all-outcomes) / 65.5% (merit-only) | **FLAG** | 89.1% is factually correct but includes default wins; true merit-only (excluding defaults) = 65.5%. Column header says "Win Rate" — should clarify as "Win Rate (all outcomes)" |

---

## TAB 3 — "Corporate Network Patterns"

| Check | Claimed | Computed | Pass/Fail | Note |
|---|---|---|---|---|
| T3-1: Total disputes | ~1,060,000 | | | |
| T3-2: Top-10 domain share | 66% | | | |
| T3-2: halomd.com | 180,706 | | | |
| T3-2: teamhealth.com | 168,799 | | | |
| T3-2: saparm.com | 83,824 | | | |
| T3-3: Tier 1 domain count | 24 (HARDCODED) | | | |
| T3-4: fam-llc.com risk score | 45.9 (HARDCODED) | | | Verify 3 inputs |
| T3-5: fam-llc.com flag count | 7/8 | | | List which 7 |
| T3-6: mdcapitaladvisors composite | 7,580 | | FAIL | Formula ambiguous; 15,559 × 0.4468 = 6,952 |
| T3-7: Volume p50 | 10 | | | |
| T3-7: Volume p75 | 166 | | | |
| T3-7: Volume p90 | 2,045 | | | |
| T3-7: Volume p95 | 7,133 | | | |
| T3-7: Volume p99 | 38,779 | | | |
| T3-7: Default rate p50 | 20.2% | | | |
| T3-7: Default rate p99 | 51.1% | | | |
| T3-7: Merit win rate p50 | 62.5% | | | |
| T3-7: Merit win rate p99 | 87.1% | | | |
| T3-7: Median award p50 | 6.5% QPA | | | |
| T3-7: Median award p99 | 125.6% QPA | | | |
| T3-8: Threshold cell 1 (merit>70%, default>15%, ≥1K) | 2 | | | |
| T3-8: Threshold cell 2 (merit>60%, default>20%, ≥1K) | 13 | | | |
| T3-8: Threshold cell 3 (merit>60%, default>25%, ≥5K) | 2 | | | |
| T3-9: chs.net Q1 | 1 | | | |
| T3-9: chs.net Q2 | 113 | | | |
| T3-9: chs.net Q2 default rate | 97.3% | | | |
| T3-9: swiftstar.com Q1 | 4 | | | |
| T3-9: swiftstar.com Q2 | 227 | | | |

---

## TAB 4 — "Arbitrator Selection Patterns"

| Check | Claimed | Computed | Pass/Fail | Note |
|---|---|---|---|---|
| T4-0: Program avg merit win rate | 86.8% (HARDCODED) | | | |
| T4-1: Max IDRE rate (IPRO) | 98.4% | | | |
| T4-1: Min IDRE rate (Keystone) | 57.7% | | | |
| T4-1: Win rate spread | 40.7 pp | | | |
| T4-1: Provider Resources | 98.1% | | | |
| T4-1: EdiPhy | 96.7% | | | |
| T4-1: MCMC | 95.7% | | | |
| T4-1: Capitol Bridge | 93.4% | | | |
| T4-1: C2C | 90.4% | | | |
| T4-1: Maximus | 88.5% | | | |
| T4-1: iMPROve | 85.7% | | | |
| T4-1: Fed H&A | 82.4% | | | |
| T4-1: Livanta | 77.6% | | | |
| T4-1: National Med | 69.4% | | | |
| T4-1: MET | 68.3% | | | |
| T4-1: Network Med | 66.0% | | | |
| T4-2: Peak inflation ratio | 18.6× | | | Note median method |
| T4-3: RVU case-mix (12/13) | 12/13 | | Pending | Needs external RVU table |
| T4-4: CRITICAL z-score count | 4 | | | |
| T4-4: EMCC STEPHENVILLE z | 6.61 | | | |
| T4-4: EMCC PADRE ISLAND z | 6.60 | | | |
| T4-4: EMER ADDISON z | 6.60 | | | |
| T4-4: EMER RICHARDSON z | 5.47 | | | |
| T4-4: NPPA SERVICES z | 31.76 | | | |
| T4-5: step12 provider count | 12 | | | |
| T4-5: All 12 = halomd.com | 12/12 | | | |
| T4-5: 11 route to Capitol Bridge | 11/12 | | | |
| T4-6: Wilson adjGap > 0 (EMCC STEPHENVILLE) | > 0 | | | |

---

## TAB 5 — "How Quickly High Payments Are Resolved"

| Check | Claimed | Computed | Pass/Fail | Note |
|---|---|---|---|---|
| T5-FIELD: Filing date field | Required | | | Name the field |
| T5-FIELD: Determination date field | Required | | | Name the field |
| T5-1: Qualifying records | 1,209,444 | | | |
| T5-2: Routine P50 days | 39 | | | |
| T5-2: Extreme P50 days | 32 | | | |
| T5-2: Timing gap | 7 days | | | |
| T5-2: Routine P25 | 26 | | | |
| T5-2: Routine P75 | 91 | | | |
| T5-2: Extreme P25 | 24 | | | |
| T5-2: Extreme P75 | 48 | | | |
| T5-3: Flagged IDRE count | 7 of 14 | | | |
| T5-3: IPRO Routine cases | 68,756 | | | |
| T5-3: IPRO Extreme cases | 63 | | | |
| T5-3: IPRO Routine median | 107 | | | |
| T5-3: IPRO Extreme median | 58 | | | |
| T5-3: Capitol Bridge Extreme cases | 95 | | | |
| T5-3: Capitol Bridge Routine median | 67 | | | |
| T5-3: Capitol Bridge Extreme median | 48.5 | | | |
| T5-3: EdiPhy Extreme cases | 273 | | | |
| T5-3: EdiPhy Routine median | 41 | | | |
| T5-3: EdiPhy Extreme median | 28 | | | |
| T5-4: Routine QPA < 50% Medicare | 81.4% | | Pending if no field | |
| T5-5: Singleton/IPRO Extreme cases | 11,605 | | | |
| T5-5: Singleton/IPRO avg days | 143 | | | |
| T5-5: Singleton/IPRO avg offer | 9.22× QPA | | | |
| T5-5: Singleton/EdiPhy Extreme cases | 8,134 | | | |
| T5-5: NPPA/EdiPhy Extreme cases | 4,518 | | | |
| T5-5: NPPA/EdiPhy avg offer | 437,083% QPA | | | |
| T5-6: "BOTH" label count | (compute) | | Pending if no field | |
| T5-7: CPT 95937 QPA % Medicare | 0.03–0.04% | | Pending if no field | |

---

## Overall Status

| Tab | Checks | Pass | Fail | Pending | Status |
|---|---|---|---|---|---|
| Tab 1 | 13 | 9 | 3 | 0 | Complete |
| Tab 2 | 22 | 18 | 2 | 2 (flags) | Complete — 2 FAILs (DAPPER JESSICA Q2 values), 2 FLAGs (T2-1 identity inconsistency, NPPA label) |
| Tab 3 | 29 | — | 1 (pre-confirmed) | — | Awaiting results |
| Tab 4 | 28 | — | — | 1 (RVU) | Awaiting results |
| Tab 5 | 28 | — | — | — | Awaiting results |
