---
name: spec-status
description: Read-only board of every spec in specs/ — state, implementation progress (plan and criteria checkboxes), dependencies — with flags for stale or blocked specs and a suggested next action for each one. Use to see where the whole project stands without opening files.
disable-model-invocation: true
argument-hint: (no arguments)
allowed-tools: Bash(ls:*), Bash(cat:*)
---

# /spec-status — Board of all specs

One command, the whole picture: which specs exist, what state each one is in, how much of each plan is implemented, and what the next action is. Read-only — it never edits a spec or changes a state.

## Session context

Specs available in this folder:
!`ls specs/ 2>/dev/null || echo "The specs/ folder does not exist"`

---

## Instructions

### Phase 1 — Inventory

List everything in `specs/`:

- `NN-slug.md` → a spec.
- `NN-slug.brief.md` → a product brief (from `/product-spec`).
- `.spec-config.yml` and any other dotfiles → configuration, not specs.
- Ignore anything that is not `.md`.

Pair briefs with their specs by `NN-slug`. A brief with no spec is a valid, pending state — note it, do not flag it as an error.

If there are no specs at all: say so and suggest starting with `/spec`.

### Phase 2 — Read each spec

For each spec file, read:

- The **header**: state (`**Status:**` / `**Estado:**` / equivalent), `Depends on:` / `Depende de:`, date, and the one-sentence objective.
- The **implementation plan**: count checked and unchecked steps (`- [x]` / `- [ ]`), including per-group progress if it is useful.
- The **acceptance criteria**: same count.

Do not modify anything while reading.

### Phase 3 — Render the board

Show a table sorted by `NN`, plus a briefs section:

```
📋 Specs — N found

| # | Slug | State | Plan | Criteria | Depends on | Next |
| --- | --- | --- | --- | --- | --- | --- |
| 01 | mvp-arkanoid | Implementado | 6/6 | 5/5 | — | done |
| 02 | powerups | Borrador | 0/4 | 0/3 | 01 | review and approve |
| 03 | levels | En revisión | 2/5 | 0/4 | 01, 02 | keep drafting with /spec |
| 04 | billing | Aprobado | 0/6 | 0/5 | — | run /spec-impl |

Briefs without a spec:
- 05-push-notifications.brief.md (optional step: run /spec 05)
```

If a table gets too wide, keep the columns above; they are the ones that matter.

**Flags** — mark with `⚠` and explain:

- `Implementado` with unchecked acceptance criteria → "implementation says done, criteria say otherwise — run /spec-verify".
- `Depends on` a spec that is not `Implementado` → "waiting on NN".
- Missing or unrecognized state value → "fix the header".
- Plan fully checked but state still `Aprobado`/`In review` → "ready to close: verify, then set Implementado".

Match states by meaning, in any language — the same rule the other skills follow.

### Phase 4 — Summary and next actions

Close with:

- Counts per state (e.g. "2 Borrador · 1 Aprobado · 1 Implementado").
- One suggested next action per spec that is not done, in the same words of the table.

Do not execute any of those actions. This skill is a board; the actions belong to the other skills.

---

## Hard rules

- **Read-only.** Never edit a spec, never change a state, never write code, never commit.
- **No invented progress.** Counts come from the actual lines in the file, not from assumptions.
- **Briefs are not specs.** They are shown in their own section, never mixed into the table.
- **If the user asks for an action** (approve, implement, verify), point to the right skill: `/spec-edit`, `/spec-impl`, `/spec-verify`.

## Summary of expected behavior

```
/spec-status

  Phase 1  →  Inventories specs/ (specs, briefs, config)
  Phase 2  →  Reads each spec: state, dependencies, checkbox counts
  Phase 3  →  Renders the table with flags + briefs without a spec
  Phase 4  →  Counts per state + a next action per pending spec
```
