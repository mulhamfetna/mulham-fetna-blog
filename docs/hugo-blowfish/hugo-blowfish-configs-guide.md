---
showInRecent: false
---


# guide for all files other than params -> for params guide check its own guide

I'll go through each config file in sequence, explaining every setting with what it does, what enabling/disabling changes, and specific recommendations for your setup.

---

## `hugo.toml` — Core Site Configuration

### `theme = "blowfish"`
**What it does:** Tells Hugo which theme to use.

**Current state:** Commented out (`#`).

**When enabled (uncommented):**
- Hugo loads Blowfish theme
- All Blowfish features available
- Required for site to build with Blowfish styling

**When disabled (commented):**
- Site uses default Hugo templates (ugly, unstyled)
- **Must uncomment this** — your site won't look right without it

**Action:** Remove the `#` at the start of this line.

---

### `baseURL = "https://molhamfeneh.github.io"`
**What it does:** The canonical URL of your site.

**Current value:** Your GitHub Pages domain.

**Important:** You mentioned using **Cloudflare Pages** with `https://mulham-fetna-blog.pages.dev/`, but this says GitHub Pages.

**Which one is correct?**

| If hosting on | Set baseURL to |
|-------------|----------------|
| GitHub Pages | `https://molhamfeneh.github.io` |
| Cloudflare Pages | `https://mulham-fetna-blog.pages.dev/` |
| Custom domain | `https://yourdomain.com` |

**Critical:** This affects:
- RSS feed links
- Sitemap URLs
- Canonical meta tags
- Social sharing links

**Action:** Update to match your actual hosting domain.

---

### `defaultContentLanguage = "en"`
**What it does:** Primary language for the site.

**Current:** English.

**Effect:** Sets HTML `lang="en"` attribute, affects SEO, date formatting.

**Keep as is** unless you add Arabic content later.

---

### `# pluralizeListTitles = "true"`
**What it does:** Automatically pluralizes section titles.

**Currently commented out:** Hugo's default behavior.

**When enabled (`true`):**
- "Category" becomes "Categories" in titles
- "Tag" becomes "Tags"
- Useful for non-English languages with complex pluralization

**When commented out:**
- Uses literal titles from your content

**Your case:** Keep commented. English pluralization works fine by default.

---

### `enableRobotsTXT = true`
**What it does:** Generates `/robots.txt` file.

**When enabled (`true`):**
- Creates `robots.txt` at site root
- Allows all search engines by default
- Sitemap reference included automatically

**When disabled (`false`):**
- No `robots.txt` generated
- Search engines may crawl less efficiently

**Keep `true`** — essential for SEO.

---

### `summaryLength = 0`
**What it does:** Controls automatic summary generation.

**Current (`0`):**
- No automatic summary
- Uses `<!--more-->` manual break in content
- Or `description` frontmatter field

**If set to `30`:**
- First 30 words become summary automatically

**Your preference:** `0` gives you control. Keep it.

---

### `buildDrafts = false`
**What it does:** Whether to build content marked `draft: true`.

**When `false`:**
- Drafts excluded from production build
- Only visible in local development (`hugo server -D`)

**When `true`:**
- Drafts published live
- Risk: unfinished content goes public

**Keep `false`** for safety. Use `hugo server -D` locally to preview drafts.

---

### `buildFuture = false`
**What it does:** Whether to build content with future dates.

**When `false`:**
- Posts dated tomorrow won't build until that date
- Good for scheduled publishing

**When `true`:**
- Future posts appear immediately
- Risk: accidental early publishing

**Keep `false`**. Use future dates to schedule posts.

---

### `enableEmoji = true`
**What it does:** Converts emoji shortcodes to Unicode.

**When enabled (`true`):**
- `:smile:` becomes 😄 in content
- `:thinking_face:` becomes 🤔
- Works in Markdown

**When disabled (`false`):**
- Shortcodes remain as text
- Must use actual Unicode emoji

**Keep `true`** — convenient for writing.

---

### `# googleAnalytics = "G-XXXXXXXXX"`
**What it does:** Google Analytics 4 tracking ID.

**Currently commented out:** No analytics.

**When enabled:**
- Tracking script injected on every page
- Visitor data sent to Google Analytics
- Privacy implications (cookies, consent banner needed in many regions)

**Your case:** Keep commented if you don't want tracking. Uncomment and add your `G-XXXXXXXX` ID if you want analytics.

---

### `[pagination]` Section

```toml
[pagination]
  pagerSize = 100
```

**What it does:** Controls list page splitting.

**Current (`100`):**
- 100 posts per page before "Next" button appears
- Long scroll, fewer clicks

**If set to `10`:**
- 10 posts per page
- More pagination, faster loading

**Recommendation:** `100` is high for a blog. Consider `20` or `30` for better performance.

---

### `[imaging]` Section

```toml
[imaging]
  anchor = 'Center'
```

**What it does:** Default crop anchor for image resizing.

**Current (`'Center'`):**
- When Hugo crops images, keeps center
- Faces in middle of photo stay visible

**Options:** `'Top'`, `'Bottom'`, `'Left'`, `'Right'`, `'Center'`, `'Smart'` (experimental face detection)

**Your case:** You wanted top-focused profile photo. This affects automatic crops, but **not** profile images directly. For profile, pre-crop or use CSS.

---

### `[taxonomies]` Section

```toml
[taxonomies]
  tag = "tags"
  category = "categories"
  author = "authors"
  series = "series"
```

**What it does:** Defines content organization systems.

| Taxonomy | Purpose | Your Use |
|----------|---------|----------|
| `tag` | Flexible topics | STM32, Python, AI |
| `category` | Broad sections | Tutorials, Projects, Media |
| `author` | Multi-author support | Just you (can remove) |
| `series` | Ordered sequences | Course workshops |

**All enabled:** Blowfish creates pages for each automatically (`/tags/stm32/`, `/series/neurobotics-workshops/`).

**To disable one:** Remove the line. If single-author, remove `author = "authors"`.

---

### `[sitemap]` Section

```toml
[sitemap]
  changefreq = 'daily'
  filename = 'sitemap.xml'
  priority = 0.5
```

**What it does:** Controls `sitemap.xml` generation.

| Setting | Meaning | Your Value |
|---------|---------|------------|
| `changefreq` | How often content updates | `daily` (optimistic for blog) |
| `filename` | Output file name | `sitemap.xml` (standard) |
| `priority` | Default page importance | `0.5` (middle, 0.0-1.0 scale) |

**Effect:** Search engines use this as hint, not command.

**Recommendation:** `changefreq = 'weekly'` more realistic for personal blog. `daily` is fine if you post frequently.

---

### `[outputs]` Section

```toml
[outputs]
  home = ["HTML", "RSS", "JSON"]
```

**What it does:** Output formats for homepage.

| Format | Purpose | File Generated |
|--------|---------|----------------|
| `HTML` | Web page | `index.html` |
| `RSS` | Feed readers | `index.xml` |
| `JSON` | Headless CMS, search | `index.json` |

**Blowfish uses `JSON`** for its Fuse.js search index.

**Keep all three** — search breaks without JSON.

---

### `[related]` Section

```toml
[related]
  threshold = 0
  toLower = false
```

**What it does:** Configures "Related Posts" feature.

| Setting | Effect | Your Value |
|---------|--------|------------|
| `threshold` | Minimum similarity score (0-100) | `0` (show all, no filtering) |
| `toLower` | Case-insensitive matching | `false` (case-sensitive) |

**Indices below** define how similarity is calculated:

```toml
[[related.indices]]
    name = "tags"
    weight = 100        # Most important: shared tags
```

- `weight = 100`: Tags are primary matching factor
- `weight = 50`: Series secondary
- `weight = 10`: Date least important

**Effect:** Posts with same tags appear as "Related" at bottom of articles.

**Your case:** `threshold = 0` means even weakly related posts show. Consider `80` for stricter matching.

---

## `languages.en.toml` — English Language Config

### `disabled = false`
**What it does:** Whether this language is active.

**Keep `false`** (meaning enabled — confusing double-negative).

If `true`, English section is disabled.

---

### `languageCode = "en"`
**What it does:** HTML `lang` attribute and RSS feeds.

**Standard:** `en`, `en-US`, `en-GB`.

**Keep `en`** — fine for international English.

---

### `languageName = "English"`
**What it does:** Display name in language switcher.

**Effect:** If you add Arabic later, shows "English | العربية" dropdown.

---

### `weight = 1`
**What it does:** Order in language switcher.

**Lower = first.** English as primary = `1`.

---

### `title = "Mulham Fetna Blog"`
**What it does:** Site title in browser tab, RSS, search results.

**Current:** Your name + Blog.

**Appears as:** `<title>Mulham Fetna Blog</title>` on homepage.

---

### `[params]` Section

#### `displayName = "EN"`
**What it does:** Short language code in switcher.

**Compact version** of `languageName`.

---

#### `isoCode = "en"`
**What it does:** ISO 639-1 language code.

**Standardization** for meta tags, Open Graph.

---

#### `rtl = false`
**What it does:** Right-to-left text direction.

**Current (`false`):** Left-to-right (English).

**When `true`:** For Arabic, Hebrew. Flips entire layout.

**Future:** If you add Arabic (`languages.ar.toml`), set `rtl = true` there.

---

#### `dateFormat = "2 January 2006"`
**What it does:** How dates display.

**Go reference date:** `2 January 2006` = `31 March 2026` format.

**Options:**
- `"January 2, 2006"` → March 31, 2026
- `"2006-01-02"` → 2026-03-31 (ISO)
- `"02/01/2006"` → 31/03/2026 (European)

**Keep current** — clear, readable.

---

#### `# logo = "img/logo.png"`
**What it does:** Site logo image.

**Currently commented:** Uses text title instead.

**When enabled:**
- Image replaces text in header
- Should be SVG or transparent PNG
- Height ~40px recommended

**Your case:** Keep commented for text-based header, or create simple logo.

---

#### `# secondaryLogo = "img/secondary-logo.png"`
**What it does:** Alternative logo for dark mode.

**Use case:** White logo for dark backgrounds.

**Keep commented** unless you have dark-mode-specific logo.

---

#### `description = "Mulham Fetna Blog"`
**What it does:** Meta description for SEO, social sharing.

**Current:** Generic.

**Improve to:** `"Robotics, AI, and embedded systems tutorials by Mulham Fetna"`

---

#### `# copyright = "Copy, _right?_ :thinking_face:"`
**What it does:** Footer copyright text.

**Currently commented:** Uses default `© 2026 Mulham Fetna`.

**When enabled:** Custom text with emoji support.

**Keep commented** for professional default, or uncomment for personality.

---

### `[params.author]` Section

#### `name = "Mulham Fetna"`
**What it does:** Your display name across site.

**Used in:** Article bylines, profile card, RSS feed.

---

#### `# email = "youremail@example.com"`
**What it does:** Contact email.

**Currently commented:** No email displayed.

**When enabled:**
- Shows in profile
- Used for `mailto:` links if you enable email in `links` array

**Your choice:** Add professional email or keep private.

---

#### `image = "/img/mulham_professional_photo.jpg"`
**What it does:** Your profile photo.

**Path:** `/img/` = `assets/img/` in your repo.

**Critical:** This is the photo you wanted top-cropped. As discussed, pre-crop to square or use CSS `object-position`.

---

#### `imageQuality = 96`
**What it does:** JPEG/WebP quality for processed images.

**Range:** 1-100.

**Current (`96`):** High quality, larger files.

**Trade-off:** `75` = smaller, faster; `96` = crisp, slower.

**Keep `96`** for professional photo, or reduce to `85` for speed.

---

#### `headline = "Mechatronics Engineer"`
**What it does:** Job title under your name in profile.

**One-line summary** of who you are.

---

#### `bio = "Renaissance Engineer"`
**What it does:** Short description in profile card.

**Current:** Philosophical.

**Alternative:** Skills-focused — `"Building robots, training AI models, teaching embedded systems"`

---

#### `links` Array

Your active links (uncommented):
- Facebook: `https://www.facebook.com/mulham.robotics`
- GitHub: `https://github.com/molhamfetneh`
- GitLab: `https://gitlab.com/molhamfetneh`
- Instagram: `https://instagram.com/mulham.robotics`
- LinkedIn: `https://linkedin.com/in/molham-fetnah`
- Telegram: `https://t.me/Mulham_Fetna_official`
- WhatsApp: `https://chat.whatsapp.com/EmFS1vgi9oq4EUWn2U6WIl`

**Commented out:** 30+ other platforms (Twitter/X, YouTube, Discord, etc.).

**Effect:** Icons appear in profile card, header, footer depending on Blowfish layout.

**Recommendation:** WhatsApp group link is public — consider if you want random visitors joining. Maybe comment out and keep Telegram for public contact.

---

## `markup.toml` — Content Rendering

### `[goldmark]` Section

Hugo's default Markdown parser (replacement for Blackfriday).

---

#### `wrapStandAloneImageWithinParagraph = false`
**What it does:** HTML structure for standalone images.

**When `false`:**
```html
<figure><img></figure>
```
- Clean, no `<p>` wrapper
- Better for CSS styling

**When `true`:**
```html
<p><img></p>
```
- Paragraph wrapper
- Default browser styling

**Keep `false`** — Blowfish expects this for proper figure styling.

---

#### `[goldmark.parser.attribute]` `block = true`
**What it does:** Allows Markdown attributes on blocks.

**When `true`:**
```markdown
> Quote
{.class-name}
```
Adds class to blockquote.

**For advanced styling** — keep `true`.

---

#### `[goldmark.renderer]` `unsafe = true`
**What it does:** Allows raw HTML in Markdown.

**When `true`:**
```markdown
<div class="custom">HTML works</div>
```
- Embedded HTML renders
- Required for some shortcodes, tables

**When `false`:**
- HTML escaped to `&lt;div&gt;`
- Pure Markdown only

**Keep `true`** — needed for Blowfish shortcodes, but be careful with user-generated content (not applicable for personal blog).

---

#### `[goldmark.extensions.passthrough]`
**What it does:** Math rendering (LaTeX).

```toml
enable = true
block = [['\[', '\]'], ['$$', '$$']]      # Display math
inline = [['\(', '\)']]                    # Inline math
```

**When enabled:**
- `$$E=mc^2$$` becomes rendered equation
- Requires additional math library (KaTeX or MathJax)

**Your case:** Keep enabled if you write technical content with formulas. Disable if not used (slight performance gain).

---

### `[highlight]`
```toml
[highlight]
  noClasses = false
```

**What it does:** Code syntax highlighting method.

**When `false`:**
- Uses CSS classes for highlighting
- Theme-controlled colors (Blowfish's scheme)
- Smaller HTML, cached CSS

**When `true`:**
- Inline styles
- Hardcoded colors in HTML
- Larger files, but works without CSS

**Keep `false`** — proper way, theme integration.

---

### `[tableOfContents]`
```toml
[tableOfContents]
  startLevel = 2
  endLevel = 4
```

**What it does:** Which headings appear in TOC.

| Setting | Effect | Your Value |
|---------|--------|------------|
| `startLevel = 2` | H2 and below in TOC | Skips H1 (title) |
| `endLevel = 4` | Up to H4 included | H5, H6 excluded |

**Result:** TOC shows H2, H3, H4 — good structure for technical posts.

**If `startLevel = 1`:** Would include post title in TOC (redundant).

---

## `menus.en.toml` — Navigation

### `[[main]]` Menu (Header Navigation)

```toml
[[main]]
  name = "Blog"
  pageRef = "posts"
  weight = 10
```

**What it does:** Top navigation link.

| Field | Meaning | Your Value |
|-------|---------|------------|
| `name` | Display text | "Blog" |
| `pageRef` | Target section | "posts" → `/posts/` or `/posts` |
| `weight` | Order | 10 (first) |

**Critical issue:** Your content structure earlier showed `content/blog/`, but this points to `posts`.

**Mismatch:**
- Menu → `posts`
- Content → `blog/`
- Result: 404 or empty page

**Fix:** Change to `pageRef = "blog"` or rename folder to `posts/`.

---

### Commented Examples

```toml
#[[main]]
#  name = "Parent"
#  weight = 20
#
#[[main]]
#  name = "example sub-menu 1"
#  parent = "Parent"
#  pageRef = "posts"
#  weight = 20
```

**What it shows:** Dropdown menu structure.

**When enabled:**
- "Parent" appears in header
- Hover reveals "example sub-menu 1" and "2"
- Creates nested navigation

**Your case:** Uncomment and adapt for your sections:
```toml
[[main]]
  name = "Content"
  weight = 20

[[main]]
  name = "Blog"
  parent = "Content"
  pageRef = "blog"
  weight = 10

[[main]]
  name = "Media"
  parent = "Content"
  pageRef = "media"
  weight = 20

[[main]]
  name = "Projects"
  parent = "Content"
  pageRef = "projects"
  weight = 30
```

---

### `[[subnavigation]]` (Below Header)

**Currently all commented.**

**What it does:** Secondary navigation bar under main header.

**Use case:** Filter tags, categories, featured topics.

**Example to enable:**
```toml
[[subnavigation]]
  name = "STM32"
  pageRef = "tags/stm32"
  weight = 10

[[subnavigation]]
  name = "Python"
  pageRef = "tags/python"
  weight = 20
```

---

### `[[footer]]` Menu

**All commented.**

**What it does:** Links in footer area.

**Enable for:**
- Privacy policy
- Contact
- Colophon / about site

**Example:**
```toml
[[footer]]
  name = "Contact"
  pageRef = "contact"
  weight = 10

[[footer]]
  name = "RSS"
  url = "/index.xml"
  weight = 20
```

---

## `module.toml` — Empty

**Currently empty.**

**What it's for:** Hugo Modules (theme management, imports).

**When using:**
```toml
[module]
  [[module.imports]]
    path = "github.com/nunocoracao/blowfish/v2"
```

**Your case:** You're using Git submodule or manual theme install, so empty is fine.

---

## Critical Fixes Needed

| Issue | Location | Fix |
|-------|----------|-----|
| Theme commented out | `hugo.toml` line 4 | Uncomment `theme = "blowfish"` |
| Wrong baseURL | `hugo.toml` line 5 | Change to `https://mulham-fetna-blog.pages.dev/` |
| Menu points to `posts`, content in `blog` | `menus.en.toml` | Change `pageRef = "posts"` to `"blog"` |
| WhatsApp group public | `languages.en.toml` | Consider commenting out |

---

## Recommended Active Menu Structure

Based on your earlier brainstorm:

```toml
[[main]]
  name = "Blog"
  pageRef = "blog"
  weight = 10

[[main]]
  name = "Media"
  pageRef = "media"
  weight = 20

[[main]]
  name = "Projects"
  pageRef = "projects"
  weight = 30

[[main]]
  name = "About"
  pageRef = "about"
  weight = 40
```

Create matching folders:
```
content/
├── blog/
├── media/
├── projects/
└── about/
```

---

Want me to provide the complete corrected config files with these fixes applied?

