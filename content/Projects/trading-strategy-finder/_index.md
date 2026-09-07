---
title: "Trading Strategy Finder — 16 Years of Futures Data, Honestly Tested"
description: "An open, machine-verified futures trading research programme: pre-registered studies, powered nulls, a 225-cell ORB study, the one place the economic calendar pays, and a track record that is out-of-sample by construction."
keywords: ["trading strategy research", "backtest overfitting", "pre-registration", "futures trading", "machine-verified claims", "quantitative finance", "reproducibility"]
draft: false
---

{{< github repo="mulhamfetna/trading-strategy-finder" showThumbnail=true >}}

A research programme run May–September 2026: a systematic attempt to find out what actually
survives in futures trading once you stop fooling yourself. Every study pre-registered before it
ran; every negative result required to prove statistical power; every positive required to beat a
dumb control; and **every published number machine-verified** — 79 claims that re-derive from
committed evidence on every CI run, offline, with no market data.

The code is open under AGPL-3.0 and DOI-archived
([10.5281/zenodo.21473312](https://doi.org/10.5281/zenodo.21473312)). Two companion preprints are
on SSRN: the [pre-registered 225-cell opening-range-breakout
study](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7428398) and the [machine-verified
claims ledger methodology](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7428478).

## The end-to-end series

Six articles cover the project from first principles to its frozen track record:

1. **[What 16 years of data and 3.5 months of honest testing taught us](/projects/trading-strategy-finder/overview/)** — the whole arc in one read. Start here.
2. **[We tested the most famous day-trading strategy. Zero of 225 versions survived real costs](/projects/trading-strategy-finder/orb-null-study/)** — the pre-registered ORB null.
3. **[We swept the entire economic calendar since 2010. Exactly one place pays](/projects/trading-strategy-finder/news-cpi-edge/)** — the CPI premium and the 661-cell graveyard.
4. **[We forward-tested our own strategies. They kept 17.6%](/projects/trading-strategy-finder/forward-decay/)** — honest out-of-sample decay, and the look-ahead bug the forward test caught.
5. **[Every number we publish is machine-verified. Here's the machinery](/projects/trading-strategy-finder/claims-ledger/)** — the claims ledger, falsifiers, and the self-test that distrusts itself.
6. **[Our track record can't cheat. Here's how we froze it](/projects/trading-strategy-finder/track-record/)** — the signed protocol and what comes next.

*This research was carried out at BeInMedia (Nmo AI), Kuwait • Dubai • Doha, by Mulham Fetna, with
thanks to Abd Ulfatah Esper for project leadership and support.*
