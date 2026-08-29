================================================================================
STB Data Pilot Technical Assessment — README
================================================================================
Singapore Tourism Board (STB / STAN) tourism performance analysis
Notebook: STB submission.ipynb
Date: 2026-08-29
================================================================================

This README accompanies the submission notebook. It covers: (a) the external
datasets chosen and why, (b) the assumptions made, (c) the main challenges and
their caveats / limitations, and (d) what I would investigate next given more
time.

--------------------------------------------------------------------------------
a. EXTERNAL DATASETS — what I chose and why
--------------------------------------------------------------------------------

The brief asked for at least one external public dataset meshed with STAN data.
I used several, all cached under `other data/` so the notebook runs offline:

1. ECB / Frankfurter exchange rates  (other data/fx_monthly.csv)
   Why: the single biggest driver of a visitor's "is Singapore affordable today"
   question. STB gives me arrivals, receipts and survey behaviour but no price
   context. Meshing FX tells me how much each market's SGD purchasing power has
   shifted since 2019 — e.g. Japan lost ~30% and India ~23% of SGD buying power
   by 2025, while China and the UK lost the least. This re-frames "slower
   recovery" as partly a cost problem, not just a demand problem.

2. World Bank WDI GDP per capita  (other data/wb_gdp_percap.csv)
   Why: to separate how much of each market's tourism-receipt growth is REAL
   (intrinsic attraction) versus just income growth. I strip nominal and real
   GDP/cap growth and the FX move out of receipts growth to leave an
   "intrinsic attraction residual" per country (cell 52). 7 of 9 markets show a
   positive per-arrival residual — visitors are spending more per arrival than
   income and currency alone would predict.

3. SingStat CPI (all-items)  (other data/cpi_monthly.csv)
   Why: to distinguish behaviour change from price inflation. Nominal receipts
   per visitor rose ~+34% (2019->2025) but only ~+14% in real, CPI-deflated
   terms. Without a deflator I would have over-stated the value story.

4. Open-Meteo temperature + public/school holiday calendars
   Why: to test whether arrivals seasonality is weather- or calendar-driven.
   Each market's arrival index tracks its own holiday calendar closely, with
   some temperature correspondence for China and South Korea — so seasonality
   is programme-driven, not climate-driven.

5. STB "Tourism Receipts by Place of Residence" (user-supplied workbook,
   other data/tourism_receipts_by_residence_2019_2025.xlsx)
   Why: STB's headline receipts are not market-disaggregated; this residence-
   level series (top-10 markets, ex-Sightseeing/Entertainment/Gaming) lets me
   link each market's unique-visit share to its receipts per arrival — the
   evidence that "uniqueness pays" (r = +0.77, n = 9).

6. World Bank / FX purchasing-power inputs for the 2026 uplift sensitivity
   (other data/country_purchasing_power_2026.csv, with source URLs and
   retrieval dates) — used to re-weight the country mix under income-and-FX
   sensitivity.

--------------------------------------------------------------------------------
b. ASSUMPTIONS MADE
--------------------------------------------------------------------------------

Handling of missing / irregular data:
- 2019 is the 100 baseline everywhere: last full pre-COVID year, present in
  every source. 2020-Q2 2023 is treated as a distinct `covid_gap` regime and
  excluded from "normal period" comparisons rather than silently merged.
- Visitor-Profile survey has no observations for 2020-2022; the post-reopening
  trend is therefore fit on 2023-2025 only, with 2019 kept as a reference.
- Receipts workbook is quarterly-YTD: the Q4 value is the annual total. I never
  sum quarters.
- Receipts-by-residence: only the published top-10 markets per year exist; a
  blank cell means "not in that year's top-10" (not zero). 2025 is Jan-Sep
  top-3 only, so 2024 is used as the post-COVID reference for receipts-linked
  plots. Amounts exclude Sightseeing/Entertainment/Gaming.
- Market-name mapping (Mainland China->China, UK->United Kingdom) applied
  before joining receipts to arrivals/profile data.

Metric definitions:
- Attraction survey percentages are MULTI-RESPONSE visit rates (a visitor can
  report several attractions), so they do not sum to 100%. Tier averages use
  the arithmetic mean of POI visit rates — never a sum called a "share of
  visits".
- Unique-visit share = Tier-3 visits / all tiered visits * 100, computed on a
  STABLE venue set (venues present in both reference years) to avoid the
  Singapore River survey-label change.
- Receipts per arrival = receipts (S$m) / arrivals (millions) = S$ per visitor.
- Venue tiering (Tier 1 generic / Tier 2 / Tier 3 uniquely-Singaporean) is a
  transparent theory prior from destination-uniqueness literature, not an
  estimated or externally validated score.
- Purchasing-power index = (1 + real GDP/cap growth) x (SGD per unit of home
  currency, 2026/2025), winsorized to [0.85, 1.15] and treated as a
  sensitivity, not multiplied into the headline forecast.

Pilot assumptions:
- The Tier-2 -> Tier-3 uplift is an ASSOCIATIONAL, scenario-based estimate:
  the Low/Base/High cases apply explicit 25% / 50% / 100% realization factors
  to the observed historical gap. 2026 visit rates are a 2023-2025 slope capped
  to the historical one-year change range and bounded 0-100%. 2026 arrivals use
  same-month YTD scaling where data exist, else a 2023-2025 trend capped at
  +/-20%.

--------------------------------------------------------------------------------
c. MAIN CHALLENGES, CAVEATS AND LIMITATIONS
--------------------------------------------------------------------------------

1. Data availability gaps
   - No 2020-2022 Visitor-Profile survey: the Tier-2 vs Tier-3 comparison is a
     pre/post mean-difference, not a continuous or causal series.
   - Receipts by residence are top-10-only: receipts-linked correlations run at
     n = 9-10, so precision is limited (unique-share x receipts/arrival
     r = +0.77, n = 9; repeat x receipts/arrival r = -0.54, n = 10). Direction
     is informative; levels are indicative only.
   - No nationality x age cross-tabs, no individual-level spend data.

2. Prompt / framing ambiguity
   - "Tourists prefer cultural depth" and "uniquely Singaporean" needed a
     measurable definition. I operationalised it as a venue-tier prior and
     tested whether the data aligns with it. That prior is defensible but not
     causal.
   - The survey percentage semantics were ambiguous (share vs multi-response
     visit rate); I resolved this explicitly in favour of multi-response visit
     rates, which changes how figures may be compared.

3. Technical hurdles
   - Some external APIs (e.g. MAS FX, some data.gov.sg endpoints) were
     unavailable at fetch time; ECB/Frankfurter and the World Bank API were used
     as primary sources, and everything is cached so the notebook reruns
     offline.
   - A survey-label change on Singapore River venues required a stable-venue
     set to keep 2019 vs 2024/2025 comparisons apples-to-apples.

4. Caveats of the final analysis
   - The Tier-2 vs Tier-3 gap may reflect free-vs-paid admission, accessibility,
     capacity, tour inclusion, marketing and survey wording — not uniqueness
     alone. An actual upgrade does not guarantee a POI inherits every Tier-3
     characteristic.
   - No observed POI has ever changed tier in the STB data, so a causal
     "upgrading a Tier-2 venue to Tier-3 raises visits by X" treatment effect
     CANNOT be identified from this data. The uplift is a forecasted scenario,
     not a proven causal result.
   - All segment correlations are ecological (22 market-level points), so
     individual-level or causal claims are not made.
   - Indonesia and the Philippines have negative Tier-3 minus Tier-2 gaps; the
     aggregate uplift is positive overall but should be read market-by-market.
   - Real-terms figures use a generic all-items CPI as a proxy, not a
     tourist-specific price index.

--------------------------------------------------------------------------------
d. WHAT I WOULD INVESTIGATE NEXT (given more time)
--------------------------------------------------------------------------------

1. Run a genuine causal test of the Tier-2 -> Tier-3 uplift. The natural
   experiment is an ACTUAL POI transformation: pick one Tier-2 venue, upgrade
   its product / positioning toward the uniquely-Singaporean Tier-3 profile,
   and compare its visit-rate change against a matched Tier-2 comparison site
   (difference-in-differences). This is exactly the "observed POI changes tier"
   case the current data lacks, and would convert the scenario estimate into a
   decision-grade result.

2. Collect individual-level data to test the mechanism behind the aggregate
   shift: intercept surveys (why are visitors choosing unique venues - curiosity
   vs price), and if possible hotel check-in / partner POS / GPS-opt-in data to
   observe actual visit + spend at the person level. The repeat and solo
   segments do not carry the shift in market-level data; only individual data
   can show who does.

3. Age x market cross-tabs. STB publishes age aggregate-only; a cross-tab would
   tell us which markets send younger visitors and whether youth skews toward
   uniquely-Singaporean venues.

4. A controlled marketing / message pilot on the uniquely-Singaporean food-ways
   offer (e.g. "eat where locals eat" vs a value framing) with QR-tracked
   engagement and partner POS spend, to separate uniqueness motivation from
   price motivation.

5. Extend the receipts-by-residence analysis if STB can share more markets and
   a full-year 2025 residence table, which would lift the n = 9-10 correlations
   and let the spend-pay-off claim be tested on more markets.

6. Sensitivity of the uplift to admission-price / accessibility / capacity
   controls for each Tier-2 and Tier-3 venue, to isolate how much of the gap is
   uniqueness versus practical differences in visiting.

================================================================================
Reproducibility: run STB submission.ipynb top-to-bottom. Raw STB workbooks are
in data/; cleaned tables in cleaned_data/; external API data cached in
other data/ with source URLs and retrieval dates.
================================================================================
