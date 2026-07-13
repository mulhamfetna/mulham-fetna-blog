---

showInRecent: false
---


# Google Scholar Link Integration Reference

This document records every change made to integrate a Google Scholar profile link into the existing Blowfish author links system, matching the same structure used by the built-in placeholders.

## Scope

- Goal: add Google Scholar as a visible author link under the author name, using Blowfish’s native link rendering mechanism.
- Non-goal: modify theme rendering logic or Hugo internals.

## Deep scan findings (before edits)

### 1) Theme link rendering is key-based and icon-driven

- `themes/blowfish/layouts/partials/author-links.html` iterates `params.author.links` entries as `{ key = url }` and passes the `key` to `partial "icon.html"`.
- `themes/blowfish/layouts/partials/icon.html` loads icon assets from `assets/icons/<key>.svg`.

Implication: if an icon file exists for `google-scholar.svg`, adding `{ google-scholar = "<url>" }` in `links` is sufficient.

### 2) Google Scholar icon already exists in theme assets

- Verified file: `themes/blowfish/assets/icons/google-scholar.svg`

Implication: no new icon asset is required.

### 3) Link locations where it appears on site

The same `params.author.links` data is rendered in:
- author block: `themes/blowfish/layouts/partials/author.html` (via cached partial)
- homepage profile layout: `themes/blowfish/layouts/partials/home/profile.html`
- homepage background layout: `themes/blowfish/layouts/partials/home/background.html`
- homepage hero layout: `themes/blowfish/layouts/partials/home/hero.html`

Implication: adding the link once in `config/_default/languages.en.toml` propagates to all applicable author-link locations.

## Exact file changes

## 1) `config/_default/languages.en.toml`

### A) Added commented placeholder entry (same style as existing placeholders)

Inserted directly after the existing commented `google` placeholder:

```toml
#     { google-scholar = "https://scholar.google.com/citations?user=your_id" },
```

### B) Added active visible Google Scholar link

Inserted in active links list near existing professional/research links:

```toml
{ google-scholar = "https://scholar.google.com/citations?user=h1Yvl2QAAAAJ&hl=en" },
```

## Why this is safe

- It follows the exact existing link schema (`{ platform-key = "url" }`).
- `google-scholar` maps directly to an existing icon file.
- No template logic, partial structure, or theme assets were changed.
- If removed later, behavior cleanly reverts by deleting one inline-table entry.

## Resulting behavior

- Google Scholar icon link now renders directly under the author name anywhere Blowfish shows author links.
- It opens in a new tab with the same accessibility and `rel` attributes as other links, because those behaviors come from existing shared partials.

## Validation command used

```bash
hugo --gc --minify
```

Build completes successfully; existing unrelated warnings from current repo/theme state may still appear.

## Rollback instructions

If you need to undo:
1. Remove the active line:
   ```toml
   { google-scholar = "https://scholar.google.com/citations?user=h1Yvl2QAAAAJ&hl=en" },
   ```
2. Optionally remove the commented placeholder line.

