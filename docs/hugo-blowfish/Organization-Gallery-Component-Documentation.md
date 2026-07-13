---
showInRecent: false
---


# Organization Gallery Component Documentation

## 📋 Table of Contents
1. [Overview](#overview)
2. [Architecture & Design Principles](#architecture--design-principles)
3. [File Structure](#file-structure)
4. [Developer Manual](#developer-manual)
5. [User Guide](#user-guide)
6. [API Reference](#api-reference)
7. [Troubleshooting](#troubleshooting)

---

## Overview

The **Organization Gallery** is a reusable Hugo shortcode component designed for the Blowfish theme. It displays governmental and non-governmental organizations in a responsive grid layout with optimized image handling, SEO structured data, and accessibility features.

### Key Features
- **Data-Driven**: Organizations defined in YAML, separate from presentation
- **Image Optimization**: Automatic WebP conversion and responsive sizing
- **SEO Ready**: Schema.org structured data (JSON-LD) for rich snippets
- **Accessible**: ARIA labels, keyboard navigation, semantic HTML
- **Responsive**: Mobile-first grid layout (1/2/3 columns)
- **Maintainable**: Single source of truth, DRY principle compliance

---

## Architecture & Design Principles

### OOP Principles Applied

| Principle | Implementation |
|-----------|----------------|
| **DRY** | Organization data stored once in `data/organizations.yaml`, reused across all pages |
| **Single Responsibility** | Data layer (YAML) ↔ Logic layer (Shortcode) ↔ Presentation layer (HTML/Tailwind) |
| **Open/Closed** | New organizations added via data file without code changes |
| **Encapsulation** | Complex image processing and schema generation hidden behind simple shortcode interface |
| **Inheritance** | Leverages Blowfish's existing Tailwind classes and icon partials |

### Data Flow
```
data/organizations.yaml 
    ↓
hugo.Data (Hugo v0.156+)
    ↓
layouts/shortcodes/org-gallery.html (Processing & Logic)
    ↓
Partial: Process Images → assets/org-images/ → WebP optimization
    ↓
HTML Output with Schema.org JSON-LD
```

---

## File Structure

```bash
your-blog/
├── assets/
│   ├── css/
│   │   └── custom.css              # Optional: Styling enhancements
│   └── org-images/                 # Organization logos (source files)
│       ├── ministry-edu.jpg
│       ├── ttc.png
│       ├── neurobotics.svg
│       └── boundless.jpg
├── data/
│   └── organizations.yaml          # Organization data (single source of truth)
├── layouts/
│   └── shortcodes/
│       └── org-gallery.html        # Main component shortcode
└── content/
    └── courses/
        └── _index.md               # Usage example
```

### File Responsibilities

| File | Type | Responsibility |
|------|------|----------------|
| `organizations.yaml` | Data | Stores all organization metadata (name, logo, URL, description, location) |
| `org-gallery.html` | Template | Renders grid, processes images, generates schema markup |
| `custom.css` | Styling | Additional utilities (line-clamp, hover effects) |
| `org-images/` | Assets | Source images (PNG, JPG, SVG, WebP supported) |

---

## Developer Manual

### 1. Shortcode Implementation (`layouts/shortcodes/org-gallery.html`)

#### Core Logic Structure

**A. Parameter Handling**
```go
{{ $type := .Get "type" | default "all" }}        // Filter: governmental | non-governmental | all
{{ $title := .Get "title" | default "Partner Organizations" }}  // Section heading
```

**B. Data Aggregation (Hugo v0.156+ Compatible)**
Uses `hugo.Data` instead of deprecated `site.Data`. Merges slices using `append` (not `merge`, which only works on maps).

```go
{{ $orgs := slice }}
{{ range hugo.Data.organizations.governmental }}
  {{ $orgs = $orgs | append . }}
{{ end }}
```

**C. Image Processing Pipeline**
1. **Path Construction**: Extracts filename from data, constructs `org-images/{filename}`
2. **Resource Retrieval**: `resources.Get` pulls from `assets/` directory
3. **Optimization**: `.Fit "400x300 webp q80"` maintains aspect ratio, converts to WebP, 80% quality
4. **Responsive Attributes**: Outputs width/height for CLS prevention

```go
{{ $logoPath := printf "org-images/%s" (path.Base .logo) }}
{{ $img := resources.Get $logoPath }}
{{ $processed := $img.Fit "400x300 webp q80" }}
```

**D. Schema.org Generation**
Dynamically generates JSON-LD for SEO:
- `@type`: ItemList containing Organization entities
- Properties: name, description, url, logo, areaServed, position (for ordering)

#### CSS Architecture
- **Container**: CSS Grid (`grid-cols-1 sm:grid-cols-2 lg:grid-cols-3`)
- **Card**: Flexbox column layout with hover transform effects
- **Image**: `object-contain` preserves aspect ratio, `max-h-[200px]` prevents oversized logos
- **Accessibility**: `focus:ring-2`, `aria-labelledby`, semantic roles (`list`, `listitem`)

### 2. Data Schema (`data/organizations.yaml`)

#### YAML Structure
```yaml
governmental:        # Array of government entities
  - name: string     # Display name
    logo: string     # Filename only (e.g., "logo.jpg" not "/path/logo.jpg")
    website: URL     # External link (target="_blank" applied automatically)
    description: string   # Short bio (2-3 lines max)
    location: string # City, Country or "Remote"

non_governmental:    # Array of NGOs, companies, etc.
  # Same structure as above
```

#### Validation Rules
- **Logo**: Must exist in `assets/org-images/`. Extensions: `.jpg`, `.jpeg`, `.png`, `.svg`, `.webp`
- **Website**: Must include protocol (`https://`)
- **Name**: Required, used for alt text and fallback avatars
- **Description**: Optional, displayed below title
- **Location**: Optional, displayed with map-pin icon

### 3. CSS Enhancements (`assets/css/custom.css`)

#### Tailwind Extensions
```css
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```

**Purpose**: Limits description to 2 lines with ellipsis, preventing card height inconsistency.

#### Card Hover Effects
```css
.org-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
}
```

---

## User Guide

### Adding a New Organization

#### Step 1: Prepare Image
1. Navigate to `assets/org-images/`
2. Add logo file (recommended: 400x300px or larger, PNG with transparency or JPG)
3. **Filename rules**: Use lowercase, hyphens instead of spaces (e.g., `ministry-of-edu.jpg`)

#### Step 2: Update Data File
Edit `data/organizations.yaml`:

```yaml
governmental:
  - name: "New Organization Name"
    logo: "filename.jpg"           # Must match filename in org-images/
    website: "https://example.com" # Include https://
    description: "Brief description of collaboration or services provided."
    location: "Riyadh, Saudi Arabia"
```

**Important**: 
- Only put the filename in `logo`, not the path
- Ensure proper YAML indentation (2 spaces)
- Keep description under 150 characters for optimal display

#### Step 3: Verify
Run `hugo serve` and check:
- No WARN messages about missing images
- Card appears in correct section (governmental vs non-governmental)
- Image maintains aspect ratio (not stretched)
- Link opens correct website

### Using the Shortcode in Content

#### Basic Usage
```markdown
{{< org-gallery type="governmental" >}}
```

#### With Custom Title
```markdown
{{< org-gallery type="governmental" title="Government Partners" >}}
```

#### Display All Organizations
```markdown
{{< org-gallery type="all" title="All Collaborations" >}}
```

#### Multiple Sections on Same Page
```markdown
## Governmental Partners
{{< org-gallery type="governmental" >}}

## NGOs & Private Sector
{{< org-gallery type="non-governmental" >}}
```

### Updating Existing Organizations

1. **Change Logo**: Replace file in `assets/org-images/`, keep same filename, or update `logo:` field in YAML
2. **Update Description**: Edit text in `organizations.yaml`
3. **Remove Organization**: Delete the entire list item (all 5 lines) from YAML
4. **Reorder**: Cut and paste the entire list item block to desired position in array

---

## API Reference

### Shortcode Parameters

| Parameter | Type | Default | Options | Description |
|-----------|------|---------|---------|-------------|
| `type` | string | `"all"` | `"governmental"`, `"non-governmental"`, `"all"` | Filters which organizations to display |
| `title` | string | `"Partner Organizations"` | Any string | Section heading displayed above grid |

### Data Fields (organizations.yaml)

| Field | Required | Type | Constraints | Description |
|-------|----------|------|-------------|-------------|
| `name` | Yes | string | Max 50 chars | Organization display name |
| `logo` | No | string | Filename only | Image file in `assets/org-images/` |
| `website` | Yes | URL | Must include protocol | External link |
| `description` | No | string | Max 150 chars | Short bio/services |
| `location` | No | string | Any | Geographic location |

### Image Processing Options

Edit the shortcode to change processing:

| Method | Code | Use Case |
|--------|------|----------|
| **Fit** (Default) | `$img.Fit "400x300 webp q80"` | Fit in box, maintain ratio, letterbox if needed |
| **Resize Width** | `$img.Resize "400x webp q80"` | Set width only, height auto (uncapped) |
| **Resize Exact** | `$img.Resize "400x300 webp q80"` | Force exact dimensions (may distort) |
| **Original** | `$img.RelPermalink` | No processing, use original file |

---

## Troubleshooting

### Images Not Displaying

**Symptom**: Gray circle with first letter appears instead of logo

**Checklist**:
1. Verify file exists: `ls assets/org-images/`
2. Check filename matches exactly (case-sensitive): `logo.jpg` ≠ `logo.JPG`
3. Check console for WARN: `Image not found in assets/org-images/...`
4. Ensure file extension is supported: `.jpg`, `.jpeg`, `.png`, `.svg`, `.webp`

**Fix**:
```bash
# Clear Hugo's image cache
rm -rf resources/

# Verify file type
file assets/org-images/your-logo.jpg
```

### Images Stretched/Distorted

**Cause**: Using `Resize` instead of `Fit` or missing CSS classes

**Fix**: Ensure shortcode uses:
```html
class="max-w-full max-h-[200px] w-auto h-auto object-contain"
```

And processing uses:
```go
{{ $img.Fit "400x300 webp q80" }}
```

### Deprecation Warning: `.Site.Data`

**Error**: `WARN deprecated: .Site.Data was deprecated in Hugo v0.156.0`

**Fix**: Ensure shortcode uses `hugo.Data` not `site.Data`:
```go
{{ $orgs = hugo.Data.organizations.governmental }}
```

### Schema Not Appearing in Google

**Test**: Use [Google Rich Results Test](https://search.google.com/test/rich-results)

**Requirements**:
- Must have at least 1 organization with all fields filled
- Website URLs must be valid and accessible
- JSON-LD appears in page source (view-source:)

### Build Performance

**Issue**: Slow builds with many images

**Optimization**:
- Pre-resize images before adding to `assets/org-images/` (max 800px width)
- Reduce quality: `q70` instead of `q80`
- Skip WebP conversion for already-optimized images:
  ```go
  {{ $processed := $img.Fit "400x300" }}  // No webp conversion
  ```

---

## Best Practices

### Image Preparation
1. **Format**: Use WebP or PNG for logos with transparency
2. **Size**: Upload at 2x display size (800px for 400px display) for retina screens
3. **Optimization**: Run through [TinyPNG](https://tinypng.com/) or [Squoosh](https://squoosh.app/) before adding
4. **Naming**: `organization-name.ext` (lowercase, hyphens)

### Content Management
1. **Backup**: Keep `organizations.yaml` backed up - it's your single source of truth
2. **Version Control**: Commit both the YAML and new images in same commit
3. **Descriptions**: Write for humans, not SEO. Keep it conversational.
4. **Consistency**: Use same location format for all orgs ("City, Country")

### SEO Optimization
1. **Alt Text**: Automatically generated from `name` field - keep names descriptive
2. **External Links**: All links open in new tab with `rel="noopener noreferrer"` (security best practice)
3. **Structured Data**: Test with Google's tool after each major update

---

## Example Implementation

### Complete Working Example

**File**: `content/courses/_index.md`
```markdown
---
title: "Courses & Training"
---

I have developed educational programs in collaboration with leading institutions.

## Governmental Partners
{{< org-gallery type="governmental" title="Official Partnerships" >}}

## Industry Collaborators
{{< org-gallery type="non-governmental" title="Private Sector & NGOs" >}}
```

**Result**: Two distinct sections with responsive grids, optimized images, and full schema markup.

---

**Document Version**: 1.0  
**Compatible Hugo Version**: 0.156.0+  
**Theme**: Blowfish  
**Last Updated**: 2026-04-07

This documentation ensures the component is maintainable by future developers and usable by content editors without technical knowledge.