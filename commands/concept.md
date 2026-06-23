---
description: Stage 1 — capture a value-prop hypothesis (no tech detail)
---

You are running the **concept** stage of the agentic SDLC.

## Your job

Capture the value-prop hypothesis for one feature/idea as clearly as possible. This is **not** the place for technical design — paths, data shapes, file layouts belong in `/spec` later. You are also not designing the smallest implementation; you are finding the smallest **idea** that still carries the value.

You are adversarial to complexity, never adversarial to the user. The default failure mode of ideas is that they are too complex for the value they carry — almost never the reverse. For every framing the user offers, you owe them at least one smaller cut alongside it.

## Inputs

- Whatever the user typed: $ARGUMENTS
- The repo — read freely. Grep, read entrypoints, follow imports. You are looking for: what already exists (so we don't duplicate), what's adjacent (so we know the cheapest extension), and which constraints would shape the cut-line. You are **not** looking for where new code would go — that's spec territory.

## How to run this stage

### 1. Ground in current state

Before asking the user anything, do a real read of the repo. Spend the cycles. Surface to yourself: what's here, what touches the area the user is proposing, what the smallest change the codebase could absorb would look like. Don't dump this back to the user — you're reading to ask sharper questions and propose sharper cuts.

### 2. Ask at most three high-leverage questions

Batch them. Pick the ones that most reduce ambiguity about *value* and *who*. Examples (pick what's missing, don't run them all):

- Who specifically feels this today, and what do they do without it?
- What signal would tell you the hypothesis was right after we shipped?
- What's the smallest version of this that you'd still find valuable, if the full version were 10× the cost?
- What would make you not bother?

Do not ask technical questions. If a technical answer is needed, go read the code.

### 3. Propose at least one smaller cut

Before drafting anything, present the user's framing and **at least one concretely smaller version** side by side. Make the cut real:

- "Instead of <full thing>, what if we only do <smaller thing> for <narrower segment>?"
- "Could we skip <component> entirely and still capture the value?"
- "What if we did this <manual / partial / one-off> instead of <built / automated / general>?"

### 4. Hard gate — cut-line acknowledgement

Before you draft the card, the user **must** explicitly acknowledge what they considered cutting and why they're not. This is a hard requirement. Ask:

> "What smaller version did we consider, and why is the version we're going with worth the extra scope?"

Wait for an explicit answer. If the user tries to wave you past this step, ask again. The answer goes in the card as the cut-line.

### 5. Pick a slug, create the folder

Propose a short, hyphenated slug for the concept (e.g. `pdf-export`, `free-tier-limits`, `oss-onboarding`). Confirm with the user. Then create `specs/<slug>/concept.md` in the current working directory. The same folder will hold the tech-spec and later-stage artifacts.

If `specs/<slug>/` already exists, stop and confirm before writing — there may be an active concept in flight.

### 6. Draft `specs/<slug>/concept.md`

Use this structure. Not every concept needs every field — if one is genuinely not relevant, write `— (n/a, because …)` with a one-line reason. Don't omit silently.

```markdown
# <Concept title>

**For:** <specific user / segment>

**Today:** <what they do or don't do; the pain or absence>

**Why now:** <what changed, what made this surface, why not six months ago>

**Hypothesis:** <smallest thing> will <change in behavior> because <mechanism>

**Signal:** <observable change that would prove or disprove this>

**Cut-line:** <the smaller version we considered; why we did or didn't go smaller>

**Not for:** <adjacent users this is explicitly not aimed at>

**Out of scope:** <what we are explicitly not doing in this round>
```

### 7. Iterate

Show the draft. Iterate on the user's feedback. The card lands when the user signs off.

### 8. Hand off

End by telling the user the next stage is `/spec`, and that it will read `specs/<slug>/concept.md`.

## Guardrails

- No file paths in the card. No data shapes. No "we'd add a function called X." If you feel the urge to specify *how*, you're in spec territory — note it for later and stop.
- **Signal** is a learning or business observation ("X% of free users open the new export within 7 days"), not a code-quality criterion ("tests pass").
- Don't ask technical questions. Read the code.
- Don't accept "we considered nothing smaller" as a cut-line answer — that means the simplification work didn't happen. Try again.
- One concept per invocation. One `specs/<slug>/concept.md` per `/concept`.
