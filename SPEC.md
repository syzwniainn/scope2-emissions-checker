# Scope 2 Emissions Checker: Spec

## 1. Problem
Companies receive electricity bills in inconsistent formats. Cleaning them by hand
is slow, and errors flow into GHG reports. This tool cleans monthly bill data,
converts it to kWh, calculates location-based Scope 2 emissions, and flags anything
a human should review before the numbers are used.

## 2. Users
A sustainability analyst preparing a client's annual GHG inventory. They use the clean
table for calculations and the flags list to chase the client for corrections or
missing bills.

## 3. Input
- File: CSV with columns `site, month, usage, unit, bill_rm`
- Known problems expected:
  - Mixed units and unit spelling (kWh, kwh, MWh, kW)
  - Thousands separators in numbers ("13,250")
  - Mixed date formats (Jan-2025, 2025-03)
  - Site-name variants (case, trailing spaces, typos)
  - Exact duplicate rows
  - Missing usage values (N/A) and missing months
  - Suspicious values: spikes, negatives

## 4. Output
a) **Clean table:** site, month (YYYY-MM), kwh, bill_rm, source_row, needs_review
b) **Emissions:** tCO2e per site and in total, with kWh, plus coverage
   (verified site-months out of expected site-months)
c) **Flags list:** site, month, issue, suggested_action, source_row, implied_kwh_from_bill
   (review only, never inserted into the data)
d) **Changes log:** every automatic fix, with the source row it applied to
e) **Summary:** about 150 words, written for the client, using only verified numbers

## 5. Rules (non-negotiable)
1. Never fill a missing value silently.
2. Never invent an emission factor. It must come from a cited official source.
3. Flagged rows are excluded from totals until a human approves them.
4. Every number must trace back to a source row or factor.
5. If the tool cannot proceed safely (missing factor year, unmapped grid region),
   it stops with an error instead of guessing.
6. Every automatic fix is logged. Fuzzy matching is limited to names seen 3+ times.
7. The AI-written summary must pass a script that checks every number against the data.

## 6. Method
Emissions (tCO2e) = kWh x emission factor (kg CO2e/kWh) / 1000
- Location-based method, using the grid average emission factor
- Python does all calculations and all checks
- AI writes code and drafts text only, and its output is verified by script
- Thresholds: spike if usage > 3x site median; tariff mismatch if implied
  tariff differs from the site median by more than 25%
- Medians, not means, so a spike does not distort its own baseline

## 7. Out of scope (for this version)
- Scope 1 and Scope 3
- Market-based Scope 2 (supplier-specific factors, certificates)
- Multiple reporting years in one run
- Estimating or extrapolating missing months
- Live utility or email integrations (CSV input only)
- Real client data (sample data is synthetic)

## 8. Success criteria
- Catches 11 of 11 planted errors in the sample data
- No false alarms on valid rows
- No flagged row enters the emissions totals
- Does not crash on bad input, and stops cleanly when it should
- Summary contains no number that is not in the data
- Output prints the emission factor, year, status and source URL
