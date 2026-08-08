University of Hawaiʻi at Mānoa · Shidler College of Business
FIN-321 International Finance & Securities
FX Transaction Hedging Project — Technical Specification

**Aerospace EUR Receivable — FX Transaction Hedge Model · Technical Specification**

> Technical specification for the FX transaction hedge model — the named-range contract, calculation flow, and validation checks, precise enough that an AI or a colleague could build the workbook from this document alone. This spec is the input the Stage 3 AI-assisted build works from.

| Field | Value |
|---|---|
| Created by | Kailani Phillips |
| Updated by | Kailani Phillips |
| Date Created | 2026-07-31 |
| Date Updated | 2026-07-31 |
| Version | 0.1 |
| LLM Used (optional) | Claude — used as drafter from Stage 1 memo + scenario data; corrected by author (see prompt-log.md) |
| Role | Treasury Analyst / FP&A Analyst |
| Audience | CFO / Director of Treasury |
| Companion Workbook | Student build, Stage 3 (`models/builds/`) |

## 1. Problem Statement

Our aerospace division holds a contracted **receivable of EUR 20,000,000**, quoted USD per EUR, due for settlement in **365 days**. The exposure is a straightforward transaction exposure: the firm will convert the EUR proceeds to USD at whatever spot rate prevails at settlement, and USD is the functional currency for costs, budgeting, and reporting. The objective of this model is to protect the USD value of the receivable against adverse EUR depreciation while keeping the true cost of that protection visible, so Treasury can present a board-approvable recommendation rather than an unhedged bet on currency direction. This is a corporate treasury decision, evaluated against a board-approved policy of managing material transaction exposures rather than speculating on FX.

## 2. Inputs (Known Variables)

All inputs are exposed as workbook named ranges so the Calculation Flow (§5) reads identically whether implemented in Excel, Python, or an AI prompt. Market inputs (spot, forward, rates, premia) are the only cells an analyst should adjust for scenario work; sources and access timestamps are recorded on the Notes tab.

### 2.1 Core Inputs

| Name | Description | Unit | Placeholder value | Stage-4 data source |
|---|---|---|---|---|
| `FC_AMT` | Foreign-currency receivable | EUR | 20,000,000 | Fixed by sales contract; unchanged at Stage 4 |
| `S0_in` | Spot rate at inception | USD per EUR | 1.1505 *(indicative)* | EURUSD spot, Yahoo Finance/Bloomberg, at Stage-4 market close |
| `F0_in` | Forward rate to settlement | USD per EUR | 1.0935 *(indicative)* | Scenario-assigned; replaced with a live 1-year outright forward quote at Stage 4 |
| `R_USD` | USD interest rate to settlement | Annual % | 3.63% *(indicative)* | Fed H.15 effective federal funds rate at Stage 4 |
| `R_FC` | EUR interest rate to settlement | Annual % | 2.25% *(indicative)* | ECB deposit facility rate / EURIBOR equivalent at Stage 4 |
| `T_DAYS` | Days to settlement | Days | 365 | Fixed by contract |
| `BASIS_USD` | USD-leg day-count denominator | Days | 360 (ACT/360) | Standard USD money-market convention |
| `BASIS_FC` | EUR-leg day-count denominator | Days | 365 (ACT/365, per course convention) | Standard course convention for EUR legs; see §4 note on real-world EURIBOR basis |
| `K_PUT` | Put option strike (receivable hedge) | USD per EUR | 1.1505 *(indicative, exactly at `S0_in`, no rounding)* | Reset to exactly equal live `S0_in` at Stage 4, same 4-decimal precision, unless the options desk's quote sheet specifies a rounding increment |
| `K_CALL` | Call option strike | USD per EUR | 1.1505 *(retained for template symmetry — not used in this receivable-only build)* | N/A for this scenario |
| `PREM_PUT` | Put premium, USD per 1 EUR | USD | 0.019 *(indicative)* | Scenario-assigned; replaced with live options-desk quote at Stage 4 |
| `PREM_CALL` | Call premium, USD per 1 EUR | USD | 0.024 *(retained for template symmetry — not used in this receivable-only build)* | N/A for this scenario |
| `STEP_FRAC` | Sensitivity grid step size | % | 1% | Fixed design parameter, drives `S_T_grid` |

### 2.2 Derived / Intermediate Values

| Name | Description | Source |
|---|---|---|
| `DF_USD` | USD accumulation factor to settlement | `1 + R_USD × T_DAYS/BASIS_USD` |
| `DF_FC` | EUR accumulation factor to settlement | `1 + R_FC × T_DAYS/BASIS_FC` |
| `FV_PREM_PUT` | Future value of put premium at settlement | `−PREM_PUT × FC_AMT × DF_USD` |
| `S_T_grid` | Sensitivity spot grid at settlement | `S0_in × (1 + n × STEP_FRAC)` for `n = −5…+5` |
| `USD_NO_HEDGE` | USD proceeds under no hedge | `S_T × FC_AMT` |

## 3. Tab Architecture

| Tab | Purpose |
|---|---|
| **Cover** | Scenario name, firm, receivable summary, prepared-by/date |
| **Legend/Key** | Color-coding key per Appendix B.3: yellow = input, blue = assumption, black = formula, UH Green = cross-tab link |
| **Inputs** | All named ranges from §2.1 and §2.2, each in its own labeled, colored cell |
| **Forward** | Step 2 forward-hedge calculation |
| **MoneyMarket** | Step 3 three-step money-market synthetic hedge + parity check |
| **Options** | Step 4 put-floor calculation |
| **Sensitivity** | `S_T_grid`, the Step 5 table, and the comparison chart (§8) |
| **Notes & Assumptions** | §4 restated in-workbook, plus data source/access-date log |

## 4. Assumptions & Constraints

- **Quote convention:** All rates expressed as USD per EUR. A higher quote means EUR appreciation.
- **Horizon:** Single-maturity model; `T_DAYS = 365`.
- **Day-count basis:** `BASIS_USD = 360` (ACT/360, standard USD money-market convention) and `BASIS_FC = 365` per this course's stated convention for EUR legs. *Note: real-world EURIBOR quoting is typically ACT/360, not ACT/365 — this spec follows the course's stated simplification rather than market convention, and this divergence should be footnoted in the workbook's Notes tab.*
- **Parity:** The money-market hedge is assumed to replicate the forward hedge under covered interest-rate parity; per the Auditability Checklist (§7.3), any gap beyond the stated tolerance is a test of parity, not a model error — **except** that this draft's placeholder `F0_in` (scenario-assigned) was set independently of the placeholder `S0_in`, `R_USD`, and `R_FC` pulled today, so a real, material gap is expected until Stage 4 replaces all four with mutually consistent, same-timestamp live data.
- **Option premium:** Paid upfront in USD, quoted per 1 unit of EUR (no contract multiplier). Premia are a negative cash flow at t₀, carried forward at `R_USD` via `DF_USD` to place them on the same footing as settlement-date USD proceeds.
- **Counterparty / credit risk:** Excluded. All derivatives assumed frictionless and creditworthy.
- **Transaction costs & bid-ask spreads:** Excluded from the base case; flagged as a sensitivity candidate in §9.
- **Tax / accounting treatment:** Excluded. Model reports pre-tax cash outcomes only.
- **Scenario construction:** `S_T` is varied deterministically across a grid; no probability weights or implied-volatility distribution are applied.

## 5. Calculation Flow

Written in named-range pseudocode, portable across Excel, Python, and AI prompts. Formulas below are for the receivable exposure; Step 7 notes the payable mirror (not used in this scenario).

**Step 1 — Derived inputs**
```
DF_USD       = 1 + R_USD × T_DAYS / BASIS_USD
DF_FC        = 1 + R_FC  × T_DAYS / BASIS_FC
FV_PREM_PUT  = −PREM_PUT × FC_AMT × DF_USD
```

**Step 2 — Forward hedge (certainty benchmark)**
```
USD_FWD = FC_AMT × F0_in
```
Locked at inception; invariant across the `S_T` grid.

**Step 3 — Money-market hedge (parity check)**
```
Borrow FC_AMT / DF_FC euros today (the discounted PV of the receivable).
Convert to USD at spot:      (FC_AMT / DF_FC) × S0_in
Invest USD to maturity:      USD_MM = (FC_AMT / DF_FC) × S0_in × DF_USD
```
At maturity, the EUR receivable repays the euro borrowing exactly, so the USD deposit is the locked-in proceeds.

**Parity check:** `USD_MM ≈ USD_FWD` within the tolerance in §7.3. A persistent gap beyond that tolerance indicates a violation of covered interest-rate parity in the quoted inputs (expected in this draft — see §4).

**Step 4 — Option hedge (put floor)**
```
USD_PUT(S_T) = MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT
```
Below `K_PUT`, the put pays the difference and the floor holds. Above it, the option expires worthless and the firm sells at the better market rate, net of the premium already paid either way.

**Step 5 — Sensitivity table** (one row per `S_T` in `S_T_grid`)

| Column | Output | Formula |
|---|---|---|
| No hedge | `USD_NO_HEDGE(S_T)` | `S_T × FC_AMT` |
| Forward | `USD_FWD` | constant across rows |
| Money market | `USD_MM` | constant across rows |
| Option (put) | `USD_PUT(S_T)` | `MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT` |
| Hedge profit | `HEDGE_PROFIT_k(S_T)` | `USD_k − USD_NO_HEDGE`, one sub-column per strategy k ∈ {forward, money market, option} |
| Overall winner | label | `ARGMAX(USD_NO_HEDGE, USD_FWD, USD_MM, USD_PUT)` |
| Best active hedge | label | `ARGMAX(USD_FWD, USD_MM, USD_PUT)` |

**Step 6 — Summary metrics (scalar outputs)**
```
USD_FLOOR_PUT = MIN(USD_PUT) across S_T_grid
USD_BASE_k    = USD_k evaluated at S_T = S0_in, for each strategy k
```

**Step 7 — Payable variant (not used in this scenario)**
This model is receivable-only; `K_CALL` and `PREM_CALL` are retained in §2.1 for template symmetry with the payable mirror (buy the forward, borrow USD/invest FC, use a call struck at `K_CALL`) but are not exercised in this build.

## 6. Outputs

| Output | Description | Format | Purpose |
|---|---|---|---|
| Input panel | All named-range inputs with units, sources, access dates | Top of Inputs tab | Single source of truth |
| Strategy summary | `USD_FWD`, `USD_MM`, `USD_BASE_k` per strategy, plus `USD_FLOOR_PUT` | Table above sensitivity grid | Executive at-a-glance |
| Sensitivity table | USD proceeds per strategy across `S_T_grid` ± 5% | Table on Sensitivity tab | Core analytical evidence |
| Hedge-profit columns | `USD_k − USD_NO_HEDGE` per strategy per row | Sub-table | Isolates hedge value-add |
| Winner / best-hedge labels | ARGMAX labels per row | Two label columns | Quick-read decision cue |
| Sensitivity chart | Line chart, USD proceeds vs. `S_T`, all four series | Embedded chart, styled per Appendix B.4 | Visual comparison |
| Executive summary (Stage 5) | Narrative with explicit recommendation | Separate memo | Downstream deliverable |

### 5.1 Computed Base-Case Values

*To be recorded once the Stage 3 workbook is built (`S_T = S0_in`); serves as a regression checkpoint.*

| Strategy | USD Proceeds (Receivable) | Hedge Profit vs. No Hedge |
|---|---|---|
| No hedge | *(pending build)* | — |
| Forward | *(pending build)* | *(pending build)* |
| Money market | *(pending build)* | *(pending build)* |
| Option (put) | *(pending build)* | *(pending build)* |

## 7. Model Review — Designed Against Known Pitfalls

This model is being built fresh in Stage 3, not repaired from a legacy tab, so this section is a **known-risks register**: common pitfalls in this class of model, and the design choice this spec makes to avoid each one from the start.

### 6.1 Design decisions adopted proactively
- **Standardized named ranges only.** No legacy names, no bare cell references (`$F$7`-style) anywhere in the formula layer.
- **Rigorous day-count split.** `BASIS_USD` and `BASIS_FC` are separate named ranges from the outset, rather than one shared `BASIS` value.
- **Explicit, unrounded strike.** `K_PUT` is set to the exact `S0_in` value (1.1505), not a rounded placeholder — closing a gap identified while drafting this spec (see prompt-log.md).
- **`STEP_FRAC`-driven grid.** `S_T_grid` is generated from a single `STEP_FRAC` input so the ±5% range stays symmetric and consistent, rather than hard-coded step sizes.
- **Chart included from the start**, styled per Appendix B.4, rather than added later.
- **Both a baseline and a floor reported** for the option strategy (`USD_BASE_k` at `S0_in` and `USD_FLOOR_PUT` as the grid minimum), not just the worst case.

### 6.2 Risks flagged for future iteration
- **Transaction-cost sensitivity is absent from the base case** (per §4) — a bid-ask/commission knob on the forward and a spread on the option premium is a strong candidate for Stage 4, since real treasury desks never transact at the mid.
- **Implicit-intersection formulas** (e.g., relying on Excel's automatic row-matching) are avoided in favor of explicit references throughout — this must be verified during the Stage 3 build itself, since it's easy to reintroduce accidentally when copying formulas across rows.

### 6.3 Auditability Checklist
- [ ] Every input has a standardized named range from §2.1
- [ ] Every formula in §5 uses named ranges — no bare cell references
- [ ] Money-market hedge ties to forward hedge within **0.05%** (parity check) — *expected to fail in this placeholder draft; must pass once Stage 4 supplies mutually consistent live data (see §4)*
- [ ] Put payoff at `S_T = K_PUT` equals `K_PUT × FC_AMT + FV_PREM_PUT` (kink verification)
- [ ] Sensitivity grid is symmetric around `S_T = S0_in` and driven by `STEP_FRAC`
- [ ] Notes tab records spot / forward / rate sources with access dates
- [ ] Cell colors match the legend: yellow inputs, blue assumptions, black formulas, UH Green cross-tab links (Appendix B.3)
- [ ] Zero error cells (`#DIV/0!`, `#VALUE!`, `#REF!`, `#N/A`) anywhere in the workbook

## 8. Sensitivity Plan

- **Grid:** `S_T_grid` spans `S0_in × (1 ± 5%)` in 1% increments (`STEP_FRAC = 1%`) → 11 rows including the baseline.
- **Strategies plotted:** no hedge, forward, money market, option (put).
- **Primary chart:** line chart, `S_T` on the x-axis, USD proceeds on the y-axis. Forward and money-market series are horizontal by construction; no-hedge is a straight line through the origin; the option is piecewise-linear with a kink at `K_PUT`. Styled per Appendix B.4 (UH Green solid forward line, UH Green 700 dashed money-market line, Neutral-600 dotted option line, black no-hedge line).
- **Secondary table:** hedge profit vs. no hedge for each strategy, making the visual intuition numeric.
- **What the chart should communicate:** the trade-off between certainty (forward/money-market, flat lines), optionality (put, kinked payoff with a floor), and naked exposure (no hedge, unbounded downside).

