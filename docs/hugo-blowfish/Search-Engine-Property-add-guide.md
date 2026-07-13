---
showInRecent: false
---


# Search Engine Property add
## google
For your setup (GitHub repo + Cloudflare Pages + `*.pages.dev` domain), here's the ranking of verification methods from best to worst:

---


- Survives theme updates
- No external dependencies

**How to implement:**

1. Go to [Google Search Console](https://search.google.com/search-console) → Add property → URL prefix → Enter `https://mulham-fetna-blog.pages.dev/`

2. Select **"HTML tag"** verification method

3. Copy the meta tag (looks like):
   ```html
   <meta name="google-site-verification" content="ABC123DEF456GHI789" />
   ```

4. In your `config.toml`, add:
   ```toml
   [verification]
     google = "ABC123DEF456GHI789"
   ```

5. Rebuild and deploy:
   ```bash
   hugo --minify
   git add .
   git commit -m "Add Google verification"
   git push
   ```

6. Back in Search Console, click **Verify**

**Done.** The meta tag appears in your `<head>` automatically via Blowfish's built-in verification support.

---

## Avoid These for Your Setup

| Method | Why Not Ideal |
|--------|---------------|
| **Google Analytics** | Requires GA4 setup, cookie banner, privacy policy updates, overkill for just verification |
| **Google Tag Manager** | Same complexity as Analytics, unnecessary overhead |
| **Domain name provider** | Only works for root domains (`example.com`), not `*.pages.dev` subdomains |

---

## Critical Note About Your Domain

Your URL is `https://mulham-fetna-blog.pages.dev/`

**Two verification paths:**

### Option A: Verify the `pages.dev` subdomain (Easier)
- Add property as **URL prefix**: `https://mulham-fetna-blog.pages.dev/`
- Use HTML tag or file method above
- **Limitation:** Only covers this specific subdomain

### Option B: Verify all `*.pages.dev` (Not Recommended)
- Would require DNS TXT record
- Cloudflare Pages doesn't give you DNS control for `pages.dev` subdomains
- **Not possible** with your current setup

### Option C: Custom Domain (Future)
If you later add a custom domain (e.g., `mulhamfetna.com`):

1. Add property as **Domain** (not URL prefix) in Search Console
2. Use **DNS TXT record** verification
3. Add TXT record in Cloudflare DNS dashboard
4. Covers all subdomains automatically (`www`, `blog`, etc.)

---

## My Recommendation for You Right Now

Use **HTML Tag** method:

```toml
# config.toml
[verification]
  google = "your-verification-code-here"
```

**Why:**
- Clean, one-line config
- Blowfish handles the rest
- No extra files to manage
- Easy to migrate if you get a custom domain later

**After verification**, submit your sitemap:
1. In Search Console → Sitemaps
2. Submit: `https://mulham-fetna-blog.pages.dev/sitemap.xml`

Hugo generates this automatically at `/sitemap.xml`.

---

## Quick Checklist

- [ ] Add Search Console property (URL prefix: your `pages.dev` URL)
- [ ] Copy meta tag verification code
- [ ] Paste into `config.toml` under `[verification]`
- [ ] Commit, push, wait for Cloudflare Pages build (~1-2 min)
- [ ] Click "Verify" in Search Console
- [ ] Submit sitemap.xml
- [ ] (Optional) Add Bing Webmaster Tools with same meta tag approach

---
***
---