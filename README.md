# Data Anonymizer

**▶ Live: https://framewerx.github.io/data-anonymizer/**

A **100% client-side** tool for detecting and redacting PII (personally identifiable
information) from text before it's sent to AI agents — or anywhere else sensitive
data shouldn't go.

Paste in logs or free text, the tool scans it with regex-based detectors, builds a
list of "variables" (e.g. `jane.doe@example.com → [UserEmail]`), and produces a
sanitized output with every match replaced by its token. You stay in control: rename
tokens, toggle matches on/off, remove false positives, or add your own selectors.

> **No server. No build. No dependencies.** Everything runs in the browser. Nothing
> you paste ever leaves the page. Just open `index.html`.

## Usage

Double-click `index.html` (or serve the folder with any static server) and open it in
a browser. Click **Load sample** to see it in action.

## What it detects

| Detector       | Example match                          | Default token   |
| -------------- | -------------------------------------- | --------------- |
| Email          | `jane.doe@example.com`                 | `[UserEmail]`   |
| IP address     | `9.2.12.33`                            | `[IPAddress]`   |
| MAC address    | `00:1B:44:11:3A:B7`                    | `[MacAddress]`  |
| UUID / token   | `a7d6dfs-sdf9234-sdaf8ff`              | `[UUID]`        |
| Numeric ID     | `4815162342`                           | `[NumericID]`   |
| Server/host    | `server ABC`                           | `[ServerName]`  |
| Name           | `Han` (labelled fields + email-derived)| `[Name]`        |

Server/host and Name detection are **heuristic** (marked with `~`) — they're best
guesses and the most likely to need manual correction.

Names are found two ways: from the local part of detected emails
(`jane.doe@…` → suggests `jane`), and from labelled fields — `FirstName`,
`First Name`, `LastName`, `Username`, `User:`, etc. (the `: `/`=` separator is optional
for the strong name-field labels). Once a name value is found, it's
replaced everywhere it appears — so `Han` detected from `First Name: Han` will also be
redacted in a sentence like "Another person involved is Han Solo".

## The workflow

1. **Paste** text into the input. Detection runs automatically as you type.
2. **Review** the variables list. Each unique value gets one token.
3. **Refine:**
   - ✏️ **Rename** any token (e.g. `[UUID]` → `[EmailJobId]`).
   - 🔘 **Toggle** a variable off to leave it untouched.
   - 🗑️ **Remove** a false positive (e.g. a timestamp mistaken for an ID). Removed
     auto-detections won't come back on re-scan.
   - ➕ **Add your own** — type a value + token, or select text in the input and click
     *Add selection as variable*. Per-variable **Whole word** and **Ignore case**
     options control how aggressively it matches.
   - 🎛️ **Disable a whole detector** via the chips at the top.
   - 🔤 **Change the wrapper** — tokens are wrapped in `[ ]` by default. If square
     brackets are awkward to parse in your data, click either bracket affix to pick a
     different style: `{ }`, `{{ }}`, `( )`, `< >`, `<< >>`, `« »`, `= =`, or `% %`.
     The choice applies to the output and the exported mapping.
4. **Copy** the sanitized output, or **Copy mapping** to grab the `token → value` map
   as JSON (useful if you need to re-hydrate the AI's response later).

## How matching works

- Each unique value becomes one variable; identical values share a token.
- Replacement is overlap-safe (no double-replacement) and applied in **list order**:
  the variable higher in the list claims its matches first and wins any conflict. Use
  the ▲ / ▼ controls in the **Order** column to set precedence — e.g. put `server ABC`
  above `ABC` so the longer phrase wins, or below it so only `ABC` is replaced.
- Custom variables are added at the top (highest precedence) by default.
- Manual ordering is preserved across re-scans; newly detected values are appended.
- Name detection is derived from the local part of detected emails
  (`jane.doe@…` → suggests `jane`, `doe`) and only suggested when the name
  also appears standalone elsewhere in the text.

## Why

Built as a governance/sanitization wrapper step: scrub identifiers out of logs and
text before feeding them to an LLM, while keeping a human in the loop over exactly
what gets redacted.

## Project layout

```
index.html   # the entire app — markup, PrimeNG-inspired styles, and logic
```

That's it. No package.json, no node_modules, no build step.
