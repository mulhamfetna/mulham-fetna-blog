# Website Refactor Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure mulhamfetna.com into a truthful, audience-routed, AI-citable professional site that works as a LinkedIn replacement.

**Architecture:** Hugo + Blowfish, no theme fork. Six audience lanes replace a 12-item menu; existing URLs are preserved and lanes are hub pages that route into them. A canonical `data/facts.yaml` ends the site's self-contradictions. GEO comes from `Person` + `FAQPage` JSON-LD via Blowfish's sanctioned `extend-head` hook and one new `faq` shortcode.

**Tech Stack:** Hugo v0.154.5+extended, Blowfish v2.100.0, TOML front matter, Cloudflare Pages.

**Spec:** `docs/superpowers/specs/2026-07-13-website-refactor-design.md`

## Global Constraints

- Stay on Hugo + Blowfish. No theme fork, no JS/CSS frameworks, no edits inside `themes/blowfish/`.
- Images: `assets/img/...` → referenced as `/img/...` (never `/assets/img/...`). Page-bundle images by bare relative filename. `static/` files as `/file.ext`.
- New content via `archetypes/default.md` (TOML front matter).
- New templates use `hugo.Data`, never deprecated `.Site.Data`.
- Section renames require lockstep `menus.en.toml` update **and** `aliases` front matter for old URLs.
- No tests exist. **Verification for every task = `hugo --gc --minify` builds with no NEW warnings, plus grep assertions against `public/`.**
- Baseline (pre-change): build succeeds, **140 pages**, 58 aliases. Only warnings are the GitHub remote-fetch failures fixed in Task 4.
- Branch: `refactor/site-restructure`. Commit after every task.

## Canonical facts (from spec §3 — never contradict these)

| Key | Value |
|---|---|
| teaching_years | 4 (since 2022) |
| learners_total | 500+ across all programs, 2022–2026 (**always state with basis**) |
| medals | 65 total — 12 gold, 21 silver, 32 bronze (2024) |
| src1 | 75 enrolled → 50 continued → 35+ graduated (2025) |
| src2 | 50 researchers (2026, ongoing) |
| paper_airplanes | 60+ women across 6 countries |
| personal_branding_101 | 270 online + 75 in person (2026) |
| papers | 3 under peer review (NOT published) |
| open_source | 18+ PRs (OpenCV, Ultralytics, OpenDR) |
| orcid | 0009-0006-4432-798X |

---

# PHASE 0 — Truth & hygiene

## Task 1: Canonical facts data file + `fact` shortcode

**Files:**
- Create: `data/facts.yaml`
- Create: `layouts/shortcodes/fact.html`

**Interfaces:**
- Produces: `{{< fact key="medals_total" >}}` renders a single canonical value as text. Later tasks use this on the homepage proof strip and About.

**Note on custom-code count:** the spec §8 claimed the custom surface stays flat at 7 files. Adding `fact.html` makes it **8**. This is a deliberate, justified deviation: `fact.html` is the only mechanism that structurally prevents the number-drift that this whole phase exists to fix. Hardcoding the corrected numbers would let them drift again.

- [ ] **Step 1: Create `data/facts.yaml`**

```yaml
# Canonical facts for mulhamfetna.com — SINGLE SOURCE OF TRUTH.
# Every stat rendered on the site MUST come from here via {{< fact key="..." >}}.
# Do not hand-type these numbers into content. See docs/superpowers/specs/2026-07-13-website-refactor-design.md §3

teaching_years: "4"
teaching_since: "2022"

learners_total: "500+"
learners_basis: "across all programs, 2022–2026"

medals_total: "65"
medals_gold: "12"
medals_silver: "21"
medals_bronze: "32"
medals_year: "2024"

src1_enrolled: "75"
src1_continued: "50"
src1_graduated: "35+"
src1_year: "2025"
src1_hours: "40"
src1_workshops: "11"

src2_researchers: "50"
src2_year: "2026"

paper_airplanes_women: "60+"
paper_airplanes_countries: "6"

pb101_online: "270"
pb101_inperson: "75"
pb101_year: "2026"

papers_under_review: "3"
open_source_prs: "18+"
orcid: "0009-0006-4432-798X"
```

- [ ] **Step 2: Create `layouts/shortcodes/fact.html`**

Uses `hugo.Data` per repo convention (never `.Site.Data`). Fails loudly on a bad key so a typo can never silently render an empty stat.

```go-html-template
{{- $key := .Get "key" -}}
{{- $facts := index hugo.Data "facts" -}}
{{- with index $facts $key -}}
  {{- . -}}
{{- else -}}
  {{- errorf "fact shortcode: unknown key %q in data/facts.yaml (called from %s)" $key $.Page.File.Filename -}}
{{- end -}}
```

- [ ] **Step 3: Verify the shortcode resolves and errors correctly**

Temporarily append to `content/_index.md`: `TESTFACT={{< fact key="medals_total" >}}`

Run: `hugo --gc --minify 2>&1 | tail -5`
Expected: build succeeds.

Run: `grep -o 'TESTFACT=65' public/index.html`
Expected: `TESTFACT=65`

Now change the key to a bogus one (`key="nope"`) and run `hugo` again.
Expected: build FAILS with `fact shortcode: unknown key "nope"`.

Remove the temporary `TESTFACT=` line from `content/_index.md` before committing.

- [ ] **Step 4: Commit**

```bash
git add data/facts.yaml layouts/shortcodes/fact.html
git commit -m "feat: canonical facts data file and fact shortcode"
```

---

## Task 2: Correct every contradictory number

**Files:**
- Modify: `content/Testimonials/_index.md`
- Modify: `content/About/timeline/_index.md`
- Modify: `content/_index.md`
- Modify: `content/About/_index.md`
- Modify: `data/organizations.yaml`

**Interfaces:**
- Consumes: `{{< fact >}}` from Task 1.

- [ ] **Step 1: Find every stale number**

```bash
grep -rn "29 Olympic\|Olympic Medals (as Coach) | 29\|75 graduates\|+50 mentees\|200+ students\|200+ mentees\|+500 mentees" content/ data/
```
Record each hit — every one must be resolved in this task.

- [ ] **Step 2: Apply the canon**

- Medal totals → `65` (12 gold / 21 silver / 32 bronze). The "29" was only the younger-group subtotal. In `content/About/timeline/_index.md` the sentence "29 Olympic medals (12 gold, 21 silver, 32 bronze as coach)" is internally contradictory (12+21+32=65) — rewrite as: `65 Olympic medals as coach (12 gold, 21 silver, 32 bronze)`.
- `content/Testimonials/_index.md` stats row `Olympic Medals (as Coach) | 29` → `65`.
- SRC graduates: any "75 graduates" → `75 enrolled, 35+ graduated`.
- `data/organizations.yaml`: Boundless SRC "+50 mentees" → `75 enrolled, 35+ graduated (2025)`.
- Learners: standardize to `500+ learners across all programs (2022–2026)` — **always with the basis**, never a bare "500+".

Where a stat sits in prose, use the shortcode: e.g. `{{< fact key="medals_total" >}} Olympic medals as coach`.

- [ ] **Step 3: Verify no stale number survives**

```bash
hugo --gc --minify 2>&1 | grep -iE "^ERROR|^WARN" | grep -v "github shortcode" ; echo "build-warnings-above"
grep -rniE "29 olympic|olympic medals \(as coach\) \| 29|75 graduates|\+50 mentees|\+500 mentees" public/ && echo "FAIL: stale number still rendered" || echo "PASS: no stale numbers in output"
```
Expected: `PASS: no stale numbers in output`

- [ ] **Step 4: Commit**

```bash
git add content/ data/organizations.yaml
git commit -m "fix: reconcile all site statistics to canonical facts"
```

---

## Task 3: Delete dead and duplicate files

**Files:**
- Delete: `arbic-header.html`
- Delete: `layouts/shortcodes/org-gallery-debug.html`
- Delete: `content/About/porofolio.md`
- Delete: `content/About/past_main_page.md`
- Delete: `content/About/homepage-archive/index.md` (and the now-empty `homepage-archive/` dir)

- [ ] **Step 1: Confirm nothing links to them**

```bash
grep -rn "porofolio\|past_main_page\|homepage-archive\|arbic-header\|org-gallery-debug" content/ config/ layouts/ data/
```
Expected: the only hit is the "Homepage Archive (v1)" link inside `content/About/_index.md`. Remove that link line as part of this task. If any other hit appears, resolve it before deleting.

- [ ] **Step 2: Delete**

```bash
git rm arbic-header.html layouts/shortcodes/org-gallery-debug.html \
       content/About/porofolio.md content/About/past_main_page.md \
       content/About/homepage-archive/index.md
```

- [ ] **Step 3: Remove the dangling archive link from About**

Edit `content/About/_index.md` — delete the line referencing "Homepage Archive (v1)".

- [ ] **Step 4: Verify**

```bash
hugo --gc --minify 2>&1 | tail -3
grep -rn "homepage-archive\|porofolio" public/ && echo "FAIL: dead link still rendered" || echo "PASS"
```
Expected: build succeeds, `PASS`. Page count drops from 140 to ~137.

- [ ] **Step 5: Commit**

```bash
git commit -am "chore: remove dead and duplicate About/legacy files"
```

---

## Task 4: Fix the broken GitHub username (LIVE BUG)

**Files:**
- Modify: `content/Projects/ai-cup-2026-bird-classification/_index.md`
- Modify: `content/Projects/kaggle-playground-series-s6e4/_index.md`
- Modify: `content/Projects/portfolio/_index.md`
- Modify: `config/_default/params.toml:81`

**Background:** the account was renamed `molhamfetnah` → `mulhamfetna`. The old username now 404s. Verified:
`api.github.com/users/molhamfetnah` → **404**; `api.github.com/users/mulhamfetna` → **200**.
Consequence: `{{< github >}}` and `{{< mdimporter >}}` fail at build time, so those two Projects pages render **without their README content in production**, and `editURL` points at a dead account.

- [ ] **Step 1: Replace every occurrence**

```bash
grep -rln "molhamfetnah" content/ config/ | xargs sed -i 's/molhamfetnah/mulhamfetna/g'
grep -rn "molhamfetnah" content/ config/ && echo "FAIL: occurrences remain" || echo "PASS: none remain"
```

- [ ] **Step 2: Verify the remote fetches now succeed**

```bash
hugo --gc --minify 2>&1 | grep -c "github shortcode.*failed to fetch"
```
Expected: `0` (baseline was 8). This is the proof the bug is fixed.

- [ ] **Step 3: Confirm README content actually renders**

```bash
grep -c "." public/projects/ai-cup-2026-bird-classification/index.html
```
Expected: substantially more content than before — the imported README body is now present.

- [ ] **Step 4: Commit**

```bash
git commit -am "fix: correct GitHub username molhamfetnah -> mulhamfetna (repos 404'd, README imports failed)"
```

---

## Task 5: Move internal dev docs out of public content; kill deprecated API

**Files:**
- Move: `content/Tutorials-Guides/Hugo-BlowFish/` → `docs/hugo-blowfish/`
- Modify: `layouts/shortcodes/org-gallery.html`
- Modify: `content/Tutorials-Guides/_index.md`

- [ ] **Step 1: Move the dev docs out of `content/`**

These 11 files (up to 1,445 lines: theme guides, changelogs, component docs) are internal notes currently servable on the public site.

```bash
mkdir -p docs/hugo-blowfish
git mv content/Tutorials-Guides/Hugo-BlowFish/* docs/hugo-blowfish/
rmdir content/Tutorials-Guides/Hugo-BlowFish
```

- [ ] **Step 2: Fix `.Site.Data` → `hugo.Data`**

In `layouts/shortcodes/org-gallery.html`, replace `site.Data.organizations` / `.Site.Data.organizations` with `index hugo.Data "organizations"`. This is a repo convention and removes a build deprecation warning.

- [ ] **Step 3: Give `content/Tutorials-Guides/_index.md` real front matter**

It currently has **none** (no `title:`), relying on a filename fallback. Add TOML front matter with `title`, `description`.

- [ ] **Step 4: Verify**

```bash
hugo --gc --minify 2>&1 | grep -i "deprecat" && echo "FAIL: deprecation remains" || echo "PASS: no deprecation warnings"
test -d public/tutorials-guides/hugo-blowfish && echo "FAIL: dev docs still public" || echo "PASS: dev docs no longer served"
```
Expected: both `PASS`.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "chore: move internal dev docs out of public content; migrate to hugo.Data"
```

---

# PHASE 1 — IA + Home + About

## Task 6: Six-lane navigation

**Files:**
- Modify: `config/_default/menus.en.toml`

**Interfaces:**
- Produces: routes `/about/`, `/research/`, `/learn/`, `/ventures/`, `/work-with-me/`, `/blog/`. Tasks 7–9 create the three new hub pages these point at; Task 15 creates `/blog/`.

- [ ] **Step 1: Replace the 12-item main menu with 6 lanes + submenus**

Blowfish supports `parent` for dropdowns. Replace the entire `[[main]]` block set in `config/_default/menus.en.toml`:

```toml
[[main]]
  name = "About"
  pageRef = "about"
  weight = 1

[[main]]
  name = "Research"
  pageRef = "Research"
  weight = 2

  [[main]]
    name = "Publications"
    pageRef = "Research"
    parent = "Research"
    weight = 1

  [[main]]
    name = "Open Source"
    pageRef = "Open-Source"
    parent = "Research"
    weight = 2

[[main]]
  name = "Learn"
  pageRef = "learn"
  weight = 3

  [[main]]
    name = "Courses"
    pageRef = "Courses"
    parent = "Learn"
    weight = 1

  [[main]]
    name = "Workshops"
    pageRef = "Workshops-Camps"
    parent = "Learn"
    weight = 2

  [[main]]
    name = "Roadmaps"
    pageRef = "Roadmaps"
    parent = "Learn"
    weight = 3

  [[main]]
    name = "Tutorials"
    pageRef = "Tutorials-Guides"
    parent = "Learn"
    weight = 4

[[main]]
  name = "Ventures"
  pageRef = "ventures"
  weight = 4

  [[main]]
    name = "Services"
    pageRef = "Services"
    parent = "Ventures"
    weight = 1

[[main]]
  name = "Work With Me"
  pageRef = "work-with-me"
  weight = 5

  [[main]]
    name = "Projects"
    pageRef = "Projects"
    parent = "Work With Me"
    weight = 1

  [[main]]
    name = "Skills"
    pageRef = "Skills"
    parent = "Work With Me"
    weight = 2

  [[main]]
    name = "CV"
    pageRef = "work-with-me/cv"
    parent = "Work With Me"
    weight = 3

[[main]]
  name = "Blog"
  pageRef = "blog"
  weight = 6
```

Leave the `[[footer]]` block unchanged.

- [ ] **Step 2: Verify (expect failures until Tasks 7–9, 15 land)**

```bash
hugo --gc --minify 2>&1 | tail -5
```
Hugo does not hard-fail on a `pageRef` to a missing page, but the nav link will 404. This is expected until the hub pages exist. Do **not** commit a broken nav on its own — complete Tasks 7, 8, 9 and 15, then commit them together with this one.

- [ ] **Step 3: Do not commit yet**

Proceed to Task 7. Commit happens at the end of Task 9 (and Blog at Task 15).

---

## Task 7: `/learn/` hub (audience: students & learners)

**Files:**
- Create: `content/learn/_index.md`

- [ ] **Step 1: Create the hub page**

```markdown
---
title: "Learn With Me"
description: "Courses, workshops, and roadmaps in Python, AI/ML, data engineering, MLOps, and mechatronics — taught in Arabic and English by Mulham Fetna."
keywords: ["Python course Arabic", "AI course Syria", "MLOps training", "mechatronics roadmap", "data engineering bootcamp"]
showDate: false
showAuthor: false
showTableOfContents: true
---

Practical, project-based technical education — taught in Arabic and English, online and in person.

I have taught {{< fact key="learners_total" >}} learners {{< fact key="learners_basis" >}}, across university cohorts, women re-skilling into tech, and nationally selected olympiad students.

## Start here

- **[Courses](/courses/)** — Python Mastery, Data Engineering, and MLOps diplomas with graduation projects.
- **[Workshops](/workshops-camps/)** — short, intensive programmes: Personal Branding 101, Mechatronics & Robotics 101, Data & AI Careers.
- **[Roadmaps](/roadmaps/)** — free self-paced paths: a 12-level Mechatronics roadmap and a Python-to-MLOps roadmap.
- **[Tutorials](/tutorials-guides/)** — free written guides.

## What students say

Read [student outcomes and testimonials](/testimonials/).
```

- [ ] **Step 2: Verify it builds and renders**

```bash
hugo --gc --minify 2>&1 | tail -3
test -f public/learn/index.html && echo "PASS: /learn/ exists" || echo "FAIL"
grep -o "500+" public/learn/index.html | head -1
```
Expected: `PASS: /learn/ exists`, and `500+` present (proves the `fact` shortcode resolved).

---

## Task 8: `/ventures/` hub (audience: clients & sponsors)

**Files:**
- Create: `content/ventures/_index.md`

- [ ] **Step 1: Create the hub page**

```markdown
---
title: "Ventures"
description: "Neurobotics (engineering and technical education) and Boundless (academic services) — Syrian ventures founded and led by Mulham Fetna, with UNFPA and SANAD partnerships."
keywords: ["Neurobotics Syria", "Boundless academic services", "robotics education Syria", "research camp Aleppo", "UNFPA SANAD partner"]
showDate: false
showAuthor: false
showTableOfContents: true
---

I founded and lead two Syrian ventures, both built to open doors that are otherwise closed to people here.

## Neurobotics

Engineering and technical education. Paid diplomas in Python, data engineering, and MLOps for Syrian learners, an engineering team building custom mechatronics solutions, and the Arabic-language ROS developer community.

## Boundless

Academic services. The **Scientific Research Camp** — {{< fact key="src1_hours" >}} training hours across {{< fact key="src1_workshops" >}} workshops, partnered with **SANAD** and funded by **UNFPA Syria**. Its first cohort ({{< fact key="src1_year" >}}) took {{< fact key="src1_enrolled" >}} enrolled researchers through to {{< fact key="src1_graduated" >}} graduates; the second ({{< fact key="src2_year" >}}) is running now with {{< fact key="src2_researchers" >}} researchers.

Boundless also runs scholarship mentorship, olympiad training, and academic personal-branding programmes.

## Partners

{{< org-gallery type="all" >}}

## Work together

Sponsorship, partnership, or delivery — [get in touch](/work-with-me/).
```

- [ ] **Step 2: Verify**

```bash
hugo --gc --minify 2>&1 | tail -3
test -f public/ventures/index.html && echo "PASS" || echo "FAIL"
grep -c "org-card\|organization" public/ventures/index.html
```
Expected: `PASS`, and the org gallery renders (non-zero count).

---

## Task 9: `/work-with-me/` hub + CV page (audience: recruiters & collaborators)

**Files:**
- Create: `content/work-with-me/_index.md`
- Create: `content/work-with-me/cv/index.md`

- [ ] **Step 1: Create the hub**

```markdown
---
title: "Work With Me"
description: "Mulham Fetna — mechatronics and AI/ML engineer available for engineering work, technical consulting, and mentorship. Aleppo, Syria. Remote worldwide."
keywords: ["hire mechatronics engineer", "AI ML engineer Syria", "robotics consultant", "technical consulting Arabic", "ROS engineer"]
showDate: false
showAuthor: false
showTableOfContents: true
---

Mechatronics and AI/ML engineer. I build robotics, computer-vision, and data systems — and I teach them.

## What I do

- **Engineering** — robotics and ROS/ROS2, embedded systems, control, computer vision, edge AI.
- **AI/ML & data** — machine learning, MLOps, data engineering, Arabic NLP.
- **Consulting & mentorship** — technical consultation sessions and one-to-one mentorship.

## Evidence

- **[Projects](/projects/portfolio/)** — 9 engineering and AI projects.
- **[Skills](/skills/)** — full technical competency matrix.
- **[Open source](/open-source/)** — {{< fact key="open_source_prs" >}} pull requests across OpenCV, Ultralytics, and OpenDR.
- **[Research](/research/)** — {{< fact key="papers_under_review" >}} papers under peer review.
- **[Full CV](/work-with-me/cv/)** — complete experience, certifications, and affiliations.

## Get in touch

Book a [technical consultation or mentorship session](/services/), or email **contact@mulhamfetna.com**.
```

- [ ] **Step 2: Create the CV page**

Move the exhaustive experience / affiliations / certifications lists **out of** `content/About/_index.md` and into this page (they are what recruiters want and what makes About unreadable). Preserve every entry; correct all numbers to canon.

```markdown
---
title: "Curriculum Vitae"
description: "Full CV of Mulham Fetna — mechatronics engineer, AI/ML practitioner, and technical trainer. Experience, research, certifications, and affiliations."
keywords: ["Mulham Fetna CV", "mechatronics engineer resume", "AI engineer Syria CV"]
showDate: false
showTableOfContents: true
---

(Full experience, affiliations, volunteering, and certifications — moved verbatim from About,
with all statistics corrected to canon. See spec §6.)
```

Populate it with the content lifted from About's `# Professional Experience`, `# Affiliations`, `# Volunteering`, and `# Licenses, Certifications, Honors & Awards` sections.

- [ ] **Step 3: Verify all three hubs + nav**

```bash
hugo --gc --minify 2>&1 | tail -3
for p in learn ventures work-with-me work-with-me/cv; do
  test -f "public/$p/index.html" && echo "PASS: /$p/" || echo "FAIL: /$p/"
done
```
Expected: four `PASS` lines.

- [ ] **Step 4: Commit Tasks 6–9 together**

```bash
git add config/_default/menus.en.toml content/learn content/ventures content/work-with-me content/About
git commit -m "feat: six-lane audience-routed navigation with Learn/Ventures/Work-With-Me hubs"
```

---

## Task 10: Rewrite the homepage as a router

**Files:**
- Modify: `content/_index.md` (full rewrite)
- Delete: `layouts/index.html`

**Background:** `content/_index.md` currently contains **two homepages concatenated** — an English "Technical Portfolio" block plus a separate Arabic "Router" homepage with its own CTAs. Its front matter declares `layout: "page"`, which is dead (the site actually renders Blowfish's `background` layout from `params.toml`). `layouts/index.html` re-implements Blowfish's own dispatch and drops the theme's author/sharing partials.

- [ ] **Step 1: Delete the redundant layout override**

```bash
git rm layouts/index.html
hugo --gc --minify 2>&1 | tail -3
test -f public/index.html && echo "PASS: homepage still renders from theme" || echo "FAIL"
```
Expected: `PASS` — Blowfish's own `layouts/index.html` takes over and picks `home/background.html` from `params.toml`.

- [ ] **Step 2: Rewrite `content/_index.md`**

Remove the dead `layout: "page"`. Remove the Arabic router block entirely (Arabic becomes a real language in Phase 4, not an inline duplicate).

```markdown
---
title: "Mulham Fetna"
description: "Mechatronics and AI/ML engineer, researcher, and technical trainer based in Aleppo, Syria. Robotics, machine learning, Arabic NLP, and research capacity building."
keywords: ["Mulham Fetna", "mechatronics engineer Syria", "AI ML engineer Aleppo", "robotics researcher", "Arabic NLP", "technical trainer Syria"]
showDate: false
showAuthor: false
showAuthorBottom: false
showDateUpdated: false
showEdit: false
showTableOfContents: false
---

Mechatronics and AI/ML engineer, researcher, and technical trainer — based in Aleppo, Syria.

I build robotics and machine-learning systems, publish research, and train the next generation of Syrian engineers and researchers.

| | |
|---|---|
| **{{< fact key="teaching_years" >}} years** teaching | **{{< fact key="learners_total" >}}** learners taught |
| **{{< fact key="papers_under_review" >}}** papers under peer review | **{{< fact key="open_source_prs" >}}** open-source contributions |

## Where would you like to go?

**Hiring or collaborating?** → **[Work With Me](/work-with-me/)** — projects, skills, CV, and consulting.

**Reviewing my research?** → **[Research](/research/)** — publications, ORCID, and open-source work.

**Looking to learn?** → **[Learn](/learn/)** — courses, workshops, and free roadmaps.

**Partner or sponsor?** → **[Ventures](/ventures/)** — Neurobotics and Boundless.

## Featured

- **[Research](/research/)** — {{< fact key="papers_under_review" >}} robotics papers under peer review (distributed MPC, path planning, inverse kinematics).
- **[Scientific Research Camp](/ventures/)** — {{< fact key="src1_hours" >}} training hours with SANAD and UNFPA Syria.
- **[Open Source](/open-source/)** — contributions to OpenCV, Ultralytics, and OpenDR.

---

Get in touch: **contact@mulhamfetna.com**
```

- [ ] **Step 3: Verify**

```bash
hugo --gc --minify 2>&1 | tail -3
grep -q "Router\|الصفحة الرئيسية" public/index.html && echo "FAIL: Arabic router block survives" || echo "PASS: single coherent homepage"
grep -q "work-with-me" public/index.html && echo "PASS: router links present" || echo "FAIL"
```
Expected: both `PASS`.

- [ ] **Step 4: Visual check in both appearances**

Run `hugo server -D`, open `http://localhost:1313/`, and confirm in **light and dark** mode (site uses `autoSwitchAppearance`): identity line reads clearly, the four audience routes are obvious, and no stray Arabic block remains.

- [ ] **Step 5: Commit**

```bash
git add content/_index.md
git commit -m "feat: rewrite homepage as audience router; remove redundant layouts/index.html"
```

---

## Task 11: Slim the About page

**Files:**
- Modify: `content/About/_index.md`

**Background:** currently 345 lines / 1,624 words of flat CV — 8 top-level sections, ~28 sub-headings — with no `description`, `summary`, or `keywords`, despite being the identity page.

- [ ] **Step 1: Add SEO front matter**

```toml
title = "About"
description = "Mulham Fetna — mechatronics engineer, AI/ML practitioner, and technical trainer in Aleppo, Syria. Founder of Neurobotics and Boundless."
keywords = ["Mulham Fetna", "about Mulham Fetna", "mechatronics engineer Aleppo", "Neurobotics founder", "Boundless founder"]
```

- [ ] **Step 2: Cut to the story (target 600–700 words)**

Keep: the narrative/mission, a condensed credibility summary (stats via `{{< fact >}}`), the mermaid org chart, education, and outbound links to the four lanes.
Remove (now on `/work-with-me/cv/`): the exhaustive `# Professional Experience`, `# Affiliations`, `# Volunteering`, and `# Licenses, Certifications, Honors & Awards` sections. Replace them with a single line: `Full experience, affiliations, and certifications: **[see my CV](/work-with-me/cv/)**.`
Also remove the commented-out dead `# Publications` heading.

- [ ] **Step 3: Verify nothing was lost**

```bash
hugo --gc --minify 2>&1 | tail -3
wc -w content/About/_index.md   # expect ~600-700, was 1624
grep -c "Aspire Leaders\|McKinsey\|CS50" public/work-with-me/cv/index.html
```
Expected: About is materially shorter, and every certification still appears — on the CV page.

- [ ] **Step 4: Commit**

```bash
git commit -am "refactor: slim About to narrative; move CV detail to /work-with-me/cv/"
```

---

# PHASE 2 — SEO / GEO foundation

## Task 12: `Person` schema via `extend-head.html` (highest-leverage GEO item)

**Files:**
- Create: `layouts/partials/extend-head.html`

**Background:** Blowfish emits only `WebSite` + `Article` schema. There is **no entity for Mulham himself**. `Person` + `sameAs` is how Google and AI assistants resolve "who is this person" and decide to cite him. `extend-head.html` is Blowfish's sanctioned extension hook — no theme file is overridden.

**Interfaces:**
- Consumes: `params.author.links` already in `config/_default/languages.en.toml` (Instagram, Facebook, Telegram, X, Bluesky, YouTube, GitHub, GitLab, Google Scholar, ResearchGate, ORCID, email). Read from config — do not hardcode.

- [ ] **Step 1: Create the partial**

Emit `Person` JSON-LD on the homepage and About page only (a `Person` entity repeated on every page is noise; these two are the identity pages).

```go-html-template
{{- if or .IsHome (eq .RelPermalink "/about/") -}}
{{- $author := site.Params.Author -}}
{{- $sameAs := slice -}}
{{- range $author.links -}}
  {{- range $k, $v := . -}}
    {{- if and $v (ne $k "email") (hasPrefix $v "http") -}}
      {{- $sameAs = $sameAs | append $v -}}
    {{- end -}}
  {{- end -}}
{{- end -}}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Person",
  "name": "Mulham Fetna",
  "url": "{{ site.BaseURL }}",
  "image": "{{ site.BaseURL }}img/mulham_professional_photo.jpg",
  "email": "contact@mulhamfetna.com",
  "jobTitle": "Mechatronics & AI/ML Engineer, Researcher, Technical Trainer",
  "description": "Mechatronics and AI/ML engineer, researcher, and technical trainer based in Aleppo, Syria.",
  "address": { "@type": "PostalAddress", "addressLocality": "Aleppo", "addressCountry": "SY" },
  "alumniOf": { "@type": "CollegeOrUniversity", "name": "University of Aleppo" },
  "worksFor": [
    { "@type": "Organization", "name": "Neurobotics" },
    { "@type": "Organization", "name": "Boundless" }
  ],
  "knowsAbout": ["Mechatronics","Robotics","ROS","Machine Learning","Computer Vision","MLOps","Data Engineering","Arabic NLP","Research Methodology"],
  "identifier": { "@type": "PropertyValue", "propertyID": "ORCID", "value": "0009-0006-4432-798X" },
  "sameAs": [
    {{- range $i, $u := $sameAs -}}{{ if $i }},{{ end }}"{{ $u }}"{{- end -}}
  ]
}
</script>
{{- end -}}
```

- [ ] **Step 2: Verify the schema is emitted and valid**

```bash
hugo --gc --minify 2>&1 | tail -3
grep -c '"@type": *"Person"' public/index.html public/about/index.html
python3 - <<'PY'
import json,re,sys
for f in ("public/index.html","public/about/index.html"):
    html=open(f,encoding="utf-8").read()
    blocks=re.findall(r'<script type="application/ld\+json">(.*?)</script>', html, re.S)
    person=[json.loads(b) for b in blocks if '"Person"' in b]
    assert person, f"no Person schema in {f}"
    p=person[0]
    assert p["name"]=="Mulham Fetna"
    assert len(p["sameAs"])>=5, f"too few sameAs links: {p['sameAs']}"
    print(f, "OK — sameAs:", len(p["sameAs"]), "links")
PY
```
Expected: valid JSON, `Person`, and ≥5 `sameAs` URLs. Invalid JSON here would silently kill the schema — this check is the whole point.

- [ ] **Step 3: Confirm it is NOT on other pages**

```bash
grep -l '"@type": *"Person"' public/research/index.html public/learn/index.html 2>/dev/null && echo "FAIL: Person leaked onto lane pages" || echo "PASS: scoped to home + about"
```

- [ ] **Step 4: Commit**

```bash
git add layouts/partials/extend-head.html
git commit -m "feat: Person JSON-LD with sameAs for entity recognition (GEO)"
```

---

## Task 13: `faq` / `faqitem` shortcodes → `FAQPage` schema

**Files:**
- Create: `layouts/shortcodes/faq.html`
- Create: `layouts/shortcodes/faqitem.html`

**Background:** Blowfish ships no FAQ shortcode and no FAQ schema. This renders an accessible accordion **and** emits `FAQPage` JSON-LD, which is what makes an answer quotable by an AI assistant.

**Interfaces:**
- Produces:
  ```
  {{< faq >}}
    {{< faqitem question="Who is Mulham Fetna?" >}}Answer prose.{{< /faqitem >}}
  {{< /faq >}}
  ```
  Task 17 uses this on every lane page.

- [ ] **Step 1: Create `layouts/shortcodes/faq.html`**

Collects children into a scratch slice, renders `<details>` elements (native, accessible, no JS), then emits one `FAQPage` block.

```go-html-template
{{- .Page.Store.Set "faqitems" slice -}}
{{- $content := .Inner -}}
{{- $rendered := $content | markdownify -}}
{{- $items := .Page.Store.Get "faqitems" -}}
<div class="faq my-6">
{{- range $items }}
  <details class="border-b border-neutral-200 dark:border-neutral-700 py-3">
    <summary class="cursor-pointer font-semibold">{{ .question }}</summary>
    <div class="mt-2 prose dark:prose-invert max-w-none">{{ .answer | safeHTML }}</div>
  </details>
{{- end }}
</div>
{{- if $items }}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {{- range $i, $it := $items -}}{{ if $i }},{{ end }}
    {
      "@type": "Question",
      "name": {{ $it.question | jsonify }},
      "acceptedAnswer": { "@type": "Answer", "text": {{ $it.answer | plainify | jsonify }} }
    }
    {{- end }}
  ]
}
</script>
{{- end -}}
```

- [ ] **Step 2: Create `layouts/shortcodes/faqitem.html`**

```go-html-template
{{- $q := .Get "question" -}}
{{- if not $q -}}{{ errorf "faqitem: missing required 'question' parameter in %s" .Page.File.Filename }}{{- end -}}
{{- $a := .Inner | markdownify -}}
{{- $items := .Page.Store.Get "faqitems" | default slice -}}
{{- $items = $items | append (dict "question" $q "answer" $a) -}}
{{- .Page.Store.Set "faqitems" $items -}}
```

- [ ] **Step 3: Smoke-test on one page**

Add temporarily to `content/learn/_index.md`:

```markdown
{{< faq >}}
{{< faqitem question="Does Mulham Fetna teach in Arabic?" >}}Yes. Courses and workshops are delivered in Arabic and English, online and in person.{{< /faqitem >}}
{{< faqitem question="Who are the courses for?" >}}Beginners through job-ready learners, including university students and career switchers.{{< /faqitem >}}
{{< /faq >}}
```

```bash
hugo --gc --minify 2>&1 | tail -3
python3 - <<'PY'
import json,re
html=open("public/learn/index.html",encoding="utf-8").read()
blocks=re.findall(r'<script type="application/ld\+json">(.*?)</script>', html, re.S)
faq=[json.loads(b) for b in blocks if '"FAQPage"' in b]
assert faq, "no FAQPage schema emitted"
qs=faq[0]["mainEntity"]
assert len(qs)==2, f"expected 2 questions, got {len(qs)}"
assert qs[0]["acceptedAnswer"]["text"].strip(), "empty answer text"
print("OK — FAQPage with", len(qs), "questions; answers non-empty")
PY
grep -c "<details" public/learn/index.html
```
Expected: valid `FAQPage` with 2 questions and non-empty answers; `<details>` elements render.

- [ ] **Step 4: Commit**

```bash
git add layouts/shortcodes/faq.html layouts/shortcodes/faqitem.html content/learn/_index.md
git commit -m "feat: faq/faqitem shortcodes emitting FAQPage JSON-LD (AEO)"
```

---

## Task 14: `description` + `keywords` on every page

**Files:**
- Modify: every `content/**/*.md` lacking a `description`

**Background:** only **17 of 51** content files have an explicit `description`. Missing ones fall back to an auto-summary or a long generic site bio.

- [ ] **Step 1: List every page still missing a description**

```bash
for f in $(find content -name "*.md"); do
  head -20 "$f" | grep -qi "^description" || echo "MISSING: $f"
done
```

- [ ] **Step 2: Write a real description for each**

≤160 characters, stating what the page **is** and **who it is for**. No boilerplate, no duplication between pages — a duplicated description is worth roughly nothing to a search engine. Add `keywords` targeting the queries that page should win.

Pattern to follow (these are the already-written lane descriptions — match this specificity):

```toml
# content/Courses/_index.md
description = "Paid technical diplomas in Python, data engineering, and MLOps for Syrian learners — cohort-based, project-assessed, taught in Arabic and English."
keywords = ["Python course Syria", "MLOps diploma Arabic", "data engineering course", "Neurobotics Academy"]

# content/Skills/_index.md
description = "Technical competency matrix for Mulham Fetna — robotics, ROS, embedded systems, machine learning, computer vision, MLOps, and Arabic NLP."
keywords = ["Mulham Fetna skills", "ROS engineer skills", "machine learning competencies"]

# content/Roadmaps/_index.md
description = "Free self-paced learning roadmaps: a 12-level mechatronics path and a Python-to-MLOps path, in Arabic and English."
keywords = ["mechatronics roadmap", "Python MLOps roadmap", "free engineering roadmap Arabic"]
```

- [ ] **Step 3: Verify 100% coverage**

```bash
total=$(find content -name "*.md" | wc -l)
have=$(for f in $(find content -name "*.md"); do head -20 "$f" | grep -qi "^description" && echo x; done | wc -l)
echo "$have / $total pages have descriptions"
test "$have" -eq "$total" && echo "PASS" || echo "FAIL: $((total-have)) still missing"
```
Expected: `PASS`.

- [ ] **Step 4: Commit**

```bash
git add content/
git commit -m "seo: add description and keywords to every content page"
```

---

## Task 15: Real `/blog/` section

**Files:**
- Create: `content/Blog/_index.md`
- Modify/Delete: `content/Social-Media-Posts/`

**Background:** the "Blog" nav item currently points at `content/Social-Media-Posts/`, which holds **one 0-byte file** (`pmp-course.md`) and **no `_index.md`**. A visitor clicking "Blog" hits an empty section. The author chose to keep Blog in the nav — so it must have a real index now, and real posts in Phase 5.

- [ ] **Step 1: Create the section index with an alias from the old URL**

```markdown
---
title: "Blog"
description: "Writing by Mulham Fetna on robotics, AI and machine learning, data engineering, research methodology, and building technical education in Syria."
keywords: ["Mulham Fetna blog", "robotics blog Arabic", "AI blog Syria", "research methodology"]
aliases: ["/social-media-posts/"]
showDate: false
showAuthor: false
---

Writing on robotics, AI and machine learning, data engineering, research methodology, and building technical education in Syria.
```

- [ ] **Step 2: Remove the empty stub**

```bash
git rm content/Social-Media-Posts/pmp-course.md
rmdir content/Social-Media-Posts 2>/dev/null || true
```

- [ ] **Step 3: Verify the nav item and the redirect both work**

```bash
hugo --gc --minify 2>&1 | tail -3
test -f public/blog/index.html && echo "PASS: /blog/ exists" || echo "FAIL"
grep -q "blog" public/social-media-posts/index.html && echo "PASS: alias redirects" || echo "FAIL: no alias"
```
Expected: both `PASS`.

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "feat: real /blog/ section with alias from /social-media-posts/"
```

---

# PHASE 3 — Lane build-out

## Task 16: Fill the three empty Workshops pages

**Files:**
- Modify: `content/Workshops-Camps/Personal-Branding-101/_index.md`
- Modify: `content/Workshops-Camps/Data-AI-Careers-Behind-it/_index.md`
- Modify: `content/Workshops-Camps/Mechatronics-Robotics-101/_index.md`

**Background:** all three are front-matter-only shells linked from a real landing page. The author chose to **fill, not cut** them. Real, verified content exists for each.

- [ ] **Step 1: Personal Branding 101**

9-hour workshop (3 sessions × 3 hours): Session 1 an interactive dialogue session, Sessions 2–3 directed training. Covers personal brand, CV/résumé building, LinkedIn profile optimization. Delivered {{< fact key="pb101_year" >}} to two cohorts: **{{< fact key="pb101_online" >}} online** (Boundless) and **{{< fact key="pb101_inperson" >}} in person** (Homs University, IT Department). Add `description`, `keywords`, outline, audience, outcomes.

- [ ] **Step 2: Data & AI Careers — "From Data to Decisions"**

4-hour on-site workshop: data and AI literacy for decision-makers. Audience: senior professionals — founders, consultants, project managers, academics. Delivered 2026 for **SyrProNet**. Add `description`, `keywords`, outline, audience, outcomes.

- [ ] **Step 3: Mechatronics & Robotics 101**

Built from the 12-level Mechatronics Roadmap (analog electronics → advanced robotics/RL). Audience: engineering students and beginners. Link to `/roadmaps/mechatronics-roadmap/`. Add `description`, `keywords`, outline.

- [ ] **Step 4: Verify no stubs remain**

```bash
hugo --gc --minify 2>&1 | tail -3
for f in content/Workshops-Camps/*/_index.md; do
  words=$(sed '1,/^---$/d;1,/^---$/d' "$f" | wc -w)
  echo "$words words — $f"
  test "$words" -gt 120 || echo "FAIL: still a stub -> $f"
done
```
Expected: every workshop page well over 120 words of body.

- [ ] **Step 5: Commit**

```bash
git add content/Workshops-Camps/
git commit -m "content: fill the three empty workshop pages with real programme detail"
```

---

## Task 17: FAQ blocks on every lane page

**Files:**
- Modify: `content/_index.md`, `content/About/_index.md`, `content/Research/_index.md`, `content/learn/_index.md`, `content/ventures/_index.md`, `content/work-with-me/_index.md`

**Interfaces:**
- Consumes: `{{< faq >}}` / `{{< faqitem >}}` from Task 13.

**Rule:** 4–6 questions per page, phrased as the questions that audience actually asks an AI assistant. Each answer must be **self-contained prose** — an assistant quoting one answer in isolation must still be correct and attributable. No answer may contradict `data/facts.yaml`.

- [ ] **Step 1: Home** — Who is Mulham Fetna? What does he do? Where is he based? How do I contact him?

- [ ] **Step 2: Research** — What does Mulham Fetna research? Has he published? What is his ORCID? Who does he collaborate with? (Answer honestly: {{< fact key="papers_under_review" >}} papers **under peer review**, not published.)

- [ ] **Step 3: Learn** — What courses does he teach? Does he teach in Arabic? Who are the courses for? How do I enroll? (Replaces the Task 13 smoke-test block.)

- [ ] **Step 4: Ventures** — What is Neurobotics? What is Boundless? Who has he partnered with? How do I sponsor or partner?

- [ ] **Step 5: Work With Me** — Is he available for hire? What are his skills? Does he consult? How do I book a session?

- [ ] **Step 6: About** — a short 3-question block: background, education, languages.

- [ ] **Step 7: Verify every lane emits valid FAQPage schema**

```bash
hugo --gc --minify 2>&1 | tail -3
python3 - <<'PY'
import json,re
pages=["public/index.html","public/about/index.html","public/research/index.html",
       "public/learn/index.html","public/ventures/index.html","public/work-with-me/index.html"]
for f in pages:
    html=open(f,encoding="utf-8").read()
    blocks=re.findall(r'<script type="application/ld\+json">(.*?)</script>', html, re.S)
    faq=[json.loads(b) for b in blocks if '"FAQPage"' in b]
    assert faq, f"NO FAQPage on {f}"
    n=len(faq[0]["mainEntity"])
    assert 3<=n<=6, f"{f}: {n} questions (want 3-6)"
    for q in faq[0]["mainEntity"]:
        assert q["acceptedAnswer"]["text"].strip(), f"{f}: empty answer for {q['name']}"
    print(f"OK {f} — {n} questions")
PY
```
Expected: every page reports OK.

- [ ] **Step 8: Commit**

```bash
git add content/
git commit -m "feat: FAQ blocks with FAQPage schema on all six lane pages (AEO)"
```

---

## Task 18: Reconnect Testimonials; final orphan sweep

**Files:**
- Modify: `content/Testimonials/_index.md`
- Modify: `content/Services/academic-mentroship/` → `academic-mentorship/`
- Modify or delete: `content/Projects/Smart-robot.md`

**Background:** `Testimonials` is complete and strong but has **zero inbound links and no menu entry** — reachable only by direct URL. Tasks 7 and 9 already link it from `/learn/` and `/work-with-me/`; this task confirms and finishes the sweep.

- [ ] **Step 1: Confirm Testimonials now has inbound links**

```bash
grep -rn "/testimonials/" content/ | grep -v "^content/Testimonials"
```
Expected: at least the `/learn/` and `/work-with-me/` links from earlier tasks. Add a link from `/courses/` too.

- [ ] **Step 2: Fix the folder typo, with an alias**

```bash
git mv content/Services/academic-mentroship content/Services/academic-mentorship
```
Add to that page's front matter: `aliases = ["/services/academic-mentroship/"]` so the old URL still resolves.

- [ ] **Step 3: Resolve the empty project file**

`content/Projects/Smart-robot.md` is empty. Either write it properly or delete it — do not ship an empty page.

```bash
git rm content/Projects/Smart-robot.md   # if not being written
```

- [ ] **Step 4: Full-site orphan and emptiness sweep**

```bash
hugo --gc --minify 2>&1 | tail -3
# any rendered page with a suspiciously empty body?
for f in $(find public -name "index.html"); do
  words=$(sed 's/<[^>]*>//g' "$f" | wc -w)
  test "$words" -lt 60 && echo "THIN ($words words): $f"
done
```
Expected: no thin pages other than legitimate list/taxonomy pages.

- [ ] **Step 5: Commit**

```bash
git add -A
git commit -m "fix: reconnect Testimonials, fix academic-mentorship typo, remove empty project"
```

---

## Task 19: Final verification of the whole refactor

- [ ] **Step 1: Clean build from scratch**

```bash
rm -rf public resources
hugo --gc --minify 2>&1 | tee /tmp/build.log | tail -12
grep -icE "^ERROR" /tmp/build.log && echo "FAIL: errors present" || echo "PASS: no errors"
grep -c "failed to fetch" /tmp/build.log   # expect 0 (was 8 at baseline)
grep -ci "deprecat" /tmp/build.log         # expect 0
```

- [ ] **Step 2: Assert the acceptance criteria from spec §12**

```bash
# nav is 6 items
grep -c "^\[\[main\]\]" config/_default/menus.en.toml   # top-level entries

# no stale statistics anywhere in the output
grep -rniE "29 olympic|75 graduates|\+50 mentees|\+500 mentees" public/ && echo "FAIL" || echo "PASS: canon holds"

# every page has a description meta tag
missing=0
for f in $(find public -name "index.html"); do
  grep -q '<meta name="description"' "$f" || { echo "NO DESC: $f"; missing=1; }
done
test "$missing" -eq 0 && echo "PASS: all pages have descriptions"

# schema present
grep -l '"@type": *"Person"' public/index.html public/about/index.html
grep -rl '"@type": *"FAQPage"' public/ | wc -l   # expect >= 6

# old URLs still resolve
for u in social-media-posts services/academic-mentroship; do
  test -f "public/$u/index.html" && echo "PASS alias: /$u/" || echo "FAIL alias: /$u/"
done
```

- [ ] **Step 3: Visual review in light and dark**

`hugo server -D` and walk: `/`, `/about/`, `/research/`, `/learn/`, `/ventures/`, `/work-with-me/`, `/work-with-me/cv/`, `/blog/`. Confirm in **both appearances** (site uses `autoSwitchAppearance`): nav shows 6 items with working dropdowns, homepage routes clearly, no empty pages, FAQ accordions open and close.

- [ ] **Step 4: Validate schema externally**

Paste the rendered `/` and `/learn/` HTML into Google's Rich Results Test and schema.org validator. Confirm `Person` and `FAQPage` parse with no errors.

- [ ] **Step 5: Regenerate `KNOWLEDGE_BASE.md`** (spec §9)

It currently documents a **stale** menu ("Hire Me", "Posts") and a content topology that no longer exists — and after this refactor it would be wrong in even more ways. Rewrite it to describe the site as it now stands: the six lanes, `data/facts.yaml` as the canonical-facts contract, the custom shortcode inventory (`fact`, `faq`/`faqitem`, `org-gallery`, `mdimporter`, `cal-embed`, `page-section`), the `extend-head.html` schema hook, and the `hugo --gc --minify` verification loop.

```bash
git add KNOWLEDGE_BASE.md
git commit -m "docs: regenerate KNOWLEDGE_BASE to match refactored site"
```

- [ ] **Step 6: Merge**

```bash
git checkout main
git merge --no-ff refactor/site-restructure -m "feat: audience-routed site restructure with canonical facts and GEO schema"
```
Cloudflare Pages auto-deploys on push to `main`. **Do not push until the author approves the merge.**

---

# Deferred (designed in spec, not in this plan)

- **Phase 4 — Bilingual EN/AR** (spec §10): real Hugo multilingual — `languages.ar.toml`, `menus.ar.toml`, `i18n/`, RTL, `.ar.md` translations. Needs its own plan.
- **Phase 5 — Blog import** (spec §11): Facebook / Instagram / Telegram exports → normalized posts. Blocked on the author providing the data. Needs its own plan.
