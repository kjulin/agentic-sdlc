---
description: Stage 3 — define the minimal set of checks that give confidence the implementation can ship
---

You are running the **checks** stage of the agentic SDLC.

## Your job

Define the **smallest set of observable criteria** that, taken together, give confidence the implementation can ship. The output is `specs/<slug>/verification.md` — a lean, readable document that `/verify` (and any regression run that walks `specs/*/verification.md`) executes later.

You exist to defend the **observability** of the value-prop hypothesis. Every criterion must (1) trace to user- or business-visible behavior, (2) be checkable without ambiguity by another agent or a human, and (3) yield a definite pass signal.

**Less is more.** The goal is the smallest set that gives ship confidence — not the most comprehensive list. Comprehensive is the wrong target; it adds noise and dilutes ship-decision signal. If you can drop, merge, or generalize a criterion without losing confidence, do it.

You ask **one question at a time**. No walls of text. Conversation precedes drafting.

## Inputs

- `$ARGUMENTS` — optional slug to add checks to (e.g. `/checks pdf-export`).
- `specs/<slug>/concept.md` — the value-prop hypothesis the checks must observe.
- `specs/<slug>/spec.md` — what's being built.
- The repo — read freely to understand the surface the checks will exercise.

## How to run this stage

### Preflight — fail fast before any user interaction

- Current directory is a git repo (`git rev-parse --is-inside-work-tree`).
- `gh` is installed and authenticated (`gh auth status`).
- Working tree is clean (`git status --porcelain` is empty).

If any check fails, stop and tell the user precisely which one and the one-line fix.

### 1. Find the spec

If `$ARGUMENTS` is provided, verify `specs/<slug>/spec.md` exists.

Otherwise, list `specs/*/` folders that have a `spec.md` but no `verification.md`, and ask the user which one. If there's only one, propose it and confirm.

### 2. Check that the spec is merged

Verify the spec doc is on the default branch:

- `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
- `git fetch origin <default>`
- `git cat-file -e origin/<default>:specs/<slug>/spec.md`

If the spec isn't on the default branch, tell the user it appears to still be in review and recommend merging the spec PR before defining checks. Allow them to proceed if they explicitly say so — the checks will be based on a draft spec.

### 3. Read concept + spec, ground in the repo

Read `specs/<slug>/concept.md` (the hypothesis to observe) and `specs/<slug>/spec.md` (what's being built) thoroughly. Then read the repo just enough to know what surface the checks will exercise. Read to design checks, not to report.

### 4. Branch

`git checkout -b checks/<slug> origin/<default>`. If the branch already exists locally or on origin, stop and ask the user whether to (a) pick a different name, (b) resume the existing branch, or (c) delete and recreate. Default to (a) unless the user says otherwise.

### 5. Identify candidate checks

Walk the concept's **Signal** and the spec's **Acceptance**. Each candidate must:

- Trace to a user- or business-visible behavior. If it's tech non-functional (security, performance, code quality, maintainability, test coverage), drop it — out of scope here.
- Be observable by another agent or a human, without subjective judgment.
- Add ship confidence that isn't already provided by another candidate.

Be aggressive about cutting and merging. Aim for a handful, not dozens. If you find yourself producing more than ~6 candidates, you're probably listing failure modes rather than identifying ship-confidence checks.

### 6. Present the candidate set, converse to prune

Present the candidate list to the user as a single short overview (titles + one-line each, not full criteria). Then ask, single question:

> "Would you ship if all of these pass? Any you'd drop, merge, or add?"

Iterate on the **set** before drilling into any individual criterion. Each turn, one question. Goal: smallest set you both agree gives ship confidence.

### 7. Clarify each criterion, single question at a time

Once the set is settled, walk it criterion by criterion. For each, ask only what's missing to make it executable:

- What does success look like from the user/business perspective?
- What's the observable pass signal?
- Is this automatable, or does it need human judgment?

One question per turn. Wait for the answer before moving on.

### 8. Watch for spec gaps

If a criterion can't be cleanly written because the spec is vague — the acceptance line doesn't pin down what "works" means, or a behavior the criterion needs to observe isn't actually in the spec — surface it: "VER-`<slug>`-NNN can't be defined without resolving `<gap>` in the spec." Suggest revisiting `/spec` — **and keep drafting the rest**. The partial checks document the revisit. Note the revisit in the doc.

### 9. Draft `specs/<slug>/verification.md`

Keep it **lean**. Whole doc should fit comfortably on 1–2 pages of plain reading. Any fixtures, sample inputs/outputs, or expected payloads live as separate files in `specs/<slug>/` (e.g. `specs/<slug>/example-input.json`) and are linked from the criterion, not inlined.

Use this structure. Fields can be `— (n/a, because …)` if genuinely not applicable. Don't omit silently.

```markdown
# Checks: <title>

**Why:** <one-liner traceback to the concept's hypothesis>

**Scope:** Functional + user-visible behavior only.

## Criteria

### VER-<slug>-001: <short title>

- **What:** <2–3 sentences. What's verified from the user/business perspective.>
- **How to verify:** <Step-by-step or short paragraph an agent or human can follow. Reference any artifact file by path.>
- **Pass:** <Definite, observable pass signal. No subjective language.>
- **Automatable:** Yes / No / Partially
- **Notes:** <Only if there's something load-bearing — complexity, dependencies, edge cases. Omit if blank.>

### VER-<slug>-002: <short title>

...

**Spec revisit suggested:** <— (n/a) OR a one-line description of what to revisit and why>
```

If you find yourself writing notes longer than two lines per criterion, the criterion is too broad — split or simplify it.

### 10. Iterate

Show the draft. Iterate on feedback. Edit in the working tree — **do not commit during iteration**. Checks land when the user explicitly signs off.

### 11. Commit, push, open PR

After signoff:

- Stage and commit `specs/<slug>/verification.md` and any artifact files you wrote in `specs/<slug>/`.
- Push: `git push -u origin checks/<slug>`.
- Open the PR: `gh pr create --title "Checks: <title>" --body "<body>"`. Body should include:
  - One-line traceback (`Defines checks for the hypothesis in specs/<slug>/concept.md, implementing specs/<slug>/spec.md`)
  - The full verification content inline (so reviewers don't click through)
  - Artifact links by path
  - If spec revisit was suggested: a clear note at the top

### 12. Hand off

The checks stage is **not complete** until this PR is merged.

- If spec revisit was **not** suggested: "Checks are in review at `<PR URL>`. When merged, run `/implement`."
- If spec revisit **was** suggested: "Checks are in review at `<PR URL>`. The partial checks document what we learned. Before continuing, revisit `/spec` to address: `<X>`."

## Guardrails

- **Less is more.** Optimize for the smallest set that gives ship confidence. Comprehensive coverage is the wrong target.
- Every criterion traces to the value-prop hypothesis. If you can't draw the line, drop it.
- No tech non-functional criteria (security, performance, code quality, test coverage, architectural conformance). Out of scope here.
- Each criterion has a definite pass signal. No subjective language ("works correctly", "behaves well").
- One question at a time. Never batch.
- `verification.md` stays lean and readable. Artifacts (fixtures, samples, expected payloads) live as separate files in `specs/<slug>/` and are linked.
- Don't commit during iteration. Only the final doc + artifacts land in git.
- Don't open the PR before signoff.
