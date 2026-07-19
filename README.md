# Portfolio IRR Tracker

A single-file, no-dependency web app for tracking your investment portfolio's
money-weighted return (IRR) across years, comparing it to benchmarks, and
projecting future scenarios.

**Privacy: all data stays on your computer.** Everything is stored in your
browser's localStorage. Nothing is ever uploaded — even if you host the page on
GitHub Pages, the *page* is public but *your data* never leaves your browser.

## Running it

- **Locally:** just double-click `index.html` (opens in Safari/Chrome), or run
  `python3 -m http.server` in this folder and visit `http://localhost:8000`.
- **GitHub Pages:** push this folder to a GitHub repo → repo Settings → Pages →
  deploy from the main branch. Your tracker will be at
  `https://<username>.github.io/<repo>/`.

> Note: localStorage is per-browser, per-URL. The local file and the GitHub
> Pages copy each have their own data. Use **Backup / Restore** (top right) to
> move data between them, and back up periodically — clearing browser data
> clears the tracker too.

## Automatic updates from GitHub

The app version lives in an `APP_VERSION` constant at the top of the script —
bump it whenever you push a new build to GitHub. The app then keeps itself
current:

- **Hosted on GitHub Pages:** nothing to configure. Once a day (and via the
  footer's "Check for updates" button) the app re-fetches its own URL, and if
  the published version is newer it shows an *Update available* banner — one
  click reloads into the new version. Your data is untouched (it lives in the
  browser, not the page).
- **Running locally:** paste the GitHub URL of `index.html` (the raw URL or
  your Pages URL) into the footer field once. When an update is found,
  "Update now" rewrites your local `index.html` in place — you pick the file
  the first time (Chrome/Edge), after that it's one click. Safari falls back
  to downloading a fresh `index.html` to replace manually.
- "Skip this version" mutes the banner until the next release. A fetched copy
  is sanity-checked (complete HTML, right app, version marker) before it is
  ever written.
- Privacy: the check is a read-only download from GitHub; nothing about your
  portfolio is sent anywhere.

## The four tabs

1. **Portfolio Overview** — headline stats (current balance, total deposits,
   total earnings, overall IRR), your IRR vs. editable benchmarks, a chart of
   your actual balances against the same cash flows grown at each benchmark's
   rate, and the year-by-year table. Every row can be locked 🔒, edited, or
   deleted (with confirmation).
2. **Data Entry** — define your accounts (401k, Roth IRA, brokerage…), pick a
   year from the dropdown, and enter each account's deposits and year-end
   balance. Totals flow automatically into the matching Overview row. Years can
   be locked here too so old data can't be fat-fingered. "Add new year" asks
   which year — **prior years (2015, 2016, …) are fine**; balances chain from
   the nearest neighbors, and gap years are handled in the IRR math. The
   **Paste balances** button fills the whole grid from spreadsheet rows
   (`Account name, deposits, ending` — tabs/commas, header row OK, matched by
   name case-insensitively, unknown accounts offered for creation), then you
   review and Save.
3. **Analysis** — project forward from your current balance (auto-filled) with
   your chosen annual return, annual deposits, deposit growth, and inflation.
   Shows 5/10/20/30-year outcomes (nominal and today's dollars), a milestone
   ("when do I hit $1M?"), and a 30-year chart of balance vs. contributions.
4. **Saved Scenarios** — snapshot any Analysis projection. Cards can be locked
   and deleted (with confirmation), and the **Expected vs. actual** section
   charts how reality is tracking against a saved projection as the years fill
   in.

## Keeping your data safe

- **Connect data file** (Chrome/Edge) — auto-saves every change to a real
  `.json` file you pick. Put it in iCloud Drive or Dropbox and it syncs across
  Macs and gets picked up by Time Machine. If the file has newer changes on
  load (edited on another Mac), the app offers to load them. Safari doesn't
  support this API — use Backup/Restore there.
- **History** — an automatic snapshot is kept for each day you change data
  (last 20). Restore any of them; the current state is snapshotted first.
- **Backup / Restore** — full JSON export/import. Backups can optionally be
  **encrypted with a passphrase** (AES-256-GCM via WebCrypto, entirely local —
  no recovery if forgotten). Restore offers **Replace** or **Merge** (merge
  adds years/accounts/scenarios you don't already have, overwrites nothing).
- **Staleness indicator** — the header shows how long since your last backup
  and turns amber past 30 days (or if you've never backed up).
- The app also requests persistent storage so the browser won't evict its data.
- Reminder: browser data is still per-browser, per-URL — the data file or
  regular backups are what make it durable.

## Analysis extras

- **Real benchmark returns** — the S&P 500 / 60-40 / bonds / cash benchmarks
  use actual calendar-year total returns (2000–2025; 2025 approximate), so the
  benchmark lives through the same crash years you did. The comparison chips
  show the money-weighted IRR each benchmark would have earned *on your exact
  cash flows*. Each benchmark can be switched back to a flat rate, which also
  fills years outside the data range.
- **Monte Carlo** — set a volatility and toggle the 10th–90th percentile band
  (400 simulations) with a median line, alongside the deterministic projection.
- **Withdrawals** — "contribute for N years, then withdraw $X/yr" (withdrawals
  rise with inflation). A depletion warning shows the year the money runs out.
- **Sensitivity table** — 30-year balances across a 5–9% return × deposit-level
  grid, your closest cell highlighted.
- **Per-account analytics** — allocation bar, per-account money-weighted IRR,
  and an account-growth chart, derived from your Data Entry history.
- **Notes** — attach a note to any year ("bought house, paused 401k") or
  scenario; shown inline.
- **Scenario overlays** — compare up to 3 saved scenarios against actuals on
  one chart.
- **CSV import** — paste `Year, Deposits, Ending` (or with a Starting column)
  rows to bulk-load history; existing years are never overwritten.
- **Print** — ⌘P prints the current tab cleanly (controls hidden).
- **Export CSV**, light/dark/auto theme, colorblind-safe palette, sample data.

## Methodology

- **Per-year IRR** solves `start·(1+r) + deposits·(1+r)^½ = ending` — deposits
  are treated as arriving evenly through the year (mid-year convention).
- **Overall IRR** is the money-weighted return: the initial balance at time 0,
  each year's deposits at that year's midpoint, and the final balance at the
  end; the discount rate that zeroes the NPV is reported, annualized.
- **Benchmark lines** apply your identical cash flows at the benchmark's
  returns — real calendar-year returns where the series has data, the flat
  rate elsewhere — so the comparison is apples-to-apples.
- **Projections** use the same mid-year convention:
  `balance = balance·(1+r) + deposits·(1+r)^½`, and in withdrawal years
  `balance = balance·(1+r) − withdrawal·(1+r)^½`.
- **Monte Carlo** draws each year's return from a normal distribution with
  your mean and volatility (floored at −95%).
