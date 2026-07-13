---
title: Markdown‑First Workflow (for Engineers)
date: 2026-03-31
description: "A markdown-first workflow for engineers: writing, slides, and documentation from plain text you actually own."
keywords: ["markdown workflow", "engineering documentation", "Marp slides", "plain text writing"]

---

# Markdown‑First Workflow (for Engineers)

This is a professional, **Markdown‑first** authoring workflow for technical documentation, lectures, blogs, and slides.  
All content is written in Markdown, and outputs (PDF, HTML, slides, static site) are generated automatically.

## 1. Core tools

- **VS Code** – main editor.  
- **Markdown** – single source format for docs, lectures, blog posts.  
- **LaTeX** – inline and block math inside Markdown.  
- **Mermaid** – diagrams inside code blocks.  
- **Marp** – Markdown‑based slides.  
- **Hugo** – static site generator for SEO‑friendly blog / docs.  
- **Pandoc** + **markdown-pdf** – PDF / HTML / DOCX generation from Markdown.

***

## 2. Writing layer (what you type)

All content is written in Markdown:

```markdown
$$
\ddot{x} + 2\zeta\omega_n\dot{x} + \omega_n^2 x = 0
$$
```
{{<katex>}}
\(\ddot{x} + 2\zeta\omega_n\dot{x} + \omega_n^2 x = 0\)

Diagrams:

```mermaid
graph LR
  A[FIXED] --> B[Mermaid Works]
```
{{<mermaid>}}
graph LR
  A[FIXED] --> B[Mermaid Works]
{{</mermaid>}}


Slides (Marp):

```markdown
---
marp: true
theme: default
paginate: true
math: mathjax
---

# Lecture 1 – Introduction to Robotics
```

***

## 3. Editor setup (VS Code)

Install these (or similar):

- **Marp extension** – live preview + export of slides.  
- **Mermaid extensions**  
  - `Mermaid Chart` or `Mermaid Graphical Editor`  
  - `vscode-mermaid-preview` / `markdown-mermaid`  
- **Latex‑related**  
  - `LaTeX Workshop`  
- **Pandoc + PDF**  
  - `vscode-pandoc`  
  - `markdown-pdf`  
- **Markdown enhancements**  
  - `markdown-all-in-one`  
  - `markdownlint`  

You can:

- Edit Markdown + LaTeX + Mermaid in one pane.  
- Open a **Mermaid‑editing webview** attached to any `mermaid` block.  
- Preview Mermaid and math in the editor or side panel.

***

## 4. Outputs (what gets published)

From the same Markdown, generate:

- **Static site / blog**  
  - Hugo builds SEO‑friendly static pages from Markdown + LaTeX + Mermaid.
- **PDFs**  
  - `markdown-pdf` → quick Chromium‑style PDFs.  
  - `pandoc` + LaTeX backend → high‑quality PDFs (for reports, notes, books).  
- **DOCX / other formats**  
  - `pandoc` can export to DOCX, EPUB, HTML, etc.
- **Slides**  
  - Marp exports the same Markdown file as PDF / HTML slides.

***

## 5. Folder structure (example)

```text
project/
├── docs/             # docs, guides, tutorials
├── lectures/         # Markdown lecture notes
├── slides/           # Marp decks
├── blog/             # Hugo content (or Hugo subdir)
├── pdf/              # output PDFs
├── html/             # output HTML versions
└── .vscode/          # workspace settings + tasks
```

- Git‑native, repo‑first.  
- All source is Markdown + LaTeX + Mermaid.  
- Generated outputs are not hand‑edited.

***

## 6. Bilingual + SEO (Hugo)

- Use Hugo with `en` / `ar` (or other languages):  
  ```text
  content/en/posts/...
  content/ar/posts/...
  ```
- In Hugo templates, add:
  - `canonical` URL  
  - `hreflang="en"` / `hreflang="ar"`  
- This gives you SEO‑friendly static pages with proper language signals.

***

## 7. One‑command workflow

You can automate with `just` / `make` / scripts:

```bash
just hugo            # build Hugo site
just pdf file.md     # build PDF from Markdown
just slideshow.md    # export Marp deck to PDF
```

Plus CI/CD (e.g., GitHub Actions) that:

- Builds Hugo site  
- Runs `pandoc` / `markdown-pdf`  
- Deploys static site to GitHub Pages / Netlify.

***

## 8. Key philosophy

- **One source format**: Markdown + LaTeX + Mermaid.  
- **Many outputs**: HTML, PDF, DOCX, slides, static site.  
- **Git‑first**: all content is in version‑controlled Markdown.  
- **Tool‑agnostic style**: you can swap Hugo ↔ others later; Markdown stays.
