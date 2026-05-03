---
name: setup-github
description: Automatically detect GitHub folder location, discover documentation repositories, and prompt for selection
---

# Setup GitHub Skill

Automatically detect or configure the GitHub folder path, discover all documentation repositories, and prompt the user for repository selection (All/Specific/Single).

## Usage

```
/setup-github
```

**Parameters:** None (interactive)

**Returns:** JSON object with:
```json
{
  "github_folder": "/Users/username/Documents/GitHub/",
  "discovered_repos": ["docs-product-b", "docs-product-a", ...],
  "selected_repos": ["docs-product-a"],
  "mode": "single|specific|all"
}
```

---

## Auto-Detection Process

### Step 1: Check for Saved Configuration (Optional)

- Look for `~/Desktop/Video Workflow/.config` file
- If exists and valid, use the saved GitHub folder path
- Prompt: "Using GitHub folder: `[saved-path]` (detected previously). Continue? (Y/n)"
- If user confirms, skip to repository discovery
- If user declines or file doesn't exist, proceed to step 2

### Step 2: Search Common Locations

Scan the following directories for folders containing multiple `docs-*` repositories:

**macOS/Linux:**
- `~/Documents/GitHub/`
- `~/GitHub/`
- `~/github/`
- `~/dev/GitHub/`
- `~/Development/GitHub/`
- `~/Code/GitHub/`
- `~/projects/`

**Windows:**
- `C:\Users\[username]\Documents\GitHub\`
- `C:\Users\[username]\GitHub\`
- `C:\Users\[username]\github\`

### Step 3: Validate Candidates

- Each location must contain at least 3 `docs-*` folders to qualify
- Exclude the 4 excluded folders when counting:
  - `docs-internal-only`
  - `docs-archived`
  - `docs-site-config`
  - `docs-release-notes`
- Count only valid documentation repositories

### Step 4: Present Detection Results

**Scenario A: One Location Found**
```
✅ Found GitHub folder at /Users/username/Documents/GitHub/ with 38 documentation repositories.
Use this location? (Y/n)
```
- If user confirms (Y), proceed to repository discovery
- If user declines (n), prompt for manual path entry

**Scenario B: Multiple Locations Found**
```
Found multiple potential GitHub folders:
  1. /Users/username/Documents/GitHub/ (38 repos)
  2. /Users/username/github/ (12 repos)
  3. /Users/username/Code/GitHub/ (5 repos)

Select location (1-3) or enter custom path:
```
- Accept numeric selection or full path
- Validate selection before proceeding

**Scenario C: No Location Found**
```
Could not automatically detect GitHub folder.
Please provide the full path to your GitHub folder:
(Example: /Users/username/Documents/GitHub/)
```
- Accept manual path entry
- Validate path exists and contains `docs-*` repositories

### Step 5: Save Configuration (Optional)

- After successful detection/validation, offer to save path:
  - "Save this location for future runs? (Y/n)"
- If yes, write path to `~/Desktop/Video Workflow/.config` as plain text
- On next run, skip detection and use saved path (with confirmation prompt)

### Step 6: Validate Final Path

- Verify the path exists and is accessible
- Verify it contains at least 1 `docs-*` repository (excluding the 4 excluded folders)
- If validation fails, prompt for manual path entry with helpful error message

---

## Repository Discovery

Once the GitHub folder is validated, discover all documentation repositories:

### Discovery Process

1. Scan the GitHub folder for all directories that match the pattern `docs-*`
2. Exclude the following folders:
   - `docs-internal-only`
   - `docs-archived`
   - `docs-site-config`
   - `docs-release-notes`
3. Build a list of available repositories from the remaining `docs-*` folders
4. Display discovery results: "Discovered X documentation repositories in [path]"

**Example Discovery Output:**
```
Discovered documentation repositories in /Users/username/Documents/GitHub/:
  1. docs-product-a
  2. docs-product-b
  3. docs-product-c
  4. docs-product-d
  5. docs-product-a
  6. docs-studio
  ... (total: 38 repositories found)
```

---

## Repository Selection

Prompt the user for repository selection mode:

```
Which repositories would you like to process?

Options:
  A) All repositories (all discovered repos, excluding 4 excluded folders)
  B) Select specific repositories (you will choose from the discovered list)
  C) Single repository (provide the repository name, e.g., docs-product-a)

Please type A, B, or C to continue.
```

### Option A: All Repositories

- Process all discovered `docs-*` repositories (excluding the 4 excluded folders)
- Confirm with user: "Process all 38 repositories? (Y/n)"
- Return full list of discovered repos

### Option B: Specific Repositories

- Display the discovered list with numbers
- Allow the user to select multiple repositories by:
  - **Name:** "docs-product-b,docs-product-a"
  - **Number:** "2,5,10"
  - **Mixed:** "docs-product-b,5,10"
- Parse input and validate all selections exist
- Return selected repo list

### Option C: Single Repository

- Prompt: "Enter repository name (e.g., docs-product-a):"
- Validate it exists in discovered list
- If not found, show similar matches and prompt again
- Return single-item list with selected repo

---

## Exclusion Enforcement

The following folders must **never** be included in the discovered list or selection options:

**Excluded by name (exact match):**
- `docs-internal-only`
- `docs-archived`
- `docs-site-config`
- `docs-release-notes`

**Why excluded:** These folders do not contain documentation content relevant to video script generation.

**Implementation:**
1. When discovering repositories, skip any folder that matches the excluded list
2. When Option A (All repositories) is selected, process all discovered `docs-*` repositories except these 4
3. These excluded folders should never appear in the selection list presented to the user
4. If a user explicitly requests an excluded repository (Option C), warn them it's excluded and prompt for a different selection

---

## Output Format

Return structured JSON with:
- **github_folder:** Full path to validated GitHub folder
- **discovered_repos:** Array of all discovered repository names (excluding 4 excluded folders)
- **selected_repos:** Array of user-selected repository names
- **mode:** Selection mode ("all", "specific", or "single")
- **config_saved:** Boolean indicating if path was saved to .config file

**Example:**
```json
{
  "github_folder": "/Users/username/Documents/GitHub/",
  "discovered_repos": ["docs-product-b", "docs-product-a", "docs-product-c", ...],
  "selected_repos": ["docs-product-a"],
  "mode": "single",
  "config_saved": true
}
```

---

## Error Handling

If setup fails:
- **Invalid path:** "Path does not exist or is not accessible. Please try again."
- **No repos found:** "No docs-* repositories found in this location. Please check the path."
- **Invalid selection:** "Repository '[name]' not found. Available repositories: [list]"
- **Excluded repo requested:** "Repository '[name]' is excluded from processing. Please select a different repository."

---

## Validation Checklist

Before returning results:
- ✅ GitHub folder path exists and is accessible
- ✅ At least 1 valid `docs-*` repository discovered
- ✅ All 4 excluded folders filtered out
- ✅ User selection is valid (repos exist in discovered list)
- ✅ Mode is set correctly (all/specific/single)
- ✅ .config file saved if user requested
