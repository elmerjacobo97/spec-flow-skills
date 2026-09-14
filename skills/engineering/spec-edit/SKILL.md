---
name: spec-edit
description: Edits an existing spec in place, guided and minimal. Use when you are re-reading a spec — draft or approved — and something does not fit, or you want to change scope, plan, criteria, or decisions without regenerating the whole document. It maps the change across sections, shows a preview, applies after one confirmation, and never touches the state line.
disable-model-invocation: true
argument-hint: <NN-spec-name> [what-to-change]
allowed-tools: Bash(ls:*), Bash(date:*)
---

# /spec-edit — Guided editor of existing specs

This skill revises a spec that already exists. It is not a generator and not an implementer:

- `/spec` creates a spec.
- `/spec-edit` changes one, in place and minimally.
- `/spec-impl` implements it.

The mental model: a spec is the live plan, not a signed contract. Editing it is normal. But here the edit is **guided**: the skill maps the change across sections, shows exactly what will change, and waits for one confirmation before writing.

## Session context

Specs available in this folder:
!`ls specs/ 2>/dev/null || echo "The specs/ folder does not exist"`

Today's date:
!`date +%F`

---

## Instructions

Follow these phases in order.

### Phase 1 — Identify the spec

The received argument is: `$ARGUMENTS`

Split it into two parts:

- The **first token** is the spec name (or empty).
- **Everything after it** is the change request. It may be empty — that is fine.

If the spec name is empty:

- List the files available in `specs/` (you already have them above).
- Ask the user which spec they want to edit, and what they want to change.
- Stop and wait. Do not continue.

If the spec name has a value, find the file in `specs/` using the same flexible matching the other skills use: full name (`03-levels-and-highscores`), only the number (`03`), or only the slug (`levels-and-highscores`). If you do not find it, show the available specs and ask for the correct name.

### Phase 2 — Read and index the spec

Read the spec file completely before touching anything. Build a mental index of:

- Its **state** (`**Status:**` / `**Estado:**` / equivalent, any language) — you need it before applying any edit and in the final advisory.
- Its **sections by meaning**, not by exact wording: objective, scope (in / out), data model, implementation plan (groups and checkbox steps), acceptance criteria, decisions, risks, "what is not in".
- Its **progress**: how many plan steps are already checked (`- [x]`) and how many acceptance criteria are checked. Those lines are implementation history — handle with care.
- A matching `specs/NN-slug.brief.md`, if it exists: read it only for context. **Never edit it** — this skill edits the technical spec only.

### Phase 3 — Understand the change

If the change request is already clear and specific (e.g. "remove the combo system from the plan"), skip to Phase 4.

If it is vague or missing, ask **one block of 2–4 concrete questions** — not a long interview. Examples of the right shape:

1. What exactly changes — scope, plan, criteria, decisions, or several?
2. What must stay exactly as it is?
3. Does this change anything the spec currently puts out of scope?
4. Is this still the same work refined, or different work?

**Update vs new spec check.** Apply this rule and say so if it triggers:

- **Same goal, better approach, or narrowed scope** → edit this spec.
- **The intent fundamentally changed, or the scope exploded into different work** → recommend a new spec with `/spec`, and stop there unless the user insists on editing.

Reply in the same language as the initial prompt.

### Phase 4 — Impact analysis and preview

This is the core of the skill. Map the change to the spec's sections before writing a single character.

**Affected sections.** For each section the change touches, show a concise preview:

```
## Preview

### Scope — in
- <old line>
+ <new line>

### Implementation plan — Group 2
- <step 2.2 as it is now>
+ <step 2.2 as it would be>
```

**Cascades.** A change in one section usually forces another. Detect and include them in the preview — never apply them silently:

- Scope changes → check the implementation plan and the acceptance criteria still cover the new scope.
- A new or removed plan step → check acceptance criteria still match.
- A reverted decision → the decisions section needs its "no" and reason recorded.
- A new risk → propose the risks entry.

**Conflicts.** Flag anything that touches work already done:

- If an edit would change or remove a step marked `- [x]`, or an acceptance criterion already checked → warn, and offer three options: leave it untouched, uncheck it with explicit permission, or record it as a follow-up. **Never uncheck `- [x]` on your own initiative.**
- If the state is `Approved` or the equivalent: announce that after this edit the spec needs the human to re-approve it (the state line is human-owned; see Phase 5).
- If the state is `Implemented` or the equivalent: warn that the code may now diverge from the spec, and that the spec should be re-approved before `/spec-impl` runs again.

**One confirmation.** Show the complete preview — edits plus cascades — and ask once:

```
Apply these edits to specs/NN-slug.md? [Y/n]
```

Do not apply anything before this confirmation. Do not apply a partial version.

### Phase 5 — Apply the edits

Once confirmed:

- Apply **minimal diffs**. Do not regenerate the whole document, do not reorder steps, do not rewrite sections that were not in the preview.
- Preserve everything of the spec's voice and format: its language, its heading style, its table style, and especially the checkbox format (`### Group N` + `- [ ] N.M`).
- Update the header's `**Date:**` / `**Fecha:**` / equivalent to today's date (the date shown in the session context).
- **Never edit the state line** (`**Status:**` / `**Estado:**` / equivalent). That transition belongs to the human.

### Phase 6 — Coherence check (report only)

After applying, run a quick consistency pass and report what you find:

- Every scope item still has plan coverage; every plan step still serves the scope.
- Acceptance criteria are still verifiable booleans, still aligned with the plan.
- Group sizes stay reasonable (2–5 steps). If a group grew too big, suggest splitting it.
- No TODOs or placeholder text was introduced.
- The "out of scope" and "what is not in" sections still tell the truth.

Fix nothing beyond the confirmed preview without asking first.

### Phase 7 — Summary and advisory

Close with a compact summary in chat, in the same language as the conversation:

```
✅ Spec updated: specs/NN-slug.md

Changed:  <section>: <what changed> — per affected section
Why:      <the motivating change, one line>
Cascades: <sections updated to stay coherent, or "none">
Advisory: <"re-approval needed (was Approved)" | "code drift risk (was Implemented)" | "none">
Next:     re-read the file. If it was Approved, re-approve it manually when you are happy.
          To continue implementation, run /spec-impl NN-slug — it resumes from the first unchecked step.
```

---

## Hard rules

- **Never write code or implement anything.** This skill edits one Markdown file and stops.
- **Never edit the state line.** Say what the state requires (re-approval, re-review) — the human performs it.
- **Never regenerate the whole spec.** Minimal diffs, language and format preserved.
- **Never uncheck `- [x]`** steps or criteria without explicit permission.
- **Never edit the `.brief.md`.** The product brief is out of this skill's scope.
- **One spec per invocation.** If the change affects another spec, say so and suggest running `/spec-edit` on it separately.
- **If the scope exploded**, recommend a new spec instead of stretching this one.

## Summary of expected behavior

```
/spec-edit 03-levels-and-highscores "remove the combo system from the plan"

  Phase 1  →  Finds specs/03-levels-and-highscores.md
  Phase 2  →  Indexes sections; notes state and checked steps
  Phase 3  →  Request is clear; update vs new spec → edit wins
  Phase 4  →  Preview: plan edits + criteria cascade + conflict check
              One confirmation
  Phase 5  →  Applies minimal edits, updates Fecha, leaves Estado alone
  Phase 6  →  Coherence report
  Phase 7  →  Summary + advisory (re-approval if it was Approved)

/spec-edit 03-levels-and-highscores

  Phase 1  →  Finds the spec
  Phase 3  →  Asks one block of 2–4 concrete questions, waits
  Phase 4  →  Same flow from there

/spec-edit 09-billing  (state: Implemented)

  Phase 4  →  Warns: code may diverge; re-approval needed before /spec-impl runs again
              Applies only if the user confirms
```
