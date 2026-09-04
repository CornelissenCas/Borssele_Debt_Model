
# Borssele 1 & 2: Non-Recourse Debt Financing Model

A project finance debt model for the Borssele 1 & 2 offshore wind farm (752 MW, Netherlands), built from the perspective of the bank financing the 2016 tender. Companion project to the equity DCF and Monte Carlo analysis in [Project 1](../borssele-dcf-model).

**Read the notebook first**: `Borssele_Debt_Model.ipynb` contains the full analysis, validated line by line against the companion Excel model.

## Contents

| File | Description |
|---|---|
| `Borssele_Debt_Model.ipynb` | Full model: debt sizing, cash flow and waterfall, stress tests, Monte Carlo, sensitivity dashboard |
| `Borssele_Debt_Model.xlsx` | Companion Excel model, used to validate the Python implementation |
| `data/NL_wholesale_electricity_prices.csv` | NL wholesale day-ahead electricity prices (ENTSO-E), 2015 to 2026 |

## Methodology

Debt is sized off the lowest of three market-standard tests, not a single formula:

1. **DSCR-sculpted capacity**: the present value of a debt service stream set to a target coverage ratio (1.30x), discounted at the loan's own interest rate. This is solved with a closed-form NPV rather than an iterative circular calculation, because the sizing-stage cash flow deliberately excludes the interest tax shield, a conservative lender convention.
2. **Leverage cap**: 70% of total project cost.
3. **LLCR-implied capacity**: the present value of the same cash flow at a separate, higher discount rate (7% real), divided by the minimum required loan life coverage ratio (1.40x).

For Borssele, the LLCR test binds, not the leverage cap or the DSCR target. That single finding shapes most of the analysis: the facility is smaller and more conservatively sized than the leverage cap alone would suggest, which is why it clears every stress scenario tested, including a combined P90 production and 20% price shock.

The actual annual cash flow (as opposed to the sizing-stage approximation) applies the real Dutch corporate income tax brackets together with the ATAD earnings-stripping cap on interest deductibility, using the correct historical rate for each year (30% from 2019, 20% from 2022, 24.5% from 2025). A DSRA reserve, sized at six months of forward debt service, and a distribution lock-up test (both historic and forward DSCR above 1.20x) complete the waterfall.

The debt tenor (15 years) matches the SDE+ subsidy period exactly, so the facility fully amortises with no refinancing required inside the model horizon.

## Key results

| Metric | Value |
|---|---|
| Actual initial debt balance | EUR 920m |
| Binding constraint | LLCR |
| Realised DSCR, base case | 1.85x |
| LLCR at financial close | 1.40x |
| Minimum DSCR, combined stress (P90 + price shock) | 1.44x |
| Monte Carlo probability of a covenant breach | 4.4% |

## Key sources

Green Giraffe (2016 offshore wind financing terms, and the Gemini transaction benchmark), Fitch and LOFOTR (DSCR and LLCR covenant conventions), Asian Development Bank Treasury (2016 EUR swap rate), Forvis Mazars and Financial Edge (DSRA structuring), Belastingdienst and PwC (Dutch CIT and ATAD rates), European Investment Bank (Borssele co-financing), ENTSO-E (wholesale price data).

Full parameter-level sourcing is documented in the Assumptions sheet of the Excel model and the first section of the notebook.

## Running the notebook

Place `NL_wholesale_electricity_prices.csv` in a `data/` folder alongside the notebook. Restart the kernel, then run all cells in order from the top. Running cells out of order will raise `NameError`, since later cells depend on variables defined earlier.

The sensitivity dashboard in the final section includes an interactive, slider-driven version that only renders live inside a running Jupyter session.

## Limitations

- The distribution lock-up mechanism traps cash when it fails, but does not dynamically re-amortise the remaining schedule. A fully dynamic waterfall would require iterative solving.
- LLCR is evaluated at financial close only, not retested every year.
- A construction cost overrun is assumed to be funded entirely by additional sponsor equity, not additional debt.
- Years 16 to 25, after the debt is fully repaid, continue at the fixed ex-ante price for consistency with Project 1's ex-ante lens. Project 1's ex-post section already covers realised pricing in that period.
- The Monte Carlo price process is calibrated on yearly, pre-tender (2015 to 2019) data, consistent with Project 1. It reflects the price volatility a 2016 analyst could reasonably have expected, and by construction does not generate the scale of the 2021 to 2022 energy crisis, which was a multi-standard-deviation event relative to that historical calibration.
- The mean-reversion speed of the price process is a documented literature assumption, not fitted, because the pre-tender window is too short to estimate it reliably.
