# CLAUDE.md

## Repository Overview

This is a **GitHub Skills course template** titled "Introduction to GitHub". It is an interactive, self-paced learning course that teaches learners the fundamental GitHub workflow: creating a branch, committing a file, opening a pull request, and merging it. The course is driven entirely by GitHub Actions that detect learner actions and advance them through steps automatically.

This is **not a traditional software project** — there is no application code, build system, or test suite. The "code" here is the course infrastructure (GitHub Actions workflows, step content, and documentation).

## Repository Structure

```
.
├── README.md                        # Active course instructions shown to the learner (auto-updated)
├── LICENSE                          # MIT License
├── .gitignore                       # Standard gitignore (OS files, binaries, archives, logs)
├── images/                          # Screenshots used in course instructions
│   ├── profile-readme-example.png
│   ├── code-tab.png
│   ├── main-branch-dropdown.png
│   ├── create-branch-button.png
│   ├── create-new-file.png
│   ├── my-profile-file.png
│   ├── commit-full-screen.png
│   ├── compare-and-pull-request.png
│   ├── pull-request-branches.png
│   ├── Pull-request-description.png
│   ├── Actions-to-step-4.png
│   ├── Green-merge-pull-request.png
│   └── delete-branch.png
└── .github/
    ├── dependabot.yml               # Monthly auto-updates for GitHub Actions
    ├── steps/                       # Markdown content for each course step
    │   ├── -step.txt                # Current step tracker (contains: 0, 1, 2, 3, 4, or X)
    │   ├── 0-welcome.md             # Step 0 content (template placeholder)
    │   ├── 1-create-a-branch.md     # Step 1 instructions
    │   ├── 2-commit-a-file.md       # Step 2 instructions
    │   ├── 3-open-a-pull-request.md # Step 3 instructions
    │   ├── 4-merge-your-pull-request.md # Step 4 instructions
    │   └── X-finish.md              # Completion/finish content
    └── workflows/                   # GitHub Actions that drive the course
        ├── 0-welcome.yml            # Triggers on push to main; advances step 0→1
        ├── 1-create-a-branch.yml    # Triggers on branch creation; advances step 1→2
        ├── 2-commit-a-file.yml      # Triggers on push to my-first-branch; advances step 2→3
        ├── 3-open-a-pull-request.yml # Triggers on PR open/reopen; advances step 3→4
        └── 4-merge-your-pull-request.yml # Triggers on push to main; advances step 4→X
```

## How the Course Works

### Step Tracking

The current step is stored in `.github/steps/-step.txt` as a single integer (`0`, `1`, `2`, `3`, `4`) or the letter `X` (course complete). Every workflow reads this file first to ensure it only runs when the learner is on the correct step.

### Workflow Pattern

All five workflows follow the same pattern:

1. **`get_current_step` job** — Checks out the repo and reads `-step.txt`, exposing it as an output.
2. **Main job** — Only runs if:
   - The repository is not the template itself (`!github.event.repository.is_template`)
   - The current step number matches the expected step
   - Additional conditions specific to each step (branch name, event type, etc.)
3. **Update action** — Calls `skills/action-update-step@v2` with `GITHUB_TOKEN`, `from_step`, `to_step`, and `branch_name: my-first-branch`.

The `skills/action-update-step@v2` action updates both `.github/steps/-step.txt` and `README.md` on the `main` branch with the content of the next step's markdown file from `.github/steps/`.

### Step Progression

| Step | File | Trigger | Expected Learner Action |
|------|------|---------|------------------------|
| 0 → 1 | `0-welcome.yml` | Push to `main` | Repository created from template |
| 1 → 2 | `1-create-a-branch.yml` | Branch created | Create branch named `my-first-branch` |
| 2 → 3 | `2-commit-a-file.yml` | Push to `my-first-branch` | Commit `PROFILE.md` to `my-first-branch` |
| 3 → 4 | `3-open-a-pull-request.yml` | PR opened/reopened | Open PR from `my-first-branch` → `main` |
| 4 → X | `4-merge-your-pull-request.yml` | Push to `main` | Merge the PR |

### Critical Branch Name

The learner branch **must** be named `my-first-branch` exactly. This name is hard-coded in multiple workflow conditions and is the branch checked out by the `skills/action-update-step@v2` action.

## Key Conventions

### README.md Is Auto-Managed

`README.md` is dynamically rewritten by GitHub Actions as the learner progresses. Do not edit it manually to change course content — edit the appropriate file in `.github/steps/` instead. The README wraps step content in `<header>` and `<footer>` HTML comment blocks; these are preserved across updates.

### Step Content Files

Each `.github/steps/*.md` file contains the raw Markdown content for one step. These files use HTML comment blocks (`<!-- ... -->`) for author notes that are not shown to learners (the template repo renders them as HTML comments, which are invisible in the rendered view).

### Permissions

All workflows request:
```yaml
permissions:
  contents: write
```
This is the minimum required to check out the repo and for the update action to commit changes back to `main`.

### Dependabot

GitHub Actions dependencies are kept up to date automatically via Dependabot on a **monthly** schedule (configured in `.github/dependabot.yml`). Accept Dependabot PRs that update action versions (e.g., `actions/checkout`, `skills/action-update-step`).

## Development Workflow

### Branch Strategy

- `main` — The default branch; holds the active course state shown to learners. The step tracker and README live here.
- `my-first-branch` — The learner's practice branch. Created and deleted by the learner during the course. The `skills/action-update-step@v2` action also uses this branch name when committing updates.
- `claude/claude-md-docs-d9ipqk` — The current AI assistant development branch.

### Making Changes to Course Content

1. Edit the relevant `.github/steps/<N>-<step-name>.md` file for content changes.
2. Edit the corresponding `.github/workflows/<N>-<step-name>.yml` for logic/trigger changes.
3. Update images in `images/` if screenshot references need updating.
4. Test by manually triggering workflows via `workflow_dispatch` (all workflows support this).

### Resetting the Course

To reset a learner's progress, set `.github/steps/-step.txt` back to `0` on the `main` branch and restore `README.md` to the step 0 welcome state.

### Testing Workflows

All workflows include `on: workflow_dispatch` to support manual triggering from the GitHub Actions tab. Use this to test workflow logic without needing to perform the actual learner actions. The step guard (`needs.get_current_step.outputs.current_step == N`) will still be evaluated, so set `-step.txt` to the appropriate value first.

## What Not to Change

- Do not rename `my-first-branch` in workflows — the branch name is load-bearing across multiple files.
- Do not remove the `!github.event.repository.is_template` condition from workflows — this prevents the workflows from running on the template repository itself.
- Do not alter the `<header>` / `<footer>` HTML structure in `README.md` or step files — the update action relies on this structure for replacement.
- Do not add package managers, build tools, or application code — this is a pure documentation/workflow repository.
