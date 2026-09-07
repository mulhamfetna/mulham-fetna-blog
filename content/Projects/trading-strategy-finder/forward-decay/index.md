---
title: "We forward-tested our own strategies on data they'd never seen. They kept 17.6%"
slug: "forward-decay"
date: 2026-09-07
draft: false
description: "3,733 fresh trades across 54 deployed configurations: +$29,807 raw, −$63,518 at $25 per round trip — 17.6% of the calibration promise. Plus the vendor look-ahead bug the forward test caught."
keywords: ["forward testing", "out-of-sample decay", "walk-forward", "look-ahead bias", "futures trading", "backtest overfitting"]
tags: ["quantitative-finance", "trading", "backtesting", "data-quality"]
categories: ["Research"]
series: ["Trading Strategy Finder"]
series_order: 4
showDate: true
showAuthor: true
showTableOfContents: true
---

Everyone forward-tests other people's strategies. The uncomfortable experiment is doing it to your
own — the 54 optimizer-selected configurations you actually believe in — on a window of the future
that arrived *after* every parameter was fixed, with a falsifier written before the run.

We did it twice (round two after a data correction — more on that below). Here's what honest
out-of-sample performance looks like.

## The headline table

3,733 entirely new trades across all 54 configurations, nine futures markets:

| Fresh window, all 54 | trades | raw | at $10/round-trip | at $25/round-trip |
|---|---|---|---|---|
| total | 3,733 | **+$29,807** | −$7,523 | **−$63,518** |

The raw positive is statistically indistinguishable from zero (t = 0.88). Against what the
calibration window *promised*, the fleet kept **17.6%** of its per-trade rate — a decay that is
itself statistically significant (t = −2.53). The survivors of $25 costs are few and nameable: the
4-hour timeframe (+$10,106 net) and ES as an instrument (+$17,119). Fifteen of 54 configurations
stayed positive at $25; most of the rest are small-timeframe cells whose $4–7 gross per trade is a
commission illusion.

If you've read [our opening-range-breakout
post](/projects/trading-strategy-finder/orb-null-study/), you'll recognize the pattern from the
other side: this time it's *our* system paying the spread.

## The bug the forward test caught

Round one of this test produced ES results that looked wrong in a specific way — and chasing that
led to a genuine find: the vendor-derived reference levels for ES had been **shifted by one
business day at week and month boundaries**, a look-ahead. The corrected data moved ES's
full-history book from $74,237 down to $40,432 on one timeframe, and flipped another from +$12,042
to −$435. Every ES champion had been *selected* on the shifted data.

Two lessons we now operate by: **a forward test is also a data audit** — look-ahead hides
comfortably in-sample and dies loudly out-of-sample; and **after any data correction, re-run the
selection and check the incumbents** (we did, under a pre-registered decision rule: all six ES
incumbents were retained on the corrected data — the re-optimization found nothing better).

## What we did with the answer

We didn't average the pain away or wait for a better window. The five pre-registered survival
criteria were applied to the forward books as a frozen rule, and they admit **9 of 54** slots —
which beat a random-set control. Those nine became the project's live universe, hash-frozen under a
signed protocol ([that story](/projects/trading-strategy-finder/track-record/)). The other 45 are
retired from live consideration, with their verdicts on the record. Where the sample was too thin
to judge — 44 of 54 slots are individually under-powered, a physics problem, not a diligence
problem — the record says "no verdict", not "fine".

Every number above is a claim in the public machine-verified ledger
([github.com/mulhamfetna/trading-strategy-finder](https://github.com/mulhamfetna/trading-strategy-finder));
the decay table re-derives offline from the committed trade books on every CI run.

The series continues with [how that ledger
works](/projects/trading-strategy-finder/claims-ledger/).

*BeInMedia (Nmo AI), Kuwait • Dubai • Doha.*
