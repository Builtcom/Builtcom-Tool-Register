# BUILTCOM — Tool & Asset Hire Portal

An internal staff portal for browsing tools/assets, submitting hire requests, managing approvals, invoicing, and tracking service. Single-file HTML application — no build step, no server required to get started.

![No dependencies](https://img.shields.io/badge/dependencies-none-success)
![Single file](https://img.shields.io/badge/files-1-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Features

### Staff
- **🏠 Home** — landing page with clickable category tiles (one per category), quick-search bar, and shortcut cards
- **Catalog** — search by name/code/category, sort, browse with images
- **My Requests** — track pending / approved / declined / returned hires (now with sortable columns)
- **✦ Special Request** — request items not in the catalog with PO, project code, dates, details, budget
- **All Activity** — see the whole team's hire history with search and sortable columns

### Admin (additional)
- **📊 Dashboard** — top KPIs (hires, revenue, billable, units on hire), donut chart of status breakdown, tool availability table, most-hired bar chart, plus action cards for pending approvals, due-back-this-week, service required, and uninvoiced groups. Live "Latest Activity" feed on the right panel.
- **Approvals** — detailed cards highlighting requester notes, PO number, stock check, and one-click Approve / Decline
- **Manage Tools** — full CRUD with image upload (auto-resized), daily/weekly rates, service interval tracking, test & tag fields, plus **bulk Excel/CSV import** and **Excel export**
- **Invoicing** — generate invoices from approved/returned hires with automatic cheapest-rate calculation (daily vs weekly), damage charges, tax, and print-to-PDF. Searchable by project, client, or category.
- **Users** — manage admin accounts and view staff. Send invites via shareable links with editable email templates.
- **⚙ Settings** — five sub-sections:
  - **General**: portal title/tagline, date format, auto-refresh interval, default landing per role, **tab visibility and labels** (rename or hide any tab), **form field labels** (rename Project Code, PO, Department, Quantity to your terminology)
  - **Categories**: CRUD for tool categories with custom emoji icons
  - **Templates**: company billing details, invite email template (with `{name}`, `{firstName}`, `{role}`, `{dept}`, `{inviter}`, `{link}` variables), decline-reason quick-pick templates
  - **Bulk Edit Tools**: select many tools, then apply category change, percentage rate adjustment, set rate, set service interval, flag service, mark tested-and-tagged, or bulk delete
  - **Overall Data**: data summary, JSON export/import (full backup or per-entity), and a danger zone for clearing data

### Quality of life
- **Live availability** — green / amber / red status driven by active hires
- **Smart rate calculation** for invoices — picks cheapest of (days × daily) / (weeks × weekly) / (mix of full weeks + remainder days)
- **Actual return dates** — when admin marks returned, prompts for the real return date (auto-uses today, editable) and asks if the tool needs follow-up action (repair, replacement, service, inspection) — flagging the tool automatically
- **Decline reasons** — admin selects a quick template or types a custom reason; visible to the requester in their My Requests view
- **PO number required** on every hire request and special request
- **Sortable columns** on all data tables — click headers to sort, click again to reverse
- **Stock auto-reduction** option when a tool is written off due to damage/replacement
- **Service auto-flag** — tools with a `serviceIntervalDays` set automatically appear on the dashboard service list when overdue
- **Test & tag overdue** detection — tools flagged for service when next-test-due date passes
- **CSV/Excel export** of catalog and hire activity for accounting workflows

---

## Quick Start

### Option 1 — Open locally
Just download `index.html` and open it in any modern browser.

### Option 2 — Deploy on GitHub Pages
1. Fork or clone this repository.
2. Go to **Settings → Pages**.
3. Under **Source**, select `main` branch, `/ (root)`. Save.
4. Live at `https://<your-username>.github.io/<repo-name>/` within a minute.

### Option 3 — Any static host
Drop `index.html` onto Netlify, Vercel, Cloudflare Pages, S3, or any web server. No backend required.

---

## Default Admin Credentials

For initial setup, two demo admin accounts are pre-loaded:

| Username   | Password       |
|------------|----------------|
| `admin`    | `admin123`     |
| `m.kahara` | `builtcom2026` |

**⚠️ Change these immediately** before sharing the link with anyone. In **Users → Admin Accounts**, edit each account and set a new password.

---

## Data Storage

The app uses a `window.storage` API. Two operating modes:

**1. Standalone mode (GitHub Pages, local file, any static host)**
The included polyfill maps storage to the browser's `localStorage`:
- ✅ Data persists between visits on the same browser
- ✅ Works fully offline after first load
- ❌ Data is **not shared between users** — each browser has its own copy
- ❌ ~5–10 MB total quota per origin

**2. Shared multi-user mode**
For true multi-user shared data, replace the polyfill at the top of the `<script>` block in `index.html` with calls to your own backend. The API surface is four methods:

```js
window.storage = {
  async get(key, shared) { /* → {key, value, shared} | null */ },
  async set(key, value, shared) { /* → {key, value, shared} */ },
  async delete(key, shared) { /* → {key, deleted, shared} */ },
  async list(prefix, shared) { /* → {keys, prefix, shared} */ },
};
```

Any backend that exposes these four operations works — Firebase, Supabase, Express, Cloudflare Workers + KV, etc. Shared keys are passed with `shared: true`, per-user keys with `shared: false`.

### Storage Keys

| Key                 | Scope  | Contents                                              |
|---------------------|--------|-------------------------------------------------------|
| `catalog:v2`        | shared | All tools, images, rates, service & test-tag info     |
| `requests:v2`       | shared | All hire requests, returns, decline reasons           |
| `admins:v2`         | shared | Admin accounts (username + password)                  |
| `invites:v1`        | shared | Pending and accepted invites                          |
| `invoices:v1`       | shared | Generated invoices                                    |
| `invSettings:v1`    | shared | Company details, invoice settings, **app settings**   |
| `specialRequests:v1`| shared | Off-catalog item requests                             |
| `categories:v1`     | shared | Tool categories with emoji icons                      |
| `current_user:v2`   | user   | Currently signed-in user (per browser)                |

---

## Excel Import/Export

**Export (admin → Manage Tools → ⬇ Export to Excel)** — Generates an `.xlsx` file with all tool fields including service info and test & tag dates. Falls back to CSV if SheetJS can't load (e.g. offline).

**Import (admin → Manage Tools → ⬆ Bulk Import)** — Accepts `.xlsx`, `.xls`, or `.csv`. Flexible column-name mapping (e.g. "Item Code", "Code", "ID", "SKU" all work for ID). Shows a preview before importing: how many will be added vs updated, what will be skipped due to errors. Auto-creates unknown categories. Preserves existing tool images during updates.

**Recommended workflow:** export current catalog → edit in Excel → re-import.

---

## Security Notes

This is a **prototype suitable for internal demos and small teams**. For production with real billing data:

1. **Move authentication to a backend** — passwords are stored in plain text in `admins:v2`. Use Auth0, Firebase Auth, Clerk, or your own with bcrypt.
2. **Server-side validation** — all sensitive operations (approving, generating invoices) must be re-validated on a server.
3. **Use HTTPS** — GitHub Pages provides this automatically.
4. **Invite tokens** — currently anyone with the URL can accept. Single-use enforcement is client-side only.
5. **Back up regularly** — export JSON from Settings → Overall Data. `localStorage` can be cleared by the user or browser at any time.

---

## Customisation

### Branding
- Logo lives as a base64 data URL in the `LOGO_DATA_URL` constant near the top of the `<script>` block. Replace with your own (PNG with transparent background recommended).
- Search for `BUILTCOM` to find every brand reference if you want to rebrand.

### Colors
Defined as CSS variables at the top of the `<style>` block:

```css
:root {
  --accent: #1abc9c;       /* teal — primary / brand */
  --accent-cyan: #3498db;  /* links */
  --warn: #f39c12;         /* pending / overdue */
  --danger: #e74c3c;       /* declined / errors */
  --success: #2ecc71;      /* approved / available */
  --sidebar: #2c3e50;      /* dark navy sidebar */
  --bg: #eef1f5;           /* page background */
}
```

### Initial Data
Default tools live in the `DEFAULT_CATALOG` array near the top of the `<script>`. Edit before first deploy to seed your real inventory, or use the admin UI to add items after launch. Same for `DEFAULT_CATEGORIES` and `DEFAULT_ADMINS`.

### Currency, Tax, Tab Names
All configurable via the admin **Settings** tab — no code changes needed.

---

## Browser Support

Works in any modern browser from the last 3 years (Chrome / Edge 90+, Firefox 90+, Safari 14+, mobile Safari/Chrome). Uses ES2020 (optional chaining, async/await, nullish coalescing).

---

## Roadmap / Suggestions

- Real backend with shared multi-user storage
- Availability calendar showing tool bookings over time
- Email notifications via SendGrid / Resend / SES on approve/decline
- Barcode / QR scanning for check-in/check-out
- Maintenance log per tool (service history, condition trends)
- Project-level budget tracking dashboard
- Invoice status workflow (Draft → Sent → Paid)
- Multi-language support

---

## License

MIT — see [LICENSE](./LICENSE).

---

## Credits

Single-file portal built for BUILTCOM operations. Typography: [Poppins](https://fonts.google.com/specimen/Poppins). Excel handling via [SheetJS](https://sheetjs.com) (CE, loaded on demand from jsdelivr CDN).
