# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is the source for a GitHub Pages site (`karlwkenny.github.io`) that hosts a small set of **standalone, single-file HTML dashboards** for Westmill Industries (finance, sales/CRM, and manufacturing BOM data). There is no build system, package manager, framework, server, or test suite — every page is a self-contained `.html` file with inline `<style>` and `<script>`, viewable by opening it directly in a browser or via GitHub Pages.

There is no top-level `index.html` in this repo — it was intentionally removed (see git history) because the site's landing/gating page now lives in a separate, gated Cloudflare Worker outside this repo. This repo only holds the individual dashboard pages that Worker links to.

## Files

- `CRM_Dashboard.html` — Account & opportunity review. Data is embedded directly in the file as a top-level `const D = [...]` array (companies → opportunities → activity feed). Contains commented-out `fetch()` calls against a SharePoint REST API (`_api/web/lists/...`), showing the intended live-data integration that is currently disabled in favor of the static embedded snapshot.
- `FY27_Budget_Dashboard.html` — FY27 budget dashboard. Data is embedded as `const DB_LEAVES`, `const MNS`, `const NARRATIVES`, plus an `EMBEDDED_XLSX_B` (base64) blob decoded client-side via the `xlsx` CDN library for a raw-workbook view/export.
- `Equalizer_Chain_Dashboard.html` — Saw-chain BOM + landed-cost explorer. Data is embedded as `const DB` (a `builds` tree walked recursively by a `flatten()` helper to produce cost rows). Tabbed UI (`.tabs`/`.view.on`) with a BOM tree view and a cost-detail view.
- `parts_sales_explorer.html` — Parts sales explorer. The **only** page that loads external data: it pulls in `data.js` via `<script src="data.js?v=<BUILD_TS>">` and uses Chart.js (via CDN) for charts.
- `data.js` — Generated data file consumed only by `parts_sales_explorer.html`. Exposes `BUILD_TS`, `BUILD_INFO` (row counts, currency/date-range coverage), `FY_ORDER`, `MONTHLY`, `MONTHLY_BY_INDUSTRY`, and a large `DATA` row array (one row per invoice line: customer, FY, invoice #, description, class, qty, revenue, cost, margin, currency, dates, SO/PO numbers, warehouse, etc.). The file header says it is **auto-generated** by `python3 build_data.py` (or `--watch`) — that generator script is **not part of this repo**; treat `data.js` as a build artifact and never hand-edit it (edits will be silently discarded when it's regenerated). The `?v=<timestamp>` query string on the script tag is a cache-buster tied to `BUILD_TS`/`BUILD_INFO.generated`.
- `report-annual.html` / `report-wc.html` — Static board-report style pages (annual report, net working capital assessment). Data lives in a single `const P = {...}` object per file; charts via the Chart.js CDN build.
- `version.json` — `{"ts": <unix timestamp>, "generated": "<human timestamp>"}`. Appears to be a simple build/version stamp; not currently wired into any HTML page's cache-busting (only `data.js` uses its own `BUILD_TS`).

## Working with these files

- **No build/lint/test commands exist.** To "run" a page, just open the `.html` file in a browser (or serve the directory with any static file server, e.g. `python3 -m http.server`, if testing cross-file loading like `parts_sales_explorer.html` → `data.js`, since `file://` XHR/script loading can behave inconsistently across browsers).
- Each dashboard is fully self-contained (HTML + CSS + JS in one file) **except** `parts_sales_explorer.html`, which depends on `data.js` sitting alongside it. Don't move/rename one without the other, and keep the `?v=` query param in sync with `data.js`'s `BUILD_TS` if you regenerate it by hand.
- Embedded data blobs (`const D`, `const DB`, `const DB_LEAVES`, `const P`, `DATA`, etc.) are large, hand-formatted JSON-like literals inlined directly in `<script>` tags. When editing them, preserve valid JS/JSON syntax exactly — there is no schema or type-checking to catch mistakes, and a single syntax error breaks the entire page (all logic lives in the same file, after the data literal).
- These are financial/sales dashboards with real company and revenue data — treat the embedded data as sensitive; don't casually copy it elsewhere or expose it in ways beyond the existing site.
- Currency handling appears throughout (CAD/USD), often with an FX toggle in the UI (e.g. `FX_CAD`, `USD_TO_CAD`, the `.fx-bar` control in `FY27_Budget_Dashboard.html`) — when modifying financial figures or formatting, check for a currency-conversion path before assuming a single value/format applies everywhere.
- External dependencies are loaded via CDN `<script>`/`<link>` tags per-file (Chart.js, the `xlsx` library, Google Fonts) — there's no local vendoring or lockfile, so version pins are whatever's in each file's CDN URL.
