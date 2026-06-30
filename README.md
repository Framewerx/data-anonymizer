# Data Anonymizer

**▶ Live: https://framewerx.github.io/data-anonymizer/**

A **100% client-side** tool for detecting and redacting PII (personally identifiable
information) from text before it's sent to AI agents — and then **rehydrating** the
AI's response back to the real values for delivery to the client.

Paste in logs or free text, the tool scans it with regex-based detectors, builds a
list of "variables" (e.g. `jane.doe@example.com → [UserEmail]`), and produces a
sanitized output with every match replaced by its token. You stay in control: rename
tokens, edit or reorder rules, toggle matches on/off, remove false positives, or add
your own selectors.

> **No server. No build. No dependencies.** Everything runs in the browser. Nothing
> you paste ever leaves the page. Just open `index.html`.

> ⚠️ **This is not a perfect system.** Always review the output for missed or
> mis-tagged data before sharing it. The tool surfaces a persistent reminder for this.

## Usage

Double-click `index.html` (or serve the folder with any static server) and open it in
a browser. Click **Load sample** to see it in action.

## What it detects

| Detector       | Example match                                   | Default token   |
| -------------- | ----------------------------------------------- | --------------- |
| Email          | `jane.doe@example.com`                          | `[UserEmail]`   |
| Name           | `Darth Vader`, `First Name: Han`, `Username: x` | `[Name]`        |
| Phone          | `(480) 555-0137`, `4805550199`, `+1 480.555…`   | `[Phone]`       |
| Address        | `1138 Mos Eisley Blvd, Suite 4, Tatooine, AZ …` | `[Address]`     |
| Website / URL  | `https://sub.domain.tld/path`                   | `[Website]`     |
| IP address     | `9.2.12.33`                                     | `[IPAddress]`   |
| MAC address    | `00:1B:44:11:3A:B7`                             | `[MacAddress]`  |
| UUID / token   | `a7d6dfs-sdf9234-sdaf8ff`                       | `[UUID]`        |
| Numeric ID     | `4815162342`                                    | `[NumericID]`   |
| Server/host    | `server DS-1137`                                | `[ServerName]`  |
| Attachment     | `[cid:image001.png@01DA…]`                      | `[Attachment]`  |

**Name**, **Address**, and **Server/host** detection are **heuristic** (marked with `~`)
— best guesses, most likely to need manual correction.

Notes on the trickier detectors:

- **Names** come from three sources: the local part of an email (`first.last@…` is
  captured as the single full name `First Last` when that phrase appears, otherwise as
  the standalone part that does), and labelled fields — `FirstName`, `First Name`,
  `LastName`, `Username`, `User:`, `Name:`, etc. The `:`/`=` separator is optional for
  the strong name-field labels; with only a space the value must be Capitalized (so
  "First name is required" is ignored). Once a name value is found it's redacted
  everywhere it appears.
- **Phone** handles `(area) prefix-line`, run-together 10-digit, optional `+1`, and
  separators of space / `.` / `-` / URL-encoded `%20`.
- **Phone/Website/Address** matches are span-aware: numbers living inside a URL or
  address (and digit-groups inside a phone) won't spawn their own duplicate variables.
- **Numeric ID** skips 4-digit numbers that look like a year in a date context
  (e.g. `2024-03-14`, `March 2024`), so dates aren't redacted as IDs.

Each detector has a chip toggle; use **All** / **None** to flip them in bulk.

## The two directions

A segmented control at the top switches the whole tool between:

- **Redact** (`Raw ➞ Redacted`) — detect PII and replace it with tokens. Copy the
  output to paste into your AI agent, and **Copy mapping** to save the
  `token → value` JSON.
- **Rehydrate** (`Tokens ➞ Restored`) — paste the AI's token-bearing response and the
  tool swaps every token back to its original value, ready for the client. The current
  mapping is reused automatically; switching modes seeds the input with your redacted
  output so you can verify the round-trip. For a fresh session, use **Import mapping**
  to paste a saved `token → value` JSON.

## The workflow

1. **Paste** text into the input. Detection runs automatically as you type.
2. **Review** the variables list (right sidebar). Each unique value gets one token.
3. **Refine:**
   - ✏️ **Edit** the matched text or rename the token. Editing a matched value turns
     that rule into a **custom** rule so re-scans won't overwrite it.
   - ↕️ **Reorder** with the ▲ / ▼ controls — list order resolves conflicts (top wins).
   - 🔘 **Toggle** a variable off to leave it untouched.
   - 🗑️ **Remove** a false positive. Removed auto-detections won't come back on re-scan.
   - ➕ **Add your own** — type a value + token (lands at the top, highest precedence),
     or select text in **either** the input or output and click *Add selection*.
   - 🔤 **Change the wrapper** — click either bracket affix to pick `[ ]`, `{ }`,
     `{{ }}`, `( )`, `< >`, `<< >>`, `« »`, `= =`, or `% %`.
4. **Copy** the output, or **Copy mapping** for the round-trip.

## How matching works

- Each unique value becomes one variable; identical values share a token.
- Replacement is overlap-safe (no double-replacement) and applied in **list order**:
  the variable higher in the list claims its matches first and wins any conflict.
- Auto-detected full names rank above their parts, so `Darth Vader` wins over a bare
  `Vader` where they overlap.
- Custom variables are added at the top (highest precedence) by default.
- Manual ordering is preserved across re-scans; newly detected values are appended.

## Limits

- Input is capped at **100,000 characters** (~100 KB, ~1,500 log lines) to keep things
  responsive. The counter turns amber near the cap and red at it. Adjust `MAX_CHARS`
  in `index.html` to change it.
- The variables sidebar **paginates** at 25 per page (`PAGE_SIZE`). Redaction always
  covers every detected variable regardless of which page is shown.

## Why

Built as a governance/sanitization wrapper step: scrub identifiers out of logs and
text before feeding them to an LLM, keep a human in the loop over exactly what gets
redacted, then rehydrate the response for the end client.

## Project layout

```
index.html   # the entire app — markup, PrimeNG-inspired styles, and logic
```

That's it. No package.json, no node_modules, no build step.
