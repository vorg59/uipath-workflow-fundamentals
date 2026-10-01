# UiPath Automation Workflows: From "Hello World" to Data Scraping & Form Filling

A learning-oriented collection of **UiPath Studio** workflows (`.xaml`) that build up from basic control-flow concepts to real-world web automation. A central launcher (`MAIN.xaml`) shows a dropdown dialog and runs whichever workflow the user picks.

> Built with the modern UiPath UI Automation framework (`NApplicationCard`, `NTypeInto`, `NExtractDataGeneric`, `NFillForm`, etc.) and Excel activities (`ExcelProcessScopeX`, `ReadRangeX`, `WriteRangeX`). Target browser: **Microsoft Edge**.

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Workflow Details](#workflow-details)
  - [0. MAIN](#0-mainxaml--launcher--router)
  - [1. HELLO WORLD](#1-hello-worldxaml)
  - [2. BROWSER](#2-browserxaml)
  - [3. IF ELSE](#3-if-elsexaml)
  - [4. CASE SWITCH](#4-case-switchxaml)
  - [5. CHILD-SEQUENCE](#5-child-sequencexaml)
  - [6. FLOWCHART](#6-flowchartxaml)
  - [8. SCRAPING REDUX](#8-scraping-reduxxaml)
  - [10. FILL FORM](#10-fill-formxaml)
- [Concepts Covered](#concepts-covered)
- [Prerequisites](#prerequisites)
- [Setup & Running](#setup--running)
- [Configuration Notes](#configuration-notes)
- [Known Gaps & Troubleshooting](#known-gaps--troubleshooting)
- [Possible Improvements](#possible-improvements)

---

## Overview

| # | Workflow | Main Idea | Container Type |
|---|----------|-----------|----------------|
| 0 | `MAIN` | Dropdown launcher that routes to the other workflows | Flowchart + nested Flow Switches |
| 1 | `HELLO WORLD` | Message boxes | Sequence |
| 2 | `BROWSER` | Open Google in Edge and search | Sequence + Application Card |
| 3 | `IF ELSE` | Branch on a user choice (two options) | Sequence + `If` |
| 4 | `CASE SWITCH` | Branch on a user choice (three options + default) | Sequence + `Switch` |
| 5 | `CHILD-SEQUENCE` | Nested sequences and string input | Sequence (parent/child) |
| 6 | `FLOWCHART` | Looping greeting flow with Yes/No exit | Flowchart + Flow Switch |
| 8 | `SCRAPING REDUX` | Multi-page table scraping to Excel | Sequence + Do While |
| 10 | `FILL FORM` | Read Excel and auto-fill a web form per row | Sequence + For Each Row |

---

## Project Structure

```
.
├── 0. MAIN.xaml                 # Entry point: dropdown launcher
├── 1. HELLO WORLD.xaml
├── 2. BROWSER.xaml
├── 3. IF ELSE.xaml
├── 4. CASE SWITCH.xaml
├── 5. CHILD-SEQUENCE.xaml
├── 6. FLOWCHART.xaml
├── 7. DATA SCRAPING.xaml        # Referenced by MAIN (not included in this repo)
├── 8. SCRAPING REDUX.xaml
├── 9. DATA ENTRY THROUGH EXCEL.xaml   # Referenced by MAIN (not included in this repo)
└── 10. FILL FORM.xaml
```

> **Important:** `MAIN` invokes the other files using their original names with **spaces and dots** (e.g. `1. HELLO WORLD.xaml`). Keep those exact filenames in your UiPath project, otherwise `Invoke Workflow File` will fail. See [Known Gaps](#known-gaps--troubleshooting).

---

## How It Works

1. Run `MAIN.xaml`.
2. An **Input Dialog** titled *"Main Workflow"* appears with the label *"Select Workflow"* and these options:

   `Hello World` · `Browser` · `IF-ELSE` · `CASE-SWITCH` · `Child-Sequence` · `Flowchart` · `Data Scraping Redux` · `Fill Form`

3. The selection is stored in the `userChoice` string variable.
4. A chain of **Flow Switch** activities evaluates `userChoice` and invokes the matching workflow via **Invoke Workflow File**.
5. If nothing matches (e.g. the dialog is cancelled or returns an unexpected value), the **default branch** shows a message box (*"Invoking another sequence..."*) and then runs `5. CHILD-SEQUENCE.xaml`.

```
                 ┌──────────────────────┐
                 │ Input Dialog         │
                 │ (userChoice)         │
                 └──────────┬───────────┘
                            ▼
                 ┌──────────────────────┐
                 │ Flow Switch chain    │
                 └──────────┬───────────┘
     ┌──────┬──────┬────────┼────────┬──────────┬──────────┬────────┐
     ▼      ▼      ▼        ▼        ▼          ▼          ▼        ▼
   Hello  Browser IF-ELSE  CASE-   Child-    Flowchart  Scraping  Fill
   World                   SWITCH  Sequence             Redux     Form
                                     ▲
                                     └── also the Default branch
```

---

## Workflow Details

### 0. `MAIN.xaml`: Launcher / Router

- **Type:** Flowchart inside a Sequence named `MAIN`
- **Variable:** `userChoice` (`String`)
- **Start node:** `Input Dialog`, titled *Main Workflow*, label *Select Workflow*
- **Routing (via nested `FlowSwitch<String>`):**

| Dialog Option | Invokes |
|---------------|---------|
| `Hello World` | `1. HELLO WORLD.xaml` |
| `Browser` | `2. BROWSER.xaml` |
| `IF-ELSE` | `3. IF ELSE.xaml` |
| `CASE-SWITCH` | `4. CASE SWITCH.xaml` |
| `Child-Sequence` | `5. CHILD-SEQUENCE.xaml` |
| `Flowchart` | `6. FLOWCHART.xaml` |
| `Data Scraping Redux` | `8. SCRAPING REDUX.xaml` |
| `Fill Form` | `10. FILL FORM.xaml` |
| *(default)* | Message box *"Invoking another sequence..."* → `5. CHILD-SEQUENCE.xaml` |

- The flowchart canvas also contains `Invoke Workflow` nodes for `7. DATA SCRAPING.xaml` and `9. DATA ENTRY THROUGH EXCEL.xaml`, but they are **not connected to the dropdown** (no matching option or switch case), so they are never executed from `MAIN`.

---

### 1. `HELLO WORLD.xaml`

The simplest workflow, a first-automation starter.

- **Type:** Sequence (`Main Sequence`)
- **Activities:**
  1. Message Box → `"Hello World!"`
  2. Message Box → `"Part 2"`

**Demonstrates:** sequential execution, Message Box activity.

---

### 2. `BROWSER.xaml`

Opens Google in Edge and performs a search.

- **Type:** Sequence → *Open Browser Automation*
- **Key activities:**
  - `NApplicationCard` targeting **Edge**, URL `https://www.google.com/` (attach by instance, close mode: *Never*, interaction mode: Debugger API)
  - `NTypeInto` into the Google search `TEXTAREA`, typing: `weather[k(enter)]` (types the text then presses Enter)
  - Verify-execution is enabled (`TextChanges`), checking that the search results page loaded
- **Selector:** `<webctrl tag='TEXTAREA' />` scoped to `<html app='msedge.exe' title='Google' />`; search steps include Selector with Computer Vision fallback

**Demonstrates:** Modern Application Card, Type Into, special-key syntax, element verification.

---

### 3. `IF ELSE.xaml`

Branching with a two-way choice.

- **Type:** Sequence (`Sequence1`)
- **Variable:** `userChoice` (`String`)
- **Flow:**
  1. Input Dialog, *"Make a choice from the below options"*, options: `Google;YouTube`
  2. `If userChoice = "Google"`
     - **Then:** Application Card attached to the Edge **Google** window (`https://www.google.com/`)
     - **Else:** Application Card attached to the Edge **YouTube** tab (`https://www.youtube.com/results?search_query=n8n+tutorial`)

> The Application Card bodies are currently empty. They set up the target app/window and are ready for you to drop in your own UI actions.

**Demonstrates:** `If / Then / Else`, Input Dialog with dropdown options, string comparison.

---

### 4. `CASE SWITCH.xaml`

Branching with multiple choices plus a default.

- **Type:** Sequence (`Sequence2`)
- **Variable:** `userChoice` (`String`)
- **Flow:**
  1. Input Dialog, options: `Google;YouTube;UiPath`
  2. `Switch` on `userChoice`:

| Case | Action |
|------|--------|
| `Google` | Application Card → Edge, `https://www.google.com/` |
| `YouTube` | Application Card → Edge, `https://www.youtube.com/results?search_query=n8n+tutorial` |
| `UiPath` | Application Card → Edge, `https://www.uipath.com/` |
| *Default* | Message Box → `"You have made no choice"` |

**Demonstrates:** `Switch` activity, default branch handling, scaling an If/Else into multi-way routing.

---

### 5. `CHILD-SEQUENCE.xaml`

Shows how sequences nest inside each other and how data flows through a variable.

- **Type:** Sequence (`_5__SUB_SEQUENCE`) → `Parent-Sequence` → `Child-Sequence`
- **Variable:** `userInput` (`String`)
- **Flow:**
  1. (Parent) Message Box → `"Starting sub-sequence..."`
  2. (Child) Input Dialog → label *"Enter String"*, title *"User Input"*
  3. (Child) Message Box → `"String Entered = " + userInput`

**Demonstrates:** nested sequences, variable scope, string concatenation. This is also the workflow `MAIN` falls back to by default.

---

### 6. `FLOWCHART.xaml`

A flowchart-based flow with a loop and a Yes/No decision.

- **Type:** Flowchart
- **Variables:** `userName`, `userChoice` (both `String`)
- **Flow:**

```
 ┌────────────────────┐
 │ Input Dialog       │◄─────────────── "Yes" ──────────┐
 │ "Enter Name"       │                                  │
 └─────────┬──────────┘                                  │
           ▼                                             │
 ┌────────────────────┐                                  │
 │ Message Box        │                                  │
 │ "Hello " + name    │                                  │
 └─────────┬──────────┘                                  │
           ▼                                             │
 ┌────────────────────┐        ┌─────────────┐           │
 │ Input Dialog       │───────►│ Flow Switch │───────────┘
 │ Continue? Yes;No   │◄── default (re-ask)   │
 └────────────────────┘        └──────┬──────┘
                                      │ "No"
                                      ▼
                            Message Box "Ending Automation..."
```

- `Yes` → loops back to the **name entry** step.
- `No` → shows *"Ending Automation..."* and finishes.
- Any other value (default) → re-asks the Continue/Not question.

**Demonstrates:** Flowchart design, `Flow Switch`, looping through flow connections.

---

### 8. `SCRAPING REDUX.xaml`

Scrapes **every page** of a paginated work-items table from the UiPath ACME test site and saves everything to Excel.

- **Type:** Sequence
- **Variables:**

| Variable | Type | Purpose |
|----------|------|---------|
| `pageNum` | `Int32` (default `1`) | Current page number |
| `path` | `String` | Output Excel file path (see [Configuration Notes](#configuration-notes)) |
| `scrapedData` | `DataTable` | Data extracted from the current page |
| `allData` | `DataTable` (`New DataTable`) | Accumulator for all pages |

- **Flow:**
  1. **Application Card** attached to Edge: `ACME System 1 - Work Items` (`https://acme-test.uipath.com/work-items?page=1`)
  2. **Do While** loop (`scrapedData Is Nothing OrElse scrapedData.Rows.Count > 0`):
     1. **Extract Table Data** (`NExtractDataGeneric`) on the page's `TABLE` → `scrapedData` (continue-on-error enabled)
     2. **Merge Data Table**: `scrapedData` → `allData` (`MissingSchemaAction = Add`)
     3. **Assign**: `pageNum = pageNum + 1`
     4. **Go To URL**: `"https://acme-test.uipath.com/work-items?page=" + pageNum.ToString`
  3. **Excel Process Scope** → **Use Excel File** (`path`) → **Write DataTable to Excel**: `allData` written to `Sheet1`, starting at `A1`

- **Extracted columns:** `WIID`, `Description`, `Type`, `Status`, `Date`, `Actions` (plus an `Actions Url` column captured from the link)
- **Termination:** the loop stops once a page returns **zero rows** (i.e. you've gone past the last page).

**Demonstrates:** data scraping, pagination by URL, DataTable merging, Do While loops, writing to Excel.

---

### 10. `FILL FORM.xaml`

Reads input rows from Excel and uses each one to fill and submit a web form on the **RPA Challenge** site.

- **Type:** Sequence
- **Variable:** `excelInput` (`DataTable`)
- **Flow:**
  1. **Excel Process Scope → Use Excel File** (`challenge(Sheet1).xlsx`) → **Read Range** of `Sheet1` → `excelInput`
  2. **Application Card** attached to Edge: `Rpa Challenge` (`https://rpachallenge.com/`)
  3. **For Each Row in Data Table** (`excelInput`, row variable `CurrentRow`):
     - **Fill Form** (`NFillForm`) using `CurrentRow` as the data source
     - **Click** the `Submit` button (`<webctrl tag='INPUT' type='submit' />`)

**Demonstrates:** Excel reading, `For Each Row`, the Fill Form activity (maps data columns to form fields automatically), clicking UI elements.

---

## Concepts Covered

- **Control flow:** Sequence, Flowchart, `If`, `Switch`, `Flow Switch`, `Do While`, `For Each Row`
- **User interaction:** `Input Dialog` (free text and dropdown), `Message Box`
- **Workflow composition:** `Invoke Workflow File`, nested sequences, default/fallback routing
- **Variables & data:** `String`, `Int32`, `DataTable`, string concatenation, `Assign`
- **Browser automation:** Application Card, Type Into, Click, Go To URL, Fill Form, Extract Table Data
- **Excel integration:** Read Range, Write Range, Merge Data Table
- **Reliability:** verify-execution on typing, continue-on-error on extraction, selector + Computer Vision search steps

---

## Prerequisites

- **UiPath Studio** (a recent version with the modern design experience and `UIAutomationNext` packages)
- **Microsoft Edge** with the **UiPath Browser Extension** installed and enabled
- **Microsoft Excel** installed (the workflows use Excel scope activities)
- Windows machine (the Excel and Edge activities are Windows-based)
- Internet access to:
  - `https://www.google.com`, `https://www.youtube.com`, `https://www.uipath.com`
  - `https://acme-test.uipath.com/work-items` (scraping demo)
  - `https://rpachallenge.com/` (form-filling demo)
- Required activity packages (typically included by default): `UiPath.System.Activities`, `UiPath.UIAutomationNext.Activities`, `UiPath.Excel.Activities`

---

## Setup & Running

1. **Clone** the repository:
   ```bash
   git clone <your-repo-url>
   ```
2. In UiPath Studio, choose **Open** and select the project folder (or create a new project and copy the `.xaml` files into it).
3. Make sure the files keep their **original names** (with spaces), as listed in [Project Structure](#project-structure).
4. Set **`0. MAIN.xaml`** as the project's **Main** workflow (right-click → *Set as Main*).
5. Update the file paths described in [Configuration Notes](#configuration-notes).
6. Open the target pages in Edge where required (see below) and press **Run** (`F5`).

### Running a single workflow

Any workflow can be run on its own: right-click the file in the Project panel and choose **Run file**. This is useful for debugging individual pieces.

### Pre-conditions per workflow

| Workflow | Before you run it |
|----------|-------------------|
| `BROWSER` | Edge open on Google (`google.com`) |
| `IF ELSE` / `CASE SWITCH` | Edge windows open for the option you plan to choose (Google, the YouTube results page, or uipath.com) because the cards attach **by instance** |
| `SCRAPING REDUX` | Edge open on `acme-test.uipath.com/work-items`; output Excel file must exist at `path` |
| `FILL FORM` | Edge open on `rpachallenge.com` with the challenge started; input Excel file available |

---

## Configuration Notes

Some values are **hard-coded** to the original author's machine. Update them before running:

| Workflow | Setting | Current Value | What to Change |
|----------|---------|---------------|----------------|
| `SCRAPING REDUX` | `path` variable | `C:\Users\Dell\Documents\UiPath\FirstProject\data scraping Ui Path.xlsx` | Point to an Excel file on your machine (create an empty workbook with a `Sheet1`) |
| `FILL FORM` | Excel workbook path | `C:\Users\Dell\Documents\UiPath\FirstProject\challenge(Sheet1).xlsx` | Point to your copy of the RPA Challenge input file |

**Tip:** Replace the absolute paths with project-relative paths (e.g. `Environment.CurrentDirectory + "\Data\..."`) or Workflow arguments so the project works on any machine.

The `FILL FORM` input spreadsheet should contain one row per record, with column headers matching the labels of the form fields on `rpachallenge.com` (the RPA Challenge input sheet uses columns such as *First Name, Last Name, Company Name, Role in Company, Address, Email, Phone Number*).

---

## Known Gaps & Troubleshooting

- **Missing workflows 7 and 9.** `MAIN` contains invoke nodes for `7. DATA SCRAPING.xaml` and `9. DATA ENTRY THROUGH EXCEL.xaml`, but those files are not part of this set, and they have no dropdown option or switch case. They are harmless (never executed), but Studio may flag the missing files. Either add the files or delete those two nodes from the flowchart.
- **Dropdown labels vs. files.** The dropdown lists 8 options; workflows 7 and 9 are intentionally absent from it.
- **Filename mismatch error.** If you see *"Could not find file ... .xaml"*, check that the files are named exactly as `MAIN` expects (e.g. `1. HELLO WORLD.xaml`, not `1__HELLO_WORLD.xaml`). Some download or upload tools replace spaces and dots with underscores.
- **Default branch behavior.** Cancelling the dropdown dialog (or any value that doesn't match a case) runs the `CHILD-SEQUENCE` workflow, not a "no selection" message.
- **"Cannot find UI element" errors.** The Application Cards attach by instance. Ensure the right Edge tab/window title is open (e.g. title `Google`, `ACME System 1 - Work Items`, `Rpa Challenge`) and the UiPath Edge extension is enabled.
- **`IF ELSE` / `CASE SWITCH` appear to do nothing.** The card bodies are empty; these workflows only attach to the browser window. Add activities inside the `Do` section to act on the page.
- **Excel file errors.** The Excel file must exist, and `Sheet1` must be present. Close it in Excel before running if it is locked.
- **Selectors broke after a site update.** Re-indicate the element in UiPath's UI Explorer; Google, YouTube and the demo sites can change their markup.

---

## Possible Improvements

- Convert hard-coded paths and URLs to **In/Out arguments** or a **Config.xlsx** (REFramework style).
- Add **Try/Catch** and logging (`Log Message`) around browser and Excel steps.
- Fill in the empty `Do` bodies in `IF ELSE` and `CASE SWITCH`, for example search for the chosen site's content.
- Add `7. DATA SCRAPING` and `9. DATA ENTRY THROUGH EXCEL` to the dropdown, or remove their placeholder nodes.
- Wrap `MAIN` in a loop so the user can run several workflows in one session.
- Add input validation (empty or cancelled dialog) with a clearer message than the current fallback.

---