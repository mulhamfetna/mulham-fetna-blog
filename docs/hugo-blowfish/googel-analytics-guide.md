---

showInRecent: false
---
# Google Analytics guide
> it is not google search console -> [look at this guide](./params-guide.md/#search-engine-property-add)

I need to clarify: **Google Search Console verification** ≠ **Google Analytics tracking**. These are two completely different services with different keys.

Let me explain both and where to get each:

---

## 1. Google Search Console (Already Done)

**Purpose:** Tell Google your site exists, submit sitemap, see search performance.

**Key format:** `google-site-verification` meta tag (long random string)

**Where you got it:** Google Search Console → Verification → HTML tag

**In your config:**
```toml
[verification]
  google = "YOUR_SEARCH_CONSOLE_CODE"  # You have this
```

**Status:** ✅ Already working (you verified earlier)

---

## 2. Google Analytics 4 (What You Want Now)

**Purpose:** Track visitors, page views, session duration, traffic sources.

**Key format:** `G-XXXXXXXXXX` (starts with `G-`, 10+ characters)

**Where to get it:** Completely different website → https://analytics.google.com

---

## Step-by-Step: Get Your GA4 Key

### Step 1: Create GA4 Property

1. Go to https://analytics.google.com
2. Sign in with same Google account as Search Console
3. Click **"Start measuring"** or **Admin** (gear icon) → **Create Property**
4. Property name: `Mulham Fetna Blog`
5. Time zone: Your timezone
6. Currency: Optional
7. Click **Next** → **Create**

### Step 2: Set Up Data Stream

1. Choose **"Web"** platform
2. Website URL: `https://mulham-fetna-blog.pages.dev/`
3. Stream name: `Blog Stream`
4. Click **Create stream**

### Step 3: Get Your Measurement ID

You'll see a page with:

```
Measurement ID
G-ABC123DEF0
```

**This `G-ABC123DEF0` is your key.**

### Step 4: Copy to Hugo Config

In `hugo.toml`, uncomment and fill:

```toml
googleAnalytics = "G-ABC123DEF0"  # Replace with your actual ID
```

---

## Your Config Should Have BOTH

```toml
# hugo.toml

# For search engine visibility (you have this)
[verification]
  google = "SEARCH_CONSOLE_VERIFICATION_CODE_LONG_STRING"

# For visitor tracking (add this now)
googleAnalytics = "G-YOUR_ANALYTICS_ID"
```

---

## Verify It's Working

After deploying:

1. Visit your live site
2. Open browser DevTools (F12) → Network tab
3. Filter for `google-analytics` or `gtag`
4. Reload page
5. Should see request to `https://www.googletagmanager.com/gtag/js?id=G-...`

Or check GA4 realtime reports (updates within minutes).

---

## Quick Check: Which Key Do You Have?

| If your key looks like | It's for | Action |
|------------------------|----------|--------|
| `google-site-verification=abc123...` or long random string | Search Console | Put in `[verification]` section |
| `G-ABC123DEF0` (starts with G-) | Google Analytics 4 | Put in `googleAnalytics =` |
| `UA-123456-1` (starts with UA-) | Old Universal Analytics | **Deprecated 2023**, create new GA4 property |

---

## Summary

| Service | URL | Key Format | Config Location |
|---------|-----|-----------|-----------------|
| Search Console | search.google.com/search-console | `google-site-verification` long string | `[verification] google` |
| Analytics 4 | analytics.google.com | `G-XXXXXXXXXX` | `googleAnalytics` |

**You need BOTH for complete setup:**
- Search Console = SEO visibility
- Analytics = Traffic measurement

---

Want me to check if your current `googleAnalytics` line is commented and show exactly where to add the key?



