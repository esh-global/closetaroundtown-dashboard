# Resale Ledger

A filterable dashboard for your Poshmark, Mercari, Depop, and Vinted sales — brand, gender, type, color, ship-to destination, and days listed, plus a gender → brand → type breakdown. A **Sold / Unsold inventory** toggle at the top switches the whole page between what's sold and what's still listed, so you can track live inventory alongside sales history. Static site, no backend, no login: your data lives in eight CSV files.

## What's in here

```
index.html                page shell
styles.css                 all styling
app.js                     loads the CSVs, filters, sorts, renders
manifest.json               Add to Home Screen / PWA config
icons/                      home-screen app icons (see below)
data/poshmark.csv           your real Poshmark sold data
data/mercari.csv            your real Mercari sold data
data/poshmark-unsold.csv    your real Poshmark active listings
data/mercari-unsold.csv     sample unsold/listed rows — replace with your export
data/depop.csv               sample sold rows — replace with your export
data/vinted.csv              sample sold rows — replace with your export
data/depop-unsold.csv        sample unsold/listed rows — replace with your export
data/vinted-unsold.csv       sample unsold/listed rows — replace with your export
netlify.toml                tells Netlify this is a plain static site (no build step)
```

The eight CSVs shipped in `/data` are made-up placeholder rows so you can see the dashboard working. Every sample row has `Sample` set to `yes`; once you add real rows, a checkbox appears letting you toggle sample rows in or out (they're excluded from the numbers by default the moment real rows exist).

## Sold vs. Unsold

The **Sold** tab is your sales history — the original dashboard: revenue, items sold, avg. sale price, avg. days listed before it sold. It uses the four `data/*.csv` files and includes the Ship To filter, since only sold items have a shipping destination.

The **Unsold inventory** tab is what's currently listed and hasn't sold yet — pulled from the four `data/*-unsold.csv` files. Its KPIs are listed value, items listed, avg. list price, and avg. days listed *so far*. There's no Ship To filter here (nothing's shipped yet), and "days listed" recalculates against today's date every time you load the page — so as long as an item stays in an unsold CSV, its days-listed count keeps climbing on its own with no re-upload needed.

## Preview it locally

Opening `index.html` directly by double-clicking it won't work — browsers block a page from `fetch()`-ing local files over the `file://` protocol, and this page fetches the CSVs. Run a tiny local server from this folder instead:

```
npx serve .
```

or, with Python already installed:

```
python3 -m http.server 8000
```

then visit the URL it prints (e.g. `http://localhost:8000`).

## Add to Home Screen

Once the site is live on Netlify (has a real `https://` URL — this doesn't work on `localhost`), you can install it like an app:

- **iPhone/iPad (Safari):** open the site, tap the Share icon, then "Add to Home Screen."
- **Android (Chrome):** open the site, tap the ⋮ menu, then "Add to Home screen" or "Install app."
- **Desktop (Chrome/Edge):** an install icon appears in the address bar; click it, or use the browser's ⋮ menu → "Install Resale Ledger."

It opens in its own window without browser address bars, using the purse-and-money icon in `icons/`. This is powered by `manifest.json` and the `<link rel="apple-touch-icon">`/`<meta>` tags in `index.html` — if you ever rename the site or want a different icon, edit `manifest.json` and swap the files in `icons/` (keep the same filenames, or update the paths in both `manifest.json` and `index.html`). There's no offline caching here — every open still needs a network connection to fetch the CSVs, exactly like opening it in a normal browser tab.

## Deploy to Netlify

**Fastest — drag and drop, no account setup beyond signing in:**

1. Go to [app.netlify.com/drop](https://app.netlify.com/drop).
2. Drag this whole folder (or a zip of it) onto the page.
3. Netlify gives you a live URL immediately.

**Or, for continuous deploys from GitHub:**

1. Push this folder to a new GitHub repo (make it private if your sales numbers are sensitive).
2. In Netlify: **Add new site → Import an existing project**, pick the repo.
3. Leave the build command empty and the publish directory as `.` — `netlify.toml` already says so.
4. Every push to the repo redeploys the site automatically.

## Updating your data

Export order/sales history (for Sold) or your active listings (for Unsold inventory) as CSV from each marketplace's seller dashboard, then either:

- **Reformat to match** (most reliable): open the export in Excel/Numbers/Google Sheets, and put the columns in this order with these headers — a template `Sample` column is optional, only used to mark placeholder rows.

  For **sold** files (`poshmark.csv`, `mercari.csv`, `depop.csv`, `vinted.csv`):

  | Brand | Type | Gender | Color | Price | Ship To | Date Listed | Date Sold | Sample |
  |---|---|---|---|---|---|---|---|---|

  For **unsold** files (`poshmark-unsold.csv`, `mercari-unsold.csv`, `depop-unsold.csv`, `vinted-unsold.csv`) — no Ship To or Date Sold, since neither applies yet:

  | Brand | Type | Gender | Color | List Price | Date Listed | Sample |
  |---|---|---|---|---|---|---|

  Save as CSV, replacing the matching file in `/data` (keep the filename — `poshmark.csv` stays `poshmark.csv`, `poshmark-unsold.csv` stays `poshmark-unsold.csv`, etc.).

- **Or just drop your raw export in as-is.** The page tries to auto-detect columns by common header names — see the alias list below. If a column doesn't get picked up, you'll notice it shows as "Unknown" or blank in the dashboard; rename that column's header in the CSV to one of the recognized names and reload.

**Mercari's raw sales export is missing pieces the others have.** It has no Brand, Type, Gender, Color, or Date Listed columns — only an "Item Title" you'd need to hand-categorize, and only a sold date (so "days listed" can't be computed for Mercari rows; the dashboard shows "—" for those and leaves them out of the Avg. days listed average rather than showing a wrong number). If you're rebuilding `mercari.csv` from a fresh export, expect to fill in Brand/Type/Gender/Color yourself, or accept that those rows will show as blank in that filter.

**Sample rows only disappear automatically once real (non-sample) rows exist in that same status/platform combination** — they're identified by `Sample` = `yes`. If you replace a file with real data but rows still look off, check that `Sample` is set to `no` (or left blank) in the new rows.

Recognized header aliases (case-insensitive, partial match):

- **Brand** — `brand`, `designer`, `brand name`, `label`
- **Type** — `type`, `category`, `item category`, `item type`, `product category`
- **Gender** — `gender`, `department`, `size gender`
- **Color** — `color`, `colour`, `primary color`
- **Price** — `price`, `sale price`, `sold price`, `item price`, `order total`, `payout`, `earnings`
- **Ship To** — `ship to`, `shipping state`, `buyer state`, `state`, `country`, `buyer country`, `destination`
- **Date Listed** — `date listed`, `listed date`, `listing date`, `created date`
- **Date Sold** — `date sold`, `sold date`, `order date`, `purchase date`, `sale date`

Once your files are updated, redeploy (drag the folder in again, or push to GitHub if you set up continuous deploy) — that's the whole update cycle.

### Quick look before redeploying

The **Update my data** button on the page has a "preview only" option: pick Sold or Unsold, pick a platform, choose a CSV from your computer, and it merges into the dashboard in that browser tab only — nothing is saved or written back. Refreshing the page discards it. Use it to sanity-check a file before you commit to replacing the one in `/data`.

## Limitations to know about

- No database, no accounts, no multi-device sync — the CSVs in the deployed folder are the only source of truth, and only a redeploy changes what everyone sees.
- There's no automatic pull from Poshmark/Mercari/Depop/Vinted — none of them expose a public API for this, so exporting and dropping in a CSV is the update mechanism.
- When an item sells, move it (or re-enter it) from its `*-unsold.csv` file into the matching sold `.csv` — the two sets aren't linked automatically.
- Keep a copy of your CSVs somewhere durable (a synced folder, or commit them to a private git repo) — if you only ever edit the deployed copy, losing that deploy loses your data.
