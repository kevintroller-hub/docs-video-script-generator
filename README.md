# Video Script Generator

This repository contains automated workflows for generating video scripts from AsciiDoc (`.adoc`) documentation files. It supports two AI-powered tools:

- **Claude Code** — A multi-agent pipeline that scans documentation repositories in bulk, detects UI procedures suitable for video, writes scripts, peer-reviews them, and logs everything to a CSV tracker.
- **Cursor** — An interactive skill that generates a single video script from any `.adoc` file on demand, outputting an HTML file ready to paste into Google Docs.

Both workflows share the same `Video Assets/` reference files to keep scripts consistent.

## Table of contents

- [Prerequisites](#prerequisites)
- [Clone the repository](#clone-the-repository)
- [Running the Cursor Video Skill](#running-the-cursor-video-skill)
  - [How it works](#how-it-works)
  - [One-time setup](#one-time-setup)
  - [Using the skill](#using-the-skill)
  - [Your output files](#your-output-files)
- [Running the Claude Code Video Script Generator](#running-the-claude-code-video-script-generator)
  - [How it works](#how-it-works-1)
  - [One-time setup](#one-time-setup-1)
  - [Using the pipeline](#using-the-pipeline)
  - [Your output files](#your-output-files-1)
- [Keeping assets up to date](#keeping-assets-up-to-date)
- [Excluded repositories](#excluded-repositories)
- [Troubleshooting](#troubleshooting)
- [Project structure](#project-structure)
- [Contributing](#contributing)

---

## Prerequisites

Before you begin, make sure you have the following installed on your Mac:

- **Claude Code** — required for the bulk video script generation pipeline
- **Cursor** — required for the interactive single-file video script skill

---

## Clone the repository

```bash
git clone https://github.com/[your-username]/video-script-generator.git
cd video-script-generator
```

Verify your `Video Assets/` folder contains all the reference files both workflows need:

```
Video Assets/
  ├── video-guidelines.pdf
  ├── video-script-template.pdf
  ├── video-script-samples.pdf
  ├── cx-writing-guidelines.pdf
  └── video-script-prompt.md
```

Check that all five files are present. If any are missing, pull the latest version of the repo (see [Keeping assets up to date](#keeping-assets-up-to-date)).

---

## Running the Cursor Video Skill

### How it works

This repository includes a **Cursor Skill** for generating a single video script from an `.adoc` file directly inside Cursor. Unlike the Claude Code pipeline (which bulk-processes entire repositories), the Cursor Skill is designed for writers who want to generate a script for one specific file while they are already working in a documentation repository.

The skill lives in the `Cursor - Video Script/Cursor Skills/` folder of this repository. Because Cursor discovers skills from the `.cursor/skills/` directory inside the active workspace, you need to link it to your docs repository so Cursor can find it.

### One-time setup

After cloning this repository (see [Clone the repository](#clone-the-repository)), create a symbolic link so the skill is available in every Cursor workspace on your machine.

Run this command once in Terminal:

```bash
ln -s ~/Desktop/Video\ Workflow/Cursor\ -\ Video\ Script/Cursor\ Skills ~/.cursor/skills-cursor/generate-video-script
```

This creates a single link in your user-level Cursor skills directory. The skill is now automatically available in **every** docs repository you open in Cursor — no per-repo configuration needed.

To verify the link was created:

```bash
ls -la ~/.cursor/skills-cursor/ | grep generate-video-script
```

You should see an entry pointing back to `~/Desktop/Video Workflow/Cursor - Video Script/Cursor Skills`.

> **Note:** Because the symlink points to this repository, any updates to the skill (via `git pull`) are reflected immediately — no need to re-create the link.

### Using the skill

1. Open your docs repository in Cursor (e.g., `docs-runtime-manager`)
2. Navigate to the `.adoc` file you want to generate a script for
3. Open the Cursor agent and ask it to generate a video script for the file, for example:

```
Generate a video script for this file
```

The agent picks up the skill automatically, reads the `.adoc` content, checks video eligibility, and produces an HTML script file ready to paste into Google Docs.

### Your output files

After the skill runs, the generated script is saved to:

| Output | Location |
|--------|----------|
| Generated script | `Cursor - Video Script/Video Scripts/[Your Brand] - <Product Name> - <adoc-file-name>.html` |

The HTML file can be pasted directly into Google Docs or uploaded to Google Drive, which auto-converts it to a Google Doc.

---

## Running the Claude Code Video Script Generator

### How it works

An orchestrator agent reads your documentation files, identifies content where a video would help (specifically UI procedures), and delegates to specialist subagents and skills:

1. **Script writer** — generates a formatted video script from the `.adoc` content
2. **Peer reviewer** — checks the script against writing guidelines and edits it in place
3. **CSV logger** — appends a row to the tracker file with the script details and file path

The workflow also uses six specialized skills to keep the pipeline efficient and modular:

| Skill | Purpose |
|-------|---------|
| `/setup-github` | Auto-detect GitHub folder, discover repos, prompt for selection |
| `/analyze-docs` | Classify `.adoc` files as VIDEO_NEEDED, FLAGGED, or SKIP |
| `/write-script` | Generate scripts with mandatory 3-column table format |
| `/batch-generate` | Execute parallel script generation, peer review, and CSV logging |
| `/verify-workflow` | Verify CSV completeness, flagged reports, and data integrity |
| `/generate-completion-report` | Create comprehensive 12-section completion reports |

### One-time setup

Open Terminal, navigate to the Claude Code workflow folder, and launch Claude Code:

```bash
cd ~/Desktop/Video\ Workflow/Claude\ Code\ -\ Video\ Script
claude
```

Claude Code will automatically read `CLAUDE.md` and load the project context. The `.claude/` agents and skills are also in this folder.

### Using the pipeline

Once Claude Code is running, type:

```
Run the video script generator
```

> **Note:** Running against all repositories can consume a large number of tokens. Start with a single repository to validate the pipeline first.

You can also target a specific repository directly:

```
Run the video script generator for docs-runtime-manager
```

#### GitHub folder auto-detection

When you start the workflow, it automatically searches for your GitHub folder — no manual path entry required. The workflow:

1. **Checks for a saved configuration** at `~/Desktop/Video Workflow/.config`
   - If found, asks: "Using GitHub folder: `/path/to/GitHub/` — continue? (Y/n)"
2. **Searches common locations** on your system (macOS/Linux/Windows)
3. **Validates candidates** by confirming each contains at least 3 `docs-*` repositories
4. **Presents results** based on what was found:

| Scenario | Behavior |
|----------|----------|
| One location found | Confirms with you before proceeding |
| Multiple locations found | Shows numbered list — you select one |
| No location found | Prompts you to enter the path manually |

5. **Optionally saves the path** for future runs (skips detection on next run)

#### Repository selection

After the GitHub folder is confirmed, the workflow asks which repositories to process:

```
Which repositories would you like to process?

Options:
  A) All repositories (all discovered repos, excluding 4 excluded folders)
  B) Select specific repositories (you will choose from the discovered list)
  C) Single repository (provide the repository name, e.g., docs-runtime-manager)

Please type A, B, or C to continue.
```

**Option A** — Processes all discovered `docs-*` repositories.

**Option B** — Displays a numbered list. Select by name, number, or a mix:
```
docs-api-manager,docs-runtime-manager
2,5,10
docs-api-manager,5
```

**Option C** — Processes a single repository. The workflow validates it exists before proceeding.

> **Tip:** Always start with option C on a single repository to validate the pipeline before running it across multiple repos.

#### What happens next

The pipeline runs automatically. You will see progress updates in the terminal as each file is processed:

```
[✓ SKIP]         docs-runtime-manager / overview.adoc
[✓ VIDEO_NEEDED] docs-runtime-manager / deploy-application.adoc
[⚠ FLAGGED]      docs-runtime-manager / manage-settings.adoc
```

When the run is complete, the orchestrator prints a completion report showing how many scripts were created, skipped, flagged, execution time, verification status, and output file locations.

### Your output files

After a run, find your files here:

| Output | Location |
|--------|----------|
| Generated scripts | `Claude Code - Video Script/Video Scripts/docs-<repo-name>/` |
| CSV tracker | `Claude Code - Video Script/Video Logs/docs-<repo-name>/video-script-tracker.csv` |
| Flagged report | `Claude Code - Video Script/Video Logs/docs-<repo-name>/flagged-report-YYYY-MM-DD.csv` |
| Completion report | `Claude Code - Video Script/Video Logs/docs-<repo-name>/completion-report-YYYY-MM-DD-HHMM.md` |

For multi-repository runs, a summary completion report is also saved to `Claude Code - Video Script/Video Logs/completion-report-YYYY-MM-DD-HHMM.md`.

#### Reviewing flagged files

Files the orchestrator is unsure about are saved to the flagged report CSV. Open this file to review each entry — the `Reason` column explains why the file was flagged. Decide whether to add it to the next run as a targeted VIDEO_NEEDED file or skip it.

---

## Keeping assets up to date

The `Video Assets/` folder is maintained centrally in this repository. When guidelines, templates, or samples are updated, the changes are committed here so every writer gets them automatically.

**Before starting a new session**, pull the latest version of the repo to make sure your local assets are current:

```bash
cd ~/Desktop/Video\ Workflow
git pull origin main
```

If you need to manually replace an asset file (for example, if you receive an updated PDF directly), simply drop the new file into `Video Assets/` replacing the existing one. The filename must remain exactly the same for the workflow to find it.

---

## Excluded repositories

The following repositories are excluded from the Claude Code workflow. The orchestrator will never scan or process them:

- `dev-docs-internal-integration-internal`
- `docs-superagent`
- `docs-site-playbook`
- `docs-release-notes`

If you explicitly request an excluded repository (Option C), the workflow will warn you and prompt for a different selection.

---

## Troubleshooting

**Claude Code says "command not found"**
Add Claude Code to your PATH:
```bash
export PATH="$HOME/.local/bin:$PATH"
```
Add this line to your `~/.zshrc` file to make it permanent.

**401 Authentication error on startup**
Your API key may be missing or incorrect. Run:
```bash
claude config set apiKey YOUR_API_KEY
```
If the error persists, contact your IT team — your company proxy URL may also need to be configured.

**GitHub folder not detected automatically**
If the workflow cannot find your GitHub folder, it will prompt you to enter the path manually. Enter the full path (e.g., `/Users/username/Documents/GitHub/`) and optionally save it for future runs. The saved path is stored at `~/Desktop/Video Workflow/.config`.

**Reference file not found error**
The orchestrator could not read one of the files in `Video Assets/`. Check that all five files are present and that the filenames match exactly what is listed in the [Clone the repository](#clone-the-repository) section. Run `git pull` to restore any missing files.

**Scripts folder not created**
The `Claude Code - Video Script/Video Scripts/` and `Claude Code - Video Script/Video Logs/` folders are created automatically on the first run. If they are missing after a run, check the terminal output for file write errors.

**Context limit reached on large repositories**
If Claude Code hits its context limit mid-run, restart the session and use option B or C to process one repository at a time instead of all repositories at once.

**CSV count mismatch after a run**
The workflow includes a mandatory verification step (step 7) that automatically corrects CSV discrepancies — adding missing entries and removing orphaned ones. If the completion report shows a mismatch that was not corrected, re-run the workflow on the affected repository.

---

## Project structure

```
video-script-generator/
  ├── README.md                               ← this file
  ├── .gitignore
  │
  ├── Video Assets/                           ← shared reference files (both workflows)
  │     ├── video-guidelines.pdf
  │     ├── video-script-template.pdf
  │     ├── video-script-samples.pdf
  │     ├── cx-writing-guidelines.pdf
  │     ├── video-script-prompt.md
  │     ├── script-template.md
  │     └── script-sample.md
  │
  ├── Claude Code - Video Script/             ← Claude Code workflow
  │     ├── CLAUDE.md                         ← project rules, auto-read by Claude Code
  │     ├── .claude/                          ← agents and skills
  │     │     ├── agents/
  │     │     │     ├── orchestrator.md
  │     │     │     ├── script-writer.md
  │     │     │     ├── peer-reviewer.md
  │     │     │     └── csv-logger.md
  │     │     └── skills/
  │     │           ├── setup-github.md
  │     │           ├── analyze-docs.md
  │     │           ├── write-script.md
  │     │           ├── batch-generate.md
  │     │           ├── verify-workflow.md
  │     │           └── generate-completion-report.md
  │     ├── Video Scripts/                    ← generated .md scripts (local only)
  │     └── Video Logs/                       ← tracker, flagged reports, completion reports (local only)
  │
  └── Cursor - Video Script/                  ← Cursor workflow
        ├── Cursor Skills/                    ← skill definition (symlinked by writers)
        │     └── SKILL.md
        └── Video Scripts/                    ← generated .html scripts (local only)
              └── [Your Brand] - <Product> - <file>.html
```

---

## Contributing

To update the Video Assets, agent definitions, or skill files, create a branch and open a pull request following your team's standard Git workflow. Do not commit generated scripts, CSV tracker files, or completion reports — these are listed in `.gitignore` and should remain local to each writer's machine.
