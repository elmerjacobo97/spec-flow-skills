---
name: product-spec
description: Defines the business case for a feature before /spec — who asked for it, what metric it should move, the smallest version that tests the hypothesis, and when to kill it. Use it before writing a technical spec whose value or audience is not already obvious.
disable-model-invocation: true
argument-hint: 'short feature description or business problem'
---

# /product-spec — Business case before the technical spec

This skill runs **before** `/spec`, not instead of it. `/spec` designs *what gets built*. `/product-spec` decides *whether it should be built at all*, and by what number you will know if it worked.

## Philosophy

A feature with no metric is an opinion with a due date. The failure mode this skill targets is not "we built the wrong thing" — it's "we built a thing and nobody can say, three months later, whether it worked." That question has to be answered **before** the first line of the technical spec, not after shipping.

This is deliberately a **single fast pass**, the opposite pacing of `/spec`. If this takes longer than answering five direct questions, something is being over-designed — stop and write the brief with what you have.

Read `template.md` (in the same directory as this skill) for the exact shape of the output document.

## When to use this — and when to skip it

Not every spec needs a business case in front of it.

**Use it when:**
- The feature's value to the user is assumed, not evidenced.
- Nobody can currently name the metric this is supposed to move.
- More than one reasonable scope exists and picking the wrong one is expensive.

**Skip it and go straight to `/spec` when:**
- It's a bug fix, a security patch, or paying down technical debt.
- A user or client already stated the need with hard evidence (ticket count, a specific complaint, a contract requirement).
- It's internal tooling with no external metric to speak of.

If you are not sure which case this is, ask the user directly — do not assume it needs a brief just because the skill was invoked.

## Command flow

- Reply in the same language as the initial prompt (same rule as `/spec`).
- Everything happens in **one question block**, not phases. Ask once, wait for the answers, write the brief.

### Step 1 — Read context

1. Read the project-memory file, trying in order: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`.
2. List `specs/` to find the next sequential `NN` and to check whether a `specs/NN-slug.brief.md` already exists for a feature with the same slug — if the user is refining a brief instead of starting one, resume that file instead of creating a new number.
3. Note whether the project has any analytics/instrumentation described in its memory file. This determines whether Step 3's escape hatch is likely to trigger.

### Step 2 — One question block

Ask all of these together, numbered, with a recommendation where you offer options. Do not spread this across multiple round trips.

1. **Evidence.** Who asked for this, and how many? A single anecdote, a support ticket, and "N users hit this in the last month" are different strengths of evidence — which is it?
2. **Current workaround.** What does the user do today without this feature? If there is no workaround and no visible pain, that itself is a finding — say so instead of inventing one.
3. **Metric.** What number is this supposed to move? Reject vague answers ("better UX", "easier to use") and ask for a number: current baseline, target, and the date you'll check it. If the user cannot name a baseline, that is the signal for the escape hatch in Step 3 — don't force a fake number.
4. **The 20% version.** What is the smallest version that would still test the hypothesis? Propose one yourself and ask the user to cut it further if possible.
5. **Kill criterion.** At the check-in date, what result makes this get reverted or deprioritized rather than iterated on?

### Step 3 — Escape hatch: no baseline, no brief

If Step 2.3 comes back with no real baseline because the product has no instrumentation for the relevant behavior, **do not fabricate one and do not write a feature brief.** Write a brief whose only content is: what to instrument (concrete event names and properties), why those events answer the question this feature raises, and that the feature itself should wait until there is at least one data point of baseline. This is the correct output in that situation, not a failure of the skill.

### Step 4 — Write the brief

Once the five answers exist (or Step 3's instrumentation-only case applies):

1. Reuse the `NN-slug` found in Step 1 if a matching brief already existed; otherwise assign the next sequential `NN` and derive a short slug from the feature description, confirming the slug with the user before writing.
2. Fill in every section of `template.md` — do not skip the metric table or the kill criterion, even if the answer is short.
3. Save to `specs/NN-slug.brief.md` with `Status: Draft`.
4. Tell the user:
   - The path just written.
   - That `/spec NN-slug` is the next step, and it will read this brief automatically for business context.
   - **Stop there.** Do not draft the technical spec, do not propose a data model, do not write code.

## Hard rules

- **Never write a technical spec or code here.** Only the `.brief.md` file.
- **Never accept a metric without a baseline, a target, and a check-in date.** If any of the three is missing, keep asking or fall back to the Step 3 escape hatch.
- **Never invent evidence.** If the user says "I think users want this," write that down as what it is — an assumption, not evidence — and say the brief's confidence is low.
- **Never let this replace `/spec`.** This skill produces the *why*; `/spec` still owns the *what* and the *how*.
- **If the user wants to skip straight to `/spec`,** that's a legitimate call per the "when to skip it" section above — don't insist.

## Tone

Same as `/spec`: direct, no hedging, no apologizing for asking. A vague answer to the metric question gets pushed back on once, concretely — not accepted and softened in the brief.

## Arguments

If invoked as `/product-spec push-notifications`, use `push-notifications` as the initial slug suggestion, confirming with the user before writing the file. If invoked with no arguments, start with Step 1 and ask for a one-sentence description of the feature under consideration.
