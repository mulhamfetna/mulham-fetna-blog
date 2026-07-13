---
title: "Homepage Router Redesign - Full Change Log (Before/After + Rollback)"
date: 2026-04-12
---

# Homepage Router Redesign - Full Change Log

This document records every edit made in the homepage router redesign, with:

1. **Why** the change was made.
2. **Before / After** (Git-style view).
3. **How to reverse** each change safely.

---

## Scope of edits

Edited files:

1. `config/_default/params.toml`
2. `content/_index.md`
3. `content/Courses/_index.md`
4. `content/Services/_index.md`
5. `content/About/_index.md`
6. `layouts/index.html` (new override file)
7. `layouts/shortcodes/page-section.html` (new shortcode)
8. `content/About/homepage-archive/index.md` (new archive page)

---

## Step 1) Homepage behavior configuration

### File
`config/_default/params.toml`

### Reason / cause

The homepage needed to become a controlled **router** page:

- no automatic `Recent` section,
- no extra “Show More” button for posts,
- and no background-style homepage shell that emphasized profile visuals over routing.

### Before → After

```diff
[homepage]
-  layout = "background"
+  layout = "page"
   homepageImage = "/img/mulham_professional_photo.jpg"
-  showRecent = true
+  showRecent = false
   showRecentItems = 10
-  showMoreLink = true
+  showMoreLink = false
   showMoreLinkDest = "/posts/"
```

### Reverse this change

```bash
git checkout -- config/_default/params.toml
```

Or manually restore:

- `layout = "background"`
- `showRecent = true`
- `showMoreLink = true`

---

## Step 2) Replace homepage content with router architecture

### File
`content/_index.md`

### Reason / cause

Old homepage content was biography-heavy and duplicated About.  
New homepage needed to guide users to key actions: Courses → Form funnel, Workshops, Roadmaps, Services.

### Before → After summary

**Before:** Long founder/bio/portfolio narrative with typos and “coming soon” emphasis.  
**After:** Bilingual routing page with:

1. announcement bar,
2. start-here action paths,
3. dynamic testimonials import,
4. hot topics cards/links,
5. dynamic international payment note import,
6. final CTA.

Key front matter added:

```diff
title: "Mulham Fetna | Start Here"
layout: "page"
+showDate: false
+showAuthor: false
+showAuthorBottom: false
+showDateUpdated: false
+showEdit: false
+showTableOfContents: false
```

Dynamic blocks used:

```md
{{< page-section page="/Courses" start="HOMEPAGE_TESTIMONIALS_START" end="HOMEPAGE_TESTIMONIALS_END" >}}
{{< page-section page="/Services" start="HOMEPAGE_PAYMENT_START" end="HOMEPAGE_PAYMENT_END" >}}
```

### Reverse this change

```bash
git checkout -- content/_index.md
```

If you want to keep the new file but restore old content, copy from:

- `content/About/homepage-archive/index.md`

---

## Step 3) Add testimonial source block for homepage import

### File
`content/Courses/_index.md`

### Reason / cause

Homepage testimonial section should be **editable from source page** (Courses), not hardcoded in home.

### Before → After

Inserted marker-delimited block:

```md
<!-- HOMEPAGE_TESTIMONIALS_START -->
### Student Outcomes Snapshot
...
<!-- HOMEPAGE_TESTIMONIALS_END -->
```

This block is imported by the new `page-section` shortcode.

### Reverse this change

```bash
git checkout -- content/Courses/_index.md
```

Or delete only the marker block if you want to keep other edits.

---

## Step 4) Add international payment source block for homepage import

### File
`content/Services/_index.md`

### Reason / cause

Homepage payment note should be maintained from Services policy source, not duplicated.

### Before → After

Inserted marker-delimited block:

```md
<!-- HOMEPAGE_PAYMENT_START -->
### International Payments (Quick Note)
...
<!-- HOMEPAGE_PAYMENT_END -->
```

### Reverse this change

```bash
git checkout -- content/Services/_index.md
```

Or remove only the marker block manually.

---

## Step 5) Link About page to old homepage archive

### File
`content/About/_index.md`

### Reason / cause

You requested preserving old homepage narrative under About as an archive reference.

### Before → After

Added:

```md
For the archived long-form homepage narrative, see:
[Homepage Archive (v1)](/about/homepage-archive/).
```

### Reverse this change

```bash
git checkout -- content/About/_index.md
```

Or remove only that two-line reference.

---

## Step 6) Create archive page containing old homepage narrative

### File (new)
`content/About/homepage-archive/index.md`

### Reason / cause

Preserve the old homepage text as published historical content instead of losing it in Git history only.

### Before → After

**Before:** file did not exist.  
**After:** archive page created with corrected typos (e.g., `Global Impact`, `Coming Soon`).

### Reverse this change

```bash
git rm -r content/About/homepage-archive
```

If file is uncommitted only:

```bash
rm -rf content/About/homepage-archive
```

---

## Step 7) Add reusable shortcode for dynamic section imports

### File (new)
`layouts/shortcodes/page-section.html`

### Reason / cause

Needed a lightweight dynamic mechanism to pull a bounded markdown block from another page:

1. Accepts `page`, `start`, and `end`.
2. Reads target page `.RawContent`.
3. Extracts content between comment tokens.
4. Renders with `markdownify`.
5. Emits warning if page or tokens are missing.

### Before → After

**Before:** no local shortcode for token-based page block importing.  
**After:** reusable shortcode available site-wide.

### Reverse this change

```bash
git rm layouts/shortcodes/page-section.html
```

If uncommitted:

```bash
rm layouts/shortcodes/page-section.html
```

---

## Step 8) Override theme index template to remove forced author card

### File (new override)
`layouts/index.html`

### Reason / cause

Theme default `themes/blowfish/layouts/index.html` always appended:

```go-html-template
<section class="max-w-prose pt-8 print:hidden">
  {{ partial "author.html" . }}
  {{ partial "sharing-links.html" . }}
</section>
```

That conflicted with router-home intent by injecting a personal profile block at bottom.

### Before → After

**Before:** no local override, theme default applied.  
**After:** local override keeps homepage partial rendering only, removes author/sharing append.

### Reverse this change

```bash
git rm layouts/index.html
```

If uncommitted:

```bash
rm layouts/index.html
```

Removing this file reverts behavior to theme default immediately.

---

## Full rollback options

## Option A - rollback only this redesign (surgical)

```bash
git checkout -- \
  config/_default/params.toml \
  content/_index.md \
  content/Courses/_index.md \
  content/Services/_index.md \
  content/About/_index.md

git rm -r content/About/homepage-archive layouts/index.html layouts/shortcodes/page-section.html
```

If files are still untracked, use `rm` instead of `git rm`.

## Option B - restore exact previous repository state (all local edits)

> Use only if you intentionally want to drop all working-tree edits.

```bash
git reset --hard HEAD
git clean -fd
```

---

## Verification checklist after applying or rolling back

1. Run:
   ```bash
   hugo --gc --minify
   ```
2. Confirm homepage has expected structure (router or old bio page).
3. Confirm `/about/homepage-archive/` exists (if redesign kept).
4. Confirm dynamic sections update when editing source marker blocks.

---

## Notes

- This redesign intentionally moved content ownership:
  - testimonials source → `Courses`,
  - international payment source → `Services`,
  - homepage archive source → `About/homepage-archive`.
- Unknown untracked file `content/About/past_main_page.md` was not created by this redesign and is not part of this change log.
