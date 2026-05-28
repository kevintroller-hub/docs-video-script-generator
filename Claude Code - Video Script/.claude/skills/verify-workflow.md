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
   - Record count

2. **Count CSV entries:**
   - Read `~/Desktop/Video Workflow/Video Logs/[repo-folder]/video-script-tracker.csv`
   - Count rows with status "Script created"
   - Record count

3. **Compare counts:**
   - If counts match: verification passes ✅
   - If counts do not match: identify discrepancy ❌

4. **Handle discrepancies:**

   **Scenario A: Missing CSV entries (more script files than CSV entries)**
   - Identify script files without corresponding CSV entries
   - For each missing entry:
     - Read script file to extract metadata (title, description)
     - Create CSV entry with all required columns
     - Append to `video-script-tracker.csv`
   - Log action: "Added X missing CSV entries"

   **Scenario B: Orphaned CSV entries (more CSV entries than script files)**
   - Identify CSV entries without corresponding script files
   - For each orphaned entry:
     - Verify script file path does not exist
     - Remove row from CSV file
     - Log removed file path
   - Log action: "Removed X orphaned CSV entries"

   **Scenario C: Mixed discrepancies (both missing and orphaned)**
   - Handle missing entries first (add to CSV)
   - Then handle orphaned entries (remove from CSV)
   - Log both actions

5. **Re-verify after corrections:**
   - Re-count script files and CSV entries
   - Confirm counts now match
   - If still mismatched, log as workflow failure

**CSV Entry Format:**
```csv
Docs repo folder,adoc file name,Video name,Short description,Script file path,Status,Date logged
docs-product-a,deploy-app.adoc,Deploy an Application,Learn how to deploy applications,~/Desktop/Video Workflow/Video Scripts/docs-product-a/[YOUR BRAND] - Product A - deploy-app-video-script.md,Script created,2026-04-10
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
   - If `flagged_files` count = 0, skip this verification (report not needed)

2. **Verify report exists:**
   - Check for file at `~/Desktop/Video Workflow/Video Logs/[repo-name]/flagged-report-[YYYY-MM-DD].csv`
   - If file does not exist: verification fails ❌
   - If file exists: proceed to content verification

3. **Verify report content:**
   - Read flagged report CSV
   - Count entries (excluding header row)
   - Compare count to `flagged_files` count from analysis
   - Verify CSV has required columns: `.adoc file | Path | Reason`

4. **Handle missing report:**
   - If report is missing but files were flagged:
     - Create flagged report immediately
     - Retrieve list of flagged files from analysis results
     - Write CSV with header row and all flagged entries
     - Save to `~/Desktop/Video Workflow/Video Logs/[repo-name]/flagged-report-[YYYY-MM-DD].csv`
     - Log action: "Created missing flagged report with X entries"

5. **Report format:**
   ```csv
   .adoc file,Path,Reason
   mixed-content.adoc,/path/to/repo/modules/ROOT/pages/mixed-content.adoc,Mixed content - unclear if focused video possible (10 paragraphs concept + 2 UI steps)
   short-procedure.adoc,/path/to/repo/modules/ROOT/pages/short-procedure.adoc,Very short procedure (only 2 steps) - may not justify full video
   ```

**Critical Rule:** A missing flagged report when files were flagged is a WORKFLOW FAILURE.

---

### Step 3: Completion Report Data Preparation

**Purpose:** Verify all data needed for the completion report is available and prepared before generation.

**Process:**

1. **Confirm Video Logs folder structure exists:**
   - Check `~/Desktop/Video Workflow/Video Logs/[repo-name]/` exists
   - If not, create directory structure
   - Log action if created

2. **Prepare report data:**
   - Gather all statistics (files analyzed, scripts created, flagged, skipped)
   - Calculate execution time (end_time - start_time)
   - Collect all file counts and paths
   - Compile performance metrics (analysis time, generation time, review time)
   - Calculate percentages and averages

3. **Validate all required sections have data:**
   - Section 1: Header (date, time, timestamp) ✅
   - Section 2: Configuration (GitHub folder, mode, repos, user) ✅
   - Section 3: Execution Summary (start, end, duration) ✅
   - Section 4: Repositories Processed (count, list) ✅
   - Section 5: Analysis Results (statistics, breakdown) ✅
   - Section 6: Verification Status (CSV match, flagged reports) ✅
   - Section 7: Script Quality Metrics (format, word counts) ✅
   - Section 8: Output Summary (files, sizes) ✅
   - Section 9: Next Steps (actions) ✅
   - Section 10: Issues and Warnings (list or "No issues") ✅
   - Section 11: Workflow Statistics (performance, batches) ✅
   - Section 12: Report Metadata (version, model, config) ✅

4. **Ensure report follows exact format:**
   - Verify format structure from `/generate-completion-report` skill
   - Confirm all sections will use proper markdown formatting
   - Check that no placeholder text (TBD, N/A) will be used

5. **Flag any missing data:**
   - If any required data is missing, log warning
   - Attempt to calculate or infer missing values
   - If critical data unavailable, mark verification as failed

---

## Verification Matrix

| Check | Pass Criteria | Fail Criteria | Action on Fail |
|-------|---------------|---------------|----------------|
| CSV Completeness | Script count = CSV entry count | Counts mismatch | Add missing entries or remove orphaned entries |
| Flagged Report Exists | Report exists if files flagged | Report missing when files flagged | Create report with all flagged files |
| Flagged Report Content | Entry count matches flagged count | Entry count mismatch | Regenerate report with correct entries |
| Completion Data Prepared | All 12 sections have data | Missing critical data | Attempt to calculate/infer; log warning if impossible |
| Folder Structure | Video Logs/[repo]/ exists | Directory missing | Create directory structure |

---

## Output Format

Return structured JSON with detailed verification results:

```json
{
  "csv_verification": {
    "status": "pass",
    "script_files": 42,
    "csv_entries": 42,
    "discrepancies": [],
    "actions_taken": []
  },
  "flagged_report_verification": {
    "status": "pass",
    "flagged_files": 8,
    "report_exists": true,
    "report_path": "~/Desktop/Video Workflow/Video Logs/docs-product-a/flagged-report-2026-04-10.csv",
    "entry_count": 8,
    "actions_taken": []
  },
  "completion_report_verification": {
    "status": "pass",
    "data_prepared": true,
    "required_sections": 12,
    "missing_data": [],
    "folder_structure_verified": true
  },
  "overall_status": "pass",
  "issues": [],
  "warnings": [],
  "actions_taken": [
    "Verified CSV completeness: 42 scripts = 42 CSV entries",
    "Verified flagged report exists with 8 entries",
    "Prepared completion report data for 12 sections"
  ]
}
```

---

## Error Handling

### Critical Failures (Workflow Failure)

These issues mark the workflow as FAILED:
- CSV completeness check fails and cannot be corrected
- Flagged report missing when files were flagged and cannot be created
- Completion report data cannot be prepared (missing critical data)

### Non-Critical Issues (Warnings)

These issues are logged but do not fail the workflow:
- Minor data discrepancies that can be automatically corrected
- Missing optional metadata
- Performance metrics unavailable

### Automatic Corrections

The verification skill should automatically correct:
- Missing CSV entries (add them)
- Orphaned CSV entries (remove them)
- Missing flagged report (create it)
- Missing folder structure (create directories)

**Rule:** Only fail verification if automatic correction is impossible.

---

## Validation Checklist

Before returning verification results:
- ✅ CSV verification complete (counts compared)
- ✅ Flagged report verification complete (exists if needed)
- ✅ Completion report data prepared (all 12 sections)
- ✅ All discrepancies identified
- ✅ All automatic corrections attempted
- ✅ Overall status determined (pass/fail)
- ✅ Actions taken logged

---

## Integration with Workflow

This skill is called in **Step 7** of the main workflow:

**Step 7: Verify workflow completeness**
```
/verify-workflow [repo-name] [analysis-results]
```

After verification completes:
- If status = "pass": Proceed to Step 8 (generate completion report)
- If status = "fail": Log errors, display to user, halt workflow
- If warnings present: Log warnings, include in completion report, continue

**Verification is MANDATORY** - workflow is not complete until this step passes.

---

## Logging

All verification actions should be logged:
- **Info:** "Verified CSV completeness: X scripts = X CSV entries"
- **Warning:** "Found 2 orphaned CSV entries, removed"
- **Error:** "Flagged report missing when 8 files were flagged"
- **Action:** "Created missing flagged report with 8 entries"
- **Success:** "All verification checks passed"

These logs are included in the completion report for audit trail.
