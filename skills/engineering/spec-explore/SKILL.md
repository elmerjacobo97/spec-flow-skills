---
name: spec-explore
description: No-stakes thinking partner before a spec exists. Reads the real code, compares 2-3 concrete options with tradeoffs, and sharpens a fuzzy idea into a decision. Creates nothing — no spec files, no code — and ends by recommending /product-spec or /spec when the idea crystallizes.
disable-model-invocation: true
argument-hint: [topic]
allowed-tools: Bash(git status:*), Bash(ls:*)
---

# /spec-explore — Think it through before committing

Not every idea arrives ready for a spec. Sometimes the problem is understood but the approach is not: several options, unclear tradeoffs, or you do not yet know what the codebase already does.

This skill exists for that moment. It is a **no-stakes thinking partner**: it reads the code, compares concrete options, and helps you decide. It creates nothing and commits you to nothing. An exploration can be abandoned at any point with zero cleanup.

The rule of thumb: **explore → then write the spec.** When the idea is clear enough to draw a boundary, `/spec` takes over.

## Session context

Specs already written:
!`ls specs/ 2>/dev/null || echo "none"`

---

## Instructions

### Phase 1 — Set the topic

The received argument is: `$ARGUMENTS`

If it is empty, ask one short question: what do you want to explore? Stop and wait.

Before exploring, check the specs listed above: if the topic is already covered by an existing spec, say so and point at it instead of starting from scratch.

### Phase 2 — Investigate the real code

Never explore in the abstract when the code can answer. Investigate:

- How does the relevant part work today? Read the files; cite them (`file:line`).
- What patterns and constraints already exist in this project?
- What would each plausible approach collide with?

If the investigation changes how you understand the topic, say that before answering.

### Phase 3 — Present findings and options

Respond with this shape:

1. **What exists today** — short, with file references, no guessing.
2. **2–3 concrete options** — each with: what it means in practice, what it costs, what it risks, and how it fits this codebase. More than three options is noise; if the space is bigger, group it.
3. **A recommendation** — pick one and say why, plainly.

Then ask focused questions only if a real fork remains. Do not interrogate; the point is to think together.

### Phase 4 — Iterate until it crystallizes

Keep exploring across turns as needed: dig deeper into one option, test an assumption against the code, compare two approaches side by side.

Exploration has **no artifacts**: no files, no spec sections, no code, no checkboxes, no state. If the user asks to write something down, that is the signal a spec is being born — move to Phase 5.

### Phase 5 — Recommend the next move

When the idea is concrete enough to boundary it, say so and point at exactly one next step:

- Value or metric still unclear → `/product-spec` first.
- Clear enough to define scope, plan, and acceptance criteria → `/spec`.
- Still genuinely fuzzy → keep exploring; say what is missing to move on.

Do not create the spec. The user runs the next skill when ready.

---

## Hard rules

- **Creates nothing.** No files, no code, no spec content, no state changes. Ever.
- **No implementation.** If the answer to the exploration is "just fix it", say that — a bug fix does not need a spec in this workflow.
- **Cite the code.** Options ground in what exists, not in generic advice.
- **Cap it at three options**, each with real tradeoffs. A recommendation with a reason is mandatory.
- **No scope creep by conversation.** If the topic splinters into several features, say it and suggest which one deserves a spec first.

## Summary of expected behavior

```
/spec-explore how to handle persistence for high scores

  Phase 1  →  Topic is set
  Phase 2  →  Reads the current state code; cites files
  Phase 3  →  3 options (localStorage / IndexedDB / JSON file) with tradeoffs
              + recommendation
  Phase 4  →  Iterates as the user digs
  Phase 5  →  "Clear enough. Run /spec NN-slug when ready." — and stops

/spec-explore   (no argument)

  Phase 1  →  Asks what to explore; waits
```
