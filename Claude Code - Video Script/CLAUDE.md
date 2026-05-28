# Video Script Generator — Project Guidelines

## Project overview

This project automates the creation of video scripts from AsciiDoc (`.adoc`) documentation files. An orchestrator agent scans documentation repositories, identifies content that would benefit from a video (specifically UI procedures), and delegates to specialist subagents that write, review, and log each script. All scripts are saved as local `.md` files and tracked in a central `video-script-tracker.csv` file.

The workflow supports three modes of operation:
- **All repositories:** Process all 40 documentation repositories in one run
- **Specific repositories:** Process a user-selected subset of repositories
- **Single repository:** Process one repository (ideal for testing or focused updates)

The goal is to produce high-quality, on-brand video scripts at scale, following established video and writing guidelines, without manual triage of every documentation file.

---

## GitHub folder configuration

Use the `/setup-github` skill to automatically detect or configure the GitHub folder path, discover documentation repositories, and prompt for repository selection.

**Skill:** `/setup-github`  
**Returns:** GitHub folder path, discovered repositories list, user selection (All/Specific/Single)

**Summary:** The skill uses an automatic detection strategy that:
1. Checks for saved configuration in `.config` file
2. Searches common locations (macOS/Linux/Windows)
3. Validates candidates (must have ≥3 `docs-*` repos)
4. Presents detection results and prompts for confirmation
5. Discovers all `docs-*` repositories (excluding 4 excluded folders)
6. Prompts for selection mode: All (A) | Specific (B) | Single (C)
7. Optionally saves path for future runs

**Excluded folders (never processed):**
- `docs-internal-only`
- `docs-archived`
- `docs-site-config`
- `docs-release-notes`

See `.claude/skills/setup-github.md` for complete documentation.

---

## Folder structure

### Documentation repositories

All documentation repositories are cloned locally under a single GitHub parent folder. The exact path varies by user and is configured at runtime.

**Structure example:**
```
[GitHub-folder]/
  ├── docs-product-a/
  ├── docs-product-b/
  └── [other docs-* folders...]
      └── **/*.adoc
```

**Repository pattern:** `docs-*` (e.g., `docs-product-a`, `docs-product-b`)  
**Discovery:** Orchestrator scans GitHub folder for `docs-*` directories at runtime

### Excluded folders

The following folder patterns must **never** be scanned or processed, even if they exist in the GitHub folder:

**Excluded by name (exact match):**
- `docs-internal-only`
- `docs-archived`
- `docs-site-config`
- `docs-release-notes`

**Why excluded:** These folders do not contain documentation content relevant to video script generation.

**Implementation:**
1. When discovering repositories, skip any folder that matches the excluded list
2. When Option A (All repositories) is selected, process all discovered `docs-*` repositories except these 4
3. These excluded folders should never appear in the selection list presented to the user
4. If a user explicitly requests an excluded repository (Option C), warn them it's excluded and prompt for a different selection

### Reference files and outputs

All reference materials and generated outputs are organized as follows:

```
~/Desktop/Video Workflow/
  ├── Video Assets/                          # Shared reference materials (both workflows)
  │     ├── video-guidelines.pdf
  │     ├── video-script-template.pdf
  │     ├── video-script-samples.pdf
  │     ├── cx-writing-guidelines.pdf
  │     ├── video-script-prompt.md
  │     ├── script-template.md
  │     └── script-sample.md
  │
  ├── Claude Code - Video Script/            # Claude Code workflow (this folder)
  │     ├── CLAUDE.md                        # This file — project rules
  │     ├── .claude/                         # Agents and skills
  │     │     ├── agents/
  │     │     └── skills/
  │     ├── Video Scripts/                   # Generated video scripts (auto-created per repository)
  │     │     ├── docs-product-a/
  │     │     ├── docs-product-b/
  │     │     └── [one folder per processed repository...]
  │     │           └── [Your Brand] - [repo-name] - [topic]-video-script.md (example)
  │     └── Video Logs/                      # Tracking files organized by repository
  │           ├── docs-product-a/
  │           │     ├── video-script-tracker.csv
  │           │     └── flagged-report-[date].csv
  │           ├── docs-product-b/
  │           │     ├── video-script-tracker.csv
  │           │     └── flagged-report-[date].csv
  │           └── [one folder per processed repository...]
  │
  └── Cursor - Video Script/                 # Cursor workflow
        ├── Cursor Skills/                   # Skill definition (symlinked by writers)
        │     └── SKILL.md
        └── Video Scripts/                   # Generated HTML scripts
              └── [Your Brand] - [Product] - [topic].html (example)
```

**Note on folder locations:**
- **Video Workflow folder:** Fixed location at `~/Desktop/Video Workflow/` (can be on user's Desktop)
- **GitHub folder:** Variable location auto-detected or configured at runtime (step 0a)
- **Repository folders:** Dynamically created based on processed repositories

> **Important:** Before starting any run, the orchestrator must load all five reference files from the Video Assets folder into its context. The `Claude Code - Video Script/Video Scripts/` and `Claude Code - Video Script/Video Logs/[repo-name]/` folders are created automatically on the first run for each repository.

---

## Workflow steps

The orchestrator follows these steps in order for every run:

0. **Configure GitHub folder path and discover repositories.**

   Invoke `/setup-github` skill to:
   - Auto-detect or configure GitHub folder path
   - Discover all `docs-*` repositories (excluding 4 excluded folders)
   - Prompt user for repository selection (All/Specific/Single)
   
   Skill returns:
   - GitHub folder path
   - List of discovered repositories
   - User-selected repositories
   - Selection mode (all/specific/single)

**IMPORTANT:** Do not proceed with any file scanning or processing until the user provides their selection. This step is mandatory for every run.

**TIME TRACKING:** Record the start time (timestamp) immediately after the user confirms their repository selection. This will be used to calculate total execution time in step 8.

1. **Load reference files.** Read `video-guidelines.pdf`, `video-script-template.pdf`, `video-script-samples.pdf`, `cx-writing-guidelines.pdf`, and `video-script-prompt.md` from the `~/Desktop/Video Workflow/Video Assets/` folder into context.

2. **Scan documentation repositories.** Based on the user's selection from step 0, recursively walk every `.adoc` file in the selected repository/repositories. Build a queue of all files found.

3. **Analyse each file (Parallel).** Invoke `/analyze-docs [repo-name] [github-folder-path]` to classify all .adoc files. Skill applies video detection criteria and returns three categorized lists:
   - `VIDEO_NEEDED` — clear UI procedure detected, meets video guidelines
   - `FLAGGED` — content is ambiguous; needs human review before proceeding
   - `SKIP` — no video warranted

4. **Handle flagged files.** If ANY files are flagged, you MUST create a flagged report:
   - **Mandatory:** Save all `FLAGGED` files to a dated CSV report at `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-name]/flagged-report-[YYYY-MM-DD].csv`
   - CSV format with three columns: `.adoc file` | `Path` | `Reason`
   - `.adoc file` column contains just the filename
   - `Path` column contains the full file path
   - `Reason` column contains the flagging reason with any relevant metrics
   - Include a header row
   - This report is REQUIRED even if only 1 file is flagged
   - After creating the report, continue processing `VIDEO_NEEDED` files without waiting for human review

5. **Delegate confirmed video files.** Invoke `/batch-generate [video-needed-files-list] [repo-name] [phase]` to process VIDEO_NEEDED files through three parallel phases:
   
   - **Phase A:** Script generation (4-8 agents per batch)
   - **Phase B:** Peer review (6-10 agents per batch)
   - **Phase C:** CSV logging (batch operation)
   
   Skill handles all batching, parallel execution, and verification. Returns completion status with counts.

6. **Log skipped files.** Add a row to `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-name]/video-script-tracker.csv` for every `SKIP` file with status "No video" so the full audit trail is maintained.

7. **Verify workflow completeness.** Invoke `/verify-workflow [repo-name] [analysis-results]` to ensure:
   - CSV completeness: Script file count matches CSV entry count
   - Flagged reports exist if files were flagged
   - Completion report data is prepared
   
   Skill automatically corrects discrepancies (adds missing CSV entries, removes orphaned entries, creates missing flagged reports). Returns verification status (pass/fail) with details.

8. **Generate and save completion report.** Invoke `/generate-completion-report [run-data] [mode]` to create a comprehensive completion report with all 12 required sections. Skill saves report to appropriate Video Logs folder, displays to user, and returns file path(s).

---

## Completion report format

**MANDATORY:** At workflow completion, generate a comprehensive report using `/generate-completion-report` skill.

**Skill:** `/generate-completion-report [run-data] [mode]`  
**Returns:** File path(s) to saved completion report(s)

**Report structure:** 12 required sections including Header, Configuration, Execution Summary, Analysis Results, Verification Status, Script Quality Metrics, Output Summary, Next Steps, Issues/Warnings, Workflow Statistics, and Report Metadata.

**Location:**
- Single repo: `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-name]/completion-report-[YYYY-MM-DD-HHMM].md`
- Multiple repos: Summary + per-repo reports

See `.claude/skills/generate-completion-report.md` for complete format specification and all 12 required sections.

---

## Video detection criteria

Use the `/analyze-docs` skill to classify .adoc files as VIDEO_NEEDED, FLAGGED, or SKIP.

**Skill:** `/analyze-docs [repo-name] [github-folder-path]`  
**Returns:** JSON with three categorized lists (VIDEO_NEEDED, FLAGGED, SKIP)

**Summary:** Files are classified based on:
- **VIDEO_NEEDED:** UI procedures with 3+ steps, task topics, visible UI elements
- **FLAGGED:** Mixed content, very short procedures (1-2 steps), potential PII
- **SKIP:** Conceptual content, API references, CLI-only, already processed

See `.claude/skills/analyze-docs.md` for complete detection criteria and parallel execution strategy.

---

## Subagent roster

The orchestrator delegates to three specialist subagents. Their definition files live in `.claude/agents/`.

### `script-writer`
**Responsibility:** Generates a complete video script for a given `.adoc` file.
**Inputs:** The `.adoc` file content, the video brief from the orchestrator, the video script template, and the video script samples.
**Output:** A fully formatted video script saved as a `.md` file under `~/Desktop/Video Workflow/Claude Code - Video Script/Video Scripts/[repo-folder]/`.
**Rules:** 
- Must follow video length guidelines (300-400 words)
- Must follow the exact template format (see "Required script format" section below)
- Must follow writing guidelines
- Must not include PII or sensitive product data
- All scripts must use the same 3-column table structure consistently

### `peer-reviewer`
**Responsibility:** Reviews the draft script `.md` file against the writing guidelines and validates format compliance. Edits the file in place.
**Inputs:** The local `.md` file path from the script writer, the writing guidelines, the required script format specification.
**Output:** Edits applied directly to the `.md` file; a brief review summary returned to the orchestrator confirming format compliance.
**Rules:** 
- **Format validation (CRITICAL):** Verify script uses exact 3-column table format: `Section | Voice over | Action on screen`
- Check Section column has: `Introduction` → numbered rows → `Outro`
- Verify Action on screen column has: `Standard intro` (first row) and `Standard outro` (last row)
- Check tone, clarity, sentence length, active voice, and terminology consistency per writing guidelines
- Must not rewrite content that already complies
- If format is incorrect, restructure the entire script to match the template
- Return summary including format validation status

### `csv-logger`
**Responsibility:** Appends one row to `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-name]/video-script-tracker.csv` after each script is approved.
**Inputs:** Repo folder name, `.adoc` file name, video name, short description, script file path, status.
**Output:** One new row in the repository-specific `video-script-tracker.csv` with all six columns populated.
**Rules:**
- Each repository has its own tracker CSV file in `Claude Code - Video Script/Video Logs/[repo-name]/`
- Create the repository folder if it doesn't exist
- Never overwrite existing rows
- **Before logging:** Verify that the script file exists at the provided file path. If the file does not exist, do not create the CSV entry
- If the file is unreachable, save the row data to a local pending log and retry on the next run
- Never log a CSV entry with status "Script created" unless the corresponding `.md` file exists on disk

---


## Parallel execution strategy

Use the `/batch-generate` skill to process VIDEO_NEEDED files through three parallel execution phases:
- **Phase A:** Script generation (4-8 agents per batch)
- **Phase B:** Peer review (6-10 agents per batch)
- **Phase C:** CSV logging (batch operation)

**Skill:** `/batch-generate [video-needed-files-list] [repo-name] [phase]`  
**Returns:** JSON with completion status and counts

**Performance:**
- Sequential: ~3-5 hours for 40+ scripts
- Parallel: ~25-35 min (80-90% faster)

See `.claude/skills/batch-generate.md` for complete implementation details and batching strategies.

---

## Required script format

**CRITICAL:** All video scripts MUST use the exact 3-column table format. See `/write-script` skill for complete format specification.

**Skill:** `/write-script [adoc-file-path] [output-path] [repo-name]`  
**Returns:** Confirmation message with script file path and word count

**Mandatory structure:**
- Markdown table: `Section | Voice over | Action on screen`
- Section column: `Introduction` → numbered rows (`1`, `2`, `3`...) → `Outro`
- Action on screen: `Standard intro` (first row), UI instructions (middle rows), `Standard outro` (last row)
- Voice over: 300-400 words total, conversational tone, second person ("you")

**File naming:** `[Your Brand] - [PRODUCT] - [topic]-video-script.md`

See `.claude/skills/write-script.md` for complete format specification, validation checklist, and quality requirements.

---

## Output file configuration

### Video scripts
- **Location:** `~/Desktop/Video Workflow/Claude Code - Video Script/Video Scripts/[repo-folder]/`
- **Format:** `.md` (markdown file)
- **File naming:** `[Your Brand] - [PRODUCT] - [topic]-video-script.md`
- **Content structure:** 3-column table format (Section | Voice over | Action on screen)

### Tracking files

**Video script tracker (CSV):**
- **Location:** `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-folder]/video-script-tracker.csv`
- **Frequency:** One per repository
- **Columns:** `Docs repo folder` | `adoc file name` | `Video name` | `Short description` | `Script file path` | `Status` | `Date logged`
- **Purpose:** Complete audit trail of all analyzed files and generated scripts

**Flagged report (CSV):**
- **Location:** `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-folder]/flagged-report-[YYYY-MM-DD].csv`
- **Frequency:** One per repository per run (only if files are flagged)
- **Columns:** `.adoc file` | `Path` | `Reason`
- **Purpose:** List of files requiring human review before video decision

**Completion report (Markdown):**
- **Location (single repo):** `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-folder]/completion-report-[YYYY-MM-DD-HHMM].md`
- **Location (multiple repos):** 
  - Summary: `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/completion-report-[YYYY-MM-DD-HHMM].md`
  - Per-repo: `~/Desktop/Video Workflow/Claude Code - Video Script/Video Logs/[repo-folder]/completion-report-[YYYY-MM-DD-HHMM].md`
- **Frequency:** One per workflow run
- **Format:** Structured markdown following exact template (see "Completion report format" section)
- **Purpose:** Comprehensive summary of workflow execution, results, and verification status

---

## Important rules for all agents

- **Script format compliance (CRITICAL):** Every script MUST use the exact 3-column table format from video-script-template.pdf. No exceptions. No variations. Format consistency is mandatory for handoff to video production team. See "Required script format" section for complete specification.
- **GitHub folder path:** Always use the auto-detected or user-confirmed GitHub folder path from step 0a. The workflow automatically searches common locations and validates candidates before prompting for manual entry. Never hardcode a specific path. The workflow must be portable across different user systems.
- **Repository discovery:** Dynamically discover `docs-*` repositories in the detected/configured GitHub folder. Never rely on a hardcoded list. The number and names of repositories will vary by user.
- **Exclusion enforcement:** Always exclude the 4 specified folders (docs-internal-only, docs-archived, docs-site-config, docs-release-notes) from discovery and processing, regardless of the user's GitHub folder content.
- **Repository scope:** Always honor the user's repository selection from step 0c. Never scan or process repositories that were not selected. When Option C (single repository) is chosen, all outputs must be scoped to that repository only.
- **Time tracking:** Record the start time immediately after the user confirms repository selection (step 0c) and calculate the elapsed time when generating the final completion report (step 8). Always report execution time in the final summary to help users understand workflow performance.
- Always read the reference files before taking any action. Never rely on assumptions about the guidelines.
- Never create a script file or CSV row for a file with `SKIP` status (except the audit log row).
- Never include customer names, email addresses, account IDs, or any PII in a script.
- If a file write fails, log the error and continue to the next file. Do not halt the entire run.
- Prefer short, focused scripts over comprehensive ones. One video per distinct procedure.
- If a single `.adoc` file contains multiple distinct UI procedures, create one script per procedure and log each separately.
- **CSV integrity rule:** Never log a CSV entry before verifying the corresponding script file exists. The CSV is an audit trail — entries must reflect actual deliverables, not intended ones.
- **Mandatory verification:** After all script generation completes, verify that script file count matches CSV entry count for each repo folder. If counts mismatch, investigate and correct before considering the workflow complete.
- **Flagged report requirement:** If ANY files are flagged during analysis, you MUST create a dated flagged report file (`flagged-report-[YYYY-MM-DD].csv`). This is not optional. Verify the report exists before completing the workflow. A missing flagged report is a workflow failure.
- **Completion report requirement (MANDATORY):** At the end of every workflow run, you MUST generate and save a completion report following the exact format structure defined in "Completion report format" section. Save as markdown file (`completion-report-[YYYY-MM-DD-HHMM].md`) in appropriate Video Logs folder. Display report to user and confirm file location. A missing completion report is a workflow failure.
