# TIRE+ Orleans — Sales Reports

A local, single-file web app for filtering and printing historical sales
reports from the point-of-sale CSV exports. It reads all three Workshop
exports and turns each into a filterable, print-ready report:

| Tab | Source export | What it shows |
| --- | --- | --- |
| **Sales report** | Sales Report | Invoices/credits grouped by customer, with subtotals and a grand total — the same layout as the system's own PDF |
| **Categories** | Sales Breakup Report | Every line item grouped by category (Labour, Stock, Tires, Consumables, Sublet Repair) with per-category totals and GP% |
| **Items** | Item Sales Report | Per-product sales rolled up by item code: units sold, revenue, cost, profit and margin |

![Screenshot of the app showing a demo report](docs/screenshot.png)
*Screenshot uses the built-in fictional demo data.*

## Quick start

1. Get `index.html` onto your computer (clone this repo, or download just that
   one file). No installation, no server, no internet needed.
2. Double-click `index.html` — it opens in your browser (Chrome or Edge
   recommended for printing).
3. Drag & drop your **CSV exports** onto the page (or click *Import CSV*). You
   can drop all of them at once — each file's type is detected automatically
   from its title row, and the raw exports work as-is: the app skips title
   blocks, repeated page headers, subtotal rows and "Page x of y" lines.
4. Pick a month (or a full year, a custom date range, or all data), and switch
   tabs to see transactions, categories or items for that period.
5. Click **Print report**. In the print dialog you can send it to a printer or
   *Save as PDF*.

To try it without real data, click *"…or try it with built-in demo data"* on
the start screen, or import the fictional files in [`sample-data/`](sample-data).

## Features

- **Period filters** — month picker with ‹ › navigation, full year, custom
  date range, or everything on file. The period applies to every tab.
- **Search** — customer/ref/license on the Sales tab; also item code,
  description and vehicle on Categories; item code, description and supplier
  on Items.
- **Category filter** (Categories tab) — narrow to Labour, Stock, Tires,
  Consumables or Sublet Repair, and a *Totals only* view for a one-page
  category summary.
- **Item sorting** (Items tab) — by revenue, quantity sold, profit, margin %
  or item code, so "what sells" and "what earns" are one dropdown apart.
- **Type filter** — all types, invoices only, or credits only.
- **Two layouts** on the Sales tab — grouped by customer with subtotals (like
  the original report) or a flat chronological list.
- **Summary cards** on every tab — totals for the current selection.
- **Print-ready** — A4 layout, column headers repeat on every page, and the
  browser tab title is set so *Save as PDF* suggests a sensible file name
  (e.g. `Sales Breakup Report — January 2025`).
- **Multi-year history** — import several exports (2023 through 2026 and
  beyond); rows merge so re-importing the same or overlapping exports never
  creates duplicates (the newest import wins).
- **Export filtered** — download whatever is currently on screen as a clean,
  spreadsheet-friendly CSV.
- **Remembers your data** — imports are stored in the browser's IndexedDB, so
  the reports are ready next time you open the page, even at 68,000+ rows.
  *Clear data* removes everything.

## How the numbers are computed

The app re-computes every subtotal and total from the raw rows rather than
trusting the export's own summary lines:

**Sales report tab**
- `Total = Ex HST + HST − D/C`
- `Profit = Ex HST − D/C − Cost Ex. HST`
- `GP% = Profit ÷ (Ex HST − D/C) × 100` (shown as `0.00` when the denominator is zero)

**Categories tab** — `GP% = Profit ÷ Amount × 100`, per category and overall.

**Items tab** — rows are grouped by item code; `Profit = Revenue − Cost` and
`Margin% = Profit ÷ Revenue × 100`. Revenue and cost are line totals from the
export (unit cost is per unit).

These were verified against the source system to the cent: the app's totals
match each export's own grand-total row, and its January 2025 report matches
the sample monthly PDF exactly.

### A note on the two revenue figures

The Sales report and Categories tabs are both "sales", but they do not add up
to the same number, and that is expected — it is how the source system reports
them:

- **Sales report** grand total (Ex HST) for 2023–2026 is **$4,058,198.63**.
- **Categories** (Sales Breakup) amount for the same span is **$4,057,976.73**
  — about $222 lower, because the breakup covers invoice *line items* while
  the sales report covers whole invoices.
- **Items** revenue (**$2,545,237.10**) is lower again, because it only counts
  stocked products, not labour, fees or consumables.

Use the Sales report tab for "what did we invoice", Categories for "where did
it come from", and Items for "what moved off the shelf".

## Privacy

- **Everything stays on your machine.** The app makes no network requests;
  imported data lives only in your browser's storage.
- **This repository is public — never commit real sales exports.** They contain
  customer names, license plates and financials. `.gitignore` blocks the known
  export names (`Sales_Report*.csv`, `TPWorkshop*.csv`, …) and the `data/`
  folder as a safety net; keep your real exports there or outside the repo
  entirely. Only the fictional `sample-data/` files are tracked.

## Printing tips

- Chrome/Edge print dialog → *Margins: Default* and *Scale: Default* work well
  with the built-in A4 layout.
- Enable *"Headers and footers"* in the print dialog if you want page numbers
  and the date on every page (the report itself shows "Date Printed" and the
  transaction count at the end).
- Printing *all data* on the Categories tab prints category totals only — a
  46,000-line report is not something you want to send to a printer by
  accident. Narrow the period to print line-item detail.

## Troubleshooting

- **"No report rows found…"** — the file isn't one of the three known exports.
  The app looks for a title row (`Sales Report`, `Sales Breakup Report`,
  `Item Sales Report`) or a recognizable header row.
- **"not persisted in this browser"** — the browser blocked IndexedDB (private
  browsing, or site data disabled). The reports still work for the current
  session; re-import the CSVs next time.
- **A tab looks empty** — that tab needs its own export. The tab labels show
  the row count you have loaded for each one.
