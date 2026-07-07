# AEM Edge Delivery Services + Universal Editor Setup Guide

**Project:** AEM Boilerplate XWalk with AEM Cloud Sandbox  
**Author:** m-abhishek2027  
**Last updated:** July 2026  
**Audience:** Developers setting up EDS + Universal Editor for the first time

---

## What You Are Building

This guide connects three systems:

| System | Purpose |
|--------|---------|
| **GitHub** | Stores front-end code (blocks, CSS, JS) |
| **AEM Cloud Sandbox** | Stores content (pages, assets) |
| **Edge Delivery Services (EDS)** | Delivers fast preview (`.aem.page`) and live (`.aem.live`) sites |

**Universal Editor** lets authors edit pages visually in AEM. Code changes go through GitHub; content changes go through AEM.

```
GitHub (code)  ──►  AEM Code Sync  ──►  EDS Preview/Live
AEM Sandbox (content)  ──►  fstab.yaml  ──►  EDS Preview/Live
Universal Editor  ──►  edits content in AEM  ──►  Publish to EDS
```

---

## Prerequisites

Before you start, make sure you have:

- [ ] GitHub account
- [ ] Git installed on your machine
- [ ] Node.js 20+ and npm installed
- [ ] Google Chrome (recommended for Sidekick)
- [ ] **AEM as a Cloud Service sandbox** with Author access
- [ ] Adobe ID that works for both AEM sandbox and EDS tools

---

## URLs Reference (Example Project)

Replace values if your org/repo names differ.

| Item | Value |
|------|-------|
| GitHub repo | `https://github.com/m-abhishek2027/aem-boilerplate-xwalk` |
| AEM Author | `https://author-p139816-e1765605.adobeaemcloud.com` |
| Preview site | `https://main--aem-boilerplate-xwalk--m-abhishek2027.aem.page/` |
| Live site | `https://main--aem-boilerplate-xwalk--m-abhishek2027.aem.live/` |
| GitHub org | `m-abhishek2027` |
| GitHub repo name | `aem-boilerplate-xwalk` |

---

## Phase 1 — Create and Populate GitHub Repository

### Step 1.1 — Create empty GitHub repo

1. Go to GitHub → **New repository**
2. Name it: `aem-boilerplate-xwalk`
3. Set visibility to **Public** (recommended for EDS)
4. Do **not** add README, .gitignore, or license (keep it empty)
5. Click **Create repository**

> **Important:** Do not use plain `aem-boilerplate` for Universal Editor. Use **`aem-boilerplate-xwalk`**.

### Step 1.2 — Import official Adobe template code

An empty repo has no blocks, no `fstab.yaml`, and no component definitions. Import code from Adobe:

```powershell
cd D:\eds
git clone https://github.com/m-abhishek2027/aem-boilerplate-xwalk.git
cd aem-boilerplate-xwalk

git remote add template https://github.com/adobe-rnd/aem-boilerplate-xwalk.git
git fetch template main
git checkout -f -B main template/main
git push -u origin main
```

Verify on GitHub that these files exist:

- `fstab.yaml`
- `paths.json` (add in Phase 3 if missing — see below)
- `component-definition.json`
- `component-models.json`
- `component-filters.json`
- `xwalk.json`
- `blocks/`, `models/`, `scripts/`, `styles/`

### Step 1.3 — Local install

```powershell
cd D:\eds\aem-boilerplate-xwalk
npm install
npm run lint
```

---

## Phase 2 — Connect AEM Sandbox to GitHub (fstab.yaml)

### Step 2.1 — Get your AEM Author URL

1. Log in to AEM Cloud sandbox
2. Copy Author URL from browser, e.g.:
   ```
   https://author-p139816-e1765605.adobeaemcloud.com
   ```

### Step 2.2 — Edit fstab.yaml

Open `fstab.yaml` in repo root and update:

```yaml
mountpoints:
  /:
    url: "https://author-p139816-e1765605.adobeaemcloud.com/bin/franklin.delivery/m-abhishek2027/aem-boilerplate-xwalk/main"
    type: "markup"
    suffix: ".html"
```

**Replace:**

- `author-p139816-e1765605` → your Author host
- `m-abhishek2027` → your GitHub org/username
- `aem-boilerplate-xwalk` → your repo name

### Step 2.3 — Push to GitHub

```powershell
git add fstab.yaml
git commit -m "Connect AEM sandbox author as content source"
git push origin main
```

> **Tip:** Update `fstab.yaml` before or right after installing AEM Code Sync. If Code Sync ran first, you can fix content source later in [Site Admin](https://tools.aem.live/tools/site-admin/index.html).

---

## Phase 3 — Install AEM Code Sync

### Step 3.1 — Install GitHub App

1. Open: https://github.com/apps/aem-code-sync/installations/new
2. Choose **Only select repositories**
3. Select **`aem-boilerplate-xwalk`**
4. Click **Save**

### Step 3.2 — Confirm registration

You should see a confirmation page:

- "AEM Code Sync registered"
- Repository: `your-org / aem-boilerplate-xwalk`
- Admin email assigned

If you see *"We were not able to determine the user..."*, complete Phase 4 first.

---

## Phase 4 — EDS User Admin (Grant Access)

### Step 4.1 — Open User Admin

1. Go to: https://tools.aem.live/tools/user-admin/index.html
2. A **Projects** popup appears → select your project → **Sign in**
3. Allow pop-ups if browser blocks them
4. Sign in with the same Adobe ID used for AEM sandbox

### Step 4.2 — Add Admin user

1. **Organization:** `m-abhishek2027`
2. **Site:** `aem-boilerplate-xwalk`
3. Click **Fetch Users**
4. If 0 users → click **+ Add User(s)**
5. Enter your AEM email (e.g. `you@gmail.com`)
6. Role: **Admin**
7. Click **Add**

You should see **1 user** with **admin** badge.

---

## Phase 5 — Add paths.json (Required for Universal Editor)

Universal Editor loads path mapping from:

```
https://main--aem-boilerplate-xwalk--m-abhishek2027.aem.page/paths.json
```

If this file is missing, UE shows:

> `Path Mapping config .../paths.json could not be loaded: 404 Not Found`

### Step 5.1 — Create paths.json in repo root

```json
{
  "mappings": [
    "/content/aem-boilerplate-xwalk/:/"
  ],
  "includes": [
    "/content/aem-boilerplate-xwalk/",
    "/content/dam/aem-boilerplate-xwalk/"
  ]
}
```

> **Note:** `/content/aem-boilerplate-xwalk/` must match your **AEM site name** created in Phase 6.

### Step 5.2 — Push

```powershell
git add paths.json
git commit -m "Add paths.json for Universal Editor path mapping"
git push origin main
```

### Step 5.3 — Verify

Open in browser (wait 2–3 min after push):

```
https://main--aem-boilerplate-xwalk--m-abhishek2027.aem.page/paths.json
```

JSON should display (not 404).

**Alternative:** Configure paths via https://tools.aem.live/tools/admin-edit/index.html

---

## Phase 6 — Create AEM Site in Sandbox

### Step 6.1 — Download site template

1. Go to: https://github.com/adobe-rnd/aem-boilerplate-xwalk/releases
2. Download latest: `aem-sites-with-edge-delivery-services-template-0.2.0.zip`

### Step 6.2 — Import template in AEM (one time only)

1. Log in to AEM Author: `https://author-p139816-e1765605.adobeaemcloud.com`
2. Open **Sites** console
3. Click **Create** → **Site from template**
4. Go to **Import** tab
5. Upload the ZIP file
6. Template appears in list after import

### Step 6.3 — Create site

1. Select imported template → **Next**
2. Fill in:

| Field | Value |
|-------|-------|
| Site title | `AEM XWalk Boilerplate` (any display name) |
| Site name | `aem-boilerplate-xwalk` (must match paths.json) |
| **Project URL** | `https://main--aem-boilerplate-xwalk--m-abhishek2027.aem.page` |

3. Click **Create** → **OK**

> **Critical:** Project URL must exactly match your EDS preview URL pattern:  
> `https://main--{repo}--{org}.aem.page`

---

## Phase 7 — Universal Editor

### Step 7.1 — Open editor

1. AEM Sites → open site `aem-boilerplate-xwalk`
2. Select **`index`** page
3. Click **Edit** in toolbar
4. Universal Editor opens in new tab
5. Click **Sign in with Adobe** if prompted

### Step 7.2 — Edit content

- Add components (Cards, Hero, Columns, etc.)
- Edit text and images in Properties panel
- Changes save to AEM content

### Step 7.3 — If UE shows paths.json error

- Confirm `paths.json` is pushed and accessible at preview URL
- Wait 2–3 minutes for EDS to sync
- Hard refresh UE (Ctrl + Shift + R)

---

## Phase 8 — Technical Account & Publish

### Step 8.1 — Get technical account ID

1. AEM → **Tools** → **Cloud Services** → **Edge Delivery Services Configuration**
2. Select your site configuration → **Properties**
3. Open **Authentication** tab
4. Copy **Technical Account ID** (e.g. `xxxxx@techacct.adobe.com`)

### Step 8.2 — Add technical account in User Admin

1. https://tools.aem.live/tools/user-admin/index.html
2. Org: `m-abhishek2027`, Site: `aem-boilerplate-xwalk`
3. **+ Add User(s)** → paste technical account email
4. Role: **Config Admin**
5. Click **Add**

### Step 8.3 — Publish from Universal Editor

1. In Universal Editor → click **Publish**
2. Destination: **Preview**
3. Click **Publish**

### Step 8.4 — Verify preview

```
https://main--aem-boilerplate-xwalk--m-abhishek2027.aem.page/
```

Page should load with your content and GitHub code styling.

For **live** environment, publish to **Live** from UE or Sidekick:

```
https://main--aem-boilerplate-xwalk--m-abhishek2027.aem.live/
```

---

## Phase 9 — Local Development

### Step 9.1 — Start AEM CLI

```powershell
cd D:\eds\aem-boilerplate-xwalk
npx @adobe/aem-cli up
```

- Opens: `http://localhost:3000/`
- Proxies content from preview URL after publish
- Code changes in `blocks/`, `styles/`, `scripts/` reload live

### Step 9.2 — Local draft content (optional)

When AEM content is not ready, use `drafts/` folder:

1. Create `drafts/index.plain.html` with section/block markup
2. Run:

```powershell
npx @adobe/aem-cli up --html-folder drafts
```

3. Open: `http://localhost:3000/drafts/`

### Step 9.3 — Daily dev workflow

```
1. Edit code locally → test on localhost:3000
2. npm run lint
3. git add . && git commit -m "..." && git push
4. Wait 2–5 min → check preview URL
5. Edit content in Universal Editor → Publish
```

---

## Phase 10 — Sidekick (Optional but Recommended)

### Install

1. Chrome Web Store: [AEM Sidekick](https://chromewebstore.google.com/detail/aem-sidekick/igkmdomcgoebiipaifhmpfjhbjccggml)
2. Click **Add to Chrome**
3. Pin extension in toolbar

### Use

1. Open preview URL in Chrome
2. Click Sidekick icon → **Add this project**
3. Use **Preview** / **Publish** / **Edit** from toolbar

---

## Complete Checklist

```
Phase 1 — GitHub
[ ] Empty repo created: aem-boilerplate-xwalk
[ ] Adobe template code imported and pushed
[ ] npm install successful

Phase 2 — fstab.yaml
[ ] Author URL identified
[ ] fstab.yaml updated and pushed

Phase 3 — Code Sync
[ ] AEM Code Sync installed on repo
[ ] Registration confirmation received

Phase 4 — User Admin
[ ] Signed in to project on tools.aem.live
[ ] Admin user added

Phase 5 — paths.json
[ ] paths.json created with correct AEM site path
[ ] paths.json accessible at preview URL (not 404)

Phase 6 — AEM Site
[ ] Site template ZIP downloaded
[ ] Template imported in AEM
[ ] Site created with correct Project URL

Phase 7 — Universal Editor
[ ] index page opens in UE without errors
[ ] Content editable

Phase 8 — Publish
[ ] Technical account added as Config Admin
[ ] Preview publish successful
[ ] Preview URL loads page

Phase 9 — Local Dev
[ ] aem-cli up works on localhost:3000

Phase 10 — Sidekick (optional)
[ ] Sidekick installed and project added
```

---

## Troubleshooting

| Problem | Likely cause | Fix |
|---------|--------------|-----|
| Preview URL 404 | No content published yet | Publish from UE; check fstab.yaml |
| `paths.json` 404 in UE | File missing from repo | Add paths.json, push, wait 2–3 min |
| UE "Couldn't connect to page" | paths.json or permissions | Verify paths.json URL; check Admin role |
| Publish fails | Technical account not configured | Add tech account as Config Admin |
| Code changes not on preview | Not pushed to GitHub | `git push`; wait for Code Sync |
| `npm install` errors on Windows | ESLint/line ending issues | Use LF line endings; check package versions |
| Empty repo after clone | Manual empty repo created | Import from `adobe-rnd/aem-boilerplate-xwalk` |
| Wrong site path in paths.json | Site name mismatch | Match AEM site name exactly |

---

## Key Files Explained

| File | Purpose |
|------|---------|
| `fstab.yaml` | Links EDS to AEM Author content API |
| `paths.json` | Maps AEM content paths to public URLs (UE needs this) |
| `component-definition.json` | Defines blocks for Universal Editor |
| `component-models.json` | Field models for each block |
| `component-filters.json` | Which blocks allowed in which areas |
| `xwalk.json` | XWalk feature flags |
| `head.html` | Injected into every page `<head>` |
| `blocks/*` | Block CSS/JS (styling + decoration logic) |

---

## Official References

- [Universal Editor Tutorial](https://www.aem.live/developer/ue-tutorial)
- [AEM Authoring Docs](https://www.aem.live/docs/aem-authoring)
- [Path Mapping](https://www.aem.live/developer/authoring-path-mapping)
- [AEM CLI Reference](https://www.aem.live/developer/cli-reference)
- [XWalk Template](https://github.com/adobe-rnd/aem-boilerplate-xwalk)
- [Site Template Releases](https://github.com/adobe-rnd/aem-boilerplate-xwalk/releases)
- [User Admin Tool](https://tools.aem.live/tools/user-admin/index.html)
- [Site Admin Tool](https://tools.aem.live/tools/site-admin/index.html)
- [Admin Edit Tool](https://tools.aem.live/tools/admin-edit/index.html)

---

## Quick Command Reference

```powershell
# Clone
git clone https://github.com/m-abhishek2027/aem-boilerplate-xwalk.git
cd aem-boilerplate-xwalk
npm install

# Push changes
git add .
git commit -m "Your message"
git push origin main

# Local dev (proxy preview)
npx @adobe/aem-cli up

# Local dev (draft HTML)
npx @adobe/aem-cli up --html-folder drafts

# Lint
npm run lint
```

---

*End of guide*
