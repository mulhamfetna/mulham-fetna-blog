---
title: "Trading Strategy Validation"
date: 2026-10-30
description: "Independent, out-of-sample and cost-stressed validation of your trading strategy in Python, with a written report of what the data shows — including when a strategy has no edge after costs."
keywords: ["trading strategy validation", "backtest audit", "out-of-sample backtest", "walk-forward test", "transaction costs backtest", "Python backtesting", "Mulham Fetna"]
slug: "strategy-validation"
---

# Trading Strategy Validation

Before you trade a strategy, find out whether its edge survives data it has not seen and the costs
you will actually pay. I test your strategy in Python and give you a written report of the result,
whether it is positive or negative.

This service applies the method from my own published work:
[the pre-registered opening-range breakout study](/projects/trading-strategy-finder/orb-null-study/)
and the [machine-verified claims ledger](/projects/trading-strategy-finder/claims-ledger/), both on
SSRN.

## What I test

- **Out-of-sample performance:** the strategy is fixed on one period and judged on a later one it
  never saw.
- **Costs:** results with realistic commissions and slippage, plus a stressed-cost view.
- **Controls:** a random-entry control and a noise check, so a result is compared with chance and
  not only with zero.
- **Robustness:** walk-forward windows and parameter sensitivity, so one lucky setting does not
  carry the verdict.
- **Statistics:** bootstrap or permutation tests, with the sample size stated next to every number.

## Packages

| Package | What you get | Delivery |
|---|---|---|
| Starter | 1 strategy, 1 instrument, your data, costs applied, train/test split, 2-page report | 4 days |
| Standard | Adds walk-forward test, cost stress, random-entry control, report + code | 7 days |
| Advanced | Up to 3 instruments, parameter optimisation with an out-of-sample check, bootstrap test, report + code + call | 10 days |

Prices and ordering are on Upwork, which handles payment and contracts:
**[Mulham Fetna on Upwork](https://www.upwork.com/freelancers/~01132c092d08a2d084)**.
For questions before ordering, email **contact@mulhamfetna.com**.

## What this service is not

- No trading signals, no fund management, and no profit guarantees.
- No MetaTrader (MQL) or Pine Script development; the work is in Python.
- I report what the data shows. A strategy that loses its edge after costs is a valid and useful
  result.

## What I need from you

1. The strategy rules in writing, or the code that implements them.
2. Price data, or the instrument, timeframe and period so we can agree on a data source.
3. Your commission and typical slippage per trade.
