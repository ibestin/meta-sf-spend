# Meta Ads to Salesforce Dashboard

A static, browser-only dashboard that links Meta Ads spend and leads to Salesforce lead progress (Fresh, Qualified, Test Drive) and shows the daily funnel and per-ad performance. It is hosted on GitHub Pages and split into two pages:

| Page | File | Audience | Purpose |
|------|------|----------|---------|
| Admin | `index.html` | Owner only | Upload CSV exports, check data health, publish results |
| Public view | `view.html` | Anyone with the link | Dashboards and Detailed Review (read only) |

Published data lives in `data/data.json`, which the admin page writes and the public page reads.

## Live pages

- Admin: https://ibestin.github.io/meta-sf-spend/
- Public view: https://ibestin.github.io/meta-sf-spend/view.html

## How it works

1. On the admin page you load up to six CSV files. They are parsed and processed entirely inside your browser and stored in that browser only (IndexedDB).
2. The admin page matches leads to Salesforce files using phone number plus ad set, and credits each Fresh, Qualified and Test Drive record to the day the lead was generated on Meta.
3. Clicking Publish builds an aggregated `data.json` (per ad, per day totals only) and commits it to this repository through the GitHub API.
4. The public page loads `data/data.json` each time it opens, so everyone sees the latest published data.

## Repository structure

```
index.html        Admin page (upload, data health, publish)
view.html         Public page (Dashboards and Detailed Review)
data/data.json    Published data (created by the admin page)
README.md         This file
```

## One-time setup

1. Add `index.html`, `view.html` and this README to the root of the `main` branch.
2. In the repository, go to Settings, then Pages. Set the source to the `main` branch and the `/ (root)` folder.
3. Create a fine-grained personal access token:
   - GitHub, Settings, Developer settings, Personal access tokens, Fine-grained tokens
   - Repository access: only `meta-sf-spend`
   - Permissions: Contents, Read and write (nothing else)
   - Choose a sensible expiry and note the renewal date

## Publishing new data

1. Open the admin page.
2. Load the CSV files (see the table below). Loading a file replaces the previous copy of that file.
3. Review the "What will be published" summary and the Data health table.
4. Enter the token (optionally tick "Remember the token in this browser only") and click Publish to GitHub.
5. Wait 1 to 2 minutes for GitHub Pages to deploy, then refresh the public page.

If you prefer not to use a token, click "Download data.json instead" and upload the file to `data/data.json` in the repository manually.

## Input files

| File | Required | Columns needed |
|------|----------|----------------|
| Meta Spend | Yes | Ad set name, Ad name, Day, Amount spent (INR). Campaign name is optional |
| Meta Leads | Yes | created_time, ad_name, adset_name, phone_number. campaign_name and is_organic are optional. Organic leads are excluded |
| SF Generated | No | Phone, Ad set Name, Enquiry Source: Created Date. Used only for the Meta to Salesforce sync check |
| SF Fresh leads | No | Mobile, Ad set Name, Quality Type (needed for Invalid Leads) |
| SF Qualified | No | Mobile, Ad set Name |
| SF Test Drive file | No | Mobile, Ad set Name |

Notes:
- Files must be CSV. For Excel files, use Save As CSV.
- The Meta Leads dump should reach further back than the Salesforce files, otherwise older Salesforce records cannot be linked and are reported as "Hidden: no matching Meta lead".
- Publishing requires Meta Spend and Meta Leads. Any missing Salesforce file shows as n/a on the public page.
- Invalid Leads are Fresh leads tagged Not Enquired, Invalid Number, Non-Sales Related, Wrong number or Aftersales.
- Dates written like 05/10/2026 follow the Date format setting on the admin page. ISO dates and timestamps with a timezone are read exactly and converted to IST.

## Metrics

- Spend, Leads, Fresh Leads, Invalid Leads, Qualified, Test Drives
- Cost per Lead, Cost per Fresh Lead, Cost per Qualified Lead
- All columns are the day the lead was generated on Meta. Later stages are credited back to that day, so earlier days can still grow as leads progress.

## Privacy

- Raw CSVs and phone numbers never leave the browser and are never written to the repository.
- `data/data.json` contains only per-ad, per-day totals and spend, plus the ad, ad set and campaign names.
- If this repository is public, `data.json` and the public page are readable by anyone. Keep the repository private only if your Pages plan supports private Pages, or accept that these aggregated figures are public.
- The admin page is reachable at its public URL but shows no data to other visitors and cannot publish without your token.
- The token is stored only in your browser's local storage and is never committed.

## Troubleshooting

| Message | Likely cause |
|---------|--------------|
| 401 from GitHub | Token wrong or expired |
| 403 from GitHub | Token lacks Contents read and write, or rate limit |
| 404 from GitHub | Wrong owner, repo or branch, or the token has no access to this repo |
| "No data has been published yet" on the public page | `data/data.json` does not exist yet. Publish once from the admin page |
| Public page shows old numbers | GitHub Pages is still deploying. Wait a couple of minutes and refresh |
| Public page fails when opened from a file | Open it through GitHub Pages or a web server, not directly from disk |
| Spend or leads look low | Check the Data health table for skipped rows and ads with spend but no leads (ad names may differ between files) |

## Tech notes

- Plain HTML, CSS and JavaScript. No build step, no external libraries, no server.
- The line chart is drawn with inline SVG.
- Light and dark themes follow the system setting.
- All times are shown in IST.
