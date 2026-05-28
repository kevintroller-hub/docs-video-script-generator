---
name: batch-generate
description: Execute parallel script generation, peer review, and CSV logging in batches for optimal performance
---

# Batch Generate Skill

Process VIDEO_NEEDED files through three parallel execution phases: script generation, peer review, and CSV logging. Uses batching strategy to dramatically improve performance.

## Usage

```
/batch-generate [video-needed-files-list] [repo-name] [phase]
```

**Parameters:**
- `video-needed-files-list`: JSON array of .adoc file paths classified as VIDEO_NEEDED
- `repo-name`: Repository name (e.g., "docs-product-a")
- `phase`: Execution phase ("write", "review", "log", or "all")

**Returns:** JSON object with completion status:
```json
{
  "phase": "all",
  "scripts_generated": 42,
  "scripts_reviewed": 42,
  "csv_entries_logged": 42,
  "failed": [],
  "execution_time": "18 minutes"
}
```

---

## Performance Comparison

**Sequential (AVOID):**
- Script generation: One-by-one ~2-3 hours for 40+ scripts
- Peer review: One-by-one ~1-2 hours for 40+ scripts
- Total: ~3-5 hours

**Parallel (USE):**
- Script generation: 4-8 agents per batch ~15-20 min total
- Peer review: 6-10 agents per batch ~10-15 min total
- Total: ~25-35 min (80-90% faster)

---

## Phase A: Script Generation (Parallel)

### Implementation Steps

1. **Group VIDEO_NEEDED files into logical batches** (4-8 scripts per batch) based on topic similarity

2. **Launch each batch in a single message** with multiple Agent tool calls:
   - Each agent receives: source file path, video guidelines, script format requirements
   - Each agent executes: read source → write script → save file
   - All agents in the batch work simultaneously (not sequentially)
   - Wait for all agents in batch to complete before starting next batch

3. **Each agent prompt must include:**
   - Full source content or path to the .adoc file
   - Video script guidelines and format
   - **Mandatory requirement:** Must use the exact 3-column table format from `/write-script` skill
   - Table structure: `Section | Voice over | Action on screen`
   - Section column values: `Introduction`, numbered rows (`1`, `2`, `3`...), `Outro`
   - Action on screen for Introduction: `Standard intro`
   - Action on screen for Outro: `Standard outro`
   - CX writing guidelines
   - Output file path and naming convention: `[YOUR BRAND] - [PRODUCT] - [topic]-video-script.md`
   - Word count limit (~300-400 words total in Voice over column)

4. **After all batches complete:**
   - Verify all script files exist on disk
   - Proceed to peer review phase
   - Log any failures for retry

### Batching Strategy

- **Batch size:** 4-8 agents per batch (optimal balance between parallelism and resource usage)
- **Grouping logic:** Group by topic similarity when possible (e.g., all deployment-related scripts in one batch)
- **Error handling:** If one agent fails, continue with others; retry failed scripts separately
- **Progress tracking:** Use task system to monitor batch completion

---

## Phase B: Peer Review (Parallel)

### Implementation Steps

1. **Group generated scripts into review batches** (6-10 scripts per batch)

2. **Launch each batch in a single message** with multiple Agent tool calls:
   - Each agent receives: script file path, CX writing guidelines, format validation rules
   - Each agent executes: read script → review against guidelines → edit file → save
   - All agents in the batch work simultaneously (not sequentially)
   - Wait for all agents in batch to complete before starting next batch

3. **Each peer reviewer prompt must include:**
   - Full path to .md script file
   - CX writing guidelines reference from `~/Desktop/Video Workflow/Video Assets/cx-writing-guidelines.pdf`
   - **Mandatory format validation:** 3-column table (Section | Voice over | Action on screen)
   - Check Section column: `Introduction` → numbered rows → `Outro`
   - Verify Action on screen column: `Standard intro` (first row) and `Standard outro` (last row)
   - Review checklist: tone, clarity, sentence length, active voice, terminology consistency
   - Edit file in place (not create new)
   - Return review summary with format validation confirmation

4. **After all review batches complete:**
   - Verify all script files still exist on disk
   - Proceed to CSV logging phase
   - Log any review failures

### Batching Strategy

- **Batch size:** 6-10 agents per batch (reviewers can handle more files than writers)
- **In-place edits:** Reviewers edit existing files, not create new ones
- **Format validation:** Each reviewer MUST confirm 3-column table format compliance
- **Error handling:** If review fails, script is still usable but flagged for manual review
- **Progress tracking:** Use task system to monitor review completion

---

## Phase C: CSV Logging (Batch Operation)

### Implementation Steps

1. **After all scripts are written and reviewed**, create CSV entries in batch:
   - Launch a single `csv-logger` subagent
   - Agent receives: list of all script files with metadata
   - Agent verifies each script file exists on disk before logging
   - Agent appends all rows to `~/Desktop/Video Workflow/Video Logs/[repo-name]/video-script-tracker.csv`

2. **CSV logger must verify:**
   - Each script file exists at the provided file path
   - CSV file has proper header row if newly created
   - No duplicate entries for the same .adoc file
   - All 7 columns populated: `Docs repo folder | adoc file name | Video name | Short description | Script file path | Status | Date logged`

3. **Logging format:**
   - **Docs repo folder:** Repository name (e.g., "docs-product-a")
   - **adoc file name:** Just the filename (e.g., "deploy-app.adoc")
   - **Video name:** Descriptive title (e.g., "Deploy an Application")
   - **Short description:** 1-2 sentence summary
   - **Script file path:** Full path to .md file
   - **Status:** "Script created"
   - **Date logged:** YYYY-MM-DD format

### CSV Integrity Rules

- **NEVER log a CSV entry before verifying the corresponding script file exists**
- The CSV is an audit trail — entries must reflect actual deliverables, not intended ones
- If a script file is missing or unreachable, do not create the CSV entry
- Save failed entries to a pending log for retry on next run

---

## Key Optimization Points

### Script Generation Phase
- Batch size: 4-8 agents per batch
- Each agent writes independently to separate file
- No file conflicts (different output paths)
- Parallel execution reduces time by 75-85%

### Peer Review Phase
- Batch size: 6-10 agents per batch
- In-place edits (no new files created)
- Each reviewer works independently
- Parallel execution reduces time by 70-80%

### CSV Logging Phase
- Batch operation (not per-script)
- Single agent handles all entries
- Verification before logging
- Prevents race conditions on CSV file

---

## Reference Files

All agents must load these reference files from `~/Desktop/Video Workflow/Video Assets/`:

**For script writers:**
- `video-guidelines.pdf`
- `video-script-template.pdf`
- `video-script-samples.pdf`
- `cx-writing-guidelines.pdf`
- `video-script-prompt.md`

**For peer reviewers:**
- `cx-writing-guidelines.pdf`
- `video-script-template.pdf` (for format validation)

---

## Error Handling

### Script Generation Failures
- Log failed file path and error reason
- Continue with remaining batches
- Retry failed scripts in separate batch
- Report failures in completion report

### Peer Review Failures
- Script remains usable but flagged for manual review
- Log review failure reason
- Continue with remaining batches
- Report failures in completion report

### CSV Logging Failures
- Verify script file exists before logging
- If file missing, do not create CSV entry
- Save failed entries to pending log
- Report discrepancies in verification step (step 7)

---

## Output Format

Return structured JSON with:
- **phase:** Which phase was executed ("write", "review", "log", or "all")
- **scripts_generated:** Count of successfully generated script files
- **scripts_reviewed:** Count of successfully reviewed scripts
- **csv_entries_logged:** Count of CSV entries created
- **failed:** Array of failed operations with file paths and reasons
- **execution_time:** Human-readable duration string

**Example:**
```json
{
  "phase": "all",
  "scripts_generated": 42,
  "scripts_reviewed": 42,
  "csv_entries_logged": 42,
  "failed": [],
  "execution_time": "18 minutes",
  "batches": {
    "write": 6,
    "review": 5
  }
}
```

---

## Validation Checklist

Before considering phase complete:
- ✅ All batches executed successfully
- ✅ Script file count matches expected count
- ✅ All script files exist on disk (verified before CSV logging)
- ✅ All scripts use 3-column table format (confirmed by reviewers)
- ✅ CSV entries match script file count
- ✅ No duplicate CSV entries
- ✅ All failures logged for retry
