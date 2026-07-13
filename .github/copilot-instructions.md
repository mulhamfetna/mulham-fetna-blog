# Copilot Instructions for `my-blog`

## Build, test, and lint commands

### Site-level (this repository root)
- **Production build:** `hugo --gc --minify`
- **Local development server:** `hugo server -D`
- **Tests/lint:** There are currently **no repo-level automated test or lint commands**.

### Theme-level (only when editing `themes/blowfish/`)
- Install deps: `cd themes/blowfish && npm install`
- Build theme assets: `npm run build`
- Run example site: `npm run example`
- Run a narrower example build: `npm run example:core`

## High-level architecture

- This is a **Hugo site** that uses the **Blowfish** theme (`config/_default/hugo.toml` sets `theme = "blowfish"`).
- Site behavior is split across config files in `config/_default/`:
  - `hugo.toml` (site-level Hugo settings, outputs, taxonomies, build flags)
  - `params.toml` (theme behavior, homepage/article/list UI toggles, edit link settings)
  - `languages.en.toml` (site identity + author/social metadata)
  - `menus.en.toml` (main/footer nav)
  - `markup.toml` (Goldmark/Mermaid/TOC behavior)
- Most rendering comes from `themes/blowfish/`; repository-specific behavior is added via local overrides in `layouts/shortcodes/`.
- The main custom component is the **organization gallery**:
  - Data source: `data/organizations.yaml`
  - Renderer: `layouts/shortcodes/org-gallery.html`
  - Assets: `assets/org-images/*`
  - Usage in content: e.g. `content/Courses/_index.md` via `{{< org-gallery ... >}}`
- Content is section-driven under `content/` (e.g., `Courses`, `Roadmaps`, `Services`, `Tutorials-Guides`) and navigation points to those section roots via `pageRef`.

## Key conventions in this codebase

- **Treat root `README.md` as upstream Hugo docs**, not project-specific operating instructions.
- **Menu routing depends on section naming:** `menus.en.toml` `pageRef` values are tied to `content/` paths (including current casing/hyphenation conventions like `Tutorials-Guides`, `Workshops-Camps`, `Social-Media-Posts`).
- **Organization gallery data contract:**
  - YAML keys are `governmental` and `non_governmental`.
  - `logo` values are filenames only (not paths); files must exist in `assets/org-images/`.
  - Keep shortcode/data/asset changes in sync together.
- **Image path convention:** for files in `assets/img`, reference as `/img/...` in content/config (not `/assets/img/...`).
- **New content defaults come from `archetypes/default.md`** (TOML front matter with `draft = true` by default).
- **Use `site.Data` in new templates** (e.g. `site.Data.organizations`, `index site.Data.facts $key`). Do **not** use `hugo.Data` — no such field exists in Hugo; the `hugo` namespace exposes `Version`, `Environment`, `IsProduction`, etc., and a template referencing `hugo.Data` fails the build with `can't evaluate field Data in type interface {}`. `org-gallery.html` and `fact.html` both use `site.Data`, and the build emits no deprecation warning.
