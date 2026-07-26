---
name: qa-issue-formatter
description: Converts an informal bug description, feature request, enhancement idea, or screenshot/image of a UI problem into two strict, standardized QA outputs — a formatted GitHub issue and a single-row TSV block for pasting into Excel/QA trackers. Use this skill whenever the user describes a bug, glitch, broken feature, crash, or unexpected behavior; requests a new feature or enhancement; uploads a screenshot of a UI issue and asks what's wrong or asks to log it; or explicitly asks to "convert this to a GitHub issue," "log this bug," "write this up for QA," "make a ticket," or similar. Trigger even if the user does not explicitly mention "GitHub issue" or "Excel" — any raw bug/feature description directed at Claude is a candidate for this skill. Always process exactly one issue per invocation.
---

# QA Issue Formatter

Converts informal bug reports, feature requests, or enhancement ideas (text and/or images) into two rigidly structured outputs for a QA workflow: a GitHub issue and an Excel-pasteable TSV row.

## Core behavior

- Always produce **exactly two outputs**, in this order, and nothing else:
  1. The GitHub issue (Markdown)
  2. The TSV row, inside a ```text code block
- **No introductory text, no conversational filler, no concluding remarks.** Output starts immediately with the GitHub issue and ends immediately after the closing ``` of the TSV block. Do not say things like "Here's the formatted issue" or "Let me know if you'd like changes."
- **One issue per response.** If the user describes multiple distinct bugs/features in a single message (or uploads multiple screenshots of unrelated issues), process only the first/most prominent one, then after the TSV block add a single short line (outside the strict output, so this is the one exception to "no concluding remarks"): `Additional issue(s) detected — send separately to process them one at a time.` Do not batch multiple GitHub issues or multiple TSV rows in one response.
- If images are provided, perform careful visual analysis (broken layout, error states, misaligned elements, wrong data, console errors visible in screenshot, etc.) and fold those specific observations directly into the `Description` and `Actual result` fields of the GitHub issue — never as separate "Observations"/"Analysis"/"Root Cause" headers.

## Output 1: GitHub Issue

Use exactly this structure, with these exact field labels:

```
Title: [Brief, descriptive title. Prefix with [Bug], [Feature], or [Enhancement]]

Description: [Short overview of the issue or feature and its impact. Integrate any image-derived analysis, root cause theories, and specific visual observations directly into this paragraph — no separate sections.]

Steps to reproduce / User workflow:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Expected result: [What should happen, or the intended behavior of the new feature]

Actual result: [What is currently happening, or the current system limitation. If images are provided, be highly detailed and list the specific broken elements observed.]
```

Notes:
- For feature/enhancement requests (not bugs), "Steps to reproduce" becomes the user workflow that motivates the feature — still numbered steps.
- Keep the title short (aim for under ~12 words) and always prefixed with `[Bug]`, `[Feature]`, or `[Enhancement]`.

## Output 2: Excel TSV Row

Immediately after the GitHub issue, output a single line inside a ```text code block, with these 13 columns separated by **exactly 12 tab characters** (`\t`) — never spaces, never pipes:

| # | Column | Rule |
|---|--------|------|
| 1 | Bug/Feature ID | `TBD` if not provided by user |
| 2 | Title/Scenario | Same short title as the GitHub issue (no `[Bug]` prefix needed here) |
| 3 | Module | `TBD` if not specified |
| 4 | Environment | Always `Web (Chrome, Windows 11)` |
| 5 | Severity | For bugs: infer (Critical/High/Medium/Low) or `TBD` if unclear. For features/enhancements: always `Enhancement` |
| 6 | Priority | `TBD` if not specified by user |
| 7 | Steps | All steps combined into one sentence, e.g. `1. Click X 2. Click Y 3. Observe Z` |
| 8 | Expected Result | One sentence, no line breaks |
| 9 | Actual Result | One sentence, no line breaks |
| 10 | Status | Always `Open` |
| 11 | Responsible QA | Always `JPB` |
| 12 | Date | `TBD` if not provided by user (do not invent today's date) |
| 13 | Attachments | Always `N/A` |

### Hard formatting rules — verify before emitting

1. **Exactly 13 values, exactly 12 tabs.** Before outputting, internally count the tabs. If it's not 12, something is wrong — recount.
2. **No line breaks anywhere in the row.** Strip all `\n`, `\r`, and any literal tab characters (`\t`) that appear *within* a field's own text (e.g. inside pasted steps) before assembling the row, so they don't get confused with the column-separating tabs.
3. **No pipes or spaces as separators** — only real tab characters between columns.
4. Missing metadata → `TBD` in that single column. Never split missing data across multiple columns, never fabricate specifics (dates, IDs, module names) that weren't given or inferable.

## Example

Given: "The save button on the profile page doesn't work when the user has unsaved changes, it just spins forever."

```
Title: [Bug] Save button spins indefinitely with unsaved profile changes

Description: On the profile page, clicking the Save button when there are unsaved changes causes the button to enter a permanent loading/spinner state instead of completing the save. This blocks users from persisting profile edits.

Steps to reproduce / User workflow:
1. Navigate to the profile page
2. Modify a profile field (e.g. name or bio)
3. Click the Save button

Expected result: The save request completes and the button returns to its normal state, confirming the changes were saved.

Actual result: The Save button enters a spinner state and never resolves, leaving the user unable to confirm whether changes were saved.
```

```text
TBD	Save button spins indefinitely with unsaved profile changes	TBD	Web (Chrome, Windows 11)	TBD	TBD	1. Navigate to the profile page 2. Modify a profile field 3. Click the Save button	The save request completes and the button returns to normal, confirming the save.	The Save button spins indefinitely and never resolves.	Open	JPB	TBD	N/A
```
