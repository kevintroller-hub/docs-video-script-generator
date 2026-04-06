# Video Script Generator

This repository contains an automated workflow for generating video scripts from AsciiDoc (`.adoc`) documentation files. It uses Claude Code and a multi-agent pipeline to scan documentation repositories, detect UI procedures suitable for video, write scripts, peer-review them, and log everything to a CSV tracker.

## Table of contents

- [How it works](#how-it-works)
- [Prerequisites](#prerequisites)
- [First-time setup](#first-time-setup)
  - [Step 1 — Clone the repository](#step-1--clone-the-repository)
  - [Step 2 — Verify your Video Assets folder](#step-2--verify-your-video-assets-folder)
  - [Step 3 - Update file paths to match your local setup](#step-3--update-file-paths-to-match-your-local-setup)
- [Running the workflow](#running-the-workflow)
  - [Start Claude Code](#start-claude-code)
  - [Trigger the pipeline](#trigger-the-pipeline)
  - [What happens next](#what-happens-next)
  - [Reviewing flagged files](#reviewing-flagged-files)
- [Keeping assets up to date](#keeping-assets-up-to-date)
- [Excluded repositories](#excluded-repositories)
- [Troubleshooting](#troubleshooting)
- [Project structure](#project-structure)
- [Contributing](#contributing)

---

## How it works

An orchestrator agent reads your documentation files, identifies content where a video would help (specifically UI procedures), and delegates to three specialist subagents:

1. **Script writer** — generates a formatted video script from the `.adoc` content
2. **Peer reviewer** — checks the script against CX writing guidelines and edits it in place
3. **CSV logger** — appends a row to the tracker file with the script details and file path

Scripts are saved as `.md` files under `Video Scripts/`. A CSV tracker and flagged report are saved under `Video Logs/`.

---

## Prerequisites 

Before you begin, make sure you have already Claude Code installed and running in your MacOs.

## First-time setup

Follow these steps once after cloning the repo.

### Step 1 — Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/video-script-generator.git
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
  └── video-script-prompt.md
```

Check that all five files are present before running the workflow. If any are missing, pull the latest version of the repo (see [Keeping assets up to date](#keeping-assets-up-to-date)).

### Step 3 -  Update file paths to match your local setup
The workflow files reference specific folder paths that need to match where you cloned the repository on your machine. Before running the workflow for the first time, update the paths in these files:

CLAUDE.md
.claude/agents/orchestrator.md
.claude/agents/script-writer.md
.claude/agents/peer-reviewer.md
.claude/agents/csv-logger.md

What to change:

Open each file in any text editor and replace every occurrence of:

```
~/Desktop/Video Workflow/
```

with the actual path to where you cloned this repository. For example, if you cloned it to your Documents folder:

```
~/Documents/Video Workflow/
```

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

Note: Running the script against all your repos could consume your entire budget due to the amounts of tokens consumed. Start with one repository.

You can also ask to run the video script for a specific repository:

```
Run the video script generator for docs-product-a
```

If you run it generally for all repos, the orchestrator will ask you to choose which repositories to process:

```
Which repositories would you like to process this run?

Options:
  A) All repositories
  B) Select specific repositories
  C) Single test file (provide the full file path)

Please type A, B, or C to continue.
```

> **Tip:** Always start with option C on a single `.adoc` file to validate the pipeline before running it on a full repository. The orchestrator will prompt you to review the test output before proceeding.

### What happens next

The pipeline runs automatically. You will see progress updates in the terminal as each file is processed:

```
[✓ SKIP]         docs-product-a / overview.adoc
[✓ VIDEO_NEEDED] docs-product-a / create-user.adoc
[⚠ FLAGGED]      docs-product-a / manage-settings.adoc
```

When the run is complete, the orchestrator prints a summary report showing how many scripts were created, skipped, flagged, and any errors.

### Your output files

After a run, find your files here:

| Output | Location |
|--------|----------|
| Generated scripts | `Video Scripts/[repo-name]/` |
| CSV tracker | `Video Logs/[repo-name]/video-script-tracker.csv` |
| Flagged report | `Video Logs/[repo-name]/flagged-report.csv` |

### Reviewing flagged files

Files the orchestrator is unsure about are saved to `Video Logs/[repo-name]/flagged-report.csv`. Open this file, review each entry, and fill in the `Decision` column with either `VIDEO_NEEDED` or `SKIP`. Then re-run the pipeline — the orchestrator will process your decisions automatically.

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

The following repositories exist under `~/Documents/GitHub/` but are excluded from the workflow. The orchestrator will never scan them:

- `docs-internal-only`
- `docs-archived`
- `docs-site-config`
- `docs-release-notes`

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
Check the Claude Code documentation for authentication setup instructions.

**Reference file not found error**
The orchestrator could not read one of the files in `Video Assets/`. Check that all five files are present and that the filenames match exactly what is listed in the [Verify your Video Assets folder](#step-4--verify-your-video-assets-folder) section. Run `git pull` to restore any missing files.

**Scripts folder not created**
The `Video Scripts/` and `Video Logs/` folders are created automatically on the first run. If they are missing after a run, check the terminal output for file write errors.

**Context limit reached on large repositories**
If Claude Code hits its context limit mid-run, restart the session and use option B to process one repository at a time instead of all repositories at once.

---

## Project structure

```
video-script-generator/
  ├── README.md                        ← this file
  ├── CLAUDE.md                        ← project rules, auto-read by Claude Code
  ├── .gitignore
  ├── .claude/
  │     └── agents/
  │           ├── orchestrator.md      ← coordinates the full pipeline
  │           ├── script-writer.md     ← generates video scripts
  │           ├── peer-reviewer.md     ← reviews scripts against CX guidelines
  │           └── csv-logger.md        ← logs results to the CSV tracker
  ├── Video Assets/                    ← reference files (versioned in this repo)
  │     ├── video-guidelines.pdf
  │     ├── video-script-template.pdf
  │     ├── video-script-samples.pdf
  │     ├── cx-writing-guidelines.pdf
  │     └── video-script-prompt.md
  ├── Video Scripts/                   ← generated scripts (local only, not committed)
  └── Video Logs/                      ← tracker and flagged reports (local only)
```

---

## Contributing

To update the Video Assets or agent definition files, create a branch and open a pull request following your team's standard Git workflow. Do not commit generated scripts or CSV tracker files — these are listed in `.gitignore` and should remain local to each writer's machine.

