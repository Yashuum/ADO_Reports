# TA ExecutionPipeline Report Generator (Initially, QA Execution Reporting Portal)

> Live at - https://yashuum.github.io/ADO_Reports/
> A single-file, zero-install QA analytics and execution reporting platform.  
> Open `index.html` in any modern browser — no server, no build step, no dependencies to install.

---

## Table of Contents

1. [What It Does](#what-it-does)
2. [Quick Start](#quick-start)
3. [How to Use — Step by Step](#how-to-use--step-by-step)
4. [Supported File Formats](#supported-file-formats)
5. [Understanding the Views](#understanding-the-views)
6. [Domain & Test Case Mapping](#domain--test-case-mapping)
7. [Log File Requirements](#log-file-requirements)
8. [Exporting Reports](#exporting-reports)
9. [Libraries & Frameworks](#libraries--frameworks)
10. [Architecture](#architecture)
11. [Parser Engine Reference](#parser-engine-reference)
12. [Known Modules / Domains](#known-modules--domains)
13. [Troubleshooting](#troubleshooting)
14. [Changelog](#changelog)

---

## What It Does

The QA Execution Reporting Portal takes raw execution log files from your Azure DevOps pipelines (or any test runner) and automatically:

- Parses every test case — name, domain, type (MSG/UI), status, duration, retries, errors
- Builds a management-ready **Domain-wise Execution Summary** table (Executed / Passed / Failed / Skipped / Aborted split by Msg and UI type)
- Shows **per-pipeline-run breakdowns** with expandable test case lists
- Generates **PDF and Excel reports** ready for client/management sharing
- Stores sessions in browser local storage so you can reload previous runs
- Works 100% offline — all processing happens in the browser

---

## Quick Start

```
1. Download  →  index.html
2. Open      →  Double-click the file (or drag into Chrome / Edge / Firefox)
3. Upload    →  Drop your .log / .trx / .xml / .jtl / .zip file
4. Click     →  "Parse & Analyse"
5. Done      →  Dashboard, Domain Report, Pipeline Runs — all populated
```

That's it. No npm install. No Python server. No Azure DevOps access needed.

---

## How to Use — Step by Step

### Step 1 — Open the Portal

Double-click `index.html`. It opens directly in your default browser.  
Recommended browsers: **Chrome 100+**, **Edge 100+**, **Firefox 110+**.  
Safari works but PDF export may behave differently.

---

### Step 2 — Name Your Session (Optional)

At the top of the Upload page there is a **Pipeline / Run Name** field.  
Enter something meaningful like:

```
Sprint 42 Regression · UAT Pass 3
```

This name appears in all reports, charts, and the session history.  
If left blank, the file name is used automatically.

---

### Step 3 — Upload Your Files

**Option A — Drag & Drop**  
Drag one or more files directly onto the upload zone.

**Option B — Click to Browse**  
Click the upload zone and select files from your file system.

You can upload **multiple files at once** — each file becomes one pipeline run.  
All runs are combined into a single session for cross-run comparison.

**ZIP files** are automatically extracted. Every supported file inside the ZIP is parsed independently.

---

### Step 4 — Parse & Analyse

Click the **"Parse & Analyse"** button.  
Each file shows a status indicator:

| Indicator | Meaning |
|-----------|---------|
| 🟡 Pulsing amber | Parsing in progress |
| 🟢 Green | Parsed successfully |
| 🔴 Red | Error reading the file |

Once all files are done, you are automatically taken to the **Dashboard**.

---

### Step 5 — Explore the Views

Use the left sidebar to navigate between views. All views share the same session data.

---

### Step 6 — Export

Go to **Export Reports** in the sidebar.  
Download **PDF** or **Excel** with one click.

---

### Step 7 — Reload Later (Session History)

Every time you parse files, the session is saved automatically in your browser.  
Go to **History** in the sidebar to reload any previous session without re-uploading.

> ⚠️ Session history is stored in **browser localStorage**. Clearing browser data will remove it. Up to 20 sessions are kept.

---

## Supported File Formats

| Format | Extension | Parser Used | Notes |
|--------|-----------|-------------|-------|
| Azure DevOps Console Log | `.log`, `.txt` | Custom log parser | Primary format — structured block parser |
| Visual Studio Test Results | `.trx` | TRX (XML) parser | Full MSTest / NUnit support |
| JUnit XML | `.xml` | JUnit XML parser | Standard `<testsuites>` / `<testsuite>` format |
| JMeter Results | `.jtl` | JTL parser | Both XML and CSV JTL formats supported |
| Allure Results | `.json` | Allure JSON parser | Single result file (not the full Allure folder) |
| Archive | `.zip` | Auto-extract | All supported files inside are extracted and parsed |

---

## Understanding the Views

### Dashboard

The main overview page. Contains:

- **KPI Cards** — Total, Passed, Failed, Skipped, Aborted, Avg Duration, Retries, Auth Failures, Domains count
- **Overall Status donut chart** — Pass/Fail/Skip/Abort proportions
- **Domain Breakdown bar chart** — Passed/Failed/Skipped per domain
- **Pipeline Run Comparison chart** — visible when more than 1 file was uploaded
- **Domain-wise Execution Summary table** — the management report table with Executed / Passed / Failed / Pass Rate columns split by Msg and UI type. This table is kept simple intentionally.
- **Insight cards** — Top Failing Domains, Slowest Test Cases, Most Retried

> The Dashboard table shows only Executed / Passed / Failed. For Skipped and Aborted columns, go to the Domain Report view.

---

### Pipeline Runs

Shows one card per uploaded file. Each card is expandable.

**When expanded, you see:**
- Every individual test case in that run
- Status filter buttons: ALL / PASSED / FAILED / SKIPPED / ABORTED (with counts)
- Search box to filter by test case name or domain
- Table columns: #, Test Case Name, Domain, Type, Status, Duration, Retries, HTTP Code, Error
- Click any **FAILED** row to expand the full error message and stack trace inline
- Footer showing count, average duration, total retries, auth failures for that run

---

### Domain Report

The detailed management table view. Contains:

**Top section — Full Domain Table:**  
Same layout as the Dashboard table but **with extra columns** for Skipped and Aborted.  
These extra columns appear **dynamically** — they only show up when at least one test in the dataset has that status. If your run has no skipped tests, the Skipped column is hidden automatically.

Column layout:
```
Domain / Module | Executed (Msg/UI) | Passed (Msg/UI) | Failed (Msg/UI) | [Skipped (Msg/UI)] | [Aborted (Msg/UI)] | Pass Rate
```

**Domain selector chips:**  
Click any domain chip to drill into all test cases for that domain.  
The chip shows the total count and failure count at a glance.

**Drill-down table:**  
Full test case list for the selected domain, with status filters and search.

---

### Timeline

Visual chronological execution timeline.  
Only populated if the log files contain timestamps (e.g. `[09:10:31 INF]` format).

Shows each test case as a timeline entry with:
- Coloured dot (green = passed, red = failed, amber = skipped, grey = aborted)
- Test name, domain, time of execution
- Duration and retry count
- Error message (truncated)

---

### Export Reports

Two export options:

**Excel (.xlsx)** — 6 sheets:
1. Summary KPIs
2. Domain Report (management table)
3. All Test Cases
4. Failed Tests only
5. Skipped & Aborted
6. One sheet per pipeline run

**PDF (.pdf)** — 4 pages:
1. Cover page with KPI summary boxes
2. Domain-wise management table
3. Pipeline run comparison table
4. Failed test cases list (up to 200)

---

### History

Lists up to 20 previously analysed sessions saved in browser localStorage.  
Each entry shows the session name, date/time, test count, run count and pass rate.  
Click **Load** to restore the full session. Click the trash icon to delete.

---

## Domain & Test Case Mapping

The portal has **1,351 known test cases** pre-loaded from the `TA_HP__Details.xlsx` reference file. These are embedded directly in the HTML as `TC_LOOKUP`.

### How Domain & Type are Resolved

When a test case name is encountered, the portal resolves its domain and type using this priority order:

```
1. Exact name match in TC_LOOKUP               → uses domain and type from Excel
2. Name with trailing suffix stripped           → e.g. _Test, _FT, _TT removed, then lookup
3. Name prefix detection                        → CMS_UI_ prefix → type = UI
                                                  anything else  → type = MSG
4. ALMID abbreviation segment                   → CMS_WT_ALMID_BIL_... → BIL → Bill
5. Module field in the log line                 → "Module : SET" → Settle
6. Keyword scan of known domain names           → fallback
```

### Abbreviation → Domain Map

| Abbreviation in test name | Domain |
|--------------------------|--------|
| BIL | Bill |
| CTEA | CT/EA |
| INF | Infeed |
| MSR, MROD, ME | Measure |
| REC, C1901, BPMD | Rectify |
| RTIE | RTIE |
| SE, SET | Settle |
| STR, BJ8 | Structure |
| CRP | CrossProcess |

### Module Field in Logs

If the log line contains `Module : <value>`, that value is also used as a domain fallback:

```
Module : SET    →  Settle
Module : BIL    →  Bill
Module : RTIE   →  RTIE
```

Full module-to-domain mapping is in the `_moduleToKnownDomain()` function.

---

## Log File Requirements

### Azure DevOps Console Log (Primary Format)

The parser is trained specifically on this log structure:

```
[09:10:31 INF] Execution started for test case - CMS_UI_ALMID_SET_Portfolio_Smoketest_Test. MessageId : . Module : SET
[09:10:45 INF] Technical Name : MarketHeadpointCreation
[09:11:05 INF] Transaction In Passed. TechnicalName : MarketHeadpointCreation. Status : PROCESSED
[09:11:05 INF] Total retry count : 3
[09:17:49 ERR] Testcase failed
```

**Key rules the parser applies:**

| Log pattern | How it is handled |
|-------------|-------------------|
| `Execution started for test case - NAME. ... Module : X` | Opens a new test case block |
| Duplicate `Execution started` for the same name | Deduplicated — counted once |
| `Technical Name : X` | Recorded as an internal step label — **not** a new test case |
| `Transaction In Passed. TechnicalName : X` | Internal step (PASSED) — not a separate test case |
| `Transaction In Failed. TechnicalName : X` | Internal step (FAILED) — marks test as failed |
| `Total retry count : N` | Sets retry count on the current test case |
| `Testcase failed` / `Testcase passed` | Closes the current test case with that status |
| `Test Case NAME executed ... Test Outcome : Failed` | Authoritative close with explicit outcome |
| `[HH:MM:SS ERR] <message>` | Captured as the error message for the test case |
| `401 Unauthorized` | Sets httpCode = 401 on the test case |

> **Important:** `Technical Name` and `Transaction In Passed/Failed` lines are internal steps within a test case. They do NOT create separate test case entries. One test case in the log = one row in the report.

### TRX Files (Azure DevOps / MSTest)

Standard Visual Studio `.trx` format. The parser reads:
- `<UnitTestResult testName="..." outcome="Passed/Failed" duration="..." />`
- Error message from `<Message>` element
- Stack trace from `<StackTrace>` element

### JUnit XML

Standard `<testsuites>` or `<testsuite>` format. Reads `<testcase>` elements with `<failure>`, `<error>`, or `<skipped>` child elements.

### JTL (JMeter)

Supports both XML format (`<httpSample>` / `<sample>` elements) and CSV format (with `label`, `elapsed`, `success`, `responseCode` columns).

---

## Exporting Reports

### Excel Export

Click **Download Excel** on the Export Reports page.  
File is saved as: `QA_Report_YYYY-MM-DD.xlsx`

Sheet breakdown:

| Sheet | Contents |
|-------|----------|
| Summary | KPI metrics — total, passed, failed, pass rate, avg duration |
| Domain Report | Full domain table with all statuses |
| All Test Cases | Every test case across all runs |
| Failed Tests | Failed test cases only |
| Skipped & Aborted | Non-executed test cases |
| Run_1, Run_2, … | One sheet per uploaded file with that run's test cases |

### PDF Export

Click **Download PDF** on the Export Reports page.  
File is saved as: `QA_Report_YYYY-MM-DD.pdf`  
Format: A4 Landscape

Pages:
1. **Cover** — branding, session name, date, KPI summary boxes
2. **Domain Table** — full management summary with pass rates
3. **Pipeline Runs** — per-run comparison table
4. **Failed Tests** — up to 200 failed test cases with error messages

---

## Libraries & Frameworks

All libraries are loaded from public CDNs. No local installation required.

| Library | Version | CDN | Purpose |
|---------|---------|-----|---------|
| **React** | 18 | unpkg.com | UI component framework — all views are React functional components |
| **ReactDOM** | 18 | unpkg.com | Renders React component tree into the browser DOM |
| **Babel Standalone** | latest | unpkg.com | Transpiles JSX syntax in the browser at runtime (no build step) |
| **Tailwind CSS** | 3 (CDN) | cdn.tailwindcss.com | Utility-first CSS framework for all styling |
| **Chart.js** | latest | jsdelivr.net | Canvas-based charting — doughnut and bar charts |
| **jsPDF** | 2.5.1 | cdnjs | Client-side PDF generation |
| **jsPDF-AutoTable** | 3.5.31 | cdnjs | Table plugin for jsPDF — generates the domain summary tables in PDF |
| **SheetJS (xlsx)** | 0.18.5 | cdnjs | Excel file generation — creates multi-sheet .xlsx reports |
| **JSZip** | 3.10.1 | cdnjs | ZIP file reading — auto-extracts uploaded .zip archives |
| **Google Fonts** | — | fonts.googleapis.com | Inter (UI font) + JetBrains Mono (code/mono font) |

### Why Babel Standalone?

The file uses JSX syntax (`<Component />`) inside a `<script type="text/babel">` tag. Babel Standalone transforms this at runtime in the browser, eliminating the need for any build toolchain (Webpack, Vite, etc.). This is what makes the single-file approach work.

> **Network dependency:** The portal requires an internet connection on first load to fetch libraries from CDN. Once cached by the browser, it works offline.

---

## Architecture

The entire application is a single HTML file with three logical layers:

```
index.html
│
├── <head>
│   ├── CDN library <script> tags
│   ├── Tailwind configuration
│   └── Global CSS (custom properties, animations, table styles)
│
└── <body>
    └── <script type="text/babel">
        │
        ├── ICONS               SVG icon components (Ic.Upload, Ic.Dashboard, etc.)
        │
        ├── TC_LOOKUP           1,351 known test cases → [type, domain] map
        ├── ABBR_DOMAIN         Abbreviation → domain name map
        ├── KNOWN_DOMAINS       Authoritative list of 10 modules
        │
        ├── Parser {}           Log parsing engine
        │   ├── detectType()    Identifies file format from extension/content
        │   ├── resolveTestMeta() Resolves domain + type for any test name
        │   ├── parseTRX()      Visual Studio .trx parser
        │   ├── parseJUnitXML() JUnit XML parser
        │   ├── parseJTL()      JMeter JTL parser (XML + CSV)
        │   ├── parseAllure()   Allure JSON parser
        │   └── parseLog()      Azure DevOps console log parser (primary)
        │
        ├── buildStats()        Computes pass/fail/skip/abort totals from test array
        ├── buildDomainReport() Groups tests by domain + type into table rows
        ├── fmtDur()            Formats milliseconds → human-readable string
        │
        ├── PRIMITIVE COMPONENTS
        │   ├── Card            Styled container card
        │   ├── Pill            Status badge (PASSED/FAILED/SKIPPED/ABORTED)
        │   ├── RateBar         Progress bar for pass rate
        │   ├── KPI             Metric card with coloured gradient
        │   ├── Btn             Button with variants
        │   ├── SearchInput     Search field
        │   ├── FilterBtns      Status filter button group
        │   └── ChartBox        Chart.js canvas wrapper
        │
        ├── DomainTable         Dashboard summary table (Exec/Pass/Fail only)
        ├── DomainTableFull     Domain Report table (+ dynamic Skipped/Aborted cols)
        ├── TCRow               Individual test case row (expandable for errors)
        ├── TCTableHeader       Table header row
        ├── PipelineRunCard     Expandable accordion card per pipeline run
        │
        ├── VIEWS
        │   ├── UploadView      File upload interface
        │   ├── DashboardView   KPIs + charts + summary table
        │   ├── PipelineRunsView  List of PipelineRunCards
        │   ├── DomainView      DomainTableFull + domain drill-down
        │   ├── TimelineView    Chronological test execution timeline
        │   ├── ReportsView     PDF and Excel export
        │   └── HistoryView     Session history list
        │
        └── App                 Root component — sidebar, header, routing, session state
```

### State Management

No Redux or external state library. State lives in React hooks:

| State | Location | Description |
|-------|----------|-------------|
| `runs` | `App` | Array of pipeline run objects — the main data |
| `sessions` | `App` | History saved to/from localStorage |
| `view` | `App` | Current active view (routing) |
| `dark` | `App` | Dark/light mode toggle |
| `sidebar` | `App` | Sidebar open/closed |
| `filter`, `search` | Per-view | Local filter state inside each view |

### Data Model

A **run** object:
```js
{
  fileName: "Standard_Console_Output.log",
  pipelineName: "Sprint 42 Regression",
  uploadedAt: Date,
  tests: [ /* array of test objects */ ]
}
```

A **test** object:
```js
{
  id: "log-1",
  name: "CMS_UI_ALMID_SET_Portfolio_Smoketest_Test",
  domain: "Settle",
  type: "UI",                     // "MSG" or "UI"
  status: "FAILED",               // PASSED | FAILED | SKIPPED | ABORTED | UNKNOWN
  duration: 439000,               // milliseconds
  startTime: Date,
  retries: 3,
  httpCode: 401,                  // or null
  errorMessage: "Connection timeout",
  stackTrace: "at ...",
  source: "Standard_Console_Output.log"
}
```

---

## Parser Engine Reference

### `Parser.resolveTestMeta(name)`

Returns `{ type, domain }` for any test case name string.

```
Input:  "CMS_UI_ALMID_SET_Portfolio_Smoketest_Test"
Output: { type: "UI", domain: "Settle" }

Input:  "CMS_WT_ALMID_BIL_BillLinesCheck_Case1342Scenario"
Output: { type: "MSG", domain: "Bill" }
```

### `Parser.parseLog(content, fileName)`

The primary parser for Azure DevOps console logs.

- Splits on `\r\n` and `\n`
- Looks for `Execution started for test case - NAME. ... Module : X` to open a test block
- Deduplicates if the same test name appears twice (common in Azure DevOps logs)
- `Technical Name :` and `Transaction In Passed/Failed` lines are treated as internal steps, not new test cases
- Test is closed by: `Testcase failed`, `Testcase passed`, or `Test Case NAME executed ... Test Outcome : X`
- Falls back to synthetic scan if no structured blocks found

### `buildDomainReport(tests)`

Groups tests into domain rows for the summary table:

```js
[
  {
    domain: "Settle",
    MSG: { total: 5, passed: 4, failed: 1, skipped: 0, aborted: 0 },
    UI:  { total: 1, passed: 0, failed: 1, skipped: 0, aborted: 0 }
  },
  ...
]
```

---

## Known Modules / Domains

The portal recognises exactly these 10 modules. All test cases are mapped to one of these:

| Module | Example test name prefix |
|--------|------------------------|
| **Bill** | `CMS_WT_ALMID_BIL_` |
| **Measure** | `CMS_WT_ALMID_MSR_`, `CMS_WT_ALMID_MROD_` |
| **Rectify** | `CMS_WT_ALMID_REC_`, `CMS_WT_ALMID_BPMD_` |
| **Settle** | `CMS_WT_ALMID_SE_`, `CMS_UI_ALMID_SET_` |
| **Structure** | `CMS_WT_ALMID_STR_` |
| **Infeed** | `CMS_WT_ALMID_INF_` |
| **RTIE** | `CMS_WT_RTIE_` |
| **MarketPortal** | `CMS_WT_ALMID_MARKETPORTAL_` |
| **CrossProcess** | `CMS_WT_ALMID_CRP_` |
| **CT/EA** | `CMS_WT_ALMID_CTEA_` |

Test cases that cannot be matched to any known module are placed in **General**.

---

## Troubleshooting

### "34 test cases showing instead of 1"

**Cause:** The log file contains multiple `Technical Name :` or `Transaction In Passed` lines which the old parser wrongly treated as separate test cases.

**Fix:** This is corrected in the current version. The parser now treats these lines as internal steps within a test case, not as separate test cases. Ensure you are using the latest `index.html`.

---

### "Domain showing as General instead of the correct module"

**Cause:** Either the test case name is not in `TC_LOOKUP` and the abbreviation segment could not be extracted, or the `Module :` field in the log was not recognised.

**What to check:**
1. Does the test name follow the `CMS_WT_ALMID_<ABBR>_` pattern?
2. Is the abbreviation in the `ABBR_DOMAIN` map?
3. Does the log line include `Module : <VALUE>`?

If a new abbreviation needs to be added, find `const ABBR_DOMAIN` in the HTML and add the mapping.

---

### "Portal is blank / not loading"

**Cause:** Usually a CDN fetch failure (no internet) or a browser blocking local file scripts.

**Fix:**
- Ensure you have an internet connection (required to load React, Chart.js etc. from CDN)
- Use Chrome or Edge — Firefox may require allowing local file access
- Do not open from within a ZIP — extract first, then open

---

### "PDF download does nothing"

**Cause:** Browser popup blocker may intercept the file download.

**Fix:** Allow popups/downloads from local files in your browser settings. In Chrome: Settings → Privacy → Site Settings → Additional Permissions → Automatic downloads.

---

### "Excel file opens but columns are cut off"

**Fix:** Select all cells (Ctrl+A) in Excel and use Format → AutoFit Column Width, or double-click any column border in the header row.

---

### Uploading a ZIP — some files not parsed

**Cause:** Only files with extensions `.log`, `.txt`, `.trx`, `.xml`, `.jtl`, `.json` inside the ZIP are processed. Nested ZIP files inside a ZIP are not supported.

---

## Changelog

| Version | Changes |
|---------|---------|
| v3.0 | Added `DomainTableFull` with dynamic Skipped/Aborted columns on Domain Report page. Dashboard table kept simple (Exec/Pass/Fail only). |
| v2.5 | Fixed parser incorrectly creating 34 test entries from a single log file. `Technical Name` and `Transaction In` lines now correctly treated as internal steps. Deduplication of repeated `Execution started` lines added. |
| v2.4 | Embedded 1,351 test cases from `TA_HP__Details.xlsx` as `TC_LOOKUP`. Added `resolveTestMeta()` with 5-level priority resolution. `SET` now correctly maps to `Settle`. |
| v2.3 | Added `_moduleToKnownDomain()` helper. `Module : SET` field in log now used as domain fallback. |
| v2.0 | Full rewrite. Added Pipeline Runs view with expandable per-run test case tables. Added Domain Report view with drill-down. Added Timeline view. |
| v1.0 | Initial release. Basic upload, parse, dashboard, PDF/Excel export. |

---

*Built for internal QA team use. Single HTML file — no deployment required.*
