# Knowledge Base — mulhamfetna.com

Architecture snapshot. Last regenerated: 2026-07-13, after the audience-routed refactor.

- **Stack:** Hugo (v0.154.5+extended) + Blowfish v2.100.0
- **Deploy:** Cloudflare Pages, auto-builds on push to `main`. No CI in-repo.
- **Live:** https://mulhamfetna.com/ · **Repo:** github.com/mulhamfetna/mulham-fetna-blog
- **Build:** `hugo --gc --minify` · **Dev:** `hugo server -D`
- **No tests or lint exist.** Verification = clean build + visual review in light *and* dark
  (the site uses `autoSwitchAppearance`).

## Design docs

The refactor's spec and plan live in `docs/superpowers/`. Read the spec before changing structure:

- `docs/superpowers/specs/2026-07-13-website-refactor-design.md`
- `docs/superpowers/plans/2026-07-13-website-refactor.md`

## Information architecture — six audience lanes

Each top-level nav item serves **one** audience. This is the site's organizing principle; keep it.

| Nav | URL | Audience |
|---|---|---|
| About | `/about/` | Everyone — the story |
| Research | `/research/` | Academic & scholarship panels |
| Learn | `/learn/` | Students & learners |
| Ventures | `/ventures/` | Clients & sponsors |
| Work With Me | `/work-with-me/` | Recruiters & collaborators |
| Blog | `/blog/` | Everyone (SEO surface) |

`/learn/`, `/ventures/`, and `/work-with-me/` are **hub pages** that route into the pre-existing
sections (`/courses/`, `/projects/`, `/skills/`, …). Those sections keep their original URLs, so no
SEO equity was lost. The homepage is a **router**, not a CV.

## Canonical facts — `data/facts.yaml`

**Every statistic on the site must come from here**, via `{{< fact key="..." >}}`.

Do not hand-type numbers into content. This file exists because the site previously claimed both
29 and 65 Olympic medals, both 200+ and 500+ mentees, and "75 graduates" where 75 was the
*enrolment* figure. `fact.html` **fails the build** on an unknown key — a silent empty stat would
ship a wrong claim to a reader.

Statistics that must always carry their basis (never a bare number): `learners_total` is rendered
together with `learners_basis`.

## Custom code (everything outside `themes/`)

Blowfish is not forked. Eight custom files, each justified by a capability the theme lacks:

| File | Purpose |
|---|---|
| `layouts/shortcodes/fact.html` | Renders a canonical value from `data/facts.yaml`. Errors on unknown key. |
| `layouts/shortcodes/faq.html` + `faqitem.html` | `<details>` accordions **and** `FAQPage` JSON-LD from one source. |
| `layouts/partials/extend-head-uncached.html` | `Person` JSON-LD with `sameAs` (GEO/entity binding). |
| `layouts/shortcodes/org-gallery.html` | Partner card grid from `data/organizations.yaml`. |
| `layouts/shortcodes/mdimporter.html` | Pulls a remote README into a page. |
| `layouts/shortcodes/cal-embed.html` | Cal.com booking widget. |
| `layouts/shortcodes/page-section.html` | Transcludes a marked section of another page. |
| `assets/css/custom.css` | Line-clamp + org-card styles. |

## Hugo gotchas that have already bitten (do not relearn these)

- **`site.Data`, never `hugo.Data`.** `hugo.Data` does not exist; a template using it fails with
  `can't evaluate field Data in type interface {}`.
- **`extend-head.html` receives the *Site*, not the page**, and Blowfish calls it via
  `partialCached ... .Site` — so it is cached once per build and **cannot** be scoped per page. For
  page-scoped `<head>` output use **`extend-head-uncached.html`**, which receives the page.
- **Nested shortcodes render BEFORE their parent.** `faq.html` must not reset its page store before
  reading it, or it wipes the `faqitem` children that already ran.
- Hugo errors if a shortcode has a closing tag but never evaluates `.Inner`.
- **`--minify` strips attribute quotes and JSON whitespace.** Never grep built HTML for
  `type="application/ld+json"` or `"@type": "Question"` — parse the JSON instead.

## Conventions

- **Images:** global assets live in `assets/img/…` and are referenced as `/img/…` (never
  `/assets/img/…`). Page-bundle images by bare relative filename. `static/` files as `/file.ext`.
- **New content:** create from `archetypes/default.md`.
- **Renaming a section breaks `menus.en.toml` `pageRef`s.** Update both in lockstep and add an
  `aliases` entry to the moved page so the old URL still resolves.
- **`data/organizations.yaml` contract:** keys `governmental` / `non_governmental`; `logo` is a
  filename only and must exist in `assets/org-images/`.
- Every content page has a hand-written `description` and `keywords`. Keep it that way — a page
  without one falls back to a generic site bio.

## SEO / GEO

- `Person` schema (home + `/about/`) binds 11 profiles via `sameAs` — ORCID, Google Scholar,
  ResearchGate, GitHub, GitLab, socials — into one recognized entity. This is what makes the site
  citable by an AI assistant rather than anonymous.
- `FAQPage` schema on all six lanes plus the three workshop pages. Answers must be
  **self-contained**: an assistant quoting one in isolation must still be correct and attributable.
- Google Analytics (`G-L9LCQSRM78`) and Search Console verification are active. Sitemap, RSS, and
  `index.json` (search) are generated.

## Known outstanding

- **Blowfish is vendored into git with a broken submodule link** (~2,250 files, no `.gitmodules`).
  Theme upgrades will be painful until this is resolved. Deliberately out of scope so far.
- **Bilingual EN/AR is not implemented.** The site is English-only (`languages.en.toml`); `i18n/` is
  empty. Arabic currently exists only as inline `{{% rtl %}}` blocks in Services and the
  University-Email tutorials. Phase 4 of the spec designs the real multilingual build.
- **Blog has no posts yet.** Phase 5 imports Facebook / Instagram / Telegram exports.
- **Unverified credentials still published** on `/work-with-me/cv/`: CS50 Certified Instructor,
  Modarby, Courseing Platform. No supporting artifact was found for these. Substantiate or remove.
- **Degree dates conflict:** the CV page says University of Aleppo 2023–2027, while other records
  say a 2022 start. Reconcile before the next application.
