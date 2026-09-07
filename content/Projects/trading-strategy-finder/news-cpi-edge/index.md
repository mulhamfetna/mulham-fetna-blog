---
title: "We swept the entire economic calendar since 2010. Exactly one place pays"
slug: "news-cpi-edge"
date: 2026-09-11
draft: true
description: "A complete, pre-registered sweep of scheduled economic news on nine futures markets: the CPI release on equity-index futures is the only premium that survives stressed costs. Everything else — including Retail Sales — is a graveyard."
keywords: ["economic calendar trading", "CPI release", "news trading", "futures markets", "event-driven trading", "negative results"]
tags: ["quantitative-finance", "trading", "macro-news", "research-methods"]
categories: ["Research"]
series: ["Trading Strategy Finder"]
series_order: 3
showDate: true
showAuthor: true
showTableOfContents: true
---

Scheduled economic news — inflation prints, jobs reports, rate decisions, oil inventories — moves
futures markets violently. Every trading course says so, and it's true. We wanted the question
nobody answers with a complete sample: **which of these moves can you actually get paid for, after
real costs?**

So we tested all of it. Not the promising ones — all of it.

## The sweep

Three waves, each pre-registered before running: first 612 market×news pairs on nine futures
markets against the full calendar since 2010; then the secondary series (11,822 additional news
moments); finally a literal closure census of all 661 remaining series×instrument cells, so that
*every* combination in our registry has a recorded verdict — positive, negative-with-power, or
honestly "underpowered, no verdict". Costs stressed on every table.

## The answer: one place

**The US CPI (inflation) release, on equity-index futures.** That's it. The market jumps on many
things; it *pays* on this one:

- **Long NQ (Nasdaq futures), entering 300 seconds before the release,** fixed stop and target:
  **+$133 per event net of stressed costs, t = 4.13** — and it survives a Bonferroni correction
  across all 54 cells of that study design.
- **ES (S&P futures) riding CPI alone:** 116 events, **+$151.37 per event at $52.50 of stressed
  costs, p = 0.0027**, the release jump 20.5× the quiet baseline, both sample halves positive,
  control floor and a 1,000-placebo noise check green. Recent regime (2024–2026): +$529 per event.
- The premium is ordered exactly by index sensitivity: **Nasdaq > S&P > Dow > Russell.** It is an
  equity-index phenomenon — the same event on gold or oil pays nothing.

## The graveyard (equally important)

Of the 661-cell closing census, **exactly one cell** came out exploratory-positive. 179 cells are
significant negatives — and here's the instructive part: 41% of those lose *purely to costs* (their
gross edge is within $5 of zero), and 29% are actually gross-positive before friction. The market
really does move; the move really is bigger than quiet times; and the spread eats it. **Violence is
not premium.**

Two named tombstones worth remembering:

- **Retail Sales is an anti-premium.** Trading its jump *loses* money gross — not net, gross — on
  seven of nine instruments, in both directions. It replicated across two independent study waves.
  Whatever moves price at 08:30 on Retail Sales day, you are on the wrong side of it by
  construction.
- **Natural gas's own inventory report** jumps 8.5× baseline and grosses −$4.89 per event. The most
  market-specific news there is, and it pays nothing even before costs.

## Why you can trust these verdicts

Every claim above is a machine-verified entry in our public claims ledger — the number re-derives
from committed evidence files on every CI run, each with a falsification test and a declared blind
spot (the CPI results, for instance, declare their era-concentration openly: the magnitude lives in
the recent regime). Negatives were only allowed when the test had the power to see a cost-sized
effect. The repo: [github.com/mulhamfetna/trading-strategy-finder](https://github.com/mulhamfetna/trading-strategy-finder).

The broader story of the project — including the famous strategy that failed all 225 of its
pre-registered configurations — starts here:
[the end-to-end overview](/projects/trading-strategy-finder/overview/).

*BeInMedia (Nmo AI), Kuwait • Dubai • Doha.*
