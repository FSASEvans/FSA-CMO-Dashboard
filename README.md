# FullSpeed Automotive — CMO Marketing Dashboard

**Repository:** `FSA-CMO-Dashboard` · **Version:** 2.0 · **Year:** 2026

> A single-file HTML dashboard that auto-fetches live channel data from this repository on every page load. Every user sees identical, up-to-date data — no manual uploads, no per-user file management.

---

## Table of Contents
1. [Overview](#1-overview)
2. [Data Files — Names, Structure & How to Update](#2-data-files--names-structure--how-to-update)
3. [Expected Column Structure Per Channel](#3-expected-column-structure-per-channel)
4. [How the Dashboard Works](#4-how-the-dashboard-works)
5. [Repository Structure](#5-repository-structure)
6. [Updating the Dashboard HTML](#6-updating-the-dashboard-html)
7. [GitHub Pages Setup](#7-github-pages-setup)
8. [Channel KPI Reference](#8-channel-kpi-reference)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Overview

The FSA CMO Marketing Dashboard is a self-contained HTML file (`index.html`) that aggregates performance data across all 12 marketing channels. It is served via GitHub Pages and pulls all data files directly from the `/data/` folder in this repository on every page load.

| Component | Description |
|---|---|
| **Dashboard file** | `index.html` — all CSS, JS, and HTML in one file at the repo root |
| **Data files** | `/data/` folder — 12 CSV/XLSX files, one per channel |
| **Data fetch** | `raw.githubusercontent.com` — browser fetches files directly on load, no backend |
| **Hosting** | GitHub Pages — served from the root of the `main` branch |
| **AI Analysis** | Anthropic Claude API — per-channel analysis generated on tab open |
| **Parsing library** | SheetJS (`xlsx.full.min.js`) — parses XLSX and CSV files in-browser |

> **⚠️ The repository must remain Public** for `raw.githubusercontent.com` to serve files without authentication. If the repo must be private, a GitHub Actions proxy is required.

---

## 2. Data Files — Names, Structure & How to Update

### 2.1 Expected File List

All files live in the `/data/` folder. **Filenames are hardcoded in the dashboard — do not rename them.**

| Channel | Filename | Format | Status |
|---|---|---|---|
| Paid Search | `google-ads.csv` | CSV · Google Ads export | ✅ Active |
| Web | `web-ga4.csv` | CSV · GA4 / Looker Studio | ✅ Active |
| SMS | `cinch-sms.xlsx` | XLSX · Cinch dashboard | ✅ Active |
| Email | `cinch-email.xlsx` | XLSX · Cinch dashboard | ✅ Active |
| Direct Mail | `dripdrop.xlsx` | XLSX · DripDrop export | ✅ Active |
| SEO / AI | `seo-ai.csv` | CSV · Scout / GSC export | 🟡 Stub — add file to activate |
| Listings | `listings.xlsx` | XLSX · Yext / GBP export | 🟡 Stub — add file to activate |
| Reviews | `reviews.xlsx` | XLSX · GBP / Yext / Scout | 🟡 Stub — add file to activate |
| Organic Social | `organic-social.csv` | CSV · Meta Business Suite | 🟡 Stub — add file to activate |
| CTV | `ctv.xlsx` | XLSX · Mountain vendor | 🟡 Stub — add file to activate |
| Paid Social | `paid-social.csv` | CSV · Meta Ads / agency | 🟡 Stub — add file to activate |
| Programmatic | `programmatic.csv` | CSV · DSP / agency | 🟡 Stub — add file to activate |

> **Stub channels** are handled gracefully — missing files show `○ File not yet in repo` in the status panel. The channel retains its hardcoded baseline values until the file is added. No errors, no broken dashboard.

---

### 2.2 How to Replace an Outdated Data File

This is the primary workflow for keeping the dashboard current. Do this every time you have a new export from a vendor.

1. Go to the **FSA-CMO-Dashboard** repository on GitHub.com
2. Click into the **`/data/`** folder
3. Click the file you want to replace (e.g. `google-ads.csv`)
4. Click the **pencil icon → ··· → Upload file**, or use the **Upload files** button in the folder view
5. Drag your new export file onto the upload area — GitHub will overwrite the existing file if the name matches exactly
6. Add a commit message such as:
   ```
   google-ads refresh 2026-03-05
   ```
7. Click **Commit changes**

The update is live immediately. Any user who opens the dashboard or clicks **Force Re-fetch** will pull the new file.

> **⚠️ Filename must match exactly** — including case and extension. `google-ads.CSV` will not be found; it must be `google-ads.csv`.

---

### 2.3 How to Add a Stub Channel File

When a new channel becomes active (e.g. agency onboards for Paid Social):

1. Export data from the vendor in the expected format (see [Section 3](#3-expected-column-structure-per-channel))
2. Rename the file to the exact stub filename listed in the table above (e.g. `paid-social.csv`)
3. In the repo, navigate to `/data/` → **Add file → Upload files**
4. Upload and commit — the dashboard parser activates automatically on next load

---

## 3. Expected Column Structure Per Channel

Each parser reads specific column headers. **Headers are case-sensitive.** If your vendor export uses different column names, the parser must be updated in `index.html` (see [Section 6.3](#63-updating-a-parser-for-changed-column-names)).

---

### `google-ads.csv` — Paid Search
Source: Google Ads → Reports → Campaigns. Export as CSV. The parser skips Google's title/date header rows and finds data by detecting a `Day` column.

| Column | Type | Notes |
|---|---|---|
| `Day` | Date | Used to locate the header row |
| `Campaign type` | String | Must contain `Search`, `Performance Max`, or `Demand Gen` |
| `Cost` | Currency | `$` and `,` stripped before parsing |
| `Clicks` | Integer | |
| `Store Visits` | Integer | Google-attributed store visits |

---

### `web-ga4.csv` — Web
Source: GA4 → Explore or Looker Studio export. One row per channel group.

| Column | Type | Notes |
|---|---|---|
| `Default channel group` | String | Values: `Organic Search`, `Paid Search`, etc. |
| `Sessions` | Integer | Summed per channel group |

---

### `cinch-sms.xlsx` — SMS
Source: Cinch dashboard export. May have merged cells in the Month-Year column — the parser forward-fills these automatically.

| Column | Type | Notes |
|---|---|---|
| `Month - Year` | String | e.g. `Feb 2026` — forward-filled if merged |
| `Outgoing Messages` | Integer | Total sends for the period |
| `Delivered Messages` | Integer | Used to calculate delivery rate |

---

### `cinch-email.xlsx` — Email
Source: Cinch dashboard export. One row per email template. Templates are classified as **Journey** or **Eblast** based on keywords in the Template column (`welcome`, `thank you`, `survey`, `lof`, `recommend`, `birthday`, `service`).

| Column | Type | Notes |
|---|---|---|
| `Template` | String | Keyword match determines Journey vs Eblast classification |
| `Sent` | Integer | |
| `Opened` | Integer | CTOR denominator |
| `Clicked` | Integer | CTOR numerator |

---

### `dripdrop.xlsx` — Direct Mail

| Column | Type | Notes |
|---|---|---|
| `Cards Mailed` | Integer | Total cards per row (location or campaign) |
| `Household Responses` | Integer | Summed across all rows |

---

### `seo-ai.csv` — SEO / AI

| Column | Type | Notes |
|---|---|---|
| `Date` | Date | |
| `Clicks` | Integer | From Google Search Console |
| `Impressions` | Integer | |
| `CTR` | Decimal | `0.027` or `2.7%` — parser normalizes either |
| `AI_Score` | Integer | Scout AI Visibility Score (0–100). Also accepts `AI Score` with a space. |

---

### `listings.xlsx` — Listings

| Column | Type | Notes |
|---|---|---|
| `Location` | String | Store name or ID |
| `Impressions` | Integer | GBP listing impressions |
| `Actions` | Integer | All action types combined (calls, directions, website clicks) |
| `Period` | String | Optional — display only |

---

### `reviews.xlsx` — Reviews

| Column | Type | Notes |
|---|---|---|
| `Location` | String | |
| `Total_Reviews` | Integer | Also accepts `Total Reviews` |
| `Replied` | Integer | Reviews with an owner response |
| `Star_Rating` | Decimal | Per-location avg. Also accepts `Star Rating`. Dashboard computes overall avg. |

---

### `organic-social.csv` — Organic Social
Source: Meta Business Suite export. One row per post.

| Column | Type | Notes |
|---|---|---|
| `Date` | Date | |
| `Post_Reach` | Integer | Also accepts `Reach` |
| `Reactions` | Integer | Likes, loves, etc. |
| `Comments` | Integer | |
| `Shares` | Integer | |
| `Saves` | Integer | |

---

### `ctv.xlsx` — CTV

| Column | Type | Notes |
|---|---|---|
| `Date` | Date | |
| `Impressions` | Integer | |
| `Spend` | Currency | `$` and `,` stripped |
| `Unique_HH` | Integer | Also accepts `Unique HH` |
| `URL_Conversions` | Integer | Also accepts `Conversions` |

---

### `paid-social.csv` — Paid Social

| Column | Type | Notes |
|---|---|---|
| `Date` | Date | |
| `Spend` | Currency | |
| `Impressions` | Integer | |
| `Clicks` | Integer | |
| `Video_Views` | Integer | Also accepts `Video Views` |
| `Landing_Page_Views` | Integer | Also accepts `Landing Page Views` |

---

### `programmatic.csv` — Programmatic

| Column | Type | Notes |
|---|---|---|
| `Date` | Date | |
| `Spend` | Currency | |
| `Impressions` | Integer | |
| `Clicks` | Integer | |
| `Coupon_Downloads` | Integer | Also accepts `Coupon Downloads` |

---

## 4. How the Dashboard Works

### 4.1 Auto-Fetch on Load

When the page opens, `fetchFromGitHub()` fires immediately. All 12 channel files are requested in parallel from `raw.githubusercontent.com`. The page renders with hardcoded baseline values first, then updates each channel card as files return.

### 4.2 The Fetch Flow

```
User opens dashboard URL
  ↓
Page renders with hardcoded baseline channel values
  ↓
fetchFromGitHub() fires automatically
  ↓
Promise.all() requests all 12 files in parallel
  ↓
Each file parsed by channel-specific parser → updates CHANNELS[] array
  ↓
renderKpiCards() + renderOverviewRows() re-render with live data
  ↓
Freshness indicators update with last-fetch timestamp
```

### 4.3 Missing Files

Files not yet in the repo are caught gracefully — the slot shows `○ File not yet in repo` and the channel keeps baseline values. This is the expected state for stub channels.

### 4.4 Manual Re-fetch

Click **⬡ Data Source** in the top-right header to open the status panel. Click **Force Re-fetch from GitHub** to re-run the full fetch cycle mid-session — useful after pushing a new file without closing the tab.

### 4.5 AI Analysis

Each channel tab triggers an AI-generated analysis on first open via the Anthropic API. Results are cached in-session — revisiting the tab does not re-call the API. A fresh page load generates new analysis.

### 4.6 Data Freshness Indicators

Freshness state is stored in `localStorage` and survives page reloads.

| Color | Condition | Meaning |
|---|---|---|
| 🟢 Green | 0–2 days | Data is current |
| 🟡 Yellow | 3–7 days | Aging — refresh soon |
| 🔴 Red | 8+ days | Stale — refresh needed |
| ⚫ Gray | Never fetched | No data loaded for this channel |

---

## 5. Repository Structure

```
FSA-CMO-Dashboard/
├── index.html                  ← dashboard (served by GitHub Pages)
├── README.md                   ← this file
└── data/
    ├── google-ads.csv          ← Paid Search        (active)
    ├── web-ga4.csv             ← Web / GA4          (active)
    ├── cinch-sms.xlsx          ← SMS                (active)
    ├── cinch-email.xlsx        ← Email              (active)
    ├── dripdrop.xlsx           ← Direct Mail        (active)
    ├── seo-ai.csv              ← SEO / AI           (stub)
    ├── listings.xlsx           ← Listings           (stub)
    ├── reviews.xlsx            ← Reviews            (stub)
    ├── organic-social.csv      ← Organic Social     (stub)
    ├── ctv.xlsx                ← CTV                (stub)
    ├── paid-social.csv         ← Paid Social        (stub)
    └── programmatic.csv        ← Programmatic       (stub)
```

---

## 6. Updating the Dashboard HTML

### 6.1 Hardcoded Repo Setting

The repository is set in one constant near the top of the `<script>` section in `index.html`:

```js
const GH_REPO   = 'YOUR_ORG/FSA-CMO-Dashboard';  // ← set your org/username here
const GH_BRANCH = 'main';
const GH_FOLDER = 'data';
```

If the repo is ever renamed or moved, update `GH_REPO` and commit the updated HTML.

---

### 6.2 Adding a New Channel Parser

1. Add the file to `GH_FILE_MAP`:
   ```js
   'new-channel': { file: 'new-channel.csv', type: 'csv' },
   ```
2. Write a `parseNewChannel(wb)` function — use `parseSEO` or `parsePaidSocial` as a reference template
3. Add a handler block inside `applyParsedFiles(files)` following the existing if/parse/update pattern
4. Add a slot `div` to the data status panel HTML with IDs `slot-new-channel` and `slot-status-new-channel`
5. Add the channel to `CHANNEL_SLOT_META` so its freshness pill appears in the header

---

### 6.3 Updating a Parser for Changed Column Names

Find the relevant `parse___()` function and update the column name string. Use the `||` fallback pattern to maintain backward compatibility:

```js
// Before:
totalCards += parseFloat(r['Cards Mailed']) || 0;

// After (supports both old and new column name):
totalCards += parseFloat(r['Total Cards'] || r['Cards Mailed']) || 0;
```

---

### 6.4 Deploying an Updated Dashboard

1. Edit `index.html` locally
2. Commit and push to the `main` branch
3. GitHub Pages deploys automatically within ~1 minute — no build step required
4. Verify by opening the GitHub Pages URL in a fresh tab

---

## 7. GitHub Pages Setup

> Only needed once. Skip if already configured.

1. Go to **Settings → Pages** in the FSA-CMO-Dashboard repo
2. Under **Source**, select: `Deploy from a branch` → `main` → `/ (root)`
3. Click **Save**
4. GitHub provides a URL: `https://YOUR_ORG.github.io/FSA-CMO-Dashboard/`
5. Share this URL with all dashboard users — it never changes

> **The repository must be Public** for GitHub Pages to serve at a public URL and for `/data/` files to be accessible via `raw.githubusercontent.com`.

---

## 8. Channel KPI Reference

| Channel | Primary KPI | Q1 Target | EoY Target | Owner |
|---|---|---|---|---|
| Paid Search | Store Visit Rate (SVR) | 20.0% | 25.0% | Bethany |
| Web | Sessions by Source | Q4 Baseline | Seasonality-adjusted | TBD |
| SMS | Opt-in Rate | 0.8% | 1.0% | Brian |
| Email | CTOR (Journey / Eblast) | 2.1% / 1.25% | 2.5% / 1.5% | Brian |
| Direct Mail | HH Response Rate | 2.75% (new HH) | 2.75% | Brian |
| SEO / AI | CTR + AI Visibility Score | 3.0% CTR / 95 AI | 3.5% / 97 | Bethany |
| Listings | Action Rate | 2.7% | 3.0% | Louis |
| Reviews | Star Rating + Reply % | 4.85★ | 4.85★ + 100% reply | Louis |
| Organic Social | Avg Engagement Rate | 5.0% | 6.5% | Louis |
| CTV | CPM | $50 | TBD | Bethany |
| Paid Social | CPVC / CTR | TBD | TBD | Bethany |
| Programmatic | CPC | $22.50 | $22.50 | Bethany |

---

## 9. Troubleshooting

| Symptom | Resolution |
|---|---|
| Slot shows `✗ Not found` | File is missing from `/data/` or filename doesn't match exactly (check case + extension). Paste the raw URL into a browser tab to confirm. |
| Channel shows old data after file update | Click **Force Re-fetch** in the Data Source panel, or hard refresh (`Ctrl+Shift+R` / `Cmd+Shift+R`). The browser may have cached the previous file. |
| All slots show `✗` errors | Repo is likely set to **Private**. Go to Settings → Danger Zone → Change visibility → Public. |
| AI Analysis shows "unavailable" | Transient Anthropic API issue — switch to another tab and back to retry. Check browser console for API errors if persistent. |
| KPI values didn't update after fetch | Parser could not find expected column names. Open DevTools → Console and look for `Parse error`. Compare file headers to [Section 3](#3-expected-column-structure-per-channel). |
| Freshness indicators show wrong age | `localStorage` stores the last-fetch timestamp per device. Clearing browser storage resets all indicators to gray. |
