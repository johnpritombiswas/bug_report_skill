# Bug Report - GitHub Issue & Excel Format

**Author:** John Pritom Biswas

A [Claude Skill](https://www.anthropic.com/news/skills) that converts informal bug reports, feature requests, or enhancement ideas — text or screenshots — into two rigidly structured QA outputs:

1. A formatted **GitHub issue**
2. A single-row **TSV block** ready to paste into Excel/Google Sheets QA trackers

No preamble, no follow-up chit-chat — just the two outputs, every time.

## Why

QA teams often need the same messy bug description turned into two different formats: a clean GitHub issue for engineering, and a row in a shared spreadsheet for tracking. Doing this by hand is repetitive and inconsistent. This skill enforces one strict template so every issue — regardless of who wrote the original report — comes out identically structured.

## Installation

1. Download [`bug-report-github-jira-excel-format.skill`](./bug-report-github-jira-excel-format.skill) from this repo (or clone the repo and zip the `bug-report-github-jira-excel-format/` folder yourself).
2. In Claude, upload/attach the `.skill` file and click **Save skill** on the file card. This installs it into your profile.
3. It will now trigger automatically whenever you paste a bug/feature description, or when you explicitly ask Claude to "log this bug," "convert this to a GitHub issue," "write this up for QA," etc.

## Usage

Just describe a bug or feature — with or without a screenshot:

```
The save button on the profile page doesn't work when the user has
unsaved changes, it just spins forever.
```

Claude responds with:

```
Title: [Bug] Save button spins indefinitely with unsaved profile changes

Description: On the profile page, clicking the Save button when there
are unsaved changes causes the button to enter a permanent
loading/spinner state instead of completing the save. This blocks
users from persisting profile edits.

Steps to reproduce / User workflow:
1. Navigate to the profile page
2. Modify a profile field (e.g. name or bio)
3. Click the Save button

Expected result: The save request completes and the button returns to
its normal state, confirming the changes were saved.

Actual result: The Save button enters a spinner state and never
resolves, leaving the user unable to confirm whether changes were saved.
```

```text
TBD	Save button spins indefinitely with unsaved profile changes	TBD	Web (Chrome, Windows 11)	TBD	TBD	1. Navigate to the profile page 2. Modify a profile field 3. Click the Save button	The save request completes and the button returns to normal, confirming the save.	The Save button spins indefinitely and never resolves.	Open	JPB	TBD	N/A
```

Paste the fenced `text` block directly into a spreadsheet — it splits into 13 columns automatically since it's tab-separated.

## Output rules

### GitHub issue

| Field | Rule |
|---|---|
| Title | Prefixed with `[Bug]`, `[Feature]`, or `[Enhancement]` |
| Description | Overview + impact. Any screenshot analysis is folded in here — no separate "Observations"/"Root Cause" sections |
| Steps to reproduce / User workflow | Numbered list |
| Expected result | What should happen |
| Actual result | What currently happens (detailed, element-by-element if a screenshot was provided) |

### TSV row (13 columns, 12 tabs)

| # | Column | Default / rule |
|---|---|---|
| 1 | Bug/Feature ID | `TBD` if not given |
| 2 | Title/Scenario | Short title, no prefix |
| 3 | Module | `TBD` if not given |
| 4 | Environment | Always `Web (Chrome, Windows 11)` |
| 5 | Severity | `Enhancement` for features; inferred or `TBD` for bugs |
| 6 | Priority | `TBD` if not given |
| 7 | Steps | Combined into one line, no line breaks |
| 8 | Expected Result | One line |
| 9 | Actual Result | One line |
| 10 | Status | Always `Open` |
| 11 | Responsible QA | Always `JPB` |
| 12 | Date | `TBD` if not given |
| 13 | Attachments | Always `N/A` |

Separator is a real tab character (`\t`) — never spaces or pipes. Missing metadata is `TBD`, never fabricated or split across columns.

## Edge cases

- **Multiple bugs in one message** — only the first/most prominent issue is processed. A short note follows the TSV block asking you to submit the others separately.
- **Screenshots** — visual analysis (broken layout, error states, wrong data, etc.) is woven into the `Description` and `Actual result` fields rather than given its own section.
- **Missing metadata** (ID, Module, Priority, Date) — filled with `TBD`, never guessed.

## Repo structure

```
bug-report-github-jira-excel-format/
├── SKILL.md                                    # The skill definition (source of truth)
├── bug-report-github-jira-excel-format.skill   # Packaged, installable skill file
├── evals/
│   └── evals.json                              # Test prompts + verifiable expectations
└── README.md
```

## Testing

`evals/evals.json` contains test prompts covering a standard bug, a feature request, a report with full metadata provided, and the multi-bug edge case. Each has a list of verifiable expectations (exact tab/column counts, correct defaults, etc.) used to validate the skill's output.

## Author

Built by **John Pritom Biswas**. The `Responsible QA` column defaults to `JPB` for that reason — update `SKILL.md` if you want it to default to someone else's initials instead.

## License

MIT (or update to match your repo's license).
