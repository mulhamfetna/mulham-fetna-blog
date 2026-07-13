---
showInRecent: false
---


# Edit Documentation: Global Author + Sharing Activation

This document explains the exact template edits made to ensure author blocks and sharing buttons appear more consistently across page types.

Scope covered in this change set:

1. `themes/blowfish/layouts/_default/single.html`
2. `themes/blowfish/layouts/_default/list.html`
3. `themes/blowfish/layouts/_default/simple.html`
4. `themes/blowfish/layouts/index.html`

---

## Root Cause Analysis

The site uses different Hugo templates for different page kinds:

1. **Single pages** (`_default/single.html`)
2. **List/section pages** (`_default/list.html`)
3. **Simple pages** (`_default/simple.html`)
4. **Homepage** (`layouts/index.html` + home partials)

Before this change, sharing and author rendering logic was inconsistent between these templates.  
Result: some pages showed author/share UI while others did not.

---

## 1) File: `themes/blowfish/layouts/_default/single.html`

### Cause of edit

Author rendering depended on `showAuthorBottom` branching (top *or* bottom), which created inconsistent placement behavior.

### Git-style before/after

```diff
diff --git a/themes/blowfish/layouts/_default/single.html b/themes/blowfish/layouts/_default/single.html
index d1c2719..cfacccd 100644
--- a/themes/blowfish/layouts/_default/single.html
+++ b/themes/blowfish/layouts/_default/single.html
@@ -24,9 +24,7 @@
       <div class="mt-1 mb-6 text-base text-neutral-500 dark:text-neutral-400 print:hidden">
         {{ partial "article-meta/basic.html" (dict "context" . "scope" "single") }}
       </div>
-      {{ if not (.Params.showAuthorBottom | default (site.Params.article.showAuthorBottom | default false)) }}
-        {{ template "SingleAuthor" . }}
-      {{ end }}
+      {{ template "SingleAuthor" . }}
@@ -68,9 +66,7 @@
             </strong>
           {{ end }}
         </div>
-        {{ if (.Params.showAuthorBottom | default (site.Params.article.showAuthorBottom | default false)) }}
-          {{ template "SingleAuthor" . }}
-        {{ end }}
+        {{ template "SingleAuthor" . }}
         {{ partial "series/series-closed.html" . }}
         {{ partial "sharing-links.html" . }}
         {{ partial "related.html" . }}
```

### Result after edit

1. Author block is now rendered unconditionally at both locations used by template flow.
2. Sharing links remain present in single pages.
3. Behavior is more consistent regardless of frontmatter toggles.

---

## 2) File: `themes/blowfish/layouts/_default/list.html`

### Cause of edit

List/section pages did not render sharing links by default, and did not show author block in a clear consistent place.

### Git-style before/after

```diff
diff --git a/themes/blowfish/layouts/_default/list.html b/themes/blowfish/layouts/_default/list.html
index 640def9..45de890 100644
--- a/themes/blowfish/layouts/_default/list.html
+++ b/themes/blowfish/layouts/_default/list.html
@@ -22,6 +22,9 @@
     <div class="mt-1 mb-2 text-base text-neutral-500 dark:text-neutral-400 print:hidden">
       {{ partial "article-meta/list.html" (dict "context" . "scope" "single") }}
     </div>
+    <div class="mt-4 mb-6 print:hidden">
+      {{ partial "author.html" . }}
+    </div>
   </header>
@@ -137,5 +140,9 @@
       </p>
     </section>
   {{ end }}
+  <section class="pt-8 print:hidden">
+    {{ partial "author.html" . }}
+    {{ partial "sharing-links.html" . }}
+  </section>
   {{ partial "pagination.html" . }}
 {{ end }}
```

### Result after edit

1. List pages now show author info near the header.
2. List pages now include sharing links near the footer area.
3. Section pages and taxonomy-like list contexts get consistent social/action UI.

---

## 3) File: `themes/blowfish/layouts/_default/simple.html`

### Cause of edit

Simple layout had sharing links only in footer and no explicit author rendering around header/footer.

### Git-style before/after

```diff
diff --git a/themes/blowfish/layouts/_default/simple.html b/themes/blowfish/layouts/_default/simple.html
index 7b5b9a8..38d3226 100644
--- a/themes/blowfish/layouts/_default/simple.html
+++ b/themes/blowfish/layouts/_default/simple.html
@@ -7,11 +7,15 @@
       <h1 class="mt-0 text-4xl font-extrabold text-neutral-900 dark:text-neutral">
         {{ .Title | emojify }}
       </h1>
+      <div class="mt-4 mb-4 print:hidden">
+        {{ partial "author.html" . }}
+      </div>
     </header>
     <section class="max-w-full mt-6 prose dark:prose-invert">
       {{ .Content }}
     </section>
     <footer class="pt-8">
+      {{ partial "author.html" . }}
       {{ partial "sharing-links.html" . }}
     </footer>
   </article>
```

### Result after edit

1. Simple pages now show author profile in header area.
2. Simple pages also show author + sharing in footer.
3. Standalone pages using simple layout are now consistent with social/author presentation.

---

## 4) File: `themes/blowfish/layouts/index.html`

### Cause of edit

Homepage template did not include sharing/author action block after rendering the selected home partial.

### Git-style before/after

```diff
diff --git a/themes/blowfish/layouts/index.html b/themes/blowfish/layouts/index.html
index 3b35e78..bfc33f4 100644
--- a/themes/blowfish/layouts/index.html
+++ b/themes/blowfish/layouts/index.html
@@ -5,4 +5,8 @@
   {{ else }}
     {{ partial "home/profile.html" . }}
   {{ end }}
+  <section class="max-w-prose pt-8 print:hidden">
+    {{ partial "author.html" . }}
+    {{ partial "sharing-links.html" . }}
+  </section>
 {{ end }}
```

### Result after edit

1. Homepage now has explicit author + sharing block after home partial content.
2. Home layout variants now inherit a consistent end-of-page action area.

---

## Net Functional Outcome

After these edits:

1. Sharing buttons are rendered in all key page template families.
2. Author badge/profile rendering is no longer limited to only some page kinds.
3. UI behavior is consistent across single/list/simple/home contexts.

---

## Notes

1. This change set modifies **theme layout templates** directly under `themes/blowfish/layouts/...`.
2. If the theme is updated/upgraded, these local template edits may need re-application.
