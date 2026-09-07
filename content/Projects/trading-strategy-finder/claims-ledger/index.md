---
title: "Every number we publish is machine-verified. Here's the machinery"
slug: "claims-ledger"
date: 2026-09-07
draft: false
description: "A claims ledger where every published number is an object bound to committed evidence, three verifications that must fail for different reasons, a falsifier, and a declared blind spot — replayed by CI offline with no market data."
keywords: ["machine-verified claims", "reproducibility", "research verification", "pre-registration", "falsification", "research software"]
tags: ["reproducibility", "research-methods", "software-engineering", "quantitative-finance"]
categories: ["Research"]
series: ["Trading Strategy Finder"]
series_order: 5
showDate: true
showAuthor: true
showTableOfContents: true
---

The hardest problem in trading research isn't statistics — it's that the researcher grades their
own homework. You choose what to try, what costs to assume, when to stop, and what to show. After
three and a half months of running a research programme under that temptation, we're convinced the
only defense that holds is **mechanical**: make it impossible to publish a number that doesn't
re-derive from evidence.

This post describes the system we built and now run everything through. The full methodology paper
is at https://papers.ssrn.com/sol3/papers.cfm?abstract_id=7428478; the code is public.

## A claim is an object, not a sentence

In our repository, a published number isn't prose — it's a registered object with required fields:
the statement with its numbers inline; the **evidence files** that produced it (committed to git —
a claim whose evidence isn't version-controlled is rejected structurally); an **executable
re-derivation** compared to the published value within an explicit tolerance; **three independent
verifications that must fail for different reasons**, one of which is a *falsifier* — a test built
so that a specific way of being wrong would trip it; and a **mandatory declared blind spot**,
because "verified, with no stated limitation" is itself a defect.

{{< mermaid >}}
flowchart TB
    C["📄 One published claim"] --> S["the statement,<br>numbers inline"]
    C --> EV["evidence files —<br>must be git-tracked<br>(untracked ⇒ rejected structurally)"]
    C --> R["executable re-derivation<br>must equal the published value<br>within an explicit tolerance"]
    C --> V["three verifications that<br>must fail for DIFFERENT reasons"]
    V --> F["one is a falsifier — built so a<br>specific way of being wrong<br>would trip it"]
    C --> B["mandatory declared blind spot<br>('verified, with no stated<br>limitation' is itself a defect)"]
{{< /mermaid >}}

All 79 current claims replay offline in minutes with **no market data**, and CI runs them on every
change: break a published number and the build goes red.

## The gate that distrusts itself

A verification gate that has never failed is untested. So the ledger ships a self-test that replays
**five real mistakes from our own history** — reconstructed exactly as originally published — and
requires the harness to reject every one: a figure checked against notes instead of its producing
artifact; a true statement about one data series silently generalized to 649; a "verified" claim
with nothing that could have come out false; a retracted result reused without its retraction
marker; an absolute stop that was silently a different percentage on every instrument.

And the self-test's own origin story is the best argument for it: on its first run, two replays
were "correctly rejected" — by a file-path crash, not by the defect. Green for the wrong reason is
the same disease one level up. It now demands each rejection match the *specific* expected failure.

{{< mermaid >}}
flowchart LR
    P["any push or pull request"] --> CI["CI replays all 79 claims<br>offline · minutes · no market data"]
    CI -->|"every number re-derives"| G["✅ build green"]
    CI -->|"one number off"| R["❌ build red —<br>the repo refuses to publish it"]
    ST["self-test: 5 real historical mistakes,<br>reconstructed as originally published"] --> Q{"does the harness<br>reject each one?"}
    Q -->|"rejected for the RIGHT reason"| OK["✅ the gate is certified"]
    Q -->|"rejected by a crash or<br>for the wrong reason"| BAD["❌ self-test fails —<br>green for the wrong reason<br>is the same disease"]
{{< /mermaid >}}

## Two episodes where the machinery overruled us

**The +$260k that wasn't.** A re-optimization improved its training window by +$260,346. On the
2026 holdout the challengers collapsed — and then our first measurement of that collapse was
*itself* retracted the same day: the holdout was out-of-sample for the challenger but in-sample for
the incumbent. Asymmetric comparison, worthless number. What survived was baseline-independent: six
of twelve re-optimized challengers lost money outright on data they'd never seen; zero of twelve
improved it. Nothing was adopted, and "out-of-sample must be out-of-sample for **both** sides" is
now a standing rule.

**The 8-of-8 that became 5-of-8.** A search variant beat its control in eight of eight seeded runs,
median +55%. On eight *fresh* seeds — same pre-declared criterion — it won five of eight at
−4% median. Two runs agreeing on the same seeds had felt like corroboration; it was the same eight
dice rolls counted twice. Fresh-seed replication is now a pre-registration requirement.

## The rest of the discipline, in one paragraph

No study runs without a pre-registration filed first; changes mid-run are dated, append-only
amendments stating what was known at filing time. No negative verdict without a power analysis (an
under-powered "didn't work" is labelled *no verdict*, never *proven absent*). No positive without a
dumb control and a bootstrap noise check. Costs lead every table. And failed campaigns stay in the
database under their own names — prefixes are never reused, so the graveyard is part of the record.

None of this is expensive. The [225-cell
study](/projects/trading-strategy-finder/orb-null-study/) computed in 90 seconds; the discipline
around it is organizational, not computational. If you do quantitative research of any kind — not
just trading — the pattern transfers whole:
[github.com/mulhamfetna/trading-strategy-finder](https://github.com/mulhamfetna/trading-strategy-finder).

Final post — [freezing a track record so it can't
cheat](/projects/trading-strategy-finder/track-record/).

*BeInMedia (Nmo AI), Kuwait • Dubai • Doha.*
