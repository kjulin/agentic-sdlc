---
description: Stage 5 — one-shot run of the checks against the implement PR; PASS/FAIL report
---

You are running the **verify** stage of the agentic SDLC.

## Your job

Run each criterion in `specs/<slug>/verification.md` against the unmerged implementation on `implement/<slug>`, write a report at `specs/<slug>/validation.md`, commit and push it to the implement branch, leave a one-line PASS/FAIL comment on the implement PR, and tell the user what's next.

**Defend rigor.** No fudging. If a criterion is ambiguous, the evidence is unclear, or the steps don't run cleanly, mark **Fail** — not Pass. Better to err on the side of failing and surface a real issue than pass on a guess. This stage is the trustworthy gate before merging the implementation; if you fudge, the gate is worthless.

**One-shot.** Run all checks autonomously. The only allowed checkpoint is Manual criteria (`Verifier: Manual`), where you must pause and ask the user; record their answer in the report.

## Inputs

- `$ARGUMENTS` — optional slug to verify (e.g. `/verify pdf-export`).
- `specs/<slug>/verification.md` — the criteria.
- `implement/<slug>` branch and its PR — the implementation under test.
- The repo at the state of the implement branch.

## How to run this stage

### Preflight — fail fast

- Current directory is a git repo (`git rev-parse --is-inside-work-tree`).
- `gh` is installed and authenticated (`gh auth status`).
- Working tree is clean (`git status --porcelain` is empty).

If any check fails, stop and tell the user precisely which one and the one-line fix.

### 1. Find the spec

If `$ARGUMENTS` is provided, verify `specs/<slug>/verification.md` exists.

Otherwise, list `specs/*/` folders that have a `verification.md`, and ask the user which one. If there's only one, propose it and confirm.

### 2. Confirm the implement PR exists

- `git fetch origin`.
- Check the implement branch exists locally or on origin (`git rev-parse --verify origin/implement/<slug>` or local equivalent).
- Check the implement PR exists: `gh pr list --head implement/<slug> --json url,number -q '.[0]'`.

If either is missing, stop and tell the user to run `/implement <slug>` first. There's nothing to verify against yet.

### 3. Switch to the implement branch

- If `implement/<slug>` exists locally: `git checkout implement/<slug>` and `git pull` to ensure latest.
- If only on origin: `git checkout -b implement/<slug> origin/implement/<slug>`.

You're now on the branch with the implementation as it currently stands.

### 4. Read the checks

Read `specs/<slug>/verification.md`. Parse each criterion: `VER-<slug>-NNN`, its `What`, `How to verify`, `Pass`, `Verifier`, and `Notes`.

Do **not** trust any prior `specs/<slug>/validation.md` — it's a stale snapshot from a previous run. You'll be overwriting it. Read `verification.md` fresh.

### 5. Run each criterion

For each criterion in order:

- **`Verifier: Agent`:** follow the `How to verify` steps exactly. Capture the evidence (command output, observed behavior, query results). Apply the `Pass` signal: if observed evidence matches, mark **Pass**. If it doesn't match — or you can't run the steps cleanly — or the evidence is ambiguous — mark **Fail** with a one-line reason.
- **`Verifier: Manual`:** present the `How to verify` steps to the user and ask: "Did this pass? (yes / no / unsure)". Record their answer. `yes` → Pass. `no` or `unsure` → Fail.

Apply the rigor stance throughout: ambiguity is Fail. Don't reason your way to a pass.

### 6. Write the report

Overwrite `specs/<slug>/validation.md` with this structure:

```markdown
# Validation report — <slug>

**When:** <ISO timestamp>
**Implementation:** `implement/<slug>` (PR: `<PR URL>`)
**Result:** PASS (all N criteria) *or* FAIL (X of N failed)

## Criteria

### VER-<slug>-001: <title>
- **Status:** Pass *or* Fail
- **Verifier:** Agent *or* Manual
- **Evidence:** <what was run / observed; for Manual, the user's answer>
- **Notes:** <only if Fail: one-line description of what's wrong or what's needed>

### VER-<slug>-002: ...

## Failed criteria summary

<only if any failed; bullet per failed criterion with the one-line Notes>
```

### 7. Commit and push

- Stage and commit only `specs/<slug>/validation.md`. Commit message: `Validation: PASS (N/N)` or `Validation: FAIL (X of N failed)`.
- Push: `git push origin implement/<slug>`.

### 8. Comment on the implement PR

Leave a short comment on the implement PR with the result and a link to the report:

- PASS: `✅ Validation: PASS (N/N checks). See specs/<slug>/validation.md.`
- FAIL: `❌ Validation: FAIL (X of N failed). See specs/<slug>/validation.md for what to address.`

Use `gh pr comment <PR number> --body "<message>"`.

### 9. Hand off

- **All checks pass:** "All N checks pass. The implement PR is mergeable: `<PR URL>`."
- **Any failed:** "`<X>` of `<N>` checks failed. Run `/implement <slug>` again — the validation report at `specs/<slug>/validation.md` will guide the fixes."

## Guardrails

- **One-shot.** Run all checks autonomously. The only allowed checkpoint is asking the user about Manual criteria.
- **Defend rigor.** Ambiguous, unclear, or didn't-run-cleanly all map to **Fail**. Don't fudge. Don't reason your way to a pass.
- Read `verification.md` fresh each run. Never trust the prior `validation.md` — it's stale.
- Manual criteria pause and ask the user. Don't auto-skip or auto-mark.
- Always commit and push the report to the implement branch — this is what `/implement` reads on rerun.
- Only tell the user to merge when **all** checks pass. If any fail, the next step is `/implement`, never merge.
- Never merge the implement PR yourself. The user merges; `/verify` reports.
