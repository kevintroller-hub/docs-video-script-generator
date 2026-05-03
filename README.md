# Video Script Generator

This repository contains an automated workflow for generating video scripts from AsciiDoc (`.adoc`) documentation files. It uses Claude Code and a multi-agent pipeline to scan documentation repositories, detect UI procedures suitable for video, write scripts, peer-review them, and log everything to a CSV tracker.

## Table of contents

- [How it works](#how-it-works)
- [Prerequisites](#prerequisites)
- [First-time setup](#first-time-setup)
  - [Step 1 — Clone the repository](#step-1--clone-the-repository)
  - [Step 2 — Verify your Video Assets folder](#step-2--verify-your-video-assets-folder)
- [Running the workflow](#running-the-workflow)
  - [Start Claude Code](#start-claude-code)
  - [Trigger the pipeline](#trigger-the-pipeline)
  - [GitHub folder auto-detection](#github-folder-auto-detection)
  - [Repository selection](#repository-selection)
  - [What happens next](#what-happens-next)
  - [Your output files](#your-output-files)
  - [Reviewing flagged files](#reviewing-flagged-files)
- [Keeping assets up to date](#keeping-assets-up-to-date)
- [Excluded repositories](#excluded-repositories)
- [Troubleshooting](#troubleshooting)
- [Project structure](#project-structure)
- [Contributing](#contributing)

---

## How it works

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

Scripts are saved as `.md` files under `Video Scripts/`. A CSV tracker, flagged report, and completion report are saved under `Video Logs/`.

---

## Prerequisites

Before you begin, make sure you have Claude Code installed and running on your machine.

---

## First-time setup

Follow these steps once after cloning the repo.

### Step 1 — Clone the repository

```bash
git clone https://github.com/[your-org]/video-script-generator.git
cd video-script-generator
```

### Step 2 — Verify your Video Assets folder

The `Video Assets/` folder ships with the repository and contains all the reference files the workflow needs:

```
Video Assets/
  ├── video-guidelines.pdf
  ├── video-script-template.pdf
  ├── video-script-samples.pdf
  ├── cx-writing-guidelines.pdf
  ├── video-script-prompt.md
  ├── script-template.md
  └── script-sample.md
```

Check that all seven files are present before running the workflow. If any are missing, pull the latest version of the repo (see [Keeping assets up to date](#keeping-assets-up-to-date)).

> **No manual path configuration required.** The workflow automatically detects your GitHub folder location and discovers repositories dynamically. See [GitHub folder auto-detection](#github-folder-auto-detection) for details.

---

## Running the workflow

### Start Claude Code

Open Terminal, navigate to the project folder, and launch Claude Code:

```bash
cd ~/Desktop/Video\ Workflow
claude
```

Claude Code will automatically read `CLAUDE.md` and load the project context.

### Trigger the pipeline

Once Claude Code is running, type:

```
Run the video script generator
```

> **Note:** Running against all repositories can consume a large number of tokens. Start with a single repository to validate the pipeline first.

You can also target a specific repository directly:

```
Run the video script generator for docs-[product-name]
```

### GitHub folder auto-detection

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

### Repository selection

After the GitHub folder is confirmed, the workflow asks which repositories to process:

```
Which repositories would you like to process?

Options:
  A) All repositories (all discovered repos, excluding excluded folders)
  B) Select specific repositories (you will choose from the discovered list)
  C) Single repository (provide the repository name, e.g., docs-[product-name])

Please type A, B, or C to continue.
```

**Option A** — Processes all discovered `docs-*` repositories.

**Option B** — Displays a numbered list. Select by name, number, or a mix:
```
docs-product-a,docs-product-b
2,5,10
docs-product-a,5
```

**Option C** — Processes a single repository. The workflow validates it exists before proceeding.

> **Tip:** Always start with option C on a single repository to validate the pipeline before running it across multiple repos.

### What happens next

The pipeline runs automatically. You will see progress updates in the terminal as each file is processed:

```
[✓ SKIP]         docs-product-a / overview.adoc
[✓ VIDEO_NEEDED] docs-product-a / deploy-application.adoc
[⚠ FLAGGED]      docs-product-a / manage-settings.adoc
```

When the run is complete, the orchestrator prints a completion report showing how many scripts were created, skipped, flagged, execution time, verification status, and output file locations. The report is also saved to `Video Logs/`.

### Your output files

After a run, find your files here:

| Output | Location |
|--------|----------|
| Generated scripts | `Video Scripts/docs-[product-name]/` |
| CSV tracker | `Video Logs/docs-[product-name]/video-script-tracker.csv` |
| Flagged report | `Video Logs/docs-[product-name]/flagged-report-YYYY-MM-DD.csv` |
| Completion report | `Video Logs/docs-[product-name]/completion-report-YYYY-MM-DD-HHMM.md` |

For multi-repository runs, a summary completion report is also saved to `Video Logs/completion-report-YYYY-MM-DD-HHMM.md`.

### Reviewing flagged files

Files the orchestrator is unsure about are saved to `Video Logs/docs-[product-name]/flagged-report-YYYY-MM-DD.csv`. Open this file to review each entry — the `Reason` column explains why the file was flagged. Decide whether to add it to the next run as a targeted VIDEO_NEEDED file or skip it.

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

The following repository types are excluded from the workflow by default. The orchestrator will never scan or process them:

- `docs-internal-only`
- `docs-archived`
- `docs-site-config`
- `docs-release-notes`

Update these in `CLAUDE.md` and `.claude/skills/setup-github.md` to match the excluded repositories in your own environment.

If you explicitly request an excluded repository (Option C), the workflow will warn you and prompt for a different selection.

---

## Troubleshooting

**Claude Code says "command not found"**
Add Claude Code to your PATH:
```bash
export PATH="$HOME/.local/bin:$PATH"
```
Add this line to your `~/.zshrc` or `~/.bashrc` file to make it permanent.

**401 Authentication error on startup**
Your API key may be missing or incorrect. Run:
```bash
claude config set apiKey YOUR_API_KEY
```
If the error persists, your network proxy may also need to be configured.

**GitHub folder not detected automatically**
If the workflow cannot find your GitHub folder, it will prompt you to enter the path manually. Enter the full path (e.g., `/Users/username/Documents/GitHub/`) and optionally save it for future runs. The saved path is stored at `~/Desktop/Video Workflow/.config`.

**Reference file not found error**
The orchestrator could not read one of the files in `Video Assets/`. Check that all seven files are present and that the filenames match exactly what is listed in the [Verify your Video Assets folder](#step-2--verify-your-video-assets-folder) section. Run `git pull` to restore any missing files.

**Scripts folder not created**
The `Video Scripts/` and `Video Logs/` folders are created automatically on the first run. If they are missing after a run, check the terminal output for file write errors.

**Context limit reached on large repositories**
If Claude Code hits its context limit mid-run, restart the session and use option B or C to process one repository at a time instead of all repositories at once.

**CSV count mismatch after a run**
The workflow includes a mandatory verification step (step 7) that automatically corrects CSV discrepancies — adding missing entries and removing orphaned ones. If the completion report shows a mismatch that was not corrected, re-run the workflow on the affected repository.

---

## Project structure

```
video-script-generator/
  ├── README.md                               ← this file
  ├── CLAUDE.md                               ← project rules, auto-read by Claude Code
  ├── .gitignore
  ├── .claude/
  │     ├── agents/
  │     │     ├── orchestrator.md             ← coordinates the full pipeline
  │     │     ├── script-writer.md            ← generates video scripts
  │     │     ├── peer-reviewer.md            ← reviews scripts against writing guidelines
  │     │     └── csv-logger.md               ← logs results to the CSV tracker
  │     └── skills/
  │           ├── setup-github.md             ← GitHub folder auto-detection & repo selection
  │           ├── analyze-docs.md             ← .adoc file classification (VIDEO_NEEDED/FLAGGED/SKIP)
  │           ├── write-script.md             ← script format specification & generation
  │           ├── batch-generate.md           ← parallel execution for writing, review, logging
  │           ├── verify-workflow.md          ← post-run verification of CSV, reports, integrity
  │           └── generate-completion-report.md ← 12-section completion report generation
  ├── Video Assets/                           ← reference files (versioned in this repo)
  │     ├── video-guidelines.pdf
  │     ├── video-script-template.pdf
  │     ├── video-script-samples.pdf
  │     ├── cx-writing-guidelines.pdf
  │     ├── video-script-prompt.md
  │     ├── script-template.md
  │     └── script-sample.md
  ├── Video Scripts/                          ← generated scripts (local only, not committed)
  └── Video Logs/                             ← tracker, flagged reports, completion reports (local only)
```

---

## Contributing

To update the Video Assets, agent definitions, or skill files, create a branch and open a pull request. Do not commit generated scripts, CSV tracker files, or completion reports — these are listed in `.gitignore` and should remain local to each writer's machine.
