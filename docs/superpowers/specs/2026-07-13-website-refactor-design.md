# Website Refactor — Design Spec

**Site:** mulhamfetna.com (Hugo + Blowfish v2.100.0, Cloudflare Pages)
**Repo:** github.com/mulhamfetna/mulham-fetna-blog · branch `main`
**Author:** Mulham Fetna
**Date:** 2026-07-13
**Approach:** B — staged restructure (approved). Full design first, then phased implementation.

---

## 1. Goals

1. **Professional when shared cold.** The link must function as a LinkedIn replacement: a stranger
   understands who Mulham is and what to do next within ten seconds.
2. **Audience-routed.** Each page is optimized for exactly one audience group (approved principle).
3. **SEO + GEO/AEO.** Rank in search *and* get cited by AI assistants as a source, via entity
   schema and per-page Q&A.
4. **Truthful and internally consistent.** One canonical set of facts across the site — and
   consistent with submitted applications (SyrIA/DAAD, NASA Space Apps).

## 2. Constraints

- Stay on **Hugo + Blowfish**. No theme fork, no JS frameworks, no CSS frameworks.
- **Pragmatic code line** (approved): Blowfish shortcodes/params/markdown first; our own Hugo
  shortcodes/partials only where the theme genuinely lacks a capability. The custom surface must
  stay flat or shrink — every addition must justify itself against a capability the theme lacks.
- Respect existing repo conventions (from `.github/copilot-instructions.md` and `guide.md`):
  - Images: `assets/img/...` referenced as `/img/...` (never `/assets/img/...`); page-bundle images
    by bare relative filename; `static/` files as `/file.ext`.
  - New content via `archetypes/default.md` (TOML front matter, `draft = true`).
  - New templates use `hugo.Data`, not deprecated `.Site.Data`.
  - Section folder names are wired to `menus.en.toml` `pageRef`s — renames require lockstep menu
    updates **and** `aliases` for old URLs.
- No tests/lint exist. **Verification = `hugo --gc --minify` builds clean + visual review.**
- Deployment is Cloudflare Pages on push to `main`. No CI to add.

## 3. Canonical facts — `data/facts.yaml`

Single source of truth. Every stat rendered on the site reads from here. Prevents the drift that
currently has the site contradicting itself.

| Key | Canonical value | Basis |
|---|---|---|
| `teaching_years` | 4 (since 2022) | Confirmed by author |
| `learners_total` | 500+ across all programs, 2022–2026 | Confirmed by author. **State with basis, never as a bare number.** Components below must remain able to substantiate it. |
| `medals` | 65 total — 12 gold, 21 silver, 32 bronze (2024) | Younger 4/10/15 = 29; Adolescent 8/11/17 = 36; 29+36 = 65. The "29" previously used as a total was only the younger-group subtotal. |
| `src1` | 75 enrolled → 50 continued → 35+ graduated (2025) | SRC1 closing-ceremony deck. Matches submitted SyrIA application. |
| `src2` | 50 researchers (2026, ongoing) | SRC2 concept note |
| `paper_airplanes` | 60+ women across 6 countries | Org data + testimonials |
| `personal_branding_101` | 270 online + 75 in person (2026) | Two cohorts; in-person hosted by Homs University IT Dept |
| `papers` | 3 under peer review (**not** published) | Manuscript IDs on file |
| `open_source` | 18+ PRs (OpenCV, Ultralytics, OpenDR) | PR ledger |
| `orcid` | 0009-0006-4432-798X | — |

**Corrections this forces on live content:**
- Testimonials "Olympic Medals (as Coach) | 29" → **65**.
- Timeline "29 Olympic medals (12 gold, 21 silver, 32 bronze)" → self-contradictory (sums to 65) → **65**.
- Homepage "2024: 29 Olympic medals as coach" → **65**.
- About/Testimonials "75 graduates" (SRC) → **75 enrolled, 35+ graduated**.
- `data/organizations.yaml` "+50 mentees" (Boundless SRC) → align to SRC1 canon.
- Mentee figure standardized to **500+ with basis**; the orphaned "+500" pages are deleted anyway.

## 4. Information architecture

### 4.1 Navigation: 12 items → 6

Current main menu (12): Home, About, Projects, Courses, Research, Open Source, Skills, Roadmaps,
Tutorials, Workshops, Blog, Mentorship & Consultations. This forces every visitor to self-sort.

New main menu (6 lanes), each serving one audience:

| Nav item | URL | Audience | Absorbs |
|---|---|---|---|
| About | `/about/` | Everyone — the human story | About, Timeline |
| Research | `/research/` | Academic & scholarship panels | Research, Open-Source, publications, ORCID |
| Learn | `/learn/` | Students & learners | Courses, Workshops-Camps, Roadmaps, Tutorials-Guides |
| Ventures | `/ventures/` | Clients & sponsors | Neurobotics, Boundless, Services, org gallery |
| Work With Me | `/work-with-me/` | Recruiters & collaborators | Projects, Skills, CV, hire/consult CTA |
| Blog | `/blog/` | Everyone (SEO surface) | Real posts (Phase 5 import) |

Footer keeps: Tags, Categories, IP Policy, Privacy, Terms, Authenticity & Editorial Policy.

### 4.2 URL strategy — preserve equity, avoid churn

**Existing good URLs are NOT moved.** `/research/`, `/skills/`, `/open-source/`, `/projects/`,
`/courses/`, `/roadmaps/`, `/workshops-camps/`, `/services/`, `/testimonials/`, `/about/` all stay.

Lanes are implemented as **hub pages that route into existing sections**, plus a restructured menu
with Blowfish parent/child submenus. Only three genuinely new URLs are created:

- `/learn/` (new hub)
- `/ventures/` (new hub)
- `/work-with-me/` (new hub)
- `/blog/` (new section; `/social-media-posts/` gets an `aliases` redirect to it)

Any page that does move gets Hugo `aliases` front matter so old URLs 301 correctly. No custom
redirect layer needed.

### 4.3 Orphan fixes

- **Testimonials** — currently complete but has zero inbound links and no menu entry. It becomes
  shared proof: linked from `/learn/` and `/work-with-me/`, and surfaced on the homepage proof strip.
- **Blog** — nav item currently points at `content/Social-Media-Posts/`, which holds one 0-byte file
  and no `_index.md`. A real `content/Blog/_index.md` section is created in Phase 2 so the nav item
  never leads to an empty page; it is populated for real in Phase 5.
- **Workshops sub-pages** — the three front-matter-only shells are **filled**, not cut (author
  decision). Real content exists for all three:
  - `Personal-Branding-101` → 9-hour workshop; 270 online (Boundless) + 75 in person (Homs
    University IT Dept), 2026.
  - `Data-AI-Careers-Behind-it` → "From Data to Decisions", 4-hour on-site for senior professionals,
    SyrProNet, 2026.
  - `Mechatronics-Robotics-101` → built from the 12-level Mechatronics Roadmap curriculum.

## 5. Home page — a router, not a CV

**Problem today:** `content/_index.md` is two homepages concatenated — an English "Technical
Portfolio" block followed by a separate Arabic "Router" homepage with its own CTAs. Its front matter
declares `layout: "page"` while the site actually renders Blowfish's `background` layout (set in
`params.toml`); the front-matter value is dead and misleading. A redundant `layouts/index.html`
override re-implements the theme's own dispatch.

**Design:**

1. **Identity line** — name, one-sentence positioning, location. Answers "who is this?" instantly.
2. **Proof strip** — 4 stat tiles from `data/facts.yaml` (years teaching · learners · medals ·
   papers under review). Never hand-typed numbers.
3. **Four audience cards** — the router. Each card names the visitor and their next step:
   *"Hiring or collaborating?" → /work-with-me/* · *"Reviewing my research?" → /research/* ·
   *"Looking to learn?" → /learn/* · *"Partner or sponsor?" → /ventures/*
4. **Featured work** — 3 items max.
5. **Single primary CTA** — contact / book a session.
6. **FAQ block** (see §7) — the homepage Q&A targets identity queries ("Who is Mulham Fetna?").

Built with Blowfish shortcodes (`button`, `badge`, `alert`, `article`) + markdown. **Decision:
`homepage.layout` stays `background`** — it already renders `.Content` of `content/_index.md` in
full, which is all the router needs. The four audience cards are built in markdown within that
content, not as a new layout. Switching to `custom` is explicitly rejected: it would add a layout
file for no capability we lack.

**Removed:** the Arabic router block (Arabic gets a real language in Phase 4, not an inline
duplicate), the dead `layout: "page"` front matter, and `layouts/index.html`.

## 6. About page — story, not résumé dump

**Problem today:** 345 lines / 1,624 words of flat CV — 8 top-level sections, ~28 sub-headings —
with **no `description`, no `summary`, no `keywords`** despite being the identity page. The same
content exists **four times** in the folder.

**Design:**
- `/about/` keeps: the narrative (who he is, the mission), a condensed credibility summary, the
  mermaid org chart, education, and links out to the lanes. Target ~600–700 words.
- The exhaustive experience / certifications / affiliations lists move to **`/work-with-me/cv/`**,
  where recruiters actually want them.
- `/about/timeline/` stays as-is (with corrected numbers).
- **Deleted** (approved): `About/porofolio.md` (typo'd filename, no front matter, orphaned),
  `About/past_main_page.md` (orphaned, "+500 mentees" legacy), `About/homepage-archive/index.md`
  (near-duplicate of the previous).

## 7. SEO / GEO / AEO layer

Ordered by leverage.

### 7.1 `Person` schema with `sameAs` — highest value
Blowfish emits only `WebSite` + `Article` schema. There is **no entity for Mulham himself**. A
`Person` JSON-LD block with `sameAs` binds every profile into one recognized entity, which is how
Google and AI assistants resolve "who is this person" and decide to cite him.

- Implemented in **`layouts/partials/extend-head.html`** — Blowfish's sanctioned extension hook, so
  no theme file is overridden.
- Fields: `name`, `jobTitle`, `description`, `image`, `url`, `email`, `knowsAbout` (mechatronics,
  robotics, AI/ML, data science, Arabic NLP, research methodology), `alumniOf` (University of
  Aleppo), `worksFor` (Neurobotics, Boundless), and `sameAs` = ORCID, Google Scholar, ResearchGate,
  GitHub, GitLab, X, Bluesky, YouTube, Instagram, Facebook, Telegram.
- Source of truth for links: `params.author.links` already in `languages.en.toml` — read from
  config, not hardcoded.

### 7.2 `faq` shortcode → `FAQPage` schema
Blowfish ships no FAQ shortcode and no FAQ schema. We add one shortcode that does both jobs:
renders an accessible accordion **and** emits `FAQPage` JSON-LD.

- File: `layouts/shortcodes/faq.html` (+ `faqitem.html` for each Q/A pair, mirroring Blowfish's own
  `accordion`/`accordionItem` pattern for consistency).
- Usage:
  ```
  {{< faq >}}
    {{< faqitem question="Who is Mulham Fetna?" >}}Answer in prose.{{< /faqitem >}}
  {{< /faq >}}
  ```
- **One FAQ block per lane page**, 4–6 questions, written as the questions that audience actually
  asks an AI assistant. Answers must be self-contained prose (an AI quoting one answer in isolation
  must still be correct and attributable).

Example question sets:
- **Home:** Who is Mulham Fetna? What does he do? Where is he based? How do I contact him?
- **Research:** What does Mulham Fetna research? Has he published? What is his ORCID? Who does he
  collaborate with?
- **Learn:** What courses does he teach? Does he teach in Arabic? Who are his courses for? How do I
  enroll?
- **Ventures:** What is Neurobotics? What is Boundless? Who has he partnered with?
- **Work With Me:** Is he available for hire? What are his skills? Does he consult? How do I book?

### 7.3 `description` on every page
Only **17 of 51** content files have an explicit `description` — and the missing ones include the
**homepage, About, Courses, Services, Roadmaps, and Workshops**. Those currently fall back to an
auto-summary or a long generic site bio.

Policy: every page gets a hand-written `description` (≤ 160 chars, states what the page *is* and for
whom) plus `keywords`. This is mechanical but high-yield.

### 7.4 Existing SEO assets — keep
Already in place and working: `robots.txt` (generated), `sitemap.xml`, RSS, `index.json` for search,
Google Analytics (`G-L9LCQSRM78`), Google Search Console verification, Open Graph + Twitter cards
with a default social image. No change needed.

## 8. Custom code inventory

**Add (2):**
| File | Purpose |
|---|---|
| `layouts/partials/extend-head.html` | `Person` JSON-LD (§7.1) |
| `layouts/shortcodes/faq.html` + `faqitem.html` | Accordion + `FAQPage` JSON-LD (§7.2) |

**Delete (3 files + 1 bug):**
| File | Reason |
|---|---|
| `arbic-header.html` (repo root) | Dead, unreferenced, misspelled, not a valid Hugo location |
| `layouts/shortcodes/org-gallery-debug.html` | Leftover diagnostic copy |
| `layouts/index.html` | Redundant — re-implements Blowfish's own dispatch, and drops the theme's author/sharing partials |
| deprecated `.Site.Data` in `org-gallery.html` | Migrate to `hugo.Data` (repo convention + build warning) |

**Keep (4):** `org-gallery.html`, `mdimporter.html`, `cal-embed.html`, `page-section.html`.

**Net effect — a wash, not a reduction.** Custom files before: 5 shortcodes + 1 layout override +
1 dead root file = **7**. After: 6 shortcodes (4 kept + `faq` + `faqitem`) + 1 partial
(`extend-head.html`) = **7**. We remove three and add three. The custom surface does not grow, and
the site gains `Person` + `FAQPage` schema it cannot otherwise have. (`assets/css/custom.css`
is unchanged and excluded from both counts.)

**Also fix:** `params.toml` `editURL` points at the wrong GitHub username (`molhamfetnah` vs
`mulhamfetna`).

## 9. Content hygiene

- **Move internal dev docs out of `content/`** (approved): `content/Tutorials-Guides/Hugo-BlowFish/`
  (11 files, up to 1,445 lines — theme guides, changelogs, component docs) are internal notes
  currently servable on the public site. Move to `docs/` in the repo. `Tutorials-Guides/` retains
  only genuine reader-facing tutorials.
- `content/Tutorials-Guides/_index.md` has **no front matter at all** — add title/description.
- Rename the typo'd `content/Services/academic-mentroship/` → `academic-mentorship/`, with an
  `aliases` redirect.
- `content/Projects/Smart-robot.md` is empty — fill or delete.
- Regenerate or retire `KNOWLEDGE_BASE.md` — it documents a stale menu and content topology.

## 10. Phase 4 — Bilingual EN/AR (designed now, built later)

The site is **not multilingual today**: only `languages.en.toml` exists, `i18n/` is empty, and
Arabic appears as inline `{{% rtl %}}` blocks inside English pages (Services, the homepage's Arabic
router block, and the entire `Tutorials-Guides/University-Email/` sub-section, which is 100% Arabic).

Real implementation:
- `config/_default/languages.ar.toml` (`rtl = true`, Arabic title/description/author bio),
  `menus.ar.toml`, and `i18n/ar.toml` + `i18n/en.toml`.
- Translations as `.ar.md` siblings (`_index.ar.md`) — Hugo's `defaultContentLanguage` stays `en`.
- Priority order: Home → About → the four lane pages → Services → Courses. Not every page needs a
  translation on day one.
- Existing inline-Arabic content migrates into the `ar` site; inline `{{% rtl %}}` blocks are
  removed from English pages.
- **Strategic value:** Arabic queries about ROS, mechatronics, and research methodology face far
  less competition than English ones — this materially expands GEO reach, not just accessibility.

## 11. Phase 5 — Blog from social data (designed now, built later)

Blocked on the author providing Facebook / Instagram / Telegram exports.

- Target: `content/Blog/` with real posts; `/social-media-posts/` aliased to it.
- Approach: parse exports → normalize to one post schema (date, title, body, images, source
  platform, original URL) → generate page bundles via `archetypes/default.md` → images into page
  bundles per the repo's image convention.
- Dedupe cross-posted content across platforms (same post to FB + IG + Telegram = one blog entry
  listing all sources).
- Each imported post needs a `description` and tags to be worth anything for SEO.
- This is a content-import pipeline, not a design change — it gets its own spec when the data lands.

## 12. Phasing & acceptance criteria

Each phase must leave the site **live, building, and shippable**.

| Phase | Scope | Done when |
|---|---|---|
| **0 — Truth & hygiene** | Fix all contradictory numbers to canon; delete 3 duplicate About files + `arbic-header.html` + `org-gallery-debug.html`; fix `editURL`; `.Site.Data` → `hugo.Data`; move dev docs to `docs/` | `hugo --gc --minify` clean; no number on the site contradicts `data/facts.yaml`; no dead/duplicate files remain |
| **1 — IA + Home + About** | `data/facts.yaml`; 6-lane menu w/ submenus; 3 new hub pages; rewrite home as router; split About; delete `layouts/index.html`; aliases for any moved URL | Build clean; every old URL still resolves (via alias); nav ≤ 6 items; home passes the 10-second test |
| **2 — SEO/GEO foundation** | `extend-head.html` Person schema; `faq`/`faqitem` shortcodes; `description` + `keywords` on all pages; real `/blog/` section index | Build clean; schema validates (Google Rich Results / schema.org validator); 51/51 pages have `description` |
| **3 — Lane build-out** | Fill the 3 Workshops pages; FAQ blocks on all lanes; reconnect Testimonials; fill/delete `Smart-robot.md`; fix `academic-mentroship` typo | Build clean; no orphaned or empty pages remain |
| **4 — Bilingual** | Per §10 | Build clean; `/ar/` serves RTL; language switcher works |
| **5 — Blog import** | Per §11 | Own spec, once data is provided |

## 13. Verification

No tests exist in this repo. Every phase is verified by:
1. `hugo --gc --minify` — must build with no new warnings.
2. `hugo server -D` + visual review of every changed page in light **and** dark mode (the site uses
   `autoSwitchAppearance`).
3. Old URLs spot-checked against `public/` output for alias redirects.
4. Phase 2 additionally: validate `Person` and `FAQPage` JSON-LD with a schema validator.
5. Work happens on a branch off `main`; Cloudflare Pages auto-deploys on merge.

## 14. Risks

| Risk | Mitigation |
|---|---|
| Renaming sections breaks `menus.en.toml` `pageRef`s | Existing good URLs are deliberately **not** moved (§4.2); menu and content change in lockstep; aliases for anything that does move |
| "500+ learners" is a revision **upward** from the live "200+" | State with basis and date range; keep component figures in `facts.yaml` so it can be substantiated |
| Blowfish is vendored into git with a broken submodule link (~2,250 files, no `.gitmodules`) | Out of scope for this refactor, but flagged: theme upgrades will be painful until resolved. Do not fight it now. |
| Losing SEO equity on moved pages | Hugo `aliases` on every moved page; sitemap regenerates automatically |
| Scope creep across 6 phases | Each phase ships independently; no phase depends on a later one |

---

## Open items requiring author input (none blocking Phase 0–3)
- Phase 5 needs the Facebook / Instagram / Telegram data exports.
- Phase 4 needs a decision on how much content gets Arabic translation beyond the priority list.
