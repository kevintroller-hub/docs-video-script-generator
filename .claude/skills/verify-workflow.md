---
name: verify-workflow
description: Verify workflow completeness by checking CSV entries, flagged reports, and script file integrity
---

# Verify Workflow Skill

Perform comprehensive post-execution verification to ensure workflow completeness and data integrity across all generated outputs.

## Usage

```
/verify-workflow [repo-name] [analysis-results]
```

**Parameters:**
- `repo-name`: Repository name (e.g., "docs-product-a") or "all" for multiple repos
- `analysis-results`: JSON object with analysis and generation results

**Returns:** JSON object with verification results:
```json
{
  "csv_verification": {
    "status": "pass|fail",
    "script_files": 42,
    "csv_entries": 42,
    "discrepancies": []
  },
  "flagged_report_verification": {
    "status": "pass|fail",
    "flagged_files": 8,
    "report_exists": true,
    "report_path": "~/Desktop/Video Workflow/Video Logs/docs-product-a/flagged-report-2026-04-10.csv"
  },
  "completion_report_verification": {
    "status": "pass|fail",
    "data_prepared": true,
    "required_sections": 12
  },
  "overall_status": "pass|fail",
  "issues": []
}
```

---

## Verification Steps

### Step 1: CSV Completeness Verification

**Purpose:** Ensure the number of script files matches the number of CSV entries with "Script created" status.

**Process:**

1. **Count script files:**
   - Navigate to `~/Desktop/Video Workflow/Video Scripts/[repo-folder]/`
   - Count all `.md` files in directory

2. **Count CSV entries:**
   - Read `~/Desktop/Video Workflow/Video Logs/[repo-folder]/video-script-tracker.csv`
   - Count rows with status "Script created"

3. **Handle discrepancies:**

   **Missing CSV entries (more script files than CSV entries)**
   - Identify script files without corresponding CSV entries
   - Read script file to extract metadata (title, description)
   - Create CSV entry with all required columns
   - Append to `video-script-tracker.csv`

   **Orphaned CSV entries (more CSV entries than script files)**
   - Identify CSV entries without corresponding script files
   - Remove row from CSV file

4. **Re-verify after corrections:**
   - Re-count script files and CSV entries
   - Confirm counts now match
   - If still mismatched, log as workflow failure

**CSV Entry Format:**
```csv
Docs repo folder,adoc file name,Video name,Short description,Script file path,Status,Date logged
docs-product-a,deploy-app.adoc,Deploy Application,[Short description],~/Desktop/Video Workflow/Video Scripts/docs-product-a/[BRAND] - Product A - deploy-app-video-script.md,Script created,2026-04-10
```

**Required Columns:**
1. Docs repo folder (repository name)
2. adoc file name (source file name only)
3. Video name (descriptive title)
4. Short description (1-2 sentences)
5. Script file path (full path to .md file)
6. Status ("Script created" for generated scripts, "No video" for skipped files)
7. Date logged (YYYY-MM-DD format)

---

### Step 2: Flagged Report Verification

**Purpose:** If ANY files were flagged during analysis, ensure the flagged report exists and contains all flagged files.

**Process:**

1. **Check analysis results:**
   - If `flagged_files` count > 0, flagged report is REQUIRED
   - If `flagged_files` count = 0, skip this verification

2. **Verify report exists:**
   - Check for file at `~/Desktop/Video Workflow/Video Logs/[repo-name]/flagged-report-[YYYY-MM-DD].csv`
   - If file does not exist: verification fails

3. **Handle missing report:**
   - If report is missing but files were flagged, create it immediately
   - Write CSV with header row and all flagged entries
   - Save to `~/Desktop/Video Workflow/Video Logs/[repo-name]/flagged-report-[YYYY-MM-DD].csv`

4. **Report format:**
   ```csv
   .adoc file,Path,Reason
   mixed-content.adoc,/path/to/repo/mixed-content.adoc,Mixed content - unclear if focused video possible
   short-procedure.adoc,/path/to/repo/short-procedure.adoc,Very short procedure (only 2 steps)
   ```

**Critical Rule:** A missing flagged report when files were flagged is a WORKFLOW FAILURE.

---

### Step 3: Completion Report Data Preparation

**Purpose:** Verify all data needed for the completion report is available before generation.

**Process:**

1. **Confirm Video Logs folder structure exists** — create if needed

2. **Validate all 12 required sections have data:**
   - Section 1: Header (date, time, timestamp)
   - Section 2: Configuration (GitHub folder, mode, repos, user)
   - Section 3: Execution Summary (start, end, duration)
   - Section 4: Repositories Processed (count, list)
   - Section 5: Analysis Results (statistics, breakdown)
   - Section 6: Verification Status (CSV match, flagged reports)
   - Section 7: Script Quality Metrics (format, word counts)
   - Section 8: Output Summary (files, sizes)
   - Section 9: Next Steps (actions)
   - Section 10: Issues and Warnings (list or "No issues")
   - Section 11: Workflow Statistics (performance, batches)
   - Section 12: Report Metadata (version, model, config)

---

## Automatic Corrections

The verification skill automatically corrects:
- Missing CSV entries (adds them)
- Orphaned CSV entries (removes them)
- Missing flagged report (creates it)
- Missing folder structure (creates directories)

**Rule:** Only fail verification if automatic correction is impossible.

---

## Output Format

Return structured JSON with detailed verification results including status (pass/fail), counts, discrepancies, and all actions taken.

---

## Integration with Workflow

This skill is called in **Step 7** of the main workflow:

```
/verify-workflow [repo-name] [analysis-results]
```

After verification completes:
- If status = "pass": Proceed to Step 8 (generate completion report)
- If status = "fail": Log errors, display to user, halt workflow
- If warnings present: Log warnings, include in completion report, continue

**Verification is MANDATORY** — workflow is not complete until this step passes.
