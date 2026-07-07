---
title: "Publishing Research & Software: The Complete A–Z Guide to Licenses, DOIs, and Making Your Work Citable"
date: 2026-07-07
description: "A verbose, step-by-step guide to open-science publishing: copyright vs. licenses, every license family (MIT to AGPL), DOIs and persistent identifiers, preprints vs. Zenodo vs. software journals, the full GitHub → Zenodo → DOI walkthrough, ORCID, and how to protect your work from plagiarism."
summary: "Everything I learned taking a research tool from a messy script to a licensed, permanently-archived, citable release with a DOI — the concepts, every option, and exactly when and why to use each."
tags: ["open science", "doi", "zenodo", "licensing", "research software", "github", "orcid", "arxiv", "citation"]
categories: ["Tutorials & Guides"]
showTableOfContents: true
---

# Publishing Research & Software: The Complete A–Z Guide

When you finish a piece of research or a software tool, the work is only half done. The other
half is **making it permanent, licensed, discoverable, and citable** — so that years from now
someone can find *exactly* what you made, use it under clear terms, and give you credit.

I recently took a research tool of mine from an untitled notebook to a polished, Apache-licensed,
permanently-archived release with a real DOI. This guide is the complete map of that territory:
every concept, every option, and — the part most tutorials skip — **when and why** to choose each
one. It's long on purpose. Read it once end-to-end, then keep it as a reference.

---

## Part 1 — The concepts you must get right first

Most confusion in this space comes from mixing up four different things. Let's separate them.

### 1.1 Copyright vs. license (they are not the same)

- **Copyright is automatic.** The instant you write code or text, *you own the copyright*. You do
  **not** need to register anything to "own" your work. So a license is **not** about claiming
  ownership — that's already yours.
- **A license is a grant of permission.** It tells *other people* what they may do with your work
  and what they must do in return.

Here's the counter-intuitive part: **if you publish code with *no* license, nobody may legally
use it at all.** "No license" means "all rights reserved" — the most restrictive state possible.
For research you *want* people to use and cite, that's the opposite of what you need. A license is
what *opens* your work up, on your terms.

### 1.2 A DOI (Digital Object Identifier)

A **DOI** looks like `10.5281/zenodo.21236404`. It is a **permanent address** for a specific
digital object, resolvable by prefixing `https://doi.org/`.

- A normal URL breaks when a site moves — this is called **link rot**.
- A DOI is a *promise*, maintained by a global registration agency, that the identifier will
  **always** resolve to your object, even if the file physically moves.

That permanence is why journals, theses, and grant reports require DOIs: a reviewer in ten years
must still be able to find precisely what you cited.

### 1.3 Persistent identifiers, in general

DOIs are one kind of **persistent identifier (PID)**. The ones a researcher meets:

| Identifier | Identifies | Issued by |
|-----------|-----------|-----------|
| **DOI** | an *output* (paper, dataset, software release) | DataCite / Crossref |
| **ORCID** | *you*, the researcher (a 16-digit iD) | ORCID.org |
| **SWHID** | a snapshot of *source code* | Software Heritage |
| **ROR** | a research *organization* | ROR.org |

You want your outputs (DOIs) linked to *you* (ORCID). More on ORCID in Part 6.

### 1.4 The mental model that makes it all click

| Thing | Role | Analogy |
|-------|------|---------|
| **GitHub / GitLab** | where the living, changing work happens (mutable) | your writing desk |
| **Zenodo / figshare** | a permanent archive storing a *frozen* copy (immutable) | a national library |
| **DOI** | a permanent address that always resolves to the work | the library ISBN + catalog card |

GitHub can change or be deleted; a DOI'd archive record **cannot**. That permanence, plus a fixed
date, is what turns "a repo" into "a citable, provable research artifact."

---

## Part 2 — Licensing, in full

This is where people freeze up. Let's make it a decision, not a mystery.

### 2.1 Why license at all — the four goals

1. **Enable legal use.** Without a license, others literally cannot use your work.
2. **Require attribution.** Every open license makes credit a *condition* of use — this is your
   single most practical anti-plagiarism tool.
3. **Set boundaries.** Commercial use allowed? Patents granted? Must derivatives stay open?
4. **Disclaim liability.** The "AS IS, no warranty" clause protects *you* if someone's use of your
   code causes harm.

### 2.2 The three families

**Permissive** — "do almost anything, just keep my name."

| License | Key trait | Best for |
|---------|-----------|----------|
| **MIT** | shortest; attribution only | simple projects, maximum adoption |
| **BSD-2 / BSD-3** | like MIT; 3-clause adds a no-endorsement rule | academic code |
| **Apache-2.0** | attribution **+ explicit patent grant** + `NOTICE` file | most research software; anything patent-adjacent |

**Copyleft** — "you may use it, but derivatives must stay open."

| License | Key trait | Best for |
|---------|-----------|----------|
| **LGPL** | weak copyleft; can be *linked* by closed software | libraries you want reused but kept open |
| **MPL-2.0** | file-level copyleft (weak) | middle ground |
| **GPL-3.0** | strong; distributing a derivative forces GPL | force downstream openness |
| **AGPL-3.0** | strongest; even *running it as a web service* triggers the openness requirement | SaaS-proof; maximum control |

**Content / data** — for things that aren't code.

| License | Key trait | Best for |
|---------|-----------|----------|
| **CC BY 4.0** | reuse with attribution | papers, figures, docs |
| **CC BY-SA** | attribution + share-alike | wikis, remixable content |
| **CC BY-NC** | non-commercial only | when you want to block commercial reuse |
| **CC0** | public-domain dedication (no rights kept) | datasets you want maximally reusable |

> ⚠️ **Do not use Creative Commons for source code**, and don't use software licenses for data.
> CC licenses don't address patents or source-vs-binary; software licenses don't fit prose/data.

### 2.3 How to choose — a quick decision guide

- **"I want it used as widely as possible, including by companies"** → **MIT** or **Apache-2.0**
  (Apache if patents might ever matter). Best for *citations and adoption*.
- **"Anyone who improves it must share back"** → **GPL-3.0**.
- **"…even if they only run it as a hosted service"** → **AGPL-3.0**. Strictest; note it *deters*
  corporate adoption, which can reduce reuse and citations.
- **"It's a library meant to be embedded"** → **LGPL** or a permissive license.
- **"It's data, figures, or a paper"** → a **CC** license (or **CC0** for maximum data reuse).

There's a real trade-off: **permissive = maximum reuse/citations; copyleft = maximum control.**
Pick based on which you value more for *this* project.

### 2.4 How to actually apply a license

1. Add a `LICENSE` file with the full text. Grab canonical text from
   [choosealicense.com](https://choosealicense.com) or, on the command line:
   ```bash
   gh api /licenses/apache-2.0 --jq '.body' > LICENSE
   ```
   Then fill placeholders like `[yyyy]` and `[name of copyright owner]`.
2. Use the license's **SPDX identifier** (e.g. `Apache-2.0`, `MIT`, `AGPL-3.0-or-later`)
   consistently in your `CITATION.cff`, package metadata, and archive deposit.
3. **Changing a license later?** For a solo project it's your call, but once there are outside
   contributors, relicensing may require their agreement — so choose deliberately up front.

---

## Part 3 — CITATION.cff: telling people how to cite you

A `CITATION.cff` file (Citation File Format — YAML) sits in your repo root. GitHub reads it and
renders a **"Cite this repository"** button that spits out APA/BibTeX automatically.

**The questions to answer before writing it:** title; type (software/dataset); one-sentence
abstract; version and release date; license (SPDX id); authors (name + ORCID each); 4–6 keywords.

```yaml
cff-version: 1.2.0
message: "If you use this software, please cite it as below."
title: "Your Project Title"
abstract: >-
  One or two sentences describing what it does.
type: software
authors:
  - family-names: Fetna
    given-names: Mulham
    orcid: "https://orcid.org/0009-0006-4432-798X"
version: 1.0.0
date-released: "2026-07-07"
license: Apache-2.0
url: "https://github.com/OWNER/REPO"
repository-code: "https://github.com/OWNER/REPO"
keywords: [keyword-one, keyword-two, keyword-three]
```

Once you have a DOI (Part 5), add it here too:

```yaml
doi: 10.5281/zenodo.21236404          # concept DOI (all versions)
identifiers:
  - type: doi
    value: 10.5281/zenodo.21236404
    description: Concept DOI (always latest)
  - type: doi
    value: 10.5281/zenodo.21236406
    description: Version 1.0.0
```

---

## Part 4 — The publishing landscape: every option, and when to use it

This is the section people most often get wrong: they think it's "arXiv *or* Zenodo." It isn't.
Different platforms archive **different objects**. Here's the whole map.

### 4.1 Preprint servers — for the *paper* (the narrative)

A preprint server hosts your **written manuscript** (PDF/LaTeX) *before* (or alongside) peer
review. It timestamps your findings and disseminates them fast; moderation is light, not peer
review.

| Server | Field |
|--------|-------|
| **arXiv** | physics, CS, math, engineering, quantitative bio |
| **bioRxiv / medRxiv** | biology / medicine |
| **ChemRxiv** | chemistry (relevant to materials work) |
| **TechRxiv** | engineering / IEEE |
| **EarthArXiv, SSRN, …** | earth sciences, social sciences, etc. |

*When:* you have a **paper** to share. *Why:* establish priority for your *results* and get
feedback early. (arXiv now also issues DOIs for submissions.)

### 4.2 Artifact repositories — for *software, data, figures*

These archive the **outputs behind** the paper and mint DOIs.

| Repository | Strengths | Notes |
|-----------|-----------|-------|
| **Zenodo** | free, CERN-run, GitHub integration, any file type | the default for code + data |
| **figshare** | polished UI, good for figures/posters | some features commercial |
| **Dryad** | curated datasets, strong in bio/eco | has a publication fee |
| **OSF (Open Science Framework)** | project management + storage + preregistration | free; ties a whole project together |
| **Harvard Dataverse** | institutional-grade data hosting | great for datasets |

*When:* you have **code, a dataset, figures, slides, a poster** — anything that should be
independently citable. *Why:* permanence + a DOI. **This guide's walkthrough uses Zenodo.**

### 4.3 Software Heritage — the automatic safety net

[Software Heritage](https://www.softwareheritage.org) **automatically archives all public source
code** it can find, giving each snapshot a **SWHID**. You can also "Save code now" manually.
*When:* always — it's passive insurance. *Why:* even if GitHub and you both vanish, the code
survives.

### 4.4 Software journals — peer-reviewed credit *for the code itself*

| Venue | What you get |
|-------|--------------|
| **JOSS** (Journal of Open Source Software) | free; peer-reviews your *software*; issues a short paper + DOI; very reputable |
| **SoftwareX** (Elsevier) | peer-reviewed software paper |
| **JORS** (Journal of Open Research Software) | open-access software paper |

*When:* your software is a genuine research contribution and you want a **peer-reviewed,
citable paper** about it. *Why:* it converts "a repo" into "a publication" on your CV. For most
research tools, **JOSS is the highest-value, lowest-cost option** — strongly recommended.

### 4.5 Package registries — distribution, NOT citation

**PyPI** (`pip`), **npm**, **conda**, **CRAN** exist so people can *install* your software. They
are **not archives and give no DOI.** Publish here *in addition to* Zenodo — never *instead of*.

### 4.6 The infrastructure behind the scenes (so the names stop being mysterious)

- **DataCite** — the DOI registration agency for *data and software*. Zenodo mints your DOIs
  through DataCite.
- **Crossref** — the DOI agency for *journal articles* (most published papers).
- **OpenAIRE** — *Open Access Infrastructure for Research in Europe*. An EU-funded initiative that
  aggregates metadata about publications, data, and software from thousands of repositories into
  one linked **research graph**, links outputs to EU **funding**, and powers discovery. **Zenodo
  was created by CERN under OpenAIRE**, so depositing on Zenodo automatically feeds your work into
  the European open-science graph. Think of OpenAIRE as the umbrella + discovery layer, and
  Zenodo as one concrete repository inside it.
- **ORCID** — the persistent ID for *you* (Part 6).

### 4.7 A complete modern release often uses several at once

> Manuscript on **arXiv/ChemRxiv** (or a journal) → code + data on **Zenodo** (DOI) →
> peer-reviewed via **JOSS** → installable from **PyPI** → passively archived by
> **Software Heritage** → everything linked to your **ORCID**. All cross-referenced by DOI.

---

## Part 5 — The full walkthrough: GitHub → Zenodo → DOI

This is the exact, repeatable process. Follow it top to bottom.

### 5.1 Prepare the repository

Before archiving, make the repo look like a research artifact:

- `README.md` — title, badges, install, usage, and — crucially for integrity — an **honest
  "Limitations" section** if your model/tool has them.
- `LICENSE` — chosen per Part 2.
- `CITATION.cff` — per Part 3.
- `requirements.txt` / manifest, `.gitignore`, optionally `CONTRIBUTING.md`.
- A `.zenodo.json` (optional but recommended) to control the deposit's metadata:

```json
{
  "title": "Your Project Title",
  "description": "One-sentence abstract.",
  "upload_type": "software",
  "access_right": "open",
  "license": "Apache-2.0",
  "creators": [{ "name": "Fetna, Mulham", "orcid": "0009-0006-4432-798X" }],
  "keywords": ["keyword-one", "keyword-two"]
}
```

Push it to GitHub:

```bash
git init -b main && git add -A && git commit -m "Initial release"
gh repo create REPO --public --source=. --remote=origin --description "…" --push
gh repo edit --add-topic topic-one,topic-two
```

### 5.2 One-time Zenodo setup

1. Go to [zenodo.org](https://zenodo.org) → **Log in with GitHub** → authorize.
2. Connect your **ORCID** in Zenodo profile settings (so deposits link to you).

### 5.3 Enable the repository — *and mind the order*

3. Open **zenodo.org/account/settings/github/** → find the repo → flip its switch **ON**
   (click *Sync now* if it isn't listed). This installs a **webhook** on the repo.

> ⚠️ **The #1 mistake — ordering.** You must enable the repo in Zenodo **before** publishing the
> release. Zenodo only archives releases created *after* the webhook exists. Publish first and
> Zenodo never sees it — nothing happens, and you'll think it's broken. *This is almost certainly
> why it "didn't work" the last time you tried.*

### 5.4 Publish a GitHub Release — the trigger

4. On GitHub: **Releases → Draft a new release → tag `v1.0.0`** → write notes → **Publish.**
   (Or `gh release create v1.0.0 --title "…" --notes "…"`.)

Only a **Release** triggers Zenodo — ordinary commits do not. A commit is a daily edit; a Release
is you declaring "*this* exact state is a finished, citable version."

### 5.5 Collect your DOI

5. Within ~a minute, the Zenodo GitHub page shows your record. Zenodo mints **two** DOIs:
   - a **concept DOI** — always resolves to the *latest* version (cite this for "the software");
   - a **version DOI** — points to *this* release forever (cite this for reproducibility).

Add a badge to your README. Wire it to the numeric **repo id** (`gh api repos/OWNER/REPO --jq .id`)
so it needs no DOI number in advance and auto-fills:

```markdown
[![DOI](https://zenodo.org/badge/REPO_ID.svg)](https://zenodo.org/badge/latestdoi/REPO_ID)
```

Then write the real DOIs into `CITATION.cff` and your README's BibTeX (Part 3).

### 5.6 Releasing new versions later

Just publish another Release (`v1.1.0`, …). Zenodo auto-archives it with a **new version DOI**
under the **same concept DOI**. Bump `version` and `date-released` in `CITATION.cff` first.

### 5.7 Work that isn't on GitHub

For a thesis PDF, a raw dataset, a poster: skip GitHub entirely. On Zenodo click **New upload**,
drag the files, fill in metadata, **Publish** → instant DOI. Zenodo also supports **embargoed**
records — the DOI and date are reserved *now*, but files stay hidden until a date you choose
(perfect for "I want proof I did this first, but can't reveal it until my paper is accepted").

---

## Part 6 — ORCID: a permanent ID for *you*

An **ORCID iD** (e.g. `0009-0006-4432-798X`) is a free, permanent identifier that disambiguates
you from every other researcher with a similar name, forever.

*Why it matters:* names change and collide; ORCID doesn't. Connect it to Zenodo, your GitHub
profile, journals, and your `CITATION.cff`, and every DOI you mint automatically ties into one
verifiable record of everything you've produced. **Do this once, early** — it compounds over a
career. Register at [orcid.org](https://orcid.org).

---

## Part 7 — Protecting your work from plagiarism

Be clear-eyed: **nothing technical prevents someone from copying public files.** What you *can* do
is make your authorship **provable and dated**, and copying **legally risky**. Layer these:

**1. Establish priority with a timestamp — the big one.**
A DOI'd archive record is **third-party-witnessed proof that *you* had this work on *this date*.**
If someone later claims it, you point to a permanent record predating theirs. Your **git commit
history** is a second dated authorship trail. Counter-intuitively, **publishing early protects
you**: a private file proves nothing about *when* you made it; a dated public record proves
everything.

**2. Set enforceable terms with a license.**
Every open license makes **attribution a legal condition**. Strip your name and the user is in
license violation — grounds for a takedown or a formal complaint. Copyleft (GPL/AGPL) goes
further, forcing derivatives to stay open and credited.

**3. Make citation the path of least resistance.**
`CITATION.cff` + a visible DOI badge mean the *easy, normal* thing is to cite you. Most academic
"plagiarism" is lazy non-citation, not malice — remove the excuse.

**4. If it's genuinely sensitive / unpublished.**
Keep the repo **private** until you're ready, and use an **embargoed Zenodo deposit** to lock in
the date *now* while hiding the files until your paper publishes.

**5. Enforcement, if it happens.**
GitHub has a **DMCA takedown** process for copied repos; your institution's research-integrity
office handles academic misconduct. Your dated DOI + git history are the evidence.

**Recommended posture for work you *want* credit for:** public repo + a license + Zenodo DOI +
ORCID. For secret pre-publication work: private repo + embargoed deposit.

---

## Part 8 — Putting it together: a reusable checklist

```
ONE-TIME
  □ Register an ORCID (orcid.org)
  □ Log in to Zenodo with GitHub; connect your ORCID

PER PROJECT
  □ Write README (with honest limitations), choose + add LICENSE
  □ Add CITATION.cff and .zenodo.json
  □ Push to GitHub (public), add topics
  □ Enable the repo in Zenodo  ← BEFORE the release
  □ Publish a GitHub Release (v1.0.0)  → DOI is minted
  □ Add the DOI badge; write DOIs into CITATION.cff + README
  □ (Optional) Submit to JOSS for a peer-reviewed software paper
  □ (Optional) "Save code now" on Software Heritage

EACH UPDATE
  □ Bump version/date in CITATION.cff → publish a new Release → new version DOI
```

---

## Quick reference

| I want to… | Use |
|------------|-----|
| Share a **paper/preprint** | arXiv, bioRxiv, ChemRxiv, TechRxiv |
| Make **code/data citable** with a DOI | **Zenodo** (or figshare, Dryad, OSF) |
| Get a **peer-reviewed software paper** | **JOSS**, SoftwareX, JORS |
| Let people **install** my software | PyPI, npm, conda (no DOI) |
| **Auto-archive** my source code | Software Heritage |
| Identify **myself** permanently | **ORCID** |
| Maximize **reuse/adoption** | MIT / Apache-2.0 |
| Force derivatives to **stay open** | GPL-3.0 / AGPL-3.0 |
| License **data/figures/prose** | CC BY / CC0 |

---

## Conclusion

Publishing well isn't bureaucracy — it's how your work becomes **permanent, usable, and credited**
instead of a file that quietly rots on a hard drive. The core loop is short: **license it,
archive it, get a DOI, tie it to your ORCID.** Do that once and it becomes muscle memory for every
project after.

*This guide is drawn from actually taking one of my own research tools through the entire process.
If you get stuck on the Zenodo step, re-read Part 5.3 — the ordering trap catches almost everyone.*
