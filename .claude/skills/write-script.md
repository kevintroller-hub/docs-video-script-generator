---
name: write-script
description: Generate video scripts with mandatory 3-column table format validation
---

# Write Script Skill

Generate a complete video script from an AsciiDoc (.adoc) source file following the exact 3-column table format required by the video production team.

## Usage

```
/write-script [adoc-file-path] [output-path] [repo-name]
```

**Parameters:**
- `adoc-file-path`: Full path to source .adoc file
- `output-path`: Full path where script .md file should be saved
- `repo-name`: Repository name for context (e.g., "docs-product-a")

**Returns:** Confirmation message with script file path and word count

---

## Required Script Format

**CRITICAL:** All video scripts MUST follow this exact template format. This ensures consistency across all scripts and proper handoff to the video production team.

### Mandatory Format Structure

Every script file must be a **markdown document** with this exact structure:

#### 1. Header Metadata (Optional for Automation Workflow)

Scripts should include:
- Title as a markdown heading at the top
- Short description as a subtitle

#### 2. Script Body (MANDATORY - 3-Column Table Format)

All scripts must use this **exact markdown table structure**:

```markdown
| Section | Voice over | Action on screen |
|---------|-----------|------------------|
| Introduction | [Narrator script text: "Hi there! Today we're going to..."] | Standard intro |
| 1 | [Narrator script for step 1] | [UI instructions: "Sign in to [Your Platform]..."] |
| 2 | [Narrator script for step 2] | [UI instructions: "Click [Menu Name]..."] |
| 3 | [Narrator script for step 3] | [UI instructions: "Select [Option Name]..."] |
| 4 | [Narrator script for step 4] | [UI instructions: "Click the [Tab Name] tab..."] |
| 5 | [Narrator script for step 5] | [UI instructions: "Click [Button Name]..."] |
| 6 | [Narrator script for step 6] | [UI instructions: "Click Apply Changes..."] |
| Outro | [Closing text: "To learn more, check out these resources."] | Standard outro |
```

### Column Specifications

**Column 1: Section**
- First row: `Introduction`
- Middle rows: Numbered sequentially (`1`, `2`, `3`, `4`, `5`, `6`, etc.)
- Last row: `Outro`
- Keep section numbers simple (just the number, no additional text)

**Column 2: Voice Over**
- Contains the complete narrator script text
- Written in friendly, conversational tone
- Uses second person ("you") for user actions
- Uses first person plural ("we") sparingly for product references
- 300-400 words total across all rows
- Introduction row: greeting + topic + learning objectives
- Numbered rows: step-by-step procedure guidance
- Outro row: standard closing phrase

**Column 3: Action on Screen**
- First row: `Standard intro`
- Middle rows: Specific UI instructions for video production team
  - Clear, action-oriented instructions (e.g., "Click Applications", "Select the Properties tab")
  - References to specific UI elements (buttons, menus, fields)
  - Brief descriptions of what to show on screen
- Last row: `Standard outro`

**Example Structure:**
```markdown
# [Title]
[Short description]

| Section | Voice over | Action on screen |
|---------|-----------|------------------|
| Introduction | [Greeting + topic + objectives] | Standard intro |
| 1 | [Step 1 narrator text] | [UI actions for step 1] |
| 2 | [Step 2 narrator text] | [UI actions for step 2] |
| ... | ... | ... |
| Outro | To learn more, check out these resources. | Standard outro |
```

### Format Validation Checklist

Before considering a script complete, verify:
- ✅ Uses markdown table format with 3 columns
- ✅ Column headers are: `Section | Voice over | Action on screen`
- ✅ First row Section value: `Introduction`
- ✅ Middle row Section values: Sequential numbers (`1`, `2`, `3`...)
- ✅ Last row Section value: `Outro`
- ✅ Introduction Action on screen: `Standard intro`
- ✅ Outro Action on screen: `Standard outro`
- ✅ Voice over column contains complete narrator script (300-400 words total)
- ✅ Action on screen column contains specific UI instructions for numbered sections

### Important Notes

- **Consistency is critical:** Every script must follow this exact format
- **No variations allowed:** Do not use 2-column formats, plain text formats, or other structures
- **Table alignment:** Use markdown table syntax with pipe separators (`|`)
- **Section numbering:** Use simple numbers (1, 2, 3) not "Step 1", "Section 1", etc.
- **Standard rows:** Introduction and Outro rows are mandatory for all scripts

---

## Script Generation Process

1. **Read source .adoc file** - Extract UI procedure steps
2. **Load reference materials** from `~/Desktop/Video Workflow/Video Assets/`:
   - `video-guidelines.pdf`
   - `video-script-template.pdf`
   - `video-script-samples.pdf`
   - `cx-writing-guidelines.pdf`
   - `video-script-prompt.md`
3. **Generate script** following exact 3-column table format
4. **Validate format** against checklist above
5. **Save to output path** as .md file
6. **Return confirmation** with file path and word count

---

## File Naming Convention

**Format:** `[BRAND] - [PRODUCT] - [topic]-video-script.md`

**Examples:**
- `[BRAND] - [Product A] - deploy-application-video-script.md`
- `[BRAND] - [Product B] - create-api-proxy-video-script.md`

---

## Quality Requirements

- Must follow video length guidelines (300-400 words in Voice over column)
- Must follow writing guidelines (tone, clarity, active voice)
- Must not include PII or sensitive product data
- Must use second person ("you") for user actions
- Must be clear, concise, and conversational
- Must match the learning objective from the source .adoc file

---

## Error Handling

If script generation fails:
- Log the error with file path and reason
- Continue to next file (do not halt workflow)
- Report failure in completion report
