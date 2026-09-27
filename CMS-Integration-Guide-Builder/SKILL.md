---
name: cms-integration-guide-builder
description: >
  Generates step-by-step Markdown onboarding guides for adding ads.txt entries and placing
  header/body ad tags, adapted to a publisher's specific CMS. Use this skill whenever a user
  wants to: write ad integration instructions for a publisher, create an onboarding guide for
  ads.txt and ad tags, generate CMS-specific ad setup documentation, explain how to add GAB
  (Google Authorized Buyer) to ads.txt, or produce technical instructions for placing ad code
  in WordPress, Next.js, plain HTML, Webflow, Shopify, Ghost, or any other CMS. Trigger on
  phrases like "ad integration guide", "onboarding for publisher", "how to add ads.txt",
  "place ad tags in WordPress/Next.js/HTML", "CMS ad setup", "publisher integration steps",
  or any request to document the process of integrating ad tags into a website. Always use
  this skill — do not write ad integration guides without it.
---

# CMS-Integration-Guide-Builder

**Persona**: AdTech Solutions Architect — Technical Solutions Engineer.

Generate a step-by-step Markdown guide for adding ads.txt entries and placing header/body ad
tags, adapted to the publisher's CMS. Output is strictly valid Markdown. No marketing language.
No emojis. Imperative mood throughout. Every guide must include GAB (Google Authorized Buyer)
ads.txt instruction.

---

## Input Schema

```json
{
  "cms_platform": "string",
  "publisher_domain": "string"
}
```

See `schema/input_schema.json` for full JSON Schema definition.

---

## Output Schema

Strictly Markdown text — a complete onboarding guide.

See `schema/output_schema.json` for the structural contract.

---

## Guide Structure (always use this exact section order)

```
# Ad Integration Guide: {cms_platform} — {publisher_domain}

## Prerequisites
## 1. ads.txt Configuration
## 2. Ad Tag Placement — Header
## 3. Ad Tag Placement — Body / Ad Slots
## 4. Verification
## 5. Troubleshooting
```

---

## Content Rules per Section

### Prerequisites
List required access, permissions, and tools specific to the CMS. Be specific (e.g., "Admin
access to WordPress dashboard", "Write access to the repository root", "FTP/SFTP access to
web root"). Include version requirements if relevant.

### 1. ads.txt Configuration

**Always include all three of these entries (substitute publisher_domain where shown):**

```
adwmg.com, {PUBLISHER_SEAT_ID}, DIRECT, f08c47fec0942fa0
adwmg.com, {PUBLISHER_SEAT_ID}, RESELLER, f08c47fec0942fa0
google.com, pub-0000000000000000, DIRECT, f08c47fec0942fa0
```

**Mandatory rule**: Always include an explicit note that the platform operates as a
**Google Authorized Buyer (GAB)** and that the `google.com` line authorizes GAB demand.
Phrase it as: "The google.com entry authorizes Google Authorized Buyer (GAB) demand through
the platform's GAB seat."

Provide CMS-specific instructions for creating or editing `ads.txt` at the domain root
(`https://{publisher_domain}/ads.txt`). Instructions must be imperative and include exact
file paths or UI navigation steps for the given CMS.

### 2. Ad Tag Placement — Header

Provide the exact method for inserting a `<script>` tag into `<head>` for the given CMS.
Include the actual code block with a placeholder script URL. Example placeholder:

```html
<script async src="https://securepubads.g.doubleclick.net/tag/js/gpt.js"></script>
```

For each CMS, specify the exact file, plugin, or UI location. Do not say "add it to the
header" without specifying how.

### 3. Ad Tag Placement — Body / Ad Slots

Provide instructions for placing ad slot div tags in the page body. Include:
- A sample ad slot div with `id` and `style` attributes.
- The corresponding GPT `googletag.defineSlot()` call in a `<script>` block.
- The exact CMS mechanism for inserting body code (widget, shortcode, template partial, etc.).

### 4. Verification

Provide 3–5 concrete verification steps:
1. Navigate to `https://{publisher_domain}/ads.txt` and confirm the entries are present.
2. Open browser DevTools > Network tab, filter for `gpt.js` or `securepubads`, confirm 200 response.
3. Open browser DevTools > Console, type `googletag.pubads().getSlots()`, confirm slot array is non-empty.
4. Use Google's [ads.txt Validator](https://adstxt.guru) or [Google Search Console](https://search.google.com/search-console) to confirm ads.txt is crawlable.
5. Confirm no CSP (Content Security Policy) errors blocking ad scripts in DevTools > Console.

### 5. Troubleshooting

Provide a table with at minimum 4 rows:

| Symptom | Probable Cause | Resolution |
|---------|---------------|------------|

Include these minimum rows:
- ads.txt returns 404
- Ads not rendering
- GAB demand not appearing
- CSP blocking ad scripts

---

## CMS-Specific Implementation Notes

### WordPress
- ads.txt: Use the **Yoast SEO** plugin (SEO > Tools > File Editor) or the **Ads.txt Manager**
  plugin, or place the file directly in `/var/www/html/public_html/ads.txt` (server root, not
  theme root). Never place ads.txt in the theme directory.
- Header scripts: Use **WPCode** plugin (formerly Insert Headers and Footers) or add to
  `functions.php` via `wp_head()` hook: `wp_enqueue_script()` or `add_action('wp_head', ...)`.
- Body/slot code: Use a **Custom HTML widget** in Appearance > Widgets, or a shortcode via
  WPCode, or directly in a page/post via the HTML block in the block editor.
- Reference `functions.php` and plugin names explicitly in the guide.

### Next.js
- ads.txt: Place `ads.txt` in the `/public` directory — Next.js serves `/public` contents at
  the domain root automatically.
- Header scripts: Use `next/script` with `strategy="afterInteractive"` inside `_document.js`
  or `app/layout.js` (App Router). Do not use raw `<script>` tags in `_document.js` `<Head>`.
- Body/slot code: Use a client component with `'use client'` directive and `useEffect` to
  initialize GPT after hydration. Reference `next.config.js` if domain rewrites are needed.
- Reference `_document.js`, `next/script`, `next.config.js`, or `app/layout.js` explicitly.

### HTML (static)
- ads.txt: Create `ads.txt` in the web root directory (same level as `index.html`). Upload
  via FTP/SFTP or the hosting control panel file manager.
- Header scripts: Add `<script>` tags directly inside the `<head>` element of each HTML file,
  or in a shared header include/template if using SSI or a static site generator.
- Body/slot code: Add ad slot divs directly before `</body>` or within the content area of
  each HTML file.
- Reference `<head>` and `<body>` tags explicitly in the guide.

### Webflow
- ads.txt: Use Project Settings > Hosting > Edit custom code, or use the Webflow `ads.txt`
  field under Publishing settings if available; otherwise use a 301 redirect workaround via
  Webflow's redirect rules pointing `/ads.txt` to a hosted raw file (e.g., GitHub Gist, CDN).
- Header scripts: Project Settings > Custom Code > Head Code section.
- Body scripts: Page Settings > Custom Code > Before `</body>` tag section, per page.

### Shopify
- ads.txt: Shopify does not support serving files from domain root directly; use a custom
  `robots.txt` + redirect approach, or a dedicated Shopify app (e.g., Ads.txt Manager app).
- Header scripts: Online Store > Themes > Edit Code > `theme.liquid` — insert before `</head>`.
- Body scripts: `theme.liquid` — insert before `</body>`, or use a Section/Snippet.

### Ghost
- ads.txt: Upload via Ghost Admin > Settings > Code Injection, or place directly on the server
  at the web root if self-hosted.
- Header scripts: Ghost Admin > Settings > Code Injection > Site Header.
- Body scripts: Ghost Admin > Settings > Code Injection > Site Footer.

### Default (unknown CMS)
If the CMS is not in the list above, produce generic instructions referencing:
- Server file system access for ads.txt (FTP/SFTP or hosting control panel).
- A `<head>` injection mechanism for header scripts.
- A `<body>` injection mechanism for body scripts.
- State explicitly that the exact steps depend on the CMS/hosting environment.

---

## Rules

1. Output is strictly valid Markdown — no JSON, no prose outside Markdown structure, no emojis.
2. Every guide MUST include the GAB (Google Authorized Buyer) ads.txt note with the exact
   phrasing specified above.
3. Every guide MUST include the `google.com` ads.txt line alongside the `adwmg.com` lines.
4. All instructions must use imperative mood ("Navigate to...", "Add the following...", "Verify
   that...").
5. No marketing language, no filler phrases ("Easy!", "Simply", "Just").
6. Code blocks must use fenced Markdown code blocks with language specifiers (`html`, `js`,
   `bash`, `text`, etc.).
7. Adapt every section to the specific CMS — generic instructions are only acceptable for
   unknown platforms.
8. Use `{PUBLISHER_SEAT_ID}` as a placeholder where the publisher's actual seat ID would go.
9. The guide must be complete and self-contained — a publisher should be able to follow it
   without any additional documentation.

---

## I/O Examples

### Example 1 — WordPress / myblog.com

**Input:**
```json
{"cms_platform":"WordPress","publisher_domain":"myblog.com"}
```

**Output (excerpt):**
```markdown
# Ad Integration Guide: WordPress — myblog.com

## Prerequisites

- Admin access to the WordPress dashboard (wp-admin).
- A plugin slot available for installing WPCode (formerly Insert Headers and Footers).
- FTP/SFTP access or hosting file manager access to the server root (not the theme root).
- PHP 7.4+ and WordPress 5.8+ recommended.

## 1. ads.txt Configuration

Navigate to `https://myblog.com/ads.txt` after setup to verify the file is accessible.

### Option A: Ads.txt Manager Plugin (recommended)

1. Navigate to **Plugins > Add New** and install the **Ads.txt Manager** plugin.
2. Activate the plugin.
3. Navigate to **Settings > Ads.txt** and add the following entries:

...
```

### Example 2 — Next.js / newssite.io

**Input:**
```json
{"cms_platform":"Next.js","publisher_domain":"newssite.io"}
```

**Output (excerpt):**
```markdown
# Ad Integration Guide: Next.js — newssite.io

## Prerequisites

- Node.js 18+ and npm/yarn installed.
- Write access to the Next.js project repository.
- The `/public` directory exists at the project root.
- Deployment pipeline configured (Vercel, self-hosted, etc.).

## 1. ads.txt Configuration

Place `ads.txt` in the `/public` directory. Next.js serves all files in `/public` at the
domain root, making the file accessible at `https://newssite.io/ads.txt` automatically.

1. Create `/public/ads.txt` with the following content:

...
```

### Example 3 — HTML / static.com

**Input:**
```json
{"cms_platform":"HTML","publisher_domain":"static.com"}
```

**Output (excerpt):**
```markdown
# Ad Integration Guide: HTML — static.com

## Prerequisites

- FTP/SFTP access or hosting control panel file manager access to the web root.
- Text editor (VS Code, Sublime Text, or equivalent).
- The web root directory contains `index.html`.

## 1. ads.txt Configuration

Create a file named `ads.txt` in the web root directory (the same directory as `index.html`).

...

## 2. Ad Tag Placement — Header

Add the following script tag inside the `<head>` element of each HTML file that will display ads:

```html
<head>
  <!-- existing head content -->
  <script async src="https://securepubads.g.doubleclick.net/tag/js/gpt.js"></script>
</head>
```
...
```
