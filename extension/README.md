# Universe Overview — Tableau Dashboard Extension

Live version of the "Universe Overview" dashboard we iterated on as a static preview
(`../preview/index.html`). Instead of a pre-baked JSON snapshot, this pulls data
directly from two Tableau worksheets at runtime and recomputes every chart
client-side (KPI totals, Universe/Product-Group/Brand breakdowns, monthly trend,
top products) — exactly the same aggregation logic that was validated against
`universesample.xlsx` during the preview phase, just re-run against live rows.

## Files in this folder

| File | Purpose |
|---|---|
| `index.html` | The extension itself — same visual design as the preview, now data-driven |
| `UniverseOverview.trex` | Tableau manifest — tells Tableau where to load the extension from |
| `tableau.extensions.1.latest.js` | Official Tableau Extensions API library (local copy) |

## Worksheet architecture — 2 worksheets, auto-detected

The extension looks for exactly two worksheets on the dashboard and tells them
apart automatically: whichever one has an **`Article Id`** field is treated as
**Detail**; whichever has a **`Day Month`** field but *no* `Article Id` is treated
as **Trend**. It doesn't matter what you name the worksheets themselves.

**1. Detail worksheet** — Article Id grain, no date. Feeds: SKU counts, the
Sales-by-Universe table/donut, Product Group / Product Group Mer / Brand
breakdowns, and Top Products.

Required fields (exact names, case-sensitive):
```
Sls Grp Desc          Article Id             Article Name Th
Product Group         Product Group Mer      Mc Desc
Brand                 Vendor Name            Universe
Net Inc Tax - LY      Net Inc Tax - CY
Sales Qty - LY        Sales Qty - CY
```

**2. Trend worksheet** — day grain, no Article Id. Feeds: the 4 KPI tiles, the
monthly trend sparklines, and the Flag_PrivateBrand mix in "Universe Performance".

Required fields:
```
Day Month             Sls Ofc Desc           Flag_PrivateBrand
Universe
Net Inc Tax - CY      Net Inc Tax - LY
Sales Qty - CY        Sales Qty - LY
```

If either worksheet is missing a required field, the extension shows a red banner
naming exactly which field(s) are missing on which worksheet, instead of a
blank/broken dashboard — rename the Tableau field (or update the `DETAIL_FIELDS`
/ `TREND_FIELDS` constants near the top of `index.html`'s script) to match.

### CY/LY architecture

Net Sales and Sales Qty arrive **pre-split into Current/Prior columns** via
calculated fields driven by Tableau Parameters (`Start Date`, `End Date`):

```
[Net Inc Tax - CY] = IF [Time Date] >= [Start Date] AND [Time Date] <= [End Date] THEN [Net Inc Tax] END
[Net Inc Tax - LY] = IF [Time Date] >= DATEADD('year',-1,[Start Date]) AND [Time Date] <= DATEADD('year',-1,[End Date]) THEN [Net Inc Tax] END
```
(same pattern for `Sales Qty`). The extension does **no date-range math of its
own** — it sums whichever CY/LY column Tableau already computed per row. This is
also why an arbitrary custom date range works automatically with no code changes.

The dashboard's "vs {year}" labels and the header subtitle period are derived
directly from the **min/max date actually present in the Trend worksheet** (i.e.
whatever `Start Date`/`End Date` currently produce), not hardcoded.

### Universe tiers

`Universe` is expected to hold one of `ECO` / `MASS` / `PREMIUM` / `LUXURY`. Any
row with a blank/other value is bucketed as `UNCLASSIFIED` and shown as a 5th,
visually de-emphasized row in the tables and stacked-bar mixes — the donut chart
and "Universe Performance" grid intentionally exclude UNCLASSIFIED, matching how
the dashboard was designed against the sample data (~2–3% of rows had no
Universe tag).

## 1. Push this folder to GitHub

```
cd extension
git init
git remote add origin <your-repo-url>
git add .
git commit -m "Add Universe Overview Tableau extension"
git branch -M main
git push -u origin main
```

## 2. Enable GitHub Pages

Repo → **Settings → Pages** → Source: **Deploy from a branch** → Branch: **main**,
folder **/ (root)** → Save. Once it publishes, update the `<url>` inside
`UniverseOverview.trex` to match the published address (it currently points at
`http://localhost:8765/index.html` for local testing — see below).

## 3. Build the worksheets + add the extension in Tableau Desktop

1. Connect the workbook to the real data source.
2. Add `Start Date` and `End Date` Parameters, and the CY/LY calculated fields
   described above for `Net Inc Tax` and `Sales Qty`.
3. Build the **Detail** worksheet and the **Trend** worksheet with the fields
   listed above, on Rows/Detail as appropriate (no need to match the exact shelf
   layout — the extension reads whatever the worksheet's summary data contains).
4. Add **both** worksheets to the same dashboard, then **Objects → Extension** →
   browse to `UniverseOverview.trex` → **Add**.
5. The dashboard should populate within a few seconds. Changing `Start Date`/
   `End Date` or any native Tableau filter on either worksheet automatically
   re-triggers the extension and refreshes every chart (`FilterChanged` /
   `SummaryDataChanged` listeners registered on both worksheets, debounced).

## Local testing (before deploying to GitHub Pages)

Tableau extensions must be served over http(s), not opened as a local `file://`
path. To test against `http://localhost` before pushing:

```
cd extension
node -e "require('http').createServer((req,res)=>{const fs=require('fs'),path=require('path');let p=path.join(__dirname,decodeURIComponent(req.url.split('?')[0])==='/'?'/index.html':decodeURIComponent(req.url.split('?')[0]));fs.readFile(p,(e,d)=>{if(e){res.writeHead(404);res.end('not found');return;}res.writeHead(200);res.end(d);});}).listen(8765,()=>console.log('http://localhost:8765'))"
```

`UniverseOverview.trex` already points at `http://localhost:8765/index.html`, so
you can load it directly in Tableau Desktop for local testing — swap the URL to
your GitHub Pages address once confirmed working.

## What's been verified vs. what still needs your check

**Verified without a live Tableau connection:**
- The aggregation logic (KPI totals, Universe/Product-Group-Mer/Product-Group/
  Brand/PrivateBrand breakdowns, monthly trend, Top-Products rankings) was
  ported from the Python script that built and validated the static preview,
  then re-run in Node.js against the real rows from `universesample.xlsx`
  (exported to plain JSON matching the shape the Extensions API hands back). Every
  output matched the preview's numbers exactly.
- Caught and fixed a real bug during that test: date parsing was going through
  `new Date(...)` and reading UTC parts back out, which silently shifted every
  1st-of-month row into the previous month in timezones ahead of UTC (this
  machine is UTC+7). Fixed by parsing the `YYYY-MM-DD` prefix directly out of the
  raw string instead of round-tripping through a `Date` object.
- Opened `index.html` directly in a browser (outside Tableau): the layout renders
  correctly and the extension fails gracefully with a clear error banner instead
  of a blank page or a thrown exception — expected, since extensions only
  initialize inside an actual Tableau host frame.

**Not yet verified — needs you to check inside Tableau Desktop:**
- I have no way to run this against a real Tableau workbook from here, so the
  actual `getSummaryDataReaderAsync` calls, the Detail/Trend auto-detection, the
  `FilterChanged`/`SummaryDataChanged` refresh wiring, and the on-dashboard visual
  result are all unverified against a live workbook. Please load it in Tableau
  Desktop per the steps above and confirm: both worksheets are detected correctly,
  no field-error banner appears, all four KPI tiles and every chart populate, and
  changing `Start Date`/`End Date` or a filter refreshes the dashboard.
- If a field name in the real workbook differs even slightly from the lists above
  (extra space, different casing), the named-field error banner should tell you
  exactly which one — that's the first thing to check if the dashboard doesn't load.
