---
showInRecent: false
---


# mdimporter Relative-Link Rewrite Integration (Deep Technical Documentation)

This document captures the full implementation details for fixing imported README relative links so they resolve to the source repository (not the blog URL), with trackable git-style before/after blocks.

---

## Problem Statement

When using Blowfish `mdimporter` with remote README files (for example from `raw.githubusercontent.com`), relative links such as:

- `[train.py](src/train.py)`
- `![diagram](images/arch.png)`

were rendered by Hugo as local site links. This caused internal README references to point to the blog domain instead of the GitHub repository.

---

## Root Cause Analysis

The original shortcode in theme code only did:

1. Fetch remote markdown
2. Pipe raw content directly to `markdownify`

No preprocessing step existed to rewrite relative paths before markdown rendering.

Because of that, Hugo render hooks interpreted those relative paths in the local page/site context.

---

## Scope of Change

### Files read during analysis

1. `themes/blowfish/layouts/shortcodes/mdimporter.html`
2. `themes/blowfish/layouts/_default/_markup/render-link.html`
3. `themes/blowfish/layouts/_default/_markup/render-image.html`
4. `content/Projects/ai-cup-2026-bird-classification/_index.md`

### Files changed

1. `layouts/shortcodes/mdimporter.html` (new local override)

No theme files were edited in-place. The fix is override-based and theme-update-safe.

---

## Step 1 — Baseline (Original Theme Behavior)

### File: `themes/blowfish/layouts/shortcodes/mdimporter.html`

```diff
{{ $url := .Get "url" }}
{{ with resources.GetRemote (urls.Parse $url) }}
  {{ .Content | markdownify }}
{{ else }}
  {{ warnf "mdimporter shortcode: unable to fetch %q: %s" $url .Position }}
{{ end }}
```

### Outcome

- Markdown imported successfully.
- Relative links/images were not rewritten to repository paths.

---

## Step 2 — Implement Local Override for Rewrite Logic

### Strategy

Create a project-level shortcode override at:

- `layouts/shortcodes/mdimporter.html`

This lets the site keep theme behavior while adding rewrite preprocessing for GitHub-raw README imports.

### Git-style before/after block

```diff
diff --git a/layouts/shortcodes/mdimporter.html b/layouts/shortcodes/mdimporter.html
new file mode 100644
--- /dev/null
+++ b/layouts/shortcodes/mdimporter.html
@@ -0,0 +1,34 @@
+{{ $url := .Get "url" }}
+{{ with resources.GetRemote (urls.Parse $url) }}
+  {{ $content := .Content }}
+
+  {{/* Auto-rewrite relative links/images when importing from raw.githubusercontent.com */}}
+  {{ $ghMatch := findRESubmatch `^https://raw\.githubusercontent\.com/([^/]+)/([^/]+)/([^/]+)/(.*)$` $url }}
+  {{ if gt (len $ghMatch) 0 }}
+    {{ $parts := index $ghMatch 0 }}
+    {{ $owner := index $parts 1 }}
+    {{ $repo := index $parts 2 }}
+    {{ $ref := index $parts 3 }}
+    {{ $filePath := index $parts 4 }}
+    {{ $dir := path.Dir $filePath }}
+    {{ $dirPrefix := cond (eq $dir ".") "" (printf "%s/" $dir) }}
+    {{ $blobBase := printf "https://github.com/%s/%s/blob/%s/%s" $owner $repo $ref $dirPrefix }}
+    {{ $rawBase := printf "https://raw.githubusercontent.com/%s/%s/%s/%s" $owner $repo $ref $dirPrefix }}
+
+    {{/* Rewrite Markdown links (and image links initially) */}}
+    {{ $content = replaceRE `\]\(((\./|\.\./)[^)]+|[^/#:][^)#:]*)\)` (printf "](%s$1)" $blobBase) $content }}
+    {{/* Convert Markdown image links to raw URLs so images render on-site */}}
+    {{ $content = replaceRE `(?i)!\[([^\]]*)\]\(https://github\.com/([^/]+)/([^/]+)/blob/([^/]+)/([^)]+\.(png|jpe?g|gif|svg|webp|avif|bmp|ico))\)` `![$1](https://raw.githubusercontent.com/$2/$3/$4/$5)` $content }}
+
+    {{/* Rewrite HTML links/images if README uses inline HTML */}}
+    {{ $content = replaceRE `(?i)(<a[^>]+href=['"])((\./|\.\./)[^'"]+|[^/#:][^'"]*)(['"])` (printf `${1}%s$2${4}` $blobBase) $content }}
+    {{ $content = replaceRE `(?i)(<img[^>]+src=['"])((\./|\.\./)[^'"]+|[^/#:][^'"]*)(['"])` (printf `${1}%s$2${4}` $rawBase) $content }}
+    {{/* Rewrite reference-style link definitions */}}
+    {{ $content = replaceRE `(?m)^(\[[^\]]+\]:\s*)((\./|\.\./)\S+|[^/#:\s]\S*)` (printf `${1}%s$2` $blobBase) $content }}
+  {{ end }}
+
+  {{ $content | markdownify }}
+{{ else }}
+  {{ warnf "mdimporter shortcode: unable to fetch %q: %s" $url .Position }}
+{{ end }}
```

---

## Step 3 — Regex Engine Constraint and Finalization

During implementation, negative lookahead patterns (PCRE style) were attempted first. Hugo/Go regex uses RE2 and does not support lookaheads, which caused build failure.

### Failing pattern class (for traceability)

```diff
-\]\((?!https?://|mailto:|/|#)([^)]+)\)
```

### Error observed

`error parsing regexp: invalid or unsupported Perl syntax: '(?!'`

### Resolution

Replaced lookahead-based regex with RE2-compatible alternation patterns (current final version in `layouts/shortcodes/mdimporter.html`).

---

## Rewrite Rules Implemented (Final Behavior)

When `url` matches `raw.githubusercontent.com/<owner>/<repo>/<ref>/<path>`:

1. Markdown relative links → `https://github.com/<owner>/<repo>/blob/<ref>/<dir>/<target>`
2. Markdown image links (that became GitHub blob links) → converted to `raw.githubusercontent.com` image URLs
3. HTML `<a href="...">` relative links → GitHub blob URLs
4. HTML `<img src="...">` relative images → raw GitHub URLs
5. Reference-style markdown link definitions → GitHub blob URLs

For non-GitHub-raw URLs, shortcode keeps default import behavior (no rewrite).

---

## Why This Approach Was Chosen

### Selected approach

- Local shortcode override with automatic rewrite

### Reasons

1. One-time implementation; works for future imported READMEs.
2. No per-project manual edits.
3. Avoids modifying vendored theme source directly.
4. Keeps content usage unchanged (`{{</* mdimporter url="..." */>}}` still works).

---

## Verification Target

### Page using importer

- `content/Projects/ai-cup-2026-bird-classification/_index.md`

### Result after change

- Internal README links now resolve to repository `github.com/.../blob/...`
- README images use raw GitHub URLs where needed for direct rendering
- Existing external absolute links remain external

---

## Maintenance Notes

1. This behavior is now centralized in one override file.
2. If future repositories use non-main branches/tags, rewrite remains compatible because branch/ref is parsed from the import URL.
3. If edge README formats appear (nested reference/image syntaxes), extend this single shortcode rather than editing individual content pages.

---

## Post-Implementation Hotfix (Documentation Render Safety)

### Incident

After publishing this document, Hugo attempted to execute an inline `mdimporter` example as a real shortcode, causing:

- `resources.GetRemote` to receive an empty URL
- build failure with: `unsupported protocol scheme ""`

### Root cause

The line in this document used executable shortcode syntax inside inline code text.

### Fix applied

The inline example was changed to escaped shortcode-display syntax.

### Git-style before/after block

```diff
diff --git a/content/Tutorials-Guides/Hugo-BlowFish/mdimporter-relative-link-rewrite-integration.md b/content/Tutorials-Guides/Hugo-BlowFish/mdimporter-relative-link-rewrite-integration.md
index <previous>..<current> 100644
--- a/content/Tutorials-Guides/Hugo-BlowFish/mdimporter-relative-link-rewrite-integration.md
+++ b/content/Tutorials-Guides/Hugo-BlowFish/mdimporter-relative-link-rewrite-integration.md
@@ -168,1 +168,1 @@
-4. Keeps content usage unchanged (`{{</* mdimporter url="..." */>}}` still works). <!-- originally unescaped -->
+4. Keeps content usage unchanged (`{{</* mdimporter url="..." */>}}` still works).
```

### Preventive note

When documenting Hugo shortcodes in markdown prose, always use escaped shortcode notation (`{{</* ... */>}}`) unless execution is intentional.
