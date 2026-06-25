---
description: Stage 4 — one-shot the implementation against the spec and checks; open or update the PR
---

You are running the **implement** stage of the agentic SDLC.

## Your job

Build what `specs/<slug>/spec.md` describes, in a way that lets the checks in `specs/<slug>/verification.md` pass. **One-shot.** No chat-heavy interaction, no plan-confirm gate. Read the whole spec folder, ground in the code, do the work, commit, push, open or update the PR.

`/implement` is expected to be run more than once. When checks fail in the verification stage, you'll be invoked again to fix what failed. Read any validation reports in `specs/<slug>/` so you know what's already passing and what isn't.

## Inputs

- `$ARGUMENTS` — optional slug to implement (e.g. `/implement pdf-export`).
- Everything in `specs/<slug>/`:
  - `concept.md` — the value-prop hypothesis
  - `spec.md` — what to build
  - `verification.md` — what must pass to ship
  - Any artifacts (HTML mocks, mermaid, sample payloads)
  - Any validation reports from prior verification runs
- The repo — change it freely; this is where the work happens.

## How to run this stage

### Preflight — fail fast before any work

- Current directory is a git repo (`git rev-parse --is-inside-work-tree`).
- `gh` is installed and authenticated (`gh auth status`).
- Working tree is clean (`git status --porcelain` is empty).

If any check fails, stop and tell the user precisely which one and the one-line fix.

### 1. Find the spec

If `$ARGUMENTS` is provided, verify `specs/<slug>/concept.md`, `spec.md`, and `verification.md` all exist.

Otherwise, list `specs/*/` folders that have all three files, and ask the user which one. If there's only one, propose it and confirm.

### 2. Check that the checks are merged

Verify `specs/<slug>/verification.md` is on the default branch:

- `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
- `git fetch origin <default>`
- `git cat-file -e origin/<default>:specs/<slug>/verification.md`

If checks aren't on the default branch, tell the user the checks PR appears to still be in review and recommend merging before implementing. Allow them to proceed if they explicitly say so — the implementation will be based on draft checks.

### 3. Branch — first run or continuation?

Branch before reading the spec folder. Validation reports are written by the verification stage to the `implement/<slug>` branch, not to the default branch, so you can only see them once you're on the branch.

- `git fetch origin`.
- If `implement/<slug>` exists locally or on origin, this is a continuation. Switch to it: `git checkout implement/<slug>` (or `git checkout -b implement/<slug> origin/implement/<slug>` if only on origin).
- Otherwise this is a first run. Create it: `git checkout -b implement/<slug> origin/<default>`.

### 4. Read everything in the spec folder

Now on the branch, read all files in `specs/<slug>/`:

- `concept.md` to understand the value-prop hypothesis you're delivering.
- `spec.md` to understand what to build.
- `verification.md` to understand the bar for shipping.
- Artifacts (mocks, diagrams, payloads) for the proposed shape.
- Validation reports — written by the verification stage to this branch — for what passed and failed in prior runs.

If validation reports exist, prioritize fixing the failed criteria. If they don't, you're on a first run — implement the whole spec.

**On continuation runs, also read all issue comments on the implement PR.** Fetch via `gh pr view <PR number> --json comments`. The user may have left direction there about how to handle specific failures or what approach to take — treat anything written there as priority input alongside the validation report.

### 5. Ground in the repo

Read the repo widely enough to know what the spec's `Changes` and `Sequence` actually touch. Follow the spec; don't redesign it. If you find the spec genuinely can't be followed (a file it names doesn't exist, the interface it assumes is different, etc.), stop and surface it — don't paper over.

### 6. Do the work

Implement what the spec describes. Walk the spec's `Sequence` field, doing each step. Make the changes the spec specifies in the files the spec specifies.

You may write unit or integration tests where useful for refactoring confidence — but these are not the same thing as the verification checks. The verification checks are run separately by the verification stage.

Commit as you go. Use multiple small, descriptive commits within the branch rather than one giant commit. Commit messages should describe what changed concretely, not which step of the spec is being addressed.

### 7. Surface gaps when you find them

If during implementation you discover something the spec didn't anticipate — a file that doesn't exist, a constraint the spec missed, an acceptance criterion that's ambiguous — stop and surface it. Don't guess. Suggest revisiting `/spec` or `/checks` if needed; what's already implemented stays on the branch as input to the revisit.

### 8. Push, then open or update the PR

After committing:

- Push: `git push -u origin implement/<slug>`.
- Check whether a PR already exists for this branch: `gh pr list --head implement/<slug> --json url,number -q '.[0]'`.
- If no PR exists: open one with `gh pr create --title "Implement: <title>" --body "<body>"`. Body includes:
  - One-line traceback (`Implements the spec in specs/<slug>/spec.md against the checks in specs/<slug>/verification.md`)
  - A summary of what was built (files touched, sequence steps completed)
  - If a spec/checks revisit was surfaced in step 7: a clear note at the top.
- If a PR exists: leave a short comment summarizing what this run changed (`Addressed VER-<slug>-001 and VER-<slug>-002; remaining: VER-<slug>-003`). Pushing the new commits is enough to update the PR diff itself.

### 9. Hand off

The implementation PR is **not mergeable from this stage** — it ships only after the verification stage reports a clean run. Never tell the user to merge from `/implement`.

- If first run: "Implementation opened at `<PR URL>`. **Do not merge yet** — run the verification stage to check it against `specs/<slug>/verification.md`. The PR is mergeable only after all checks pass."
- If continuation: "Pushed updates to `<PR URL>`. Re-run the verification stage to confirm the failing checks now pass. Merge only after verification reports a clean run."

## Guardrails

- **One-shot.** No "is this OK?" checkpoints before doing the work. Read, ground, do, push.
- Read the whole `specs/<slug>/` folder. Validation reports tell you what to prioritize on a rerun.
- Don't redesign the spec. Follow it. If you can't follow it, surface why; don't paper over.
- Tests are optional and separate from verification checks. Write them only when they help refactoring confidence.
- Commit in small, descriptive commits. The PR diff is review material; help future-you.
- One PR per implementation. Reruns push more commits to the same PR; do not open a second PR for the same slug.
