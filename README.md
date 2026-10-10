# Basket Option Valuation

A C# console application for pricing a **basket option**: a call or put whose payoff depends on the weighted value of several stocks at maturity. It lets you compare a fast moment-matching approximation with a Monte Carlo simulation, try your own basket, and inspect an example implied-volatility surface.

If stock prices at maturity are `S₁, …, Sₙ` and their weights add to one, the basket value is `B = Σ(weightᵢ × Sᵢ)`. A call pays `max(B − strike, 0)`; a put pays `max(strike − B, 0)`. The pricers estimate the **present value** of that future payoff under their model assumptions. This is a learning and research project, not a live market quote or trading system.

## What is included

- **H1 moment matching:** assumes constant interest rate, volatility, dividend rate, and asset correlations. It calculates the basket's first two future moments, fits a lognormal approximation, and prices a call or put from that approximation.
- **H2 moment matching:** allows deterministic interest-rate and volatility curves. The code linearly interpolates curve points and numerically integrates rates and volatility covariance.
- **H2 Monte Carlo:** simulates correlated stock-price paths using Cholesky decomposition and reports an estimated option price and standard error. An optional control-variate calculation is used in the built-in demo; treat its reported variance reduction as a diagnostic, not a guaranteed accuracy gain.
- **Interactive console menu:** enter 2–5 stocks, weights, spot prices, volatilities, dividend rates, correlations, a risk-free rate, strike, maturity, option type, and a pricing method.
- **Sample volatility-surface viewer:** reads `SX5E_OVDV_2026-01-28_MID.csv` and interpolates implied volatility by maturity and moneyness. This menu option **displays** the surface; it does not automatically pass those volatilities into the basket pricers.
- **Built-in checks:** seven unit checks and eight functional scenarios are available through the console menu.

## Requirements and quick start

Install the **.NET 9 SDK**. From the repository root, build and run the console application:

```bash
git clone https://github.com/awanishyadav967/basket-option-valuation.git
cd basket-option-valuation
dotnet build PricingBasketOption_Verdelhan_Schmitt.sln
dotnet run --project BasketOptionPricer/BasketOptionPricer.csproj
```

The app shows this menu:

| Choice | Action |
| --- | --- |
| `1` | Run the predefined H1 and H2 pricing demonstrations |
| `2` | Enter your own basket and price a call or put |
| `3` | Run the seven unit checks |
| `4` | Run the eight functional scenarios |
| `5` | Load and inspect the sample SX5E volatility surface |
| `6` | Exit |

Run from the **repository root** so menu choice 5 can find the CSV file. The unit and functional checks are custom console routines invoked through choices 3 and 4; they are not a separate `dotnet test` project.

## Using interactive mode

Choose `2` and follow the prompts. For each stock, enter its name, current price, annual volatility, annual dividend rate, and basket weight. Enter rates as decimals: `0.20` means 20%. The program then asks for a risk-free rate and pairwise correlations, followed by call/put, strike, and maturity in years. It normalizes the weights if their entered sum differs sufficiently from one.

Choose **Moment Matching** for a fast H1 approximation, or **Monte Carlo** and a simulation count from 10,000 to 1,000,000. Interactive Monte Carlo converts the entered constant parameters into H2 model objects and reports a standard error and an approximate 95% confidence interval. Results are sensitive to the correlation matrix, assumptions, and random sampling.

## How the demonstration works

Menu choice `1` runs two examples defined in `Program.cs`:

1. **H1:** a three-stock basket with weights `0.5 / 0.3 / 0.2`, constant volatilities and dividends, a fixed rate of `0.01933`, and one-year call and put options.
2. **H2:** a two-stock basket with time-dependent volatility curves and the same rate at the defined curve points. It also compares H1 with H2 under flat parameters and prints standard and control-variate Monte Carlo results using 50,000 simulations each.

The `0.01933` rate is **hard-coded in the demo**. The included ECB CSV is a reference file; the application does not load it to update rates automatically. See [ReproductionParameters.md](ReproductionParameters.md) for the exact code-defined demo inputs in English.

## Project layout

| Path | Purpose |
| --- | --- |
| `BasketOptionPricer/Program.cs` | Console menu and predefined demonstrations |
| `BasketOptionPricer/Interface/` | Interactive input flow |
| `BasketOptionPricer/Models/` | Stocks, baskets, options, and deterministic curves |
| `BasketOptionPricer/Pricers/` | H1/H2 moment matching and H2 Monte Carlo |
| `BasketOptionPricer/Data/` | CSV volatility-surface loader and interpolation |
| `BasketOptionPricer/Utils/` | Normal CDF, Black-Scholes helper, random numbers, and Cholesky decomposition |
| `BasketOptionPricer/Tests/` | Console unit and functional checks |
| `SX5E_OVDV_2026-01-28_MID.csv` | Sample implied-volatility points used by menu choice 5 |
| `ECB Data Portal_20260126121402.csv` | Historical rate reference data; not read by the program |

## Limits

Moment matching approximates the distribution of a weighted basket; Monte Carlo estimates it by sampling paths. They can differ, especially under unusual parameters. The sample surface is independent of the pricing demos, and the supplied market files are historical snapshots, not live feeds. The code does not model transaction costs, calibration to current option prices, or a production risk workflow.

The application identifies **Verdelhan and Schmitt** in its console banner and solution name. Pricing results should be independently validated before any real financial use.
