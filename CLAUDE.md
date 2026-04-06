# Video Script Generator — Project Guidelines

## Project overview

This project automates the creation of video scripts from AsciiDoc (`.adoc`) documentation files. An orchestrator agent scans documentation repositories, identifies content that would benefit from a video (specifically UI procedures), and delegates to specialist subagents that write, review, and log each script. All scripts are saved as local `.md` files and tracked in a central `video-script-tracker.csv` file.

The goal is to produce high-quality, on-brand video scripts at scale, following established video and writing guidelines, without manual triage of every documentation file.

---

## Folder structure

### Documentation repositories

All documentation repositories are cloned locally under a single parent folder:

```
~/Documents/GitHub/
  ├── docs-product-a/
  ├── docs-product-b/
  ├── docs-product-c/
  ├── docs-product-d/
  └── [additional documentation repositories...]
        └── **/*.adoc (each folder)
```

### Excluded folders

The following folders exist under `~/Documents/GitHub/` but must **never** be scanned or processed. They do not contain documentation content relevant to this workflow:

- `docs-internal-only`
- `docs-archived`
- `docs-site-config`
- `docs-release-notes`

The orchestrator must skip these folders entirely during the scan step, even if they contain `.adoc` files.

### Reference files and outputs

All reference materials and generated outputs are organized as follows:

```
~/Desktop/Video Workflow/
  ├── Video Assets/              # Reference materials
  │     ├── video-guidelines.pdf
  │     ├── video-script-template.pdf
  │     ├── video-script-samples.pdf
  │     ├── cx-writing-guidelines.md
  │     └── video-script-prompt.md
  ├── Video Scripts/             # Generated video scripts (auto-created per repository)
  │     ├── docs-product-a/
  │     ├── docs-product-b/
  │     ├── docs-product-c/
  │     └── [repository folders...]
  │           └── [BRAND] - [repo-name] - [topic]-video-script.md (example)
  └── Video Logs/                # Tracking files organized by repository (auto-created per repository)
        ├── docs-product-a/
        │     ├── video-script-tracker.csv
        │     └── flagged-report-[date].csv
        ├── docs-product-b/
        │     ├── video-script-tracker.csv
        │     └── flagged-report-[date].csv
        └── [repository folders...]
```

> **Important:** Before starting any run, the orchestrator must load all five reference files from the Video Assets folder into its context. The `Video Scripts/` and `Video Logs/[repo-name]/` folders are created automatically on the first run for each repository.

---

## Workflow steps

The orchestrator follows these steps in order for every run:

1. **Load reference files.** Read `video-guidelines.pdf`, `video-script-template.pdf`, `video-script-samples.pdf`, `cx-writing-guidelines.pdf`, and `video-script-prompt.md` from the `~/Desktop/Video Workflow/Video Assets/` folder into context.

2. **Scan documentation repositories.** Recursively walk every `.adoc` file across all folders under `~/Documents/GitHub/`. Build a queue of all files found.

3. **Analyse each file.** For each `.adoc` file, read its content and apply the video detection criteria (see section below). Assign one of three statuses:
   - `VIDEO_NEEDED` — clear UI procedure detected, meets video guidelines
   - `FLAGGED` — content is ambiguous; needs human review before proceeding
   - `SKIP` — no video warranted

4. **Handle flagged files.** If ANY files are flagged, you MUST create a flagged report:
   - **Mandatory:** Save all `FLAGGED` files to a dated CSV report at `~/Desktop/Video Workflow/Video Logs/[repo-name]/flagged-report-[YYYY-MM-DD].csv`
   - CSV format with three columns: `.adoc file` | `Path` | `Reason`
   - `.adoc file` column contains just the filename
   - `Path` column contains the full file path
   - `Reason` column contains the flagging reason with any relevant metrics
   - Include a header row
   - This report is REQUIRED even if only 1 file is flagged
   - After creating the report, continue processing `VIDEO_NEEDED` files without waiting for human review

5. **Delegate confirmed video files.** For each `VIDEO_NEEDED` file, pass a structured brief to the subagents in this sequence:
   - **Script writer subagent** → produces the draft script and saves it as a local `.md` file
   - **Peer reviewer subagent** → reviews and edits the local `.md` file against CX writing guidelines
   - **CSV logger subagent** → appends one row to `~/Desktop/Video Workflow/Video Logs/[repo-name]/video-script-tracker.csv` **only after verifying the script file exists on disk**

6. **Log skipped files.** Add a row to `~/Desktop/Video Workflow/Video Logs/[repo-name]/video-script-tracker.csv` for every `SKIP` file with status "No video" so the full audit trail is maintained.

7. **Verify CSV completeness.** After all script generation is complete, verify that the number of script files matches the number of CSV entries with "Script created" status:
   - Count `.md` files in each `~/Desktop/Video Workflow/Video Scripts/[repo-folder]/` directory
   - Count CSV rows with status "Script created" in `~/Desktop/Video Workflow/Video Logs/[repo-folder]/video-script-tracker.csv`
   - If counts do not match, identify and log the discrepancy
   - For missing CSV entries: append entries for scripts that exist but are not logged
   - For orphaned CSV entries: remove CSV rows that reference non-existent script files
   - **This verification step is mandatory and must complete before the workflow is considered done**

7a. **Verify flagged report exists.** If the analysis identified ANY flagged files:
   - Check that `flagged-report-[YYYY-MM-DD].csv` exists in the Video Logs/[repo-name]/ folder
   - If the report is missing, create it immediately with all flagged files from the analysis in CSV format
   - Report must include: `.adoc file`, `Path`, and `Reason` columns
   - **This verification is mandatory — a missing flagged report is a workflow failure**

8. **Report completion.** After all files are processed and CSV verification is complete, print a summary: total files scanned, scripts created, files flagged, files skipped, CSV entries verified.

---

## Video detection criteria

Use these rules — drawn from the video guidelines — to decide whether a `.adoc` file warrants a video:

### Strong signals for `VIDEO_NEEDED`
- The file contains a numbered step-by-step procedure that involves a UI (buttons, menus, forms, navigation)
- The file is a how-to guide or task topic (not a conceptual or reference topic)
- The steps reference visible UI elements: "Click", "Select", "Navigate to", "Enter", "Toggle", "Open the"
- The procedure has 3 or more steps
- The content would be significantly clearer as a screen recording than as written text

### Strong signals for `SKIP`
- The file is a conceptual overview, architectural explanation, or glossary
- The file is an API reference with no UI interaction
- The content is text-only with no UI procedure (e.g., a configuration file walkthrough via CLI only)
- The file has already been processed in a previous run (check `video-script-tracker.csv`)
- The file belongs to an excluded folder (see Excluded folders section above)

### Signals for `FLAGGED` (human review needed)
- The file contains a mix of conceptual content and some UI steps — unclear if a focused video is possible
- The procedure is very short (1–2 steps) — may not justify a full video
- The file references a UI that may contain PII or sensitive demo data
- The orchestrator is unsure whether the content meets the minimum length requirement per the video guidelines

---

## Subagent roster

The orchestrator delegates to three specialist subagents. Their definition files live in `.claude/agents/`.

### `script-writer`
**Responsibility:** Generates a complete video script for a given `.adoc` file.
**Inputs:** The `.adoc` file content, the video brief from the orchestrator, the video script template, and the video script samples.
**Output:** A fully formatted video script saved as a `.md` file under `~/Desktop/Video Workflow/Video Scripts/[repo-folder]/`.
**Rules:** Must follow video length guidelines, formatting from the template, and CX writing guidelines. Must not include PII or sensitive product data.

### `peer-reviewer`
**Responsibility:** Reviews the draft script `.md` file against the CX writing guidelines and edits it in place.
**Inputs:** The local `.md` file path from the script writer, the CX writing guidelines.
**Output:** Edits applied directly to the `.md` file; a brief review summary returned to the orchestrator.
**Rules:** Must check for tone, clarity, sentence length, active voice, and terminology consistency per CX guidelines. Must not rewrite content that already complies.

### `csv-logger`
**Responsibility:** Appends one row to `~/Desktop/Video Workflow/Video Logs/[repo-name]/video-script-tracker.csv` after each script is approved.
**Inputs:** Repo folder name, `.adoc` file name, video name, short description, script file path, status.
**Output:** One new row in the repository-specific `video-script-tracker.csv` with all six columns populated.
**Rules:**
- Each repository has its own tracker CSV file in `Video Logs/[repo-name]/`
- Create the repository folder if it doesn't exist
- Never overwrite existing rows
- **Before logging:** Verify that the script file exists at the provided file path. If the file does not exist, do not create the CSV entry
- If the file is unreachable, save the row data to a local pending log and retry on the next run
- Never log a CSV entry with status "Script created" unless the corresponding `.md` file exists on disk

---

## Output file configuration

- **Script files:** Saved as `.md` under `~/Desktop/Video Workflow/Video Scripts/[repo-folder]/`
- **File naming:** `[BRAND] - [PRODUCT] - [topic]-video-script.md`
- **Tracker file:** `~/Desktop/Video Workflow/Video Logs/[repo-folder]/video-script-tracker.csv` (one per repository)
- **Tracker columns:** `Docs repo folder` | `adoc file name` | `Video name` | `Short description` | `Script file path` | `Status` | `Date logged`
- **Flagged report file:** `~/Desktop/Video Workflow/Video Logs/[repo-folder]/flagged-report-[YYYY-MM-DD].csv` (one per repository per run)
- **Flagged report columns:** `.adoc file` | `Path` | `Reason`

---

## Important rules for all agents

- Always read the reference files before taking any action. Never rely on assumptions about the guidelines.
- Never create a script file or CSV row for a file with `SKIP` status (except the audit log row).
- Never include customer names, email addresses, account IDs, or any PII in a script.
- If a file write fails, log the error and continue to the next file. Do not halt the entire run.
- Prefer short, focused scripts over comprehensive ones. One video per distinct procedure.
- If a single `.adoc` file contains multiple distinct UI procedures, create one script per procedure and log each separately.
- **CSV integrity rule:** Never log a CSV entry before verifying the corresponding script file exists. The CSV is an audit trail — entries must reflect actual deliverables, not intended ones.
- **Mandatory verification:** After all script generation completes, verify that script file count matches CSV entry count for each repo folder. If counts mismatch, investigate and correct before considering the workflow complete.
- **Flagged report requirement:** If ANY files are flagged during analysis, you MUST create a dated flagged report file (`flagged-report-[YYYY-MM-DD].md`). This is not optional. Verify the report exists before completing the workflow. A missing flagged report is a workflow failure.
