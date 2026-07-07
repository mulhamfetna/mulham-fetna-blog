# installation guide
## the blog 
https://mulham-fetna-blog.pages.dev/


## the cloudflare pages project
https://dash.cloudflare.com/62d6b0abac91c436491b9e4df8dbeabc/pages/view/mulham-fetna-blog


## the github repo
https://github.com/mulhamfetna/mulham-fetna-blog


## push integrated command
git add . && git commit -m "." && git push -u origin main


## the project structure
1. the blog -> hugo -> https://gohugo.io/
2. theme -> blowfish -> https://blowfish.page/
   > make sure to edit the hugo.toml file as the configuration as the guides tell to delete the file and copy it own theme folder to the project themes and then start configureing
3. push to github repo
4. deploy using cloudeflare pages NOT workers PAGES -> choose the template as hugo and point it to the repo
5. create main page by adding _index.md
6. play with menus and creat the website

***
---
***


# asstests paths 
## images 
 Great question! This is a common point of confusion in Hugo. Here's exactly how it works with Blowfish:

---

## The Two Main Approaches

### Option 1: Page Bundles (Recommended for Blog Posts)

**Structure:**
```
content/
└── blog/
    └── my-stm32-tutorial/
        ├── index.md
        └── p1.png          # Image right next to content
```

**Reference in `index.md`:**
```markdown
![STM32 Wiring](p1.png)
```
or
```markdown
![STM32 Wiring](./p1.png)
```

**Result:** `./p1.png` or just `p1.png` — **relative to the page bundle folder**

**Pros:**
- Images processed by Hugo (resized, WebP generated)
- Travel with content if you move/rename posts
- Git tracks them together
- Blowfish optimizes them automatically

---

### Option 2: Global Assets (For Shared Images)

**Structure:**
```
assets/
└── img/
    └── p1.png              # Global image
```

**Reference in any post:**
```markdown
![Logo](/img/p1.png)
```

**Critical:** Path starts from **`/img/`** NOT `/assets/img/`

**Why?** Hugo mounts `assets/` at root. You never reference `assets/` in URLs.

---

## Your Specific Question Answered

| Location | Wrong Path | Correct Path |
|----------|-----------|--------------|
| `assets/img/p1.png` | `./assets/img/p1.png` | `/img/p1.png` |
| Page bundle `p1.png` | `/img/p1.png` | `p1.png` or `./p1.png` |

---

## Recommended Strategy for Your Site

### For Blog Posts (STM32 guides, tutorials)

Use **page bundles** — put images next to content:

```
content/blog/stm32-lcd-guide/
├── index.md
├── wiring-diagram.png
├── code-screenshot.png
└── final-result.jpg
```

In `index.md`:
```markdown
---
title: "STM32 LCD Programming"
cover:
  image: "wiring-diagram.png"    # Blowfish picks this up automatically
---

Here's the wiring:

![Wiring Diagram](wiring-diagram.png)

And the result:

![Final Result](final-result.jpg)
```

### For Shared Elements (Logo, Author Photo, Default Images)

Use **global assets**:

```
assets/
└── img/
    ├── author.jpg              # Your profile photo
    ├── default-cover.jpg       # Fallback for posts without images
    └── social-card.jpg         # For social sharing
```

Reference anywhere:
```markdown
![My Photo](/img/author.jpg)
```

Or in `params.toml`:
```toml
[homepage]
  homepageImage = "/img/author.jpg"    # NOT /assets/img/author.jpg

[article]
  defaultFeaturedImage = "/img/default-cover.jpg"
```

---

## Quick Reference Table

| Scenario | File Location | Reference Path |
|----------|-------------|----------------|
| Post-specific image | `content/blog/post-name/image.png` | `image.png` |
| Post cover image | `content/blog/post-name/cover.png` | `cover.png` (in frontmatter) |
| Site-wide logo | `assets/img/logo.png` | `/img/logo.png` |
| Author avatar | `assets/img/avatar.jpg` | `/img/avatar.jpg` |
| Default post image | `assets/img/default.jpg` | `/img/default.jpg` |
| Static file (no processing) | `static/resume.pdf` | `/resume.pdf` |

---

## The `static/` vs `assets/` Difference

| Folder | Processing | Use For |
|--------|-----------|---------|
| `assets/` | Hugo processes, optimizes, resizes | Images, CSS, JS |
| `static/` | Copied as-is | PDFs, ZIPs, robots.txt, favicon.ico |

**Example:**
```
static/
└── cv.pdf                    # Reference: /cv.pdf

assets/
└── img/
    └── project-photo.jpg     # Reference: /img/project-photo.jpg
```

---

## Your Migration from Current Structure

Looking at your files, you have images scattered in:
- `Arch-Wallpapers/`
- `DUAA/`
- `solid_works/`

**Recommended cleanup:**

1. **Move post-specific images** to page bundles:
   ```bash
   # Instead of
   DUAA/7_5_5c4a976d-c917-4049-ba8f-85bb33d52568_1024x1024.webp
   
   # Create
   content/projects/duaa-robot/
   ├── index.md
   └── images/
       └── ada-assembly.webp
   ```

2. **Move reusable images** to assets:
   ```bash
   assets/img/
   ├── logo.png
   ├── author.jpg
   └── projects/
       ├── duaa-thumb.jpg
       └── solidworks-thumb.jpg
   ```

3. **Reference correctly:**
   ```markdown
   <!-- In content/projects/duaa-robot/index.md -->
   ![ADA Assembly](images/ada-assembly.webp)
   
   <!-- Or for global image -->
   ![Project Thumbnail](/img/projects/duaa-thumb.jpg)
   ```

---

## Blowfish-Specific: Cover Images

For Blowfish to show featured images properly, use this frontmatter:

```yaml
---
title: "My Project"
cover:
  image: "/img/projects/hero.jpg"    # Global asset
  # OR for page bundle:
  # image: "hero.jpg"
  
  alt: "Project screenshot"
  caption: "The completed robot assembly"
  relative: false    # Set true if using page bundle path
---
```

**Key:** `relative: false` for `/img/` paths, `relative: true` (or omit) for page bundles.

---

## Summary

| Do This | Not This |
|---------|----------|
| `assets/img/file.png` → reference as `/img/file.png` | ❌ `./assets/img/file.png` |
| Page bundle `file.png` → reference as `file.png` | ❌ `/img/file.png` |
| `static/file.pdf` → reference as `/file.pdf` | ❌ `./static/file.pdf` |

**The rule:** `assets/` and `static/` disappear from URLs. Hugo mounts them at root.