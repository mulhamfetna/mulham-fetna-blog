---
title: "Our track record can't cheat. Here's how we froze it"
slug: "track-record"
date: 2026-09-07
draft: false
description: "Nine configurations hash-frozen under a signed public protocol: every future evaluation window is out-of-sample by construction, losing windows publish under the same rules as winning ones, and no quiet edit is possible."
keywords: ["trading track record", "out-of-sample", "signed protocol", "pre-registration", "auditable research", "futures trading"]
tags: ["quantitative-finance", "trading", "reproducibility", "research-methods"]
categories: ["Research"]
series: ["Trading Strategy Finder"]
series_order: 6
showDate: true
showAuthor: true
showTableOfContents: true
---

Every trading track record you've ever seen asks you to trust its author about one thing: that the
rules weren't adjusted along the way. Strategies quietly swapped after a bad month, windows chosen
after the fact, losers left out of the tally. You can't audit any of it, so the record is worth
exactly as much as the author's word.

For the final act of this project, we built a track record where the author's word doesn't matter.

## Out-of-sample by construction

On 2026-08-31 we signed a protocol — a public document in the repository — that froze everything an
author could later fudge:

- **The universe:** exactly 9 of our 54 configurations, admitted by five pre-registered criteria
  applied to the forward books (the selection beat a random-set control — claim
  `LIVE-ALLOWLIST-FROZEN`). Not one slot was hand-picked.
- **The parameters:** the deployed set, pinned by cryptographic hash into the signed document. The
  replay tooling *refuses to run* if the universe file differs by one byte from the signed hash.
- **The rules:** one contract always, the engine's stated fill conventions, mechanical kill rules,
  and a "never-list" (no manual overrides, no window cherry-picking, no pausing to wait out a
  drawdown) — violations void the record's claims from that point, by the protocol's own text.

{{< mermaid >}}
flowchart LR
    P["✍️ signed protocol<br>2026-08-31, public in the repo"] --> U["THE UNIVERSE<br>9 of 54 configurations,<br>admitted by five pre-registered<br>criteria (beat a random-set control)"]
    P --> PAR["THE PARAMETERS<br>pinned by cryptographic hash —<br>replay refuses to run on a<br>one-byte difference"]
    P --> RU["THE RULES<br>1 contract · stated fills ·<br>mechanical kills · never-list<br>(violations void the record)"]
{{< /mermaid >}}

Because the parameters and universe were published *before* any future data existed, every recorded
window is out-of-sample **by construction**. When new data arrives, it is first audited against the
previous delivery for retroactive changes (a repaint check), then replayed with the frozen set. The
result becomes a new claim in the machine-verified ledger — a losing window under exactly the same
rules and prominence as a winning one. The protocol says so in writing: *a negative outcome is a
publishable result of the protocol, not a failure of it.* No verdict is allowed before a
pre-registered power threshold; interim windows are descriptive only.

Changes? Dated, attributed, append-only amendments — the signature claim's falsifier literally
counts them and checks their wording. There is no quiet edit.

{{< mermaid >}}
flowchart TB
    D["📦 new market data arrives"] --> A["repaint audit against the previous<br>delivery — any retroactive changes?"]
    A --> H{"universe file hash ==<br>the signed protocol's hash?"}
    H -->|"differs by one byte"| X["⛔ the replay tooling<br>refuses to run"]
    H -->|"match"| RP["replay the 9 frozen configurations<br>under the frozen rules"]
    RP --> CL["result becomes a new claim<br>in the machine-verified ledger"]
    CL --> PUB["published — a losing window under<br>the same rules and prominence<br>as a winning one"]
    PUB --> D
{{< /mermaid >}}

## Where this honestly places us

We wrote a positioning document (`docs/POSITIONING.md` in the repo) that grades this project rung
by rung against academic practice and trading-firm practice, each cell linked to its evidence. The
honest summary: on **verification governance** — pre-registration, powered negatives, machine-
replayable claims, a tamper-evident record — we operate at a standard we have not found in any
public trading research codebase. On **proven live edge** — the thing trading firms actually have —
our record is young by construction and says so; its first windows accumulate under the frozen
protocol, and per-slot verdicts need the sample sizes the protocol pre-computed. We claim the
method, and we let the record earn the rest at the only speed honesty allows.

## The project, wrapped

That closes the arc this series opened at
[the end-to-end overview](/projects/trading-strategy-finder/overview/): the famous strategy that
failed [225 of 225 pre-registered
configurations](/projects/trading-strategy-finder/orb-null-study/) (paper:
https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7428398); the [one place in the economic
calendar that pays](/projects/trading-strategy-finder/news-cpi-edge/); our own strategies
[kept to 17.6% by an honest forward window](/projects/trading-strategy-finder/forward-decay/);
the [machinery that made all those sentences
trustworthy](/projects/trading-strategy-finder/claims-ledger/) (paper:
https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7428478); and now a track record that
continues on autopilot — auditable by anyone, cheatable by no one.

The whole thing is open under AGPL-3.0, DOI-archived
([10.5281/zenodo.21473312](https://doi.org/10.5281/zenodo.21473312)), verifiable offline without
any market data, and runnable end-to-end on your own data feed:
[github.com/mulhamfetna/trading-strategy-finder](https://github.com/mulhamfetna/trading-strategy-finder).

*Built at BeInMedia (Nmo AI), Kuwait • Dubai • Doha, by Mulham Fetna, with thanks to Abd Ulfatah
Esper for project leadership and support.*
