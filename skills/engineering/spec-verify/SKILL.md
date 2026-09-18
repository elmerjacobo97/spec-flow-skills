---
name: spec-verify
description: Audits whether an implementation matches its spec — completeness, correctness, coherence — with CRITICAL/WARNING/SUGGESTION findings and a mismatch-direction verdict. Read-only. It never edits code, checkboxes, or the state line. Use after /spec-impl, before merging, or later to catch drift.
disable-model-invocation: true
argument-hint: <NN-spec-name>
allowed-tools: Bash(git status:*), Bash(git branch:*), Bash(cat:*), Bash(ls:*)
---

# /spec-verify — Independent audit of spec against code

`/spec-impl` verifies the acceptance criteria at the end of its own run. This skill is different: it is a **second, independent look**, and it can be re-run at any time — before merging, after a refactor, or months later to catch drift.

It is an audit. It reads, searches, and runs checks; it never fixes anything.

## Session context

Current repository state:
!`git status --short`

Current branch:
!`git branch --show-current`

Specs available in this folder:
!`ls specs/ 2>/dev/null || echo "The specs/ folder does not exist"`

---

## Instructions

### Phase 1 — Identify the spec

The received argument is: `$ARGUMENTS`

If it is empty, list the specs available above and ask which one to audit. Stop and wait.

If it has a value, find the file using the same flexible matching the other skills use: full name (`03-levels-and-highscores`), only the number (`03`), or only the slug (`levels-and-highscores`). If you do not find it, show the available specs and ask for the correct name.

### Phase 2 — Read and index

Read the spec completely. Index, by meaning rather than exact wording:

- The **state** — you need it for the verdict.
- The **implementation plan**: groups, steps, and which ones are already checked (`- [x]`).
- The **acceptance criteria** and which ones are checked.
- The **decisions** section and the **data model** — both are things code can contradict.
- The areas of the codebase the spec touches (files it names, modules it describes).

If the spec shows **no implementation progress** (nothing checked, state still `Draft` / `In review` / equivalent), say so and stop: there is nothing to verify yet. Suggest `/spec-impl` when it is approved.

### Phase 3 — Gather evidence

Checkboxes are a claim, not evidence. Verify each one against the real code.

For every plan step and every acceptance criterion:

- Find the code that implements it (search the codebase for the names, files, and behaviors the spec mentions).
- Note the exact location: `file:line`.
- If the project defines a test suite, build, or linter (`package.json` scripts, `Makefile`, CI config, or language-standard commands), run the relevant ones and record the exact command and its result. If none is defined, say so — absence of tests is itself a finding.

Useful searches: symbols from the data model, strings from the decisions (storage keys, event names), file paths the plan names.

### Phase 4 — Report

Present findings in three dimensions, each finding tagged with severity and evidence:

```
🔍 spec-verify — specs/NN-slug.md (state: <STATE>)

COMPLETENESS
✓ 7/7 plan steps checked and evidenced in code
✓ 5/6 acceptance criteria found in code
⚠ WARNING  Criterion "Reloading the page preserves high-scores" —
           persistence code not found (searched localStorage/IndexedDB)

CORRECTNESS
✓ Implementation matches the spec's intent for level progression
⚠ WARNING  Edge case from the spec ("empty high-score list") not handled
           (src/ui/scores.js:44)
✗ CRITICAL  Score formula adds 5 points, spec says exactly 10
           (src/game/scoring.js:12)

COHERENCE
✓ Decisions reflected: versioned key `save:v1` used (src/persistence.js:8)
⚠ SUGGESTION  Spec names the module `src/levels.js`, code calls it `src/stages.js`
```

Severity rules:

- **CRITICAL** — the code contradicts the spec or misses something the user would notice. Broken feature, wrong numbers, missing behavior.
- **WARNING** — gap or risk: edge case unhandled, claim without evidence, missing test coverage for a criterion.
- **SUGGESTION** — naming, structure, or documentation drift. Nothing is broken.

### Phase 5 — Verdict and mismatch direction

Close with a short verdict:

```
SUMMARY
Critical: 0 · Warnings: 2 · Suggestions: 1

Verdict:        safe to merge with warnings
Mismatch:       none | spec is stale (fix with /spec-edit) | code drifted (fix the code)
Next:           <concrete step — e.g. fix the findings, then /spec-close NN-slug
```

The **mismatch direction** matters and must be explicit:

- **Code is right, the spec is stale** → the authoritative fix is `/spec-edit NN-slug` (reconcile the spec to reality).
- **Spec is right, the code drifted** → the fix is code; the biggest gaps are already listed with locations.

Never decide this silently in ambiguous cases — present what you found and let the human choose.

**Closing is a separate step.** This skill never commits, merges, or marks the spec. When the audit passes, the close request is `/spec-close NN-slug`; when it finds gaps, point at `/spec-edit` (stale spec) or the code fix first, and close after that.

---

## Hard rules

- **Read-only.** Never edit code, never edit the spec, never mark checkboxes, never touch the `**Estado:**` / `**Status:**` line.
- **Never fix while verifying.** If the user asks you to fix something found here, point them to `/spec-impl` (code) or `/spec-edit` (spec) as separate steps.
- **Checkboxes are claims, not evidence.** A checked box with no code behind it is a finding.
- **Evidence or it did not happen.** Every finding carries a `file:line` or the exact command and result.
- **No tests defined?** Say it plainly — do not pretend the absence of tests is a pass.

## Summary of expected behavior

```
/spec-verify 03-levels-and-highscores   (state: Implementado)

  Phase 1  →  Finds specs/03-levels-and-highscores.md
  Phase 2  →  Indexes plan steps, criteria, decisions, data model
  Phase 3  →  Searches code for each claim; runs tests/build/lint if defined
  Phase 4  →  Reports completeness / correctness / coherence with severities
  Phase 5  →  Verdict + mismatch direction + next step (close with /spec-close when clean)

/spec-verify 04-draft-feature   (state: Borrador, nothing checked)

  Phase 2  →  No implementation progress → says so and stops
```
