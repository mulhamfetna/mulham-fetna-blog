---

showInRecent: false
---
> https://share.google/aimode/EHKLlksvJSpzMQmGa

To get this specific code block working in your Blowfish theme, follow these steps. We will use a shortcode because Hugo usually blocks `<script>` tags inside Markdown for security.
## Step 1: Create the Shortcode File
Navigate to your Hugo project folder and create a new file for the embed code.

   1. Go to layouts/shortcodes/ (create the folder if it doesn't exist).
   2. Create a file named cal-embed.html.
   3. Paste your exact code block into that file:
```html
<!-- layouts/shortcodes/cal-embed.html -->
<div style="width:100%;height:100%;overflow:scroll" id="my-cal-inline-30min"></div>
<script type="text/javascript">
  (function (C, A, L) { let p = function (a, ar) { a.q.push(ar); }; let d = C.document; C.Cal = C.Cal || function () { let cal = C.Cal; let ar = arguments; if (!cal.loaded) { cal.ns = {}; cal.q = cal.q || []; d.head.appendChild(d.createElement("script")).src = A; cal.loaded = true; } if (ar[0] === L) { const api = function () { p(api, arguments); }; const namespace = ar[1]; api.q = api.q || []; if(typeof namespace === "string"){cal.ns[namespace] = cal.ns[namespace] || api;p(cal.ns[namespace], ar);p(cal, ["initNamespace", namespace]);} else p(cal, ar); return;} p(cal, ar); }; })(window, "https://app.cal.com/embed/embed.js", "init");
  
  Cal("init", "30min", {origin:"https://app.cal.com"});

  Cal.ns["30min"]("inline", {
    elementOrSelector:"#my-cal-inline-30min",
    config: {"layout":"month_view","useSlotsViewOnSmallScreen":"true"},
    calLink: "mulham-fetna/30min",
  });

  Cal.ns["30min"]("ui", {"hideEventTypeDetails":false,"layout":"month_view"});
</script>
```
## Step 2: Add the Shortcode to your index.md
Now, tell Hugo where you want the calendar to appear on your homepage.

   1. Open content/_index.md.
   2. Insert the shortcode name (the filename you created in Step 1) using the double curly brace syntax:
```
---title: "Welcome"
---
Check out my blog posts below, or schedule a meeting with me:

{{< cal-embed >}}
```
## Step 3: Handle the "Height" issue
Cal.com embeds sometimes appear "squashed" because they rely on the parent container's height. If the calendar looks too small:

   1. Open your layouts/shortcodes/cal-embed.html again.
   2. Change the first line to include a specific minimum height (e.g., min-height: 600px):
   ```html
   <div style="width:100%; min-height:700px; overflow:scroll" id="my-cal-inline-30min"></div>
   ```
   
## Step 4: Verify Blowfish Configuration
The Blowfish theme often uses a "profile" or "hero" layout for the homepage.

* If your _index.md content isn't showing up at all, check your params.toml or hugo.toml.
* Ensure that layout = "main" or layout = "profile" allows for Markdown content to be displayed below the header.

Pro Tip: If you want to use this for different event types later, you can replace mulham-fetna/30min in the shortcode with {{ .Get "link" }} so you can reuse the same code for other pages!
Do you want to adjust the dark/light mode of the Cal.com widget to automatically sync with your Blowfish theme?


