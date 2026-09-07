---
title: "What 16 years of data and 3.5 months of honest testing taught us about trading strategies"
slug: "overview"
date: 2026-09-07
draft: false
description: "The end-to-end story of an open futures research programme: pre-registered studies, a 225-cell ORB null, one paying place in the economic calendar, honest out-of-sample decay, and a track record that can't cheat."
keywords: ["trading strategy research", "backtest overfitting", "pre-registration", "futures trading", "out-of-sample decay", "machine-verified claims"]
tags: ["quantitative-finance", "research-methods", "trading", "reproducibility"]
categories: ["Research"]
series: ["Trading Strategy Finder"]
series_order: 1
showDate: true
showAuthor: true
showTableOfContents: true
---

This is the story of a research project my team at BeInMedia ran from May to September 2026: a
systematic attempt to find out what actually survives in futures trading once you stop fooling
yourself. We built the machinery, paid for the answers, and published everything — including the
failures. The code, the evidence, and every number below are public and machine-verifiable:
[github.com/mulhamfetna/trading-strategy-finder](https://github.com/mulhamfetna/trading-strategy-finder).

Here is the whole arc, honestly told.

{{< mermaid >}}
flowchart LR
    A["May 2026<br>machinery built:<br>backtest engine · optimizer ·<br>claims ledger"] --> B["Jun–Aug 2026<br>the studies:<br>economic-calendar sweep ·<br>225-cell ORB grid ·<br>forward test of our own fleet"]
    B --> C["Aug 31, 2026<br>track record frozen<br>under a signed protocol"]
    C --> D["Sep 2026<br>two preprints filed ·<br>everything published,<br>including the failures"]
{{< /mermaid >}}

## The rule that shaped everything

Backtests lie — not because the math is wrong, but because the person running them chooses what to
try, what costs to assume, and what to show you. Our defense was structural: **every study was
pre-registered before it ran** (design, thresholds, controls frozen in writing first), **every
negative result had to prove it had the statistical power to see an effect**, **every positive had
to beat a dumb control**, and **every published number lives in a machine-verified claims ledger**
— re-derived from committed evidence files by our CI on every change, 79 claims and counting. If a
number in this series doesn't re-derive, our own build fails.

{{< mermaid >}}
flowchart LR
    A["💡 Study idea"] --> B["📋 Pre-registration<br>design · thresholds · controls<br>frozen in writing first"]
    B --> C["▶️ The study runs<br>(changes = dated,<br>append-only amendments)"]
    C --> D{"Result?"}
    D -->|"negative"| E["⚡ Power analysis required —<br>underpowered means 'no verdict',<br>never 'proven absent'"]
    D -->|"positive"| F["🎲 Must beat a dumb control<br>+ a placebo noise check"]
    E --> G["📒 Claims ledger entry<br>evidence committed to git ·<br>falsifier · declared blind spot"]
    F --> G
    G --> H["🤖 CI re-derives all 79 claims<br>offline, no market data —<br>a broken number = red build"]
{{< /mermaid >}}

## What we found — the short version

**1. The famous stuff doesn't survive costs.** We pre-registered a 225-cell grid of opening-range
breakout — the most-taught intraday strategy there is — over 16 years and nine futures markets.
Zero cells passed at realistic costs; the grid earned +$1.57M gross and lost $6.49M net. The
"edge" is smaller than the spread. Full story:
[the ORB null](/projects/trading-strategy-finder/orb-null-study/); the paper:
https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7428398.

**2. The economic calendar pays in exactly one place.** We swept 612 market×news pairs and then a
661-cell census of every remaining combination. One place makes money after stressed costs: riding
the US inflation (CPI) release on equity-index futures — a premium ordered by index sensitivity
(Nasdaq > S&P > Dow > Russell), e.g. long NQ into the release at +$133 per event after stressed
costs, t = 4.13, and ES alone at +$151 per event across 116 events (p = 0.0027). Everything else
moves violently and pays nothing — and Retail Sales *loses* money on seven instruments in both
directions. Full story: [the calendar sweep](/projects/trading-strategy-finder/news-cpi-edge/).

**3. Our own optimized strategies decayed out-of-sample — and we published that.** On a genuinely
fresh forward window our 54 deployed configurations made 3,733 trades: +$29,807 raw, **−$63,518**
at $25 per round-trip — 17.6% of what the calibration window promised. Along the way the forward
test caught a look-ahead bug in vendor data that had been silently inflating results. Full story:
[the forward test](/projects/trading-strategy-finder/forward-decay/).

{{< chart >}}
type: 'bar',
data: {
  labels: ['Calibration window (the promise)', 'Fresh forward window (the reality)'],
  datasets: [{
    label: 'Per-trade rate kept',
    data: [100, 17.6],
    backgroundColor: ['rgba(99, 102, 241, 0.55)', 'rgba(239, 68, 68, 0.65)']
  }]
},
options: {
  plugins: { legend: { display: false } },
  scales: { y: { title: { display: true, text: '% of in-sample per-trade rate' }, max: 100 } }
}
{{< /chart >}}

**4. Honesty can be automated.** The claims ledger, the falsifiers, the self-test that replays our
own five historical mistakes and demands the gate reject each one for the right reason — the whole
verification layer costs minutes per study and changed our conclusions more than once. That's the
part of this project we think generalizes far beyond trading. Full story:
[the claims ledger](/projects/trading-strategy-finder/claims-ledger/); the methodology paper:
https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7428478.

**5. A track record you can't cheat.** The project's final act: the surviving 9 of 54
configurations were frozen — parameters hash-pinned under a signed protocol — so every future
evaluation window is out-of-sample *by construction*. The rules are public; a losing window gets
published under the same rules as a winning one. Full story:
[the frozen track record](/projects/trading-strategy-finder/track-record/).

## What we'd tell anyone doing this kind of work

Write the experiment down before you run it. State costs first, not last. Demand power before
accepting a "no" and controls before believing a "yes". Publish the nulls — ours turned out to be
the most useful results we produced. And make the honesty mechanical, because willpower doesn't
survive a promising-looking backtest.

Everything is open under AGPL-3.0 (archived releases: [DOI
10.5281/zenodo.21473312](https://doi.org/10.5281/zenodo.21473312)); the engine runs on your own
data feed, and the entire claims ledger verifies offline without any market data.

*This research was carried out at BeInMedia (Nmo AI), Kuwait • Dubai • Doha, by Mulham Fetna, with
thanks to Abd Ulfatah Esper for project leadership and support.*
