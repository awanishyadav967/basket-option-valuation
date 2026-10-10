# Reproducing the basket-option demonstrations

This guide records the inputs defined in `BasketOptionPricer/Program.cs`. It is written in English and follows the executable demo code. Prices and run times are deliberately omitted because they depend on the build and execution environment.

## Run the examples

From the repository root:

```bash
dotnet run --project BasketOptionPricer/BasketOptionPricer.csproj
```

Choose `1` for the automatic H1/H2 demonstration, `3` for unit checks, `4` for functional scenarios, or `5` to inspect the supplied volatility surface.

## H1: constant parameters

The three-stock basket uses:

| Stock | Spot | Volatility | Dividend rate | Weight |
| --- | ---: | ---: | ---: | ---: |
| A | 100 | 0.20 | 0.02 | 0.50 |
| B | 120 | 0.25 | 0.015 | 0.30 |
| C | 80 | 0.18 | 0.025 | 0.20 |

The pairwise correlations are A/B `0.30`, A/C `0.20`, and B/C `0.40`; diagonal entries are `1`. The constant risk-free rate is `0.01933`, and maturity is one year. The demo sets the call strike to 105% of the initial basket value and the put strike to 95%. The initial weighted basket value is `0.5×100 + 0.3×120 + 0.2×80 = 102`, so those strikes are `107.10` and `96.90`.

The single-stock check uses spot `100`, volatility `0.20`, dividend rate `0.02`, weight `1`, strike `100`, the same rate, and a one-year maturity.

## H2: deterministic curves

The two-stock basket uses spots `100` and `120`, dividend rates `0.02` and `0.015`, weights `0.60` and `0.40`, and pairwise correlation `0.30`. The call strike is `110`, the put strike is `105`, and maturity is one year.

The rate curve has points `(0, 0.01933)`, `(0.5, 0.01933)`, and `(1, 0.01933)`. Stock A's volatility curve has points `(0, 0.20)`, `(0.5, 0.25)`, and `(1, 0.22)`. Stock B's volatility curve has points `(0, 0.18)` and `(1, 0.28)`. Values between points are linearly interpolated.

The demo also compares a one-stock H1 call and an H2 call with identical flat parameters: spot `100`, volatility `0.20`, dividend rate `0.02`, rate `0.01933`, strike `105`, and maturity one year.

## Monte Carlo comparison

The H2 call above is passed to `MonteCarloPricerH2` with random seed `42`. The demo requests `50,000` simulations without the optional control variate and then `50,000` with it. The same pricer instance is reused, so the two runs consume successive random draws. Reported standard errors and variance reduction should be interpreted as simulation diagnostics.

## Volatility surface

Menu choice `5` reads `SX5E_OVDV_2026-01-28_MID.csv` from the repository root. The CSV loader expects `valuation_date,expiry_date,forward,moneyness,implied_vol`. It converts the date difference to years using 365 days, interpolates volatility between moneyness and maturity points, and clamps requests outside the available range to the nearest edge. This surface is displayed separately from the H1/H2 demo assumptions.

The included ECB CSV is **not read by the program**. The `0.01933` rate above is a constant value in the source code.
