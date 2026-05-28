---
name: analyze-docs
description: Analyze and classify .adoc files for video script generation using parallel execution
---

# Analyze Docs Skill

Systematically analyze all .adoc files in documentation repositories and classify them as VIDEO_NEEDED, FLAGGED, or SKIP based on video detection criteria.

## Usage

```
/analyze-docs [repo-name] [github-folder-path]
```

**Parameters:**
- `repo-name`: Repository name (e.g., "docs-product-a")
- `github-folder-path`: Full path to GitHub folder containing repositories

**Returns:** JSON object with three categorized lists:
```json
{
  "VIDEO_NEEDED": [{"file": "path/to/file.adoc", "reason": "UI procedure for deploying apps - 8 steps"}],
  "FLAGGED": [{"file": "path/to/file.adoc", "reason": "Mixed content - unclear if focused video possible"}],
  "SKIP": [{"file": "path/to/file.adoc", "reason": "Conceptual overview - no UI steps"}],
  "total_files": 150
}
```

---

## Video Detection Criteria

Use these rules — drawn from the video guidelines — to decide whether a `.adoc` file warrants a video:

### Strong Signals for `VIDEO_NEEDED`

- The file contains a numbered step-by-step procedure that involves a UI (buttons, menus, forms, navigation)
- The file is a how-to guide or task topic (not a conceptual or reference topic)
- The steps reference visible UI elements: "Click", "Select", "Navigate to", "Enter", "Toggle", "Open the"
- The procedure has 3 or more steps
- The content would be significantly clearer as a screen recording than as written text

**Examples:**
- "How to deploy an application"
- "Creating an API proxy in API Manager"
- "Configuring OAuth 2.0 authentication"
- "Setting up a load balancer"

### Strong Signals for `SKIP`

- The file is a conceptual overview, architectural explanation, or glossary
- The file is an API reference with no UI interaction
- The content is text-only with no UI procedure (e.g., a configuration file walkthrough via CLI only)
- The file has already been processed in a previous run (check `video-script-tracker.csv`)
- The file belongs to an excluded folder (see Excluded Folders section)

**Examples:**
- "Product Architecture Overview"
- "API Reference Documentation"
- "Glossary of Terms"
- "Release Notes"

### Signals for `FLAGGED` (Human Review Needed)

- The file contains a mix of conceptual content and some UI steps — unclear if a focused video is possible
- The procedure is very short (1–2 steps) — may not justify a full video
- The file references a UI that may contain PII or sensitive demo data
- The orchestrator is unsure whether the content meets the minimum length requirement per the video guidelines

**Examples:**
- File with 10 paragraphs of concept + 2 UI steps at end
- Single-step procedure ("Click Deploy")
- Tutorial using customer-specific data

---

## Excluded Folders

The following folder patterns must **never** be scanned or processed:

**Excluded by name (exact match):**
- `docs-internal-only`
- `docs-archived`
- `docs-site-config`
- `docs-release-notes`

**Repository-specific exclusions:**
- Files in `/partials/` folders (reusable content fragments)
- Files in `/attributes/` folders (variable definitions)
- Navigation files (`nav.adoc`, `navigation.adoc`)
- Index files that only list other files

---

## Parallel Execution Strategy

**IMPORTANT:** Use parallel execution to dramatically improve performance. Instead of analyzing files one at a time sequentially, delegate the entire analysis phase to a specialized agent.

**Sequential (AVOID):** File-by-file ~30-45 min for 150+ files  
**Parallel (USE):** 1 specialized agent analyzes all ~2-3 min

### Implementation Steps

1. **Delegate to specialized agent:**
   - Launch a single `general-purpose` agent with the complete file analysis task
   - Agent receives: list of all .adoc files, video detection criteria, repository context
   - Agent analyzes all files systematically and returns three categorized lists
   - Agent applies video detection criteria consistently across all files

2. **Agent prompt must include:**
   - Complete list of .adoc files to analyze
   - Video detection criteria (VIDEO_NEEDED, FLAGGED, SKIP)
   - Instructions to read sufficient content from each file (first 50-100 lines minimum)
   - Output format: three lists with file paths and classification reasons
   - Repository-specific exclusions (partials, attributes, navigation files)

3. **Analysis output format:**
   - VIDEO_NEEDED: List with file paths and brief reasons (e.g., "UI procedure for deploying apps - 8 steps")
   - FLAGGED: List with file paths and specific flagging reasons
   - SKIP: List with file paths and skip reasons (e.g., "Conceptual overview", "API reference")

4. **After analysis completes:**
   - Parse the agent's categorized lists
   - Validate counts (VIDEO_NEEDED + FLAGGED + SKIP = Total files)
   - Proceed to script generation for VIDEO_NEEDED files
   - Create flagged report if FLAGGED list is not empty

### Key Optimization Points

- Single agent handles all analysis consistently
- Agent reads and classifies files systematically
- Error handling: restart with full list if agent fails
- Use task system to track analysis phase

---

## Analysis Quality Requirements

- Read at least the first 50-100 lines of each file
- Check for UI-related keywords: click, select, navigate, enter, toggle, configure
- Count numbered steps or procedure sections
- Identify file type from heading or metadata (concept, task, reference)
- Cross-reference existing `video-script-tracker.csv` to avoid duplicates
- Apply exclusion rules consistently

---

## Output Format

Return structured JSON with:
- **VIDEO_NEEDED:** Array of objects with `file` path and `reason` string
- **FLAGGED:** Array of objects with `file` path and `reason` string
- **SKIP:** Array of objects with `file` path and `reason` string
- **total_files:** Integer count of all analyzed files
- **exclusions:** Count of files excluded by folder rules

**Validation:** Ensure VIDEO_NEEDED + FLAGGED + SKIP + exclusions = total_files

---

## Error Handling

If analysis fails:
- Log the error with repository name and failure reason
- Return partial results if some files were analyzed
- Recommend manual analysis for failed files
- Report failure in completion report
