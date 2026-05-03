---
name: generate-completion-report
description: Generate comprehensive completion report with standardized format for workflow runs
---

# Generate Completion Report Skill

Generate a comprehensive completion report at the end of every workflow run, following the exact standardized format for consistency across all runs.

## Usage

```
/generate-completion-report [run-data] [mode]
```

**Parameters:**
- `run-data`: JSON object containing all workflow execution data
- `mode`: Report mode ("single" or "multiple")

**Returns:** File path(s) to saved completion report(s)

---

## Report File Naming

### Single Repository Run
- **Filename:** `completion-report-[YYYY-MM-DD-HHMM].md`
- **Location:** `~/Desktop/Video Workflow/Video Logs/[repo-name]/`
- **Example:** `completion-report-2026-04-08-1445.md`

### Multiple Repository Run
- **Summary filename:** `completion-report-[YYYY-MM-DD-HHMM].md`
- **Summary location:** `~/Desktop/Video Workflow/Video Logs/`
- **Per-repo filename:** `completion-report-[YYYY-MM-DD-HHMM].md`
- **Per-repo location:** `~/Desktop/Video Workflow/Video Logs/[repo-name]/`

---

## Required Report Structure

Every completion report MUST include these 12 sections in exact order:

### 1. Header
```markdown
# Video Script Workflow - Completion Report

**Run Date:** 2026-04-10  
**Run Time:** 14:30:00  
**Report Generated:** 2026-04-10 15:45:23
```

### 2. Configuration
```markdown
## Configuration

**GitHub Folder:** /Users/username/Documents/GitHub/  
**Workflow Mode:** Single Repository  
**Repository:** docs-[product-name]  
**User:** [your-username]
```

### 3. Execution Summary
```markdown
## Execution Summary

**Start Time:** 2026-04-10 14:30:00  
**End Time:** 2026-04-10 15:45:00  
**Total Execution Time:** 1 hour, 15 minutes, 0 seconds
```

### 4. Repositories Processed
```markdown
## Repositories Processed

**Total Repositories:** 1

- docs-[product-name]
```

### 5. Analysis Results

#### Overall Statistics Table
```markdown
## Analysis Results

### Overall Statistics

| Metric | Count | Percentage |
|--------|-------|------------|
| Total Files Analyzed | 152 | 100.0% |
| Scripts Created | 42 | 27.6% |
| Files Flagged | 8 | 5.3% |
| Files Skipped | 102 | 67.1% |
```

#### Per-Repository Breakdown
```markdown
### Per-Repository Breakdown

#### docs-[product-name]

| Metric | Count |
|--------|-------|
| Files Analyzed | 152 |
| Scripts Created | 42 |
| Files Flagged | 8 |
| Files Skipped | 102 |

**Output Locations:**
- Scripts: `~/Desktop/Video Workflow/Video Scripts/docs-[product-name]/`
- Tracker: `~/Desktop/Video Workflow/Video Logs/docs-[product-name]/video-script-tracker.csv`
- Flagged: `~/Desktop/Video Workflow/Video Logs/docs-[product-name]/flagged-report-2026-04-10.csv`
```

### 6. Verification Status

```markdown
## Verification Status

### Script Files vs CSV Entries

| Repository | Script Files | CSV Entries | Status |
|------------|-------------|-------------|--------|
| docs-[product-name] | 42 | 42 | ✅ Match |

### Flagged Reports

| Repository | Flagged Files | Report Exists | Status |
|------------|---------------|---------------|--------|
| docs-[product-name] | 8 | Yes | ✅ Created |
```

### 7. Script Quality Metrics

```markdown
## Script Quality Metrics

### Format Compliance

- ✅ All scripts use 3-column table format
- ✅ All scripts have Introduction → numbered rows → Outro
- ✅ All scripts use Standard intro/outro
- ✅ All Voice over columns contain 300-400 words
- ✅ All Action on screen columns have UI instructions

### Word Count Distribution

| Word Count Range | Scripts | Percentage |
|------------------|---------|------------|
| 250-299 words | 5 | 11.9% |
| 300-349 words | 18 | 42.9% |
| 350-400 words | 16 | 38.1% |
| 401+ words | 3 | 7.1% |

**Average word count:** 347 words
```

### 8. Output Summary

```markdown
## Output Summary

### Generated Files

**Video Scripts:** 42 files  
**Flagged Reports:** 1 file  
**Tracker CSV:** 1 file (152 entries)  
**Completion Reports:** 1 file

### File Sizes

| File Type | Total Size | Average Size |
|-----------|------------|--------------|
| Video Scripts (.md) | 2.4 MB | 58 KB |
| Flagged Reports (.csv) | 3.2 KB | 3.2 KB |
| Tracker CSV | 28.5 KB | 28.5 KB |
```

### 9. Next Steps

```markdown
## Next Steps

### Immediate Actions

- [ ] Review 8 flagged files in `flagged-report-2026-04-10.csv`
- [ ] Verify script quality for 3 scripts over 400 words
- [ ] Review scripts in `~/Desktop/Video Workflow/Video Scripts/docs-[product-name]/`

### Recommended Actions

- [ ] Share scripts with video production team
- [ ] Update tracker CSV with production status
- [ ] Schedule video recording sessions
```

### 10. Issues and Warnings

```markdown
## Issues and Warnings

✅ No issues detected

OR

⚠️ **Warnings:**
- 3 scripts exceed 400-word limit (see Word Count Distribution)
- 8 files require human review (see Flagged Reports)
```

### 11. Workflow Statistics

```markdown
## Workflow Statistics

### Performance Metrics

**Analysis Phase:** 2 minutes, 30 seconds  
**Script Generation Phase:** 18 minutes, 15 seconds  
**Peer Review Phase:** 12 minutes, 40 seconds  
**CSV Logging Phase:** 45 seconds  
**Verification Phase:** 1 minute, 10 seconds

### Batch Execution

**Script Generation Batches:** 6  
**Scripts per Batch (avg):** 7  
**Peer Review Batches:** 5  
**Scripts per Batch (avg):** 8.4
```

### 12. Report Metadata

```markdown
## Report Metadata

**Generated by:** Claude Code Video Workflow Orchestrator  
**Workflow Version:** 2.0  
**Configuration File:** `~/Desktop/Video Workflow/CLAUDE.md`  
**Skills Used:** `/setup-github`, `/analyze-docs`, `/write-script`, `/batch-generate`, `/verify-workflow`
```

---

## Formatting Requirements

- Use markdown: `#` main heading, `##` sections, `###` subsections, `####` repo names
- Tables for structured data, bullet points for lists, checkboxes for action items
- Status emojis: ✅ success, ❌ failure, ⚠️ warning
- Date format: YYYY-MM-DD, Time: HH:MM:SS (24-hour)
- Durations: "X hours, Y minutes, Z seconds"
- Show actual values (no placeholders), include units (KB, MB), percentages to 1 decimal

---

## Report Generation Process

1. **Gather all workflow data** — statistics, timestamps, file counts, paths
2. **Calculate metrics** — percentages, averages, durations, distributions
3. **Validate completeness** — ensure all 12 sections have data
4. **Generate markdown** — follow exact format structure above
5. **Save to file** — write to appropriate Video Logs folder
6. **Verify file exists** — confirm report was saved successfully
7. **Display to user** — print report to terminal/console
8. **Return file path** — provide location for user reference

---

## Validation Checklist

Before considering report complete:
- ✅ All 12 sections present
- ✅ No placeholder text (TBD, N/A, etc.)
- ✅ All metrics calculated correctly
- ✅ Proper markdown formatting
- ✅ File saved to correct location
- ✅ File exists on disk
- ✅ Report displayed to user
