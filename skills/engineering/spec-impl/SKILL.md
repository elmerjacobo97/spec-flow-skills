---
name: spec-impl
description: Implements an approved spec group by group. Validates that the state means "Approved" (in any language), resolves the git branch (reuses the ticket work branch when one is active, otherwise creates spec-NN-slug), ticks off the implementation plan inside the spec as it progresses, stops after each group for review and commit, and verifies the acceptance criteria at the end. With --one-shot it runs every group without pauses.
disable-model-invocation: true
argument-hint: <NN-spec-name> [--one-shot]
allowed-tools: Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git symbolic-ref:*), Bash(cat:*), Bash(ls:*)
---

# /spec-impl — Implementer of approved specs

## Session context

Current repository state:
!`git status --short`

Current branch:
!`git branch --show-current`

Default branch:
!`git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo "unknown"`

Specs available in this folder:
!`ls specs/ 2>/dev/null || echo "The specs/ folder does not exist"`

Branch-creation config:
!`cat specs/.spec-config.yml 2>/dev/null || echo "AutoCreateBranch: true (default, no config file)"`

---

## Instructions

Follow these four phases in strict order. **Do not advance to the next phase if the previous one did not complete correctly.**

---

### Phase 1 — Identify the spec

The received argument is: `$ARGUMENTS`

First, parse the argument:

- If it contains `--one-shot` (anywhere), remove it and remember: **one-shot mode is on**. In one-shot mode you still work group by group internally, but you do not pause between groups — see Phase 4.
- The rest of the argument is the spec name.

If the spec name is empty:

- List the files available in `specs/` (you already have them above).
- Ask the user to specify the exact name of the spec.
- Stop and wait for an answer. Do not continue.

If the spec name has a value:

- Look for the file in `specs/`. The user may have written the full name (`01-mvp-arkanoid`), only the number (`01`), or only the slug (`mvp-arkanoid`). Try to find the correct file in any of those cases.
- If you do not find the file, show the available specs and ask the user to correct the name.
- If you do find it, continue to Phase 2.

---

### Phase 2 — Validate the spec's state

Read the spec file you located in Phase 1 using the Read tool or `cat`.

In the file's contents, look for the line that contains the spec's state. The header label is typically `**Status:**` (English) or `**Estado:**` (Spanish), but it may use any language. Match by position (status line near the top of the spec) and by the surrounding state machine, not by the exact label.

**Absolute rule:** You can only continue if the state **means "Approved"** — regardless of the language used.

Treat any of the following (and their equivalents in other languages) as the **Approved** state and continue:

- English: `Approved`
- Spanish: `Aprobado`
- Portuguese: `Aprovado`
- French: `Approuvé`
- German: `Genehmigt`
- Italian: `Approvato`
- …or any other language's word that clearly means "approved"

Anything else (Draft / Borrador, In review / En revisión, Implemented / Implementado, Obsolete / Obsoleto, or any unrecognized value) means **stop** and show the error message below.

| State category                            | Examples (any language)                           | Action                                                                     |
| ----------------------------------------- | ------------------------------------------------- | -------------------------------------------------------------------------- |
| Approved                                  | `Approved`, `Aprobado`, `Aprovado`, `Approuvé`, … | Continue to Phase 3.                                                       |
| Draft                                     | `Draft`, `Borrador`, …                            | Stop. Show the error message below.                                        |
| In review                                 | `In review`, `En revisión`, …                     | Stop. Show the error message below.                                        |
| Implemented                               | `Implemented`, `Implementado`, …                  | Stop. Show the error message below.                                        |
| Obsolete                                  | `Obsolete`, `Obsoleto`, …                         | Stop. Show the error message below.                                        |
| State line not found / unrecognized value | —                                                 | Stop. The file does not follow the expected format. Tell this to the user. |

If you are unsure whether a value means "approved", **do not assume**. Stop and ask the user to clarify or to update the spec to the canonical wording.

**Standard error message when the state does not mean Approved:**

```
❌ I cannot implement this spec.

Current state: [STATE FOUND]
I only work with specs whose state means "Approved" (e.g. `Approved`, `Aprobado`,
or the equivalent in another language).

To continue you have two options:
  1. If the spec is ready to be implemented, open it and change the state
     to "Approved" (or the equivalent term your team uses) manually.
     That change is made by the human, not the agent.
  2. If the spec still needs work, use /spec [name] to resume it.
```

Do not offer alternatives, do not suggest "I can still start if you want". The block is intentional.

---

### Phase 3 — Resolve the git branch and switch to it

Once you have confirmed the state means `Approved`:

1. Derive the branch name from the spec file's full name, without the extension. Format: `spec-NN-slug`. Examples:

   - `01-mvp-arkanoid.md` → branch `spec-01-mvp-arkanoid`
   - `02-powerups.md` → branch `spec-02-powerups`

2. Determine the **current branch** and the **default branch** from the Session context above. Strip the `origin/` prefix from the default branch. If it shows `unknown`, fall back in order to `development`, `develop`, `main`, `master` — the first one that exists locally or remotely.

3. **Existing work branch (ticket flow) — check this before anything else.**

   If the current branch is **neither the default branch nor `spec-NN-slug`**, treat it as an existing work branch (typically created by the ticket flow, e.g. `feat/<slug>`, `fix/<slug>`, `spec-NN/T<N>-<slug>`):

   - **Do not create any branch. Do not read or apply `AutoCreateBranch`.** Stay on the current branch.
   - Announce: `Using existing work branch: <branch> (ticket flow; no new branch created).`
   - Skip to step 5.

   This rule exists so `/spec-impl` cooperates with ticket-driven work: the ticket is the unit of tracking, and there is one work branch per task. Do not duplicate it with `spec-NN-slug`.

4. Read the `AutoCreateBranch` flag from the **Branch-creation config** shown in the session context above.

   - If the config file does not exist, the value is missing, or the value is unrecognized → treat it as `true` (the default).
   - Only an explicit `false` (in any capitalization) disables automatic branch creation.

   **If `AutoCreateBranch` is `true` (default):** proceed without asking.

   - If the branch **does not exist**: create it with `git checkout -b spec-NN-slug`.
   - If the branch **already exists**: inform the user that the branch already existed (it may mean previous work is being resumed).
   - If the **current branch is already** `spec-NN-slug`: stay on it and tell the user previous work is being resumed.
   - In all cases: switch to the branch with `git checkout spec-NN-slug` and confirm the change was successful before continuing.

   **If `AutoCreateBranch` is `false`:** ask before touching git. Show:

   ```
   AutoCreateBranch is set to false.
   Create and switch to the branch spec-NN-slug? [y/N]
   ```

   - If the user answers **yes**: create/switch to the branch exactly as in the `true` case above.
   - If the user answers **no** or leaves it empty: **do not create any branch.** Tell the user you will implement on the current branch (the one shown in the session context above) and ask for explicit confirmation to continue there. Do not improvise — wait for the answer.

5. Visually confirm to the user the spec is ready and which branch is active:

   ```
   ✅ Ready to implement.

   Spec:   specs/NN-slug.md
   Branch: <branch>  (active)   (← e.g. "spec-NN-slug (created)", "spec-NN-slug (resumed)", or "feat/<slug> (existing ticket branch)")
   State:  Approved   (← echo back the actual value found in the spec)
   ```

6. **Do not start implementing yet.** First show the spec summary to the user so they have it fresh. Extract and show:
   - The **objective** (the line after `**Objective:**` / `**Objetivo:**` / equivalent label).
   - The **scope** (the `## Scope` / `## Alcance` / equivalent section).
   - The **implementation plan** (the grouped checkbox steps — `## Implementation plan` / `## Plan de implementación` / equivalent). If it has no groups, Phase 4 will propose them.
   - The **acceptance criteria** (the checklist — `## Acceptance criteria` / `## Criterios de aceptación` / equivalent).

Match section headings by meaning, not by exact wording — the spec may be authored in any language.

---

### Phase 4 — Implement group by group

**First, resolve the groups.** The implementation plan may already be organized in groups (`### Group N — …`, `## Phase N`, `## Fase N`, or any heading that clearly means a group). Match by meaning, not by exact wording.

- **If the plan has groups:** use them as they are.
- **If the plan is a flat checklist:** propose a grouping of related steps — 2–5 steps per group, each group a coherent, independently reviewable chunk (e.g. schema → backend → UI → wiring). Show the proposed mapping to the user, ask for confirmation, and once confirmed write the group headings into the spec above the corresponding steps (`### Group 1 — <name>`). Do not reorder steps.
- **If the plan has neither checkboxes nor groups** (legacy numbered list): convert it once — group headings plus `- [ ] N.M ...` steps — show the result, confirm, and continue.

Then, depending on the mode:

**Default mode — group by group.** Show the group list and ask for a single start confirmation:

```
I am going to implement the plan group by group.
After each group I will stop so you can review the diff and commit.

Group 1/M: <name> — starting now.

Shall I start?
```

Wait for explicit confirmation ("yes", "go ahead", "go", or equivalent). Then, for each group, from first to last:

1. Implement the group's steps **in order**, without stopping between steps.
2. After each step completes, edit the spec and change that step's `- [ ]` to `- [x]`. One line, nothing else.
3. When the group's last step is done, give a mini-summary:

   ```
   ✅ Group N/M completed — <name>

   Files: <paths touched>
   What:  <one or two lines>
   Why:   <key decisions or deviations, each with its reason — or "no deviations">
   Next:  review the diff and commit. Say "continue" for Group N+1.
   ```

   On the **last** group, replace the `Next:` line with `Next: final verification — acceptance criteria` and continue to the verification below instead of stopping.

4. **Stop and wait.** Do not start the next group until the user explicitly says to continue ("continue", "go ahead", "siguiente", "dale", or equivalent).
5. If a group is interrupted mid-way, the first unchecked `- [ ]` step is where work resumes — including in a later session. Skip steps already marked `- [x]`.

**One-shot mode (`--one-shot`).** Same group logic, but never pause between groups: after each group's mini-summary, continue immediately to the next group. Pause only on one of the stop conditions below. Use it for small specs where reviewing group by group adds no value.

**Rules that hold in both modes:**

**One rule above all:** implement what the spec says. If something in the spec looks suboptimal to you, mention it as an observation but implement what was agreed. Changes to the spec go into the spec, not into the code by surprise (use `/spec-edit` for that).

**Only three reasons to stop mid-run:**

1. **Real ambiguity** the spec does not resolve: describe it exactly, present two or three concrete options, wait for the user's decision. Do not improvise.
2. **A step fails** or leaves the project broken (tests, build, or the step's own check fail): stop, report what failed, and do **not** mark that step `- [x]`.
3. **The user asks for something out of scope:** remind them it is out of this spec's scope, suggest noting it for the next spec, do not implement it on this branch.

**When the last group is done — verify the acceptance criteria:**

1. Go through the spec's acceptance criteria one by one.
2. For each one, look for real evidence: run the project's test suite, build, or linter, or the manual check the spec itself describes (if cheap to run). Do not mark anything you did not actually verify.
3. Mark `- [x]` only the criteria you verified with evidence. Leave the rest unchecked.
4. Never edit the `**Estado:**` / `**Status:**` line. That change is made by the human, not the agent.

**Final summary (chat only — do not append it to the spec):**

```
✅ Spec implemented — M/M groups completed.

Steps:      N/N completed (spec checkboxes updated)
Files:      <paths touched>
Why:        <key decisions or deviations during the whole run — or "no deviations">
Verified:   <acceptance criteria marked [x], with the evidence used>
Pending:    <criteria left unchecked and why they need human review>
Next:       review the final diff, commit, set the spec's state to "Implemented"
            manually, then merge.
```

---

## Summary of expected behavior

```
/impl-spec 01-mvp-arkanoid  (on the default branch)

  Phase 1  →  Finds specs/01-mvp-arkanoid.md
  Phase 2  →  Reads the state → "Approved" (or "Aprobado", etc.) → ✅ continues
  Phase 3  →  git checkout -b spec-01-mvp-arkanoid → git checkout spec-01-mvp-arkanoid
              Shows objective, scope, grouped plan and criteria
  Phase 4  →  Implements group by group: one start confirmation, ticks each step
              inside the spec, then stops after each group for review + commit
              Ends by verifying the acceptance criteria and summarizing

/impl-spec 01-mvp-arkanoid --one-shot  (small specs)

  Phase 4  →  Same group logic with no pauses: mini-summary after each group
              and continues; ends by verifying the acceptance criteria

/impl-spec 01-mvp-arkanoid  (on a ticket work branch, e.g. feat/arkanoid-levels)

  Phase 1  →  Finds specs/01-mvp-arkanoid.md
  Phase 2  →  Reads the state → "Approved" → ✅ continues
  Phase 3  →  Current branch is an existing work branch (ticket flow)
              → stays on feat/arkanoid-levels, does not create spec-01-mvp-arkanoid
  Phase 4  →  Group by group on the ticket branch, same flow

/impl-spec 02-powerups  (state: Draft / Borrador)

  Phase 1  →  Finds specs/02-powerups.md
  Phase 2  →  Reads the state → "Draft" → ❌ stops
              Shows the standard error message
              Does not create branch, does not touch code
```

**Branch creation is controlled by the `AutoCreateBranch` flag** in `specs/.spec-config.yml`. It defaults to `true` (create the branch automatically when starting from the default branch). Set it to `false` to make Phase 3 ask `[y/N]` before creating the branch. The existing-work-branch rule (ticket flow) takes precedence over `AutoCreateBranch`: in that case no branch is created and no question is asked.

**Flat plans get grouped, not guessed:** if an approved spec has a flat checklist or an older numbered list, Phase 4 proposes a grouping, writes it into the spec after one confirmation, and then implements group by group. The grouping stays in the spec, so a resumed run knows exactly where it stopped.

**`--one-shot`** is the only argument flag. It skips the between-group pauses (and the review they force) for specs small enough that a single review at the end is enough. It never skips the ambiguity and broken-step stops.
