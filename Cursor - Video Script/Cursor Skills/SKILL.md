---
name: generate-video-script
description: >-
  Generate a video script from an AsciiDoc (.adoc) documentation file. Use when
  the user asks to create a video script, generate narration for a doc page,
  write a video voiceover, or produce video content from an .adoc file.
---

# Generate Video Script from AsciiDoc

Generate a paste-ready video script for Google Docs from an `.adoc` documentation file. The script follows the [Your Brand] video script template and writing guidelines.

## Workflow

### Step 1: Read the Target `.adoc` File

Read the full contents of the `.adoc` file the user provides. Identify:

- The topic and product area
- Whether it describes a concept, procedure, or both
- Key sections, steps, and UI interactions

### Step 2: Check Video Eligibility

Evaluate the content against these criteria. At least one must apply:

| Criteria | Description |
|----------|-------------|
| **Complexity** | A concept or procedure is hard to explain or visualize using text alone |
| **Procedure Flow** | A complex procedure has multiple steps or crosses different products and UIs |
| **Length** | A topic takes many pages of text to explain but isn't difficult to grasp |

**Not eligible** if the content is primarily:
- Extensive code or reference material
- A feature likely to change within 3–6 months
- Better suited as a training or marketing video

If the content is **not eligible**, tell the user why and stop.

### Step 3: Read Reference Files

Read these reference files from the shared `Video Assets` folder to inform the script:

1. **Video script prompt** — the style and structural rules:
   [../../Video Assets/video-script-prompt.md](../../Video Assets/video-script-prompt.md)

2. **Script template** — the output structure:
   [../../Video Assets/script-template.md](../../Video Assets/script-template.md)

3. **Script sample** — a completed example:
   [../../Video Assets/script-sample.md](../../Video Assets/script-sample.md)

4. **CX writing guidelines** — tone, terminology, and style rules:
   [../../Video Assets/cx-writing-guidelines.pdf](../../Video Assets/cx-writing-guidelines.pdf)

### Step 4: Generate the Video Script

Follow these rules when writing:

#### Length and Pacing
- Target **~400 words** of narration (about 2 minutes at 150 words/min)
- Total video run time must be **3 minutes or less**

#### Voice and Tone
- Write for an off-screen narrator — a friend explaining the topic
- Use a friendly, slightly casual, helpful tone without being chatty
- Write mainly in **second person** ("you") for user actions
- Use **third person** ("we") sparingly, only to refer to the product
- Limit tech jargon to what's necessary; prefer clear, conversational language

#### Style Rules
- Write simple, declarative sentences (subject, verb, object)
- Avoid long introductory phrases
- Avoid the verbs "enabling" and "ensuring" — use stronger alternatives
- Don't use permissive language ("lets the user" or "allows the user to")
- Remove participle phrases and dangling modifiers at sentence ends
- Use adverbs and adjectives very sparingly
- Don't name the video or reference a video's name
- Avoid repetition of words, phrases, and information

#### Script Structure

1. **Greeting** — Start with "Hi there!" or similar casual greeting
2. **Introduction** — Introduce the topic the video covers
3. **Learning objectives** — Give 2–3 things the viewer will learn
4. **Prerequisites** (if applicable) — List what the viewer needs before starting
5. **Core content** — Guide the viewer through the concept or task in small, easy-to-understand increments. Use ALL-CAPS section headers (e.g., "ADD PROPERTIES", "CONFIGURE SETTINGS") to mark major segments
6. **Closing** — End with exactly: *"To learn more, check out these resources. Or "* — Nothing after this line.

### Step 5: Format the Output

Output the script in the **template format** ready to paste into Google Docs. Use this structure:

```
Title:
[Derived from the .adoc topic — concise, action-oriented]

Short Description:
[One sentence describing what the video shows]

Technical Writer:  [Leave blank]
Org ID:            [Leave blank]
Username:          [Leave blank]
Password:          [Leave blank]
Is 2FA enabled?    [Leave blank]
Video destination: Help topic
Slack channel:     #[your-video-channel]

Stakeholders:      [Leave blank]

Production Checklist:
☐ Script approved by editorial team
☐ Action On Screen column has accurate and clear instructions
☐ Test org is functional and has presentable data
☐ Final title and short description filled out
☐ All stakeholders have given feedback on the script

Resources:
● [Link to the corresponding documentation page]

---

Script

#  | Audio                              | Action On Screen                    | Comments
---|--------------------------------------|--------------------------------------|----------
1  | [Greeting + intro + objectives]     | Slide presentation                  |
2  | [Prerequisites]                     | Slide presentation                  |
3  | [SECTION HEADER                     | [Describe the UI actions            |
   |  Narration for first task...]       |  shown on screen...]                |
4  | [SECTION HEADER                     | [Describe the UI actions            |
   |  Narration for next task...]        |  shown on screen...]                |
...
N  | To learn more, check out these      | Outro Slide.                        |
   | resources.                          |                                     |
```

### Step 6: CX Style Validation

Before presenting the script, validate it against these CX guidelines:

- [ ] No use of "allow", "let", or "enable" (when referencing users)
- [ ] No permissive language
- [ ] "Agent" used only in Agentforce context; human agents called "rep", "executive", "supervisor"
- [ ] Acronyms from the common list (API, UI, URL, etc.) not spelled out
- [ ] Domain-specific acronyms spelled out at first mention
- [ ] No ampersands — spell out "and"
- [ ] Checkbox, not check box
- [ ] "Issue" or "problem", not "bug" or "defect"
- [ ] "Make sure" preferred over "ensure"; imperative verb even better
- [ ] No "above" or "below" for location references — use "preceding", "following", "next"

If any violations are found, fix them before presenting the final script.

## Output

Save the script as an **HTML file** in `../Video Scripts/` using the naming convention `[Your Brand] - <Product Name> - <adoc-file-name>.html` (e.g., `[Your Brand] - [Product Name] - deploy-application.html`). HTML tables paste cleanly into Google Docs as real tables. The user can also upload the HTML file directly to Google Drive, which auto-converts it to a Google Doc.

Use proper HTML `<table>` elements for both the metadata section and the script table. Include minimal inline CSS for readability (borders, padding, font).

After saving the file, include a brief summary:

- **Word count** of the Audio column narration
- **Estimated run time** (word count ÷ 150 wpm)
- **Eligibility criteria met** (which of the three criteria apply)
