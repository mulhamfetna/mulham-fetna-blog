---
showInRecent: false
---

# BLow fish parmas guide
I'll go through every single config in your `params.toml` in order, including all commented ones. Each explained with what it does, what happens when enabled vs disabled.

---

## 1. `colorScheme = "blowfish"`

**What it is:** The color palette for your entire site.

**How it works:** Controls accent colors, link colors, button colors, syntax highlighting themes.

**Available values:**
- `"blowfish"` - Teal/blue (default, professional tech look)
- `"avocado"` - Green (nature, embedded systems, eco-tech vibe)
- `"cherry"` - Red (bold, warnings, gaming content)
- `"ocean"` - Deep blue (corporate, trustworthy)
- `"fire"` - Orange (energetic, creative, warm)
- `"slate"` - Gray (minimalist, brutalist design)

**Example:** Change to `colorScheme = "ocean"` and your links turn blue, code blocks use blue-tinted syntax highlighting, buttons become navy.

---

## 2. `defaultAppearance = "dark"`

**What it is:** The color mode first-time visitors see.

**When enabled (set to `"dark"`):**
- New visitors see dark mode immediately
- Eyes rest, code looks better
- Preferred by developers

**When changed to `"light"`:**
- New visitors see light mode
- Better for daytime reading, photography blogs
- Some find it more "professional" for business sites

---

## 3. `autoSwitchAppearance = true`

**What it is:** Whether to respect the visitor's OS light/dark preference.

**When enabled (`true`):**
- If user's Mac/Windows is in dark mode → sees dark site
- If user's OS is in light mode → sees light site
- `defaultAppearance` only applies if OS has no preference

**When disabled (`false`):**
- Everyone sees what you set in `defaultAppearance`
- You have full control over brand experience
- OS preference is ignored

---

## 4. `enableA11y = false`

**What it is:** Accessibility (a11y = "a" + 11 letters + "y") enhancements.

**When enabled (`true`):**
- Better screen reader announcements
- Visible focus indicators (blue outline when tabbing through links)
- Skip-to-content link for keyboard users
- ARIA labels on navigation
- Respects `prefers-reduced-motion`

**When disabled (`false`):**
- Standard browser defaults
- May fail WCAG compliance audits
- Harder for visually impaired users
- Faster build (negligible difference)

**Use case:** Enable if you teach in universities, work with government, or care about inclusive design.

---

## 5. `enableSearch = true`

**What it is:** Fuse.js client-side search functionality.

**When enabled (`true`):**
- Search icon appears in header
- Clicking opens search modal
- Searches all your content instantly (no server needed)
- Works offline after first page load

**When disabled (`false`):**
- No search functionality
- Header is cleaner
- Slightly smaller JavaScript bundle

**Recommendation:** Keep `true` once you have 10+ posts. Essential for documentation sites.

---

## 6. `enableCodeCopy = true`

**What it is:** Copy button on code blocks.

**When enabled (`true`):**
- Hover over any code block → copy icon appears
- Click copies entire block to clipboard
- Shows "Copied!" toast notification

**When disabled (`false`):**
- Readers must manually select and copy code
- No visual chrome on code blocks
- Cleaner look for non-technical blogs

**Your use case:** Definitely keep `true` for STM32 code, Python snippets, Docker commands.

---

## 7. `enableStructuredBreadcrumbs = false`

**What it is:** Schema.org JSON-LD breadcrumb data for Google.

**When enabled (`true`):**
- Adds invisible structured data to HTML
- Google may show breadcrumb path in search results: `Home > Blog > Embedded > STM32`
- Helps SEO, better click-through rates

**When disabled (`false`):**
- No structured breadcrumb data
- Google shows plain URL in results
- Still have visual breadcrumbs if `showBreadcrumbs = true`

**Difference from `showBreadcrumbs`:** 
- `showBreadcrumbs` = visual navigation humans see
- `enableStructuredBreadcrumbs` = code for Googlebot only

---

## 8. `enableStyledScrollbar = true`

**What it is:** Custom CSS scrollbar vs. browser default.

**When enabled (`true`):**
- Scrollbar matches your dark/light theme
- Colors: dark gray track, lighter thumb
- Consistent across all browsers

**When disabled (`false`):**
- Uses operating system native scrollbar
- macOS: thin, transparent, auto-hide
- Windows: classic gray scrollbar
- Linux: varies by desktop environment

**Recommendation:** Keep `true` for design consistency. Disable only if you prefer native OS feel.

---

## 9. `replyByEmail = true`

**What it is:** Email reply link at bottom of articles.

**When enabled (`true`):**
- Shows envelope icon + "Reply via email" link
- Uses `email` from your `config.toml` author settings
- Opens user's default mail client

**When disabled (`false`):**
- No email contact option on posts
- Readers must find contact page manually

**Note:** Requires `params.author.email` to be set in `config.toml` or `languages.toml`.

---

## 10. `# mainSections = ["section1", "section2"]`

**What it is:** Which content sections appear on homepage "Recent Posts".

**Currently commented out:** Blowfish uses all sections with content.

**When enabled (uncommented and set):**
```toml
mainSections = ["blog", "media"]
```
- Homepage only shows posts from `blog/` and `media/`
- Projects, guides, drafts excluded from recent list
- Keeps homepage focused on writing

**When commented out (default):**
- All content sections appear
- Might mix long articles with quick notes
- Less editorial control

**Your use case:** Uncomment and set to `["blog"]` if you want homepage to only show long-form articles, not quick media posts.

---

## 11. `# robots = ""`

**What it is:** Custom robots.txt content.

**Currently commented out:** Uses Hugo default.

**When enabled (uncommented):**
```toml
robots = "User-agent: *\nDisallow: /private/\nSitemap: https://yoursite.com/sitemap.xml"
```
- Completely replaces auto-generated robots.txt
- Control exactly what search engines can crawl
- Block specific paths, set crawl delays

**When commented out:**
- Hugo generates standard permissive robots.txt
- Allows all crawlers everywhere
- Includes sitemap reference automatically

**Use case:** Only uncomment if you need to block `/admin/`, `/drafts/`, or specific user agents.

---

## 12. `disableImageOptimization = false`

**What it is:** Hugo's image processing for `static/` images.

**When enabled (set to `false`, meaning optimization IS active):**
- Hugo resizes large images to multiple sizes
- Generates WebP versions for modern browsers
- Falls back to JPEG for old browsers
- Creates responsive `srcset` attributes

**When disabled (set to `true`):**
- Images served exactly as you upload them
- No processing, faster build times
- You must manually optimize before uploading
- Risk: 5MB images slow down your site

**Trade-off:** Build time vs. performance. First build with many images is slow; subsequent builds cached.

---

## 13. `disableImageOptimizationMD = false`

**What it is:** Same as above, but specifically for images in `/content/` (Markdown files).

**When enabled (set to `false`):**
- Images referenced in `![alt](image.jpg)` get processed
- Works with page bundles (images next to .md files)

**When disabled (set to `true`):**
- Markdown images served as-is
- Faster builds for content-heavy sites
- Manual optimization required

**Difference from `disableImageOptimization`:** 
- `disableImageOptimization` = `static/` folder images
- `disableImageOptimizationMD` = `/content/` folder images

---

## 14. `disableTextInHeader = false`

**What it is:** Hides site title text if you use a logo image.

**When enabled (set to `false`, text IS shown):**
- Shows site name as text in header
- Uses `title` from `config.toml`
- Good for accessibility, SEO

**When disabled (set to `true`):**
- Removes text site name from header
- Assumes you have `logo` image configured
- Cleaner if you have stylized logo graphic

**Requires:** Must set logo image in `config.toml` or header becomes empty.

---

## 15. `# backgroundImageWidth = 1200`

**What it is:** Maximum width for background images in hero layouts.

**Currently commented out:** Uses Blowfish default (usually 1920).

**When enabled (uncommented):**
```toml
backgroundImageWidth = 1200
```
- Background images resized to 1200px wide max
- Smaller file sizes, faster loading
- May look pixelated on 4K monitors

**When commented out:**
- Uses theme default (wider, sharper)
- Larger file sizes
- Better for high-DPI displays

**Use case:** Uncomment and lower if you use `hero` or `background` homepage layout with large images and care about mobile performance.

---

## 16. `# defaultBackgroundImage = "IMAGE.jpg"`

**What it is:** Fallback background image for hero/background layouts.

**Currently commented out:** No default, must set per-page.

**When enabled (uncommented):**
```toml
defaultBackgroundImage = "images/default-bg.jpg"
```
- All pages with `layout: background` use this image
- Individual pages can override with their own `backgroundImage` frontmatter
- Consistent branding across site

**When commented out:**
- Each page must specify its own background image
- No fallback if you forget to set one
- More flexible, more work

---

## 17. `# defaultFeaturedImage = "IMAGE.jpg"`

**What it is:** Fallback image when article has no `cover.image` set.

**Currently commented out:** Articles without images have no featured image.

**When enabled (uncommented):**
```toml
defaultFeaturedImage = "images/default-post.jpg"
```
- Every article shows this image if none specified
- Consistent card appearance in lists
- Good for branding

**When commented out:**
- Articles without `cover.image` appear as text-only cards
- Cleaner, more varied layout
- Some cards have images, some don't

---

## 18. `# defaultSocialImage = "/android-chrome-512x512.png"`

**What it is:** Image used when sharing on Twitter/X, Facebook, LinkedIn.

**Currently commented out:** Uses site favicon or first article image.

**When enabled (uncommented):**
```toml
defaultSocialImage = "/images/social-card.jpg"
```
- This image appears in link previews
- Should be 1200×630px for optimal social display
- Consistent branding when people share your posts

**When commented out:**
- Blowfish tries to find article's `cover.image`
- Falls back to site icon
- Inconsistent preview images

**Important:** Create a branded image with your site name and photo for professional shares.

---

## 19. `hotlinkFeatureImage = false`

**What it is:** Use external image URLs directly without downloading.

**When enabled (set to `true`):**
- `cover.image: "https://cdn.example.com/photo.jpg"` stays as that URL
- No local processing, instant builds
- Relies on external CDN uptime
- Risk: Hotlinked images can disappear or change

**When disabled (set to `false`):**
- External images downloaded during build
- Processed and served from your domain
- Slower builds, but permanent and optimized
- No broken images if external site goes down

**Recommendation:** Keep `false` for reliability. Set `true` only if you use professional CDN (Cloudflare Images, Imgix) with permanent URLs.

---

## 20. `# imagePosition = "50% 50%"`

**What it is:** Focal point for background images.

**Currently commented out:** Centers images (50% 50%).

**When enabled (uncommented):**
```toml
imagePosition = "20% 80%"
```
- First number: horizontal position (0% = left, 100% = right)
- Second number: vertical position (0% = top, 100% = bottom)
- Keeps that point visible when image is cropped for different screen sizes

**When commented out:**
- Images centered (50% 50%)
- Faces in photos might get cropped on mobile
- Safe default, not always optimal

**Example:** For a portrait photo where face is in upper left, use `imagePosition = "30% 20%"` to keep face visible on all devices.

---

## 21. `# highlightCurrentMenuArea = true`

**What it is:** Visual indicator for which menu section you're in.

**Currently commented out:** No active state styling.

**When enabled (uncommented and `true`):**
- Current section in main menu gets bold/highlighted
- Visual feedback: "You are here"
- Helps navigation orientation

**When commented out or `false`:**
- Menu items look identical regardless of current page
- Cleaner design, less visual noise
- Users rely on page title to orient themselves

---

## 22. `# smartTOC = true`

**What it is:** Intelligent Table of Contents that highlights current section.

**Currently commented out:** Static TOC that just lists headings.

**When enabled (uncommented and `true`):**
- As you scroll, TOC highlights the section you're reading
- Visual connection between content and navigation
- Clicking TOC smoothly scrolls to section

**When commented out:**
- TOC is just a list of links
- No scroll tracking
- Less interactive but simpler

**Requires:** `showTableOfContents = true` in article settings.

---

## 23. `# smartTOCHideUnfocusedChildren = true`

**What it is:** Collapses TOC sections you're not currently reading.

**Currently commented out:** All TOC levels visible at once.

**When enabled (uncommented and `true`):**
- Only shows H2 headings by default
- Expands H3s under the H2 you're currently in
- Reduces visual clutter in long documents

**When commented out or `false`:**
- Full TOC tree visible always
- Can be overwhelming in long tutorials with many subsections
- Easier to see entire document structure at glance

**Use case:** Enable for 10,000-word technical guides with 4 levels of headings.

---

## 24. `fingerprintAlgorithm = "sha512"`

**What it is:** Hash algorithm for Subresource Integrity (SRI).

**What it does:** Generates cryptographic hashes for CSS/JS files to prevent tampering.

**Available values:**
- `"sha512"` - Most secure, longest hash (default)
- `"sha384"` - Balanced security/length
- `"sha256"` - Shortest hash, fastest, less secure (still adequate)

**Effect:** Browser verifies downloaded files match these hashes. If CDN is compromised and files modified, browser refuses to run them.

**Change only if:** You have strict content security policies requiring specific algorithms. Otherwise keep `"sha512"`.

---

## 25. `giteaDefaultServer = "https://git.fsfe.org"`

**What it is:** Default Gitea instance for social links.

**When used:** In author profile, if you set `gitea = "username"`, it links to this server.

**Change to:** Your Gitea instance, e.g., `"https://git.yourdomain.com"`

**If you don't use Gitea:** This setting does nothing, harmless to keep.

---

## 26. `forgejoDefaultServer = "https://v11.next.forgejo.org"`

**What it is:** Default Forgejo instance for social links.

**Forgejo:** A fork of Gitea, community-driven alternative.

**Same usage:** Set `forgejo = "username"` in author config, links here.

**Change to:** Your Forgejo instance if self-hosting.

---

## Header Section

```toml
[header]
  layout = "basic"
```

### `layout` options explained:

**`"basic"` (current):**
- Header scrolls away with page
- Clean, content-focused reading
- More screen space for articles
- Navigation available by scrolling up

**`"fixed"`:**
- Header always visible at top
- 60px of screen permanently occupied
- Instant access to search, menu
- Good for long articles (don't lose navigation)

**`"fixed-fill"`:**
- Fixed + solid background color
- No transparency, blocks content underneath
- Clear visual separation
- Professional, corporate feel

**`"fixed-gradient"`:**
- Fixed with gradient shadow at bottom
- Subtle fade from header to content
- Modern, polished look
- Content slightly visible under gradient edge

**`"fixed-fill-blur"`:**
- Fixed with glassmorphism effect
- Background blurred, semi-transparent
- iOS/macOS aesthetic
- Shows content scrolling behind header
- Trendy but can reduce readability

---

## Footer Section

```toml
[footer]
  showMenu = true              # Footer navigation links
  showCopyright = true         # © 2026 Your Name
  showThemeAttribution = true  # "Powered by Blowfish & Hugo"
  showAppearanceSwitcher = true # Light/dark toggle
  showScrollToTop = true       # ↑ button after scrolling
```

### Each explained:

**`showMenu = true`:**
- Shows secondary navigation in footer
- Links from `[[footer]]` menu in `config.toml`
- Useful for privacy policy, contact, colophon

**`showMenu = false`:**
- Footer has no navigation
- Cleaner, minimal
- All nav in header only

---

**`showCopyright = true`:**
- Shows "© 2026 Mulham Fetna" (uses your name from config)
- Automatic year updates
- Legal protection for your content

**`showCopyright = false`:**
- No copyright notice
- Some use Creative Commons instead in content
- Risk: content appears public domain

---

**`showThemeAttribution = true`:**
- Shows "Powered by Blowfish & Hugo" with links
- Credits theme author (Nuno Coração)
- Required by some theme licenses (MIT allows removal, but appreciated)

**`showThemeAttribution = false`:**
- No theme credit
- Looks more "professional" / less obviously templated
- Consider sponsoring theme author if you remove this

---

**`showAppearanceSwitcher = true`:**
- Moon/sun icon in footer to toggle light/dark
- Redundant if you also have it in header
- Always accessible even if header hidden

**`showAppearanceSwitcher = false`:**
- No footer toggle
- Must set appearance in header or not at all
- Cleaner footer

---

**`showScrollToTop = true`:**
- After scrolling down 300px, ↑ button appears
- Smooth scroll to top when clicked
- Essential for mobile users on long articles

**`showScrollToTop = false`:**
- No scroll-to-top button
- Users must manually scroll
- Acceptable for short pages only

---

## Homepage Section

```toml
[homepage]
  layout = "profile"
  #homepageImage = "IMAGE.jpg"
  showRecent = false
  showRecentItems = 5
  showMoreLink = false
  showMoreLinkDest = "/posts/"
  cardView = false
  cardViewScreenWidth = false
  layoutBackgroundBlur = false
  disableHeroImageFilter = false
```

### `layout` options:

**`"profile"` (current):**
- Your photo/avatar center top
- Name and bio below
- Social links row
- Recent posts section (if enabled)
- **Best for:** Personal brands, consultants, educators like you

**`"page"`:**
- Standard content page layout
- You write custom markdown in `content/_index.md`
- Full control over homepage content
- **Best for:** Landing pages, business sites, custom designs

**`"hero"`:**
- Full-width background image
- Large text overlaid on top
- Dramatic, immersive
- **Best for:** Photographers, artists, dramatic statements

**`"card"`:**
- Single large card with background image
- Content inside bordered card
- Contained, polished look
- **Best for:** Portfolio introductions, product showcases

**`"background"`:**
- Full-screen background image
- Content overlaid, minimal
- Magazine-style, high impact
- **Best for:** Visual portfolios, photography sites

**`"custom"`:**
- Uses your own `layouts/partials/home/custom.html`
- Complete control
- **You said no custom configs, so ignore this**

---

### Other homepage settings:

**`# homepageImage = "IMAGE.jpg"`:**
- Background image for `hero`, `card`, `background` layouts
- Ignored in `profile` and `page` layouts
- **Uncomment and set** if you switch to visual layout

**`showRecent = false`:**
- `true`: Shows list of recent posts on homepage
- `false`: Homepage is just profile/hero, no posts
- You have `false`, so clean profile page only

**`showRecentItems = 5`:**
- How many posts appear if `showRecent = true`
- Ignored currently since `showRecent = false`

**`showMoreLink = false`:**
- `true`: Shows "View all posts →" button
- Links to `showMoreLinkDest`
- You have `false`, so no navigation to blog from homepage

**`showMoreLinkDest = "/posts/"`:**
- Where "View all" button leads
- Should match your actual blog section name (`/blog/` not `/posts/` based on your structure)

**`cardView = false`:**
- `true`: Recent posts shown as grid cards with images
- `false`: Simple list with titles and dates
- Affects `showRecent` display style

**`cardViewScreenWidth = false`:**
- `true`: Cards span full width on mobile
- `false`: Cards have margins on mobile
- Only matters if `cardView = true`

**`layoutBackgroundBlur = false`:**
- `true`: Blurs background image in `background` layout
- Makes text more readable
- Modern glassmorphism effect
- Only for `background` layout

**`disableHeroImageFilter = false`:**
- `true`: Removes dark overlay on hero images
- `false`: Adds dark gradient so white text is readable
- Only for `hero` layout, keep `false` for readability

---

## Article Section (Blog Posts)

```toml
[article]
  showDate = true
  showViews = false
  showLikes = false
  showDateOnlyInArticle = false
  showDateUpdated = false
  showAuthor = true
  # showAuthorBottom = false
  showHero = false
  # heroStyle = "basic"
  layoutBackgroundBlur = true
  layoutBackgroundHeaderSpace = true
  showBreadcrumbs = false
  showDraftLabel = true
  showEdit = false
  # editURL = "https://github.com/username/repo/"
  editAppendPath = true
  seriesOpened = false
  showHeadingAnchors = true
  showPagination = true
  invertPagination = false
  showReadingTime = true
  showTableOfContents = false
  # showRelatedContent = false
  # relatedContentLimit = 3
  showTaxonomies = false
  showCategoryOnly = false
  showAuthorsBadges = false
  showWordCount = true
  # sharingLinks = [ ... ]
  showZenMode = false
  # externalLinkForceNewTab = false
```

### Each explained:

**`showDate = true`:**
- Shows "March 31, 2026" at top of post
- Indicates freshness/staleness of content
- Essential for technical blogs (STM32 info gets outdated)

**`showViews = false`:**
- Shows view count (e.g., "1.2k views")
- **Requires Firebase setup** (database for counters)
- Currently disabled because no Firebase configured

**`showLikes = false`:**
- Shows heart icon + like count
- **Requires Firebase setup**
- Engagement metric without comments

---

**`showDateOnlyInArticle = false`:**
- `true`: Date hidden in list views, only shows in single article
- `false`: Date shown everywhere
- Clean up list pages if you have many posts

**`showDateUpdated = false`:**
- `true`: Shows "Last updated: March 29, 2026" if you modify post
- Indicates maintained content vs. stale posts
- **Recommendation for you:** Set `true` for technical tutorials you update

**`showAuthor = true`:**
- Shows your name and avatar at top
- Good for multi-author sites, personal branding
- Links to author profile page

**`# showAuthorBottom = false`:**
- **Currently commented out**
- `true`: Author box also appears at end of article
- Good for long posts, reinforces brand after reading
- `false` (default): Author only at top

---

**`showHero = false`:**
- `true`: Enables featured image header area
- `false`: Title and meta at top, no image header
- You have `false`, so clean text-focused posts

**`# heroStyle = "basic"`:**
- **Currently commented out**
- Options: `"basic"`, `"big"`, `"background"`, `"thumbAndBackground"`
- Only works if `showHero = true`

**Hero styles explained:**
- `"basic"`: Title, then image below (standard blog)
- `"big"`: Large title, large image, more whitespace (important posts)
- `"background"`: Text overlaid on full-width image (visual impact)
- `"thumbAndBackground"`: Small thumbnail + title over background (layered)

---

**`layoutBackgroundBlur = true`:**
- Blurs background in `background` hero style
- Makes text readable over busy images
- Only applies to `heroStyle = "background"` or `"thumbAndBackground"`

**`layoutBackgroundHeaderSpace = true`:**
- Adds padding at top when using background hero
- Prevents text touching screen edge
- Only for `heroStyle = "background"`

---

**`showBreadcrumbs = false`:**
- `true`: Shows "Home > Blog > Embedded > STM32 Guide" navigation
- Helps users understand site structure
- SEO benefit (internal linking)
- **Recommendation:** Set `true` for your technical content

**`showDraftLabel = true`:**
- Shows red "DRAFT" badge on unpublished posts
- Visible in production if you accidentally deploy draft
- Safety feature, keep `true`

---

**`showEdit = false`:**
- `true`: Shows "Edit this page" link (pencil icon)
- Links to source file on GitHub/GitLab
- Encourages contributions, typo fixes

**`# editURL = "https://github.com/username/repo/"`:**
- **Currently commented out**
- Base URL for edit links
- Must set if `showEdit = true`
- Example: `"https://github.com/mulhamfetna/blog/edit/main/content/"`

**`editAppendPath = true`:**
- `true`: Automatically appends file path to `editURL`
- `false`: You must provide full URLs per-page
- Keep `true` for convenience

---

**`seriesOpened = false`:**
- `true`: Series navigation expanded by default
- `false`: Series collapsed, click to expand
- Series = grouped posts like "STM32 Course Part 1, 2, 3"

**`showHeadingAnchors = true`:**
- Shows ¶ or # icon when hovering headings
- Click to get direct link to that section
- Essential for sharing specific sections of long tutorials

---

**`showPagination = true`:**
- Shows "← Previous Post | Next Post →" at bottom
- Encourages reading more content
- Chronological navigation

**`invertPagination = false`:**
- `false`: Previous = older post, Next = newer post (standard)
- `true`: Reversed (Previous = newer, Next = older)
- Rarely changed, keep `false`

---

**`showReadingTime = true`:**
- Shows "5 min read" estimate
- Based on word count / 200 words per minute
- Helps users decide to read now or save for later

**`showTableOfContents = false`:**
- `true`: Sidebar TOC with clickable headings
- `false`: No navigation sidebar
- **Critical for your long tutorials:** Set `true` for STM32 guides

---

**`# showRelatedContent = false`:**
- **Currently commented out**
- `true`: Shows "Related Posts" at bottom
- Uses tags/categories to find similar content
- Increases page views, reduces bounce rate

**`# relatedContentLimit = 3`:**
- **Currently commented out**
- How many related posts to show
- 3 is good balance, not overwhelming

---

**`showTaxonomies = false`:**
- `true`: Shows all tags and categories at bottom of post
- `false`: Hides taxonomy links
- Navigation to similar content

**`showCategoryOnly = false`:**
- `true`: Shows only category, no tags
- `false`: Shows both (if `showTaxonomies = true`)
- Cleaner look with just categories

**Note:** Use `showTaxonomies` OR `showCategoryOnly`, not both `true`.

---

**`showAuthorsBadges = false`:**
- `true`: Shows multiple author avatars for co-authored posts
- `false`: Single author only
- For team blogs, keep `false` for personal site

**`showWordCount = true`:**
- Shows "1,234 words" 
- Alternative to reading time
- Some readers prefer word count metric

---

**`# sharingLinks = [ "linkedin", "twitter", ... ]`:**
- **Currently commented out**
- Array of social platforms for share buttons
- Options: `linkedin`, `twitter`, `bluesky`, `mastodon`, `reddit`, `pinterest`, `facebook`, `email`, `whatsapp`, `telegram`, `line`

**When enabled:**
```toml
sharingLinks = ["linkedin", "twitter", "email"]
```
- Shows icons for these platforms
- Pre-filled share text with article title and URL

**When commented out:**
- No sharing buttons
- Readers copy URL manually
- Cleaner design, less tracking

---

**`showZenMode = false`:**
- `true`: Adds "Zen Mode" button (focus icon)
- Hides everything except article text
- No header, footer, sidebars
- Distraction-free reading
- Press ESC to exit

**`# externalLinkForceNewTab = false`:**
- **Currently commented out**
- `true`: All external links open in new tab
- `false`: External links open in same tab
- Default `true` (safe to keep commented, that's the behavior)

---

## List Section (Blog Index Pages)

```toml
[list]
  showHero = false
  # heroStyle = "background"
  layoutBackgroundBlur = true
  layoutBackgroundHeaderSpace = true
  showBreadcrumbs = false
  showSummary = false
  showViews = false
  showLikes = false
  showTableOfContents = false
  showCards = false
  orderByWeight = false
  groupByYear = true
  cardView = false
  cardViewScreenWidth = false
  constrainItemsWidth = false
```

### Each explained:

**`showHero = false`:**
- `true`: Featured image at top of list pages (blog index)
- `false`: Simple title "Blog" or "Posts"
- Visual impact vs. clean listing

**`# heroStyle = "background"`:**
- **Currently commented out**
- Same options as article heroStyle
- Only works if `showHero = true`

---

**`layoutBackgroundBlur = true`:**
- Blur effect for list page hero backgrounds
- Same as article setting

**`layoutBackgroundHeaderSpace = true`:**
- Padding for list page hero
- Same as article setting

---

**`showBreadcrumbs = false`:**
- Navigation on list pages
- Usually "Home > Blog" (simple, often unnecessary)
- Set `true` if you have deep category hierarchies

**`showSummary = false`:**
- `true`: Shows article excerpt/description in list
- `false`: Just title and meta
- `true` recommended for your media section (microblog style)

---

**`showViews = false`:**
- View counts on list items
- Requires Firebase

**`showLikes = false`:**
- Like counts on list items
- Requires Firebase

**`showTableOfContents = false`:**
- TOC on list pages (rarely useful)
- Keep `false`

---

**`showCards = false`:**
- `true`: Card-based layout (image + title + excerpt)
- `false`: Simple list (title, date, meta)
- Visual vs. information density

**`orderByWeight = false`:**
- `true`: Sort by `weight` frontmatter (manual order)
- `false`: Sort by date (newest first)
- Use `weight` for "Start Here" or pinned posts

---

**`groupByYear = true`:**
- `true`: "2026", "2025" year headers in list
- `false`: Continuous chronological stream
- Archive feel vs. social media feed feel

**`cardView = false`:**
- `true`: Grid layout of cards
- `false`: Vertical list
- Magazine vs. blog style

---

**`cardViewScreenWidth = false`:**
- `true`: Cards touch screen edges on mobile
- `false`: Cards have margins
- Full-bleed vs. contained

**`constrainItemsWidth = false`:**
- `true`: Max-width on list items (readable line length)
- `false`: Full width on large screens
- Typography best practice vs. modern wide design

---

## Sitemap Section

```toml
[sitemap]
  excludedKinds = ["taxonomy", "term"]
```

**`excludedKinds`:**
- Prevents tag pages and category pages from appearing in `sitemap.xml`
- Search engines focus on actual content
- Reduces "thin content" penalties

**Options to exclude:**
- `"taxonomy"` = tag/category list pages (e.g., `/tags/`)
- `"term"` = individual tag pages (e.g., `/tags/stm32/`)
- `"section"` = section list pages (e.g., `/blog/`)
- `"page"` = regular pages

**Current setting:** Excludes taxonomy and term pages from sitemap. Good for SEO—focuses on your actual articles.

---

## Taxonomy Section (Tag/Category List Pages)

```toml
[taxonomy]
  showTermCount = true
  showHero = false
  # heroStyle = "background"
  showBreadcrumbs = false
  showViews = false
  showLikes = false
  showTableOfContents = false
  cardView = false
```

### Each explained:

**`showTermCount = true`:**
- On `/tags/` page, shows "stm32 (5)" 
- Indicates how many posts use that tag
- Helps find popular topics

**`showHero = false`:**
- Featured image on tag list page
- Usually unnecessary, keep `false`

**`# heroStyle = "background"`:**
- Same options, only if `showHero = true`

---

**`showBreadcrumbs = false`:**
- "Home > Tags" navigation
- Simple enough to often skip

**`showViews = false`:**
- Total views for tag (requires Firebase)

**`showLikes = false`:**
- Total likes for tag (requires Firebase)

**`showTableOfContents = false`:**
- TOC of tags (nonsensical, keep `false`)

**`cardView = false`:**
- Grid of tag cards vs. simple list
- List is cleaner for taxonomies

---

## Term Section (Individual Tag/Category Pages)

```toml
[term]
  showHero = false
  # heroStyle = "background"
  showBreadcrumbs = false
  showViews = false
  showLikes = false
  showTableOfContents = true
  groupByYear = false
  cardView = false
  cardViewScreenWidth = false
```

### Each explained:

**`showHero = false`:**
- Featured image on individual tag page (e.g., `/tags/stm32/`)
- Unnecessary, keep `false`

**`# heroStyle = "background"`:**
- Same options, only if `showHero = true`

---

**`showBreadcrumbs = false`:**
- "Home > Tags > STM32" navigation
- Set `true` for deep navigation clarity

**`showViews = false`:**
- Total views for this tag's posts

**`showLikes = false`:**
- Total likes for this tag's posts

---

**`showTableOfContents = true`:**
- Shows TOC of posts in this tag
- Unusual use case, probably auto-set
- Can set `false` for cleaner tag pages

**`groupByYear = false`:**
- `true`: "2026", "2025" headers on tag pages
- `false`: Simple chronological list
- Consistency with main list vs. cleaner look

---

**`cardView = false`:**
- Grid layout for posts in this tag
- Same as list setting

**`cardViewScreenWidth = false`:**
- Full-width cards on mobile for tag pages
- Same as list setting

---

## Firebase Section (Analytics/Engagement)

```toml
[firebase]
  # apiKey = "XXXXXX"
  # authDomain = "XXXXXX"
  # projectId = "XXXXXX"
  # storageBucket = "XXXXXX"
  # messagingSenderId = "XXXXXX"
  # appId = "XXXXXX"
  # measurementId = "XXXXXX"
```

**What it is:** Google Firebase project credentials.

**What it enables:**
- `showViews = true` (view counters)
- `showLikes = true` (like buttons)
- Real-time engagement metrics
- Free tier: 50k reads/day, 20k writes/day

**All commented out:** Firebase features disabled. Views and likes won't show even if `showViews = true`.

**To enable:**
1. Create Firebase project at console.firebase.google.com
2. Enable Firestore Database
3. Copy web app credentials here
4. Set `showViews = true` and/or `showLikes = true`

---

## Fathom Analytics Section

```toml
[fathomAnalytics]
  # site = "ABC12345"
  # domain = "llama.yoursite.com"
```

**What it is:** Privacy-focused analytics (paid service, $14/month).

**Features:**
- No cookies, GDPR compliant
- Simple dashboard
- Track unique visitors, page views, referrers

**All commented out:** Not using Fathom.

**To enable:**
1. Sign up at usefathom.com
2. Add site, get site ID
3. Uncomment and fill in

---

## Umami Analytics Section

```toml
[umamiAnalytics]
  # websiteid = "ABC12345"
  # domain = "llama.yoursite.com"
  # dataDomains = "yoursite.com,yoursite2.com"
  # scriptName = ""
  # enableTrackEvent = true
```

**What it is:** Self-hosted, open-source analytics (free, privacy-focused).

**Features:**
- Host on your own server/VPS
- No data leaves your infrastructure
- Lightweight (< 1KB script)
- Track events (downloads, clicks)

**All commented out:** Not using Umami.

**To enable:**
1. Self-host Umami (Docker: `docker run -d --name umami -p 3000:3000 ghcr.io/umami-software/umami:postgresql-latest`)
2. Add website in Umami dashboard, get website ID
3. Uncomment and fill in

---

## Seline Analytics Section

```toml
[selineAnalytics]
  # token = "XXXXXX"
  # enableTrackEvent = true
```

**What it is:** Newer privacy-focused analytics (similar to Fathom).

**All commented out:** Not using Seline.

---

## Buy Me a Coffee Section

```toml
[buymeacoffee]
  # identifier = ""
  # globalWidget = true
  # globalWidgetMessage = "Hello"
  # globalWidgetColor = "#FFDD00"
  # globalWidgetPosition = "right"
```

**What it is:** Tip jar for creators.

**Features:**
- Floating button on your site
- Visitors can buy you a virtual coffee ($3)
- Popular with open-source developers, educators

**All commented out:** No tip button shown.

**To enable:**
1. Sign up at buymeacoffee.com
2. Get your username/identifier
3. Uncomment and configure

**Settings explained:**
- `identifier`: Your BMC username
- `globalWidget`: Show floating button
- `globalWidgetMessage`: Hover text ("Buy me a coffee?")
- `globalWidgetColor`: Button color (default yellow)
- `globalWidgetPosition`: `left` or `right` side of screen

---

## Verification Section (Search Engine Ownership)

```toml
[verification]
  # google = ""
  # bing = ""
  # pinterest = ""
  # yandex = ""
  # fediverse = ""
```

**What it is:** Prove you own the site to search engines/web services.

**Google:** 
- Get code from Google Search Console
- Enables search performance data, sitemap submission
- Format: `google = "abc123def456"` (meta tag content)

**Bing:**
- Microsoft Webmaster Tools
- Same process as Google

**Pinterest:**
- Rich pins validation
- For image-heavy content

**Yandex:**
- Russian search engine
- Only if you have Russian audience

**Fediverse:**
- Mastodon/ActivityPub verification
- Shows green checkmark on Mastodon profiles

**All commented out:** No verification tags in HTML.

---

## RSSNext Section

```toml
[rssnext]
  # feedId = ""
  # userId = ""
```

**What it is:** RSS feed analytics service.

**All commented out:** Not tracking RSS subscribers.

---

## Advertisement Section

```toml
[advertisement]
  # adsense = ""
```

**What it is:** Google AdSense publisher ID.

**All commented out:** No ads on your site.

**To enable:**
1. Sign up for AdSense
2. Get publisher ID (pub-XXXXXXXX)
3. Uncomment and fill in
4. Ads appear in article footers

---

## Summary: Critical Settings to Change for Your Site

Based on your content (technical tutorials, media posts, projects), here are the essential changes:

```toml
# For long STM32/Python tutorials
[article]
  showTableOfContents = true
  showDateUpdated = true
  showBreadcrumbs = true

# For media section (set via cascade in _index.md)
# Keep these false globally, override per-section

# For homepage
[homepage]
  showRecent = true        # Show your latest work
  showMoreLink = true      # Link to full blog

# For navigation
[header]
  layout = "fixed"         # Always accessible on long articles
```

Everything else can stay as defaults. No custom code needed.






