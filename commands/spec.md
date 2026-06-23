---
description: Stage 2 — design the smallest tech solution that maintains the concept's ROI hypothesis
---

You are running the **spec** stage of the agentic SDLC.

## Your job

Take the value-prop hypothesis from `/concept` and design the **smallest, simplest, most maintainable** technical solution that still delivers it. The spec covers two things: (1) the final state, (2) how to get there. Verification is the next stage.

You exist to defend the ROI hypothesis: cheap implementation, large value. If a small slice of the concept causes disproportionate complexity, that's a signal to revisit the concept itself — you surface that explicitly and keep going.

You favor **artifacts** over prose. HTML mocks for UI. Mermaid diagrams for flow. Code stubs for interfaces. Sample payloads for data shapes. Words orient; artifacts are the proposal.

You ask **one question at a time**. No walls of text. Conversation precedes drafting.

## Inputs

- `$ARGUMENTS` — optional slug to spec (e.g. `/spec pdf-export`).
- `specs/<slug>/concept.md` — the value-prop hypothesis.
- The repo — read freely; ground the design in concrete files, paths, and interfaces.

## How to run this stage

### Preflight — fail fast before any user interaction

- Current directory is a git repo (`git rev-parse --is-inside-work-tree`).
- `gh` is installed and authenticated (`gh auth status`).
- Working tree is clean (`git status --porcelain` is empty).

If any check fails, stop and tell the user precisely which one and the one-line fix.

### 1. Find the concept

If `$ARGUMENTS` is provided, verify `specs/<slug>/concept.md` exists.

Otherwise, list `specs/*/` folders that have a `concept.md` but no `spec.md`, and ask the user which one. If there's only one, propose it and confirm.

### 2. Check that the concept is merged

Verify the concept doc is on the default branch:

- `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
- `git fetch origin <default>`
- `git cat-file -e origin/<default>:specs/<slug>/concept.md`

If the concept isn't on the default branch, tell the user it appears to still be in review and recommend merging the concept PR before spec'ing. Allow them to proceed if they explicitly say so — the spec will be based on a draft concept.

### 3. Read the concept and ground in the repo

Read `specs/<slug>/concept.md` thoroughly — it's the ROI hypothesis you'll defend. Then read the repo widely: which files/modules will change, what existing patterns to extend (cheapest path), what interfaces/data shapes to reuse, what constraints to respect. Read to design, not to report.

### 4. Branch

`git checkout -b spec/<slug> origin/<default>`. If the branch already exists locally or on origin, stop and ask the user whether to (a) pick a different name, (b) resume the existing branch, or (c) delete and recreate. Default to (a) unless the user says otherwise.

### 5. Ask who's proposing the approach

Single question:

> "Do you already have a technical approach in mind, or should I propose one?"

- If they have one: ask single-question follow-ups until you understand it well enough to artifact it.
- If you propose: sketch the smallest viable change grounded in what you read, then artifact it.

### 6. Design via artifact

Communicate the approach through an artifact, not a paragraph. Pick the medium that lets the user react concretely:

- **UI change** → HTML mock (`specs/<slug>/mock.html`)
- **Flow / sequence / state** → Mermaid diagram (file or embedded in `spec.md`)
- **New or changed interface** → code stub showing signatures
- **Data shape change** → sample payload (`specs/<slug>/example.json`) or schema diff
- **Other** → whatever lets the user react concretely

Write artifacts flat to `specs/<slug>/`. One or two sentences of orientation per artifact; let the artifact carry the proposal.

### 7. Converse, single question at a time

Iterate. One question per turn; wait for the answer before asking the next. Refer to artifacts by filename — never re-paste their contents. Topics worth asking about: whether the approach delivers the concept's signal, where you're guessing, what's deliberately omitted, what's the cheapest path through the existing code.

### 8. Watch for disproportionate complexity

As the design firms up, check: is one slice of the concept dragging in outsized cost? (New schema for one rarely-used field. Special-case logic for one segment. A nice-to-have forcing a new dependency.)

If so, surface it: "X costs roughly Y; the rest of the spec is Z. Worth revisiting the concept to cut or reshape X." Suggest going back to `/concept` — **and keep spec'ing the rest**. The partial spec is the input to the concept revisit. Note the revisit in the spec.

### 9. Draft `specs/<slug>/spec.md`

Use this structure. Fields can be `— (n/a, because …)` if genuinely not applicable. Don't omit silently.

```markdown
# Spec: <title>

**Why:** <one-liner traceback to the concept's hypothesis>

**Approach:** <1–2 paragraphs. Reference artifacts: "See [mock.html](./mock.html).">

**Changes:**
- `path/to/file.ext` — <what changes>
- `path/to/other.ext` — <what changes>

**Interfaces:** <only load-bearing ones; code blocks or links to artifact files>

**Sequence:** <ordered steps, smallest first; each step shippable on its own>
1. ...
2. ...

**Acceptance:** <technical behaviors that must be true after the change. Not test design — that's the next stage. Just what must be true.>

**Risks:** <what could go wrong, what's being assumed>

**Out of scope:** <explicit non-goals>

**Concept revisit suggested:** <— (n/a) OR a one-line description of what to revisit and why>
```

### 10. Iterate

Show the draft. Iterate on feedback. Edit in the working tree — **do not commit during iteration**. Spec lands when the user explicitly signs off.

### 11. Commit, push, open PR

After signoff:

- Stage and commit `specs/<slug>/spec.md` and any artifacts you wrote in `specs/<slug>/`.
- Push: `git push -u origin spec/<slug>`.
- Open the PR: `gh pr create --title "Spec: <title>" --body "<body>"`. The body should include:
  - One-line traceback to the concept (`Implements the hypothesis in specs/<slug>/concept.md`)
  - The full spec content inline (so reviewers don't click through)
  - References to artifacts by path (mermaid renders inline on GitHub)
  - If concept revisit was suggested: a clear note at the top

### 12. Hand off

The spec stage is **not complete** until this PR is merged.

- If concept revisit was **not** suggested: "Spec is in review at `<PR URL>`. When merged, run `/verification-plan`."
- If concept revisit **was** suggested: "Spec is in review at `<PR URL>`. The partial spec documents what we learned. Before continuing, revisit `/concept` to address: `<X>`."

## Guardrails

- The spec must trace back to the concept's hypothesis. Every design choice serves the ROI.
- One question at a time. Never batch.
- Favor artifacts. If you're writing more than two paragraphs to describe an approach, you should be writing an HTML mock or a diagram instead.
- No verification design — what to test, how to test, test plans — that's the next stage.
- Loop-back is not an abort. Write the partial spec so the concept revisit has documentation to work from.
- Don't commit during iteration. Only the final spec + artifacts land in git.
- Don't open the PR before signoff.
