# Scope 2 Emissions Checker

A small tool that cleans messy electricity-bill data, flags anything a human
should review, and calculates location-based Scope 2 emissions using official
Malaysian grid emission factors.

**Principle:** Python does the maths and the checks. AI writes the code and
drafts the client summary, and a script verifies the summary contains no number
that is not in the data.

## What it does
1. Cleans raw bills: unit conversion (kWh, MWh), thousands separators,
   mixed date formats, duplicate rows, site-name typos. Every automatic fix is logged.
2. Flags, never fixes: missing usage, missing months, kW (power) entered as energy,
   negative values, usage spikes (>3x site median), and bill/usage mismatches.
3. Calculates tCO2e = kWh x factor / 1000 for verified rows only.
4. Drafts a client summary with Claude, then verifies every number against the data.

## Results on the sample data
- 24 raw rows, 11 planted errors: 11 of 11 caught, no false alarms.
- No flagged row enters the totals.
- Verified total: 425.537 tCO2e from 19 of 24 site-months (79.2% coverage).

## Data and sources
- Emission factors: Energy Commission of Malaysia, Grid Emission Factor 2022-2024
  (provisional). Source URL is stored beside every factor in `emission_factors.csv`.
- `messy_bills.csv` is synthetic, built for testing. It contains no real client data.

## How to run
1. Open `scope2_checker.ipynb` in Google Colab (free).
2. Runtime -> Run all.
3. For the AI summary: copy the printed prompt into Claude, paste the reply into the
   `summary` cell, and run the verifier.

## Design decisions
- Everything is loaded as text, so pandas never guesses types on dirty data.
- The tool refuses to run without a sourced factor or a mapped grid region.
- The total is deliberately partial, and the output says so.
- Median baselines, not means, so a spike does not hide itself.

## Limitations
- No 2025 factor is published yet, so 2024 is used and a warning is printed.
- Factors are provisional, pending Government endorsement.
- Tested against one dataset I designed, not unseen data. This is the next step.
- Thresholds (3x, 25%) are judgment calls and would need tuning on real data.
- The summary verifier catches invented numbers, not wrong claims that use correct numbers.
- Location-based method only. Market-based Scope 2, Scope 1 and Scope 3 are out of scope.

## What I'd build next
- Test on unseen files and track false positives.
- Turn the cleaning and checks into a reusable Claude Code skill.
- Pull bills automatically from email or a shared drive instead of CSV upload.
