[English](README.md) · [Español](README-es.md)

<p align="center">
  <h1 align="center">Spec-Driven Skills for Claude Code</h1>
  <p align="center">Plan the feature. Approve it. Implement it group by group.</p>
</p>

<p align="center">
  <img alt="License" src="https://img.shields.io/github/license/elmerjacobo97/spec-flow-skills">
  <img alt="Latest Release" src="https://img.shields.io/github/v/release/elmerjacobo97/spec-flow-skills">
  <img alt="GitHub Stars" src="https://img.shields.io/github/stars/elmerjacobo97/spec-flow-skills?style=social">
  <img alt="Skills" src="https://img.shields.io/badge/skills-8-blue">
</p>

## Quick start

```bash
npx skills@latest add elmerjacobo97/spec-flow-skills
```

## Skills

| Skill | Description | Argument |
| --- | --- | --- |
| `/spec-explore` | Thinks through a fuzzy idea against the real code — options, tradeoffs, no artifacts | `[topic]` |
| `/product-spec` | Defines the business case before the technical spec: evidence, metric, 20% version, kill criterion | `[short topic]` |
| `/spec` | Designs the feature document by asking clarifying questions | — |
| `/spec-edit` | Edits an existing spec in place: impact analysis, one preview confirmation, minimal diff | `<NN-slug> [change]` |
| `/spec-impl` | Validates the spec is approved and implements it group by group, pausing for review and commit after each group | `<NN-slug> [--one-shot]` |
| `/spec-close` | Closes an implemented spec: state gate to `Implementado`, project-memory sync, selective commit, then `CloseMode` (local merge or MR flow). Never pushes | `<NN-slug>` |
| `/spec-status` | Read-only board of all specs: state, progress, dependencies, next action | — |
| `/spec-verify` | Audits spec ↔ code: completeness, correctness, coherence; reports, never fixes | `<NN-slug>` |

---

## Table of contents

- [What spec-driven design is](#what-spec-driven-design-is)
- [The problem it solves](#the-problem-it-solves)
- [The six-step procedure](#the-six-step-procedure)
- [Anatomy of a useful spec](#anatomy-of-a-useful-spec)
- [The optional step before /spec: /product-spec](#the-optional-step-before-spec-product-spec)
- [When to use specs and when not](#when-to-use-specs-and-when-not)
- [Rules almost nobody follows](#rules-almost-nobody-follows)
- [Installation](#installation)
- [Usage](#usage)

---

## What spec-driven design is

Spec-driven design is an approach where **the spec is the main work artifact, not the code**. The code is the consequence.

It sounds obvious. The difference from the classic "document before coding" is that in spec-driven the spec **is not optional or decorative**: it's the contract that guides execution, it's versioned in git, and it's kept alive. If the code diverges from the spec, one of the two is wrong.

Each spec captures the decisions of a single feature. Specs live in `specs/` as `.md` files numbered sequentially, and they form the project's design decision log.

---

## The problem it solves

When you work with an LLM like Claude Code, there's a very concrete phenomenon: if you ask it _"build me an Arkanoid with power-ups and levels"_, **it's going to improvise**. It's going to make 50 implicit design decisions (classes or functions? global or local state? how are entities named?) without you seeing any of them. And each one of those decisions becomes an expensive coupling to revert later.

The problem isn't new — humans improvise too — but with an LLM it's sharper:

1. **Generation speed hides the cost of decisions.** When a human takes two hours to write a module, they have time to think. When Claude does it in 30 seconds, the decisions go invisible.
2. **Every conversation starts from scratch.** Without a spec, in the next session Claude doesn't know what you decided before and is going to improvise again, possibly in the opposite direction.
3. **Context fills up fast.** Without a stable document to refer to, you end up pasting context by hand into every prompt.

The spec solves all three: it makes decisions explicit, it persists across sessions, and it loads once as a reference.

---

## The six-step procedure

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   1. DESCRIBE   │→ │  2. PLAN MODE   │→ │   3. REFINE     │
│   the problem   │  │ Claude proposes │  │ You give        │
│  not the answer │  │ doesn't edit    │  │ decisions       │
└─────────────────┘  └─────────────────┘  └─────────────────┘
        ↑                                          │
        │                                          │
        └──────── 2-3 iterations until converged ──┘
                              │
                              ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│    4. SAVE      │→ │   5. EXECUTE    │→ │   6. REVIEW     │
│ specs/NN-       │  │ Group by group, │  │ Diff + commit   │
│ feature.md      │  │ boxes ticked    │  │ per group       │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

### 1. Describe

You describe the feature to Claude in terms of **the problem**, not the solution. If you dictate the solution, Claude just formats it — you lose its ability to propose structure.

### 2. Plan mode

You activate plan mode (in plan mode Claude can't write files, only read and propose). Claude responds with a structured document: scope, data model, implementation plan, and acceptance criteria.

### 3. Refine

You read the plan with resistance and give **concrete decisions**. "Take X out of scope", "data lives in JSON, not in JS modules", "add a risks section". You iterate 2-3 times.

### 4. Save

When the spec is honed, you save it in `specs/NN-slug.md` with status `Draft`. You leave the chat, **re-read it outside the editor**, and only when you're satisfied do you change the status to `Approved` manually. That change is made by the human, not Claude.

### 5. Execute

You exit plan mode and run `/spec-impl`. Claude implements one group of the plan, ticking each step off inside the spec file, and stops with a mini-summary. You review the diff and commit. Say "continue" and it moves to the next group. It only stops mid-group on a real ambiguity or a step that breaks something. For small specs, add `--one-shot` and it runs every group without pausing.

### 6. Review

Review and commit per group: each chunk is small enough to actually read, and the history stays clean. When the last group is done, Claude verifies the acceptance criteria with real evidence and closes with a summary: what was done, why, what was verified, what is still pending. Interruptions are cheap — the unchecked boxes in the spec tell the next run exactly where to resume.

---

## Anatomy of a useful spec

Not every document does the job. A useful spec has six parts — if any of them is missing, it's probably not enough to guide execution.

### 1. Goal in one sentence

If it doesn't fit in a sentence, the feature is too big. Split it before writing anything else.

### 2. Explicit scope + what's NOT in scope

The "out of scope" is as important as the "in scope". Without it, the boundaries are blurry and scope creep appears during implementation. Capture the things that were mentioned but decided to postpone.

### 3. Data model

Concrete structures and names. If you say "the levels module", say `src/levels.js`. If you say "a key", give the exact string. This section is the one most cited later in other specs and skills.

### 4. Ordered implementation plan

Grouped checkbox steps (`### Group N` + `- [ ] N.M`). **Each step must leave the system in a working state.** If a step requires more than 30-50 lines of code, split it. The last step is not "test everything" — that's the acceptance criteria. `/spec-impl` ticks each step off as it implements it, and stops at the end of each group so you can review and commit before the next one.

### 5. Acceptance criteria

A verifiable boolean checklist. Each item can be answered yes or no.

- ❌ "Works well" — not verifiable
- ❌ "Good UX" — subjective
- ❌ "No bugs" — not operational
- ✅ "Pressing Esc pauses the game and shows the menu" — verifiable

### 6. Decisions made and discarded

What you considered and why you chose what you chose. **This is gold three months from now** when someone asks _"why does persistence use a versioned key?"_. The answer lives there.

Each decision ideally has a short reason. Decisions without a reason are the first ones questioned later.

---

## The optional step before /spec: /product-spec

`/spec` designs *what* gets built once someone has decided it's worth building. It doesn't ask *whether* it's worth building or *how you'll know* it worked — that question belongs to `/product-spec`, a separate skill that runs one step earlier.

```bash
/product-spec push-notifications   # optional — evidence, metric, 20% version, kill criterion
        ↓
/spec push-notifications           # reads the brief automatically if one exists
        ↓
/spec-impl NN-push-notifications
```

`/product-spec` asks five things in a single pass — who asked for this, what they do today without it, what metric it should move (with a real baseline, not an adjective), the smallest version that tests the hypothesis, and the number that would make you kill it — then saves `specs/NN-slug.brief.md`. If there's no baseline to measure against, it doesn't invent one: it outputs a brief that says what to instrument first, and stops there.

**Use it when** the feature's value or audience is not already obvious. **Skip it** for bug fixes, security patches, technical debt, or anything a client already requested with hard evidence — go straight to `/spec` in those cases.

See [`skills/product/product-spec/SKILL.md`](./skills/product/product-spec/SKILL.md) for the full flow.

---

## When to use specs and when not

This architecture has a cost. Don't apply it to everything.

### YES — write a spec when:

- The task will touch **more than two files**.
- There are **decisions expensive to revert** (data schemas, formats, APIs).
- The feature will take **more than one session** of Claude Code.
- There's a **contract that other artifacts will reuse** (another spec, a skill, a hook).
- It's something you'll **forget about in a week**.

### NO — use a direct prompt when:

- It's a **point bug fix**.
- It's a **mechanical refactor** (renames, file moves).
- It's an **exploratory experiment** where the goal is to discover the decision, not execute it.
- The task **fits in a prompt** and is understood at first read.
- It's a **one-off task** that won't be repeated.

### Mental rule

> **If you're tempted to open plan mode, you probably need it.** > **If planning the feature bores you, you probably don't.**

Common sense beats the rule — but common sense is trained by the two columns above.

---

## Rules almost nobody follows

Four usage patterns that distinguish the method working well from the method as decorative bureaucracy:

### 1. In the description phase, describe the problem, not the solution

❌ _"Add an array of levels loaded from JSON, a `loadLevel()` function, and persistence with versioned localStorage."_

That's already a spec poorly written by you. Claude is just going to format it.

✅ _"I want the game to stop being single-screen. The next feature is: progression through levels with increasing difficulty, and persistence of high scores across sessions."_

That second version leaves room for Claude to **decide** and you to **review**. That's the nature of the flow.

### 2. In the refine phase, give concrete decisions, not suggestions

Plan mode is where **you direct**. "Take X out", "the format is JSON", "add risks". If you say "I think maybe it would be good to...", Claude is going to leave it as is.

### 3. During execution, review by group — not by step, not by whole spec

- **Too slow:** pausing after every step turns you into a babysitter for a run that could have been one sitting.
- **Too fast:** a single pass over a big spec ends in one giant diff, and a mistake in group 2 is buried under groups 3 and 4.
- **The balance:** the groups in the spec are the unit of review. Claude implements a group, ticks its steps off inside the spec file, and stops. You read the diff, commit, and say "continue". Each chunk is small enough to actually review, and the history stays clean. An interrupted run resumes from the first unchecked box — even in a new session.

### 4. If mid-execution you want to change something, you go back to step 2 — never improvise

Mid-implementation something occurs to you. The right move is: stop, go back to plan mode, update the spec, exit, continue. **Don't improvise on the code.**

That separation is what prevents silent scope creep.

---

## Installation

### Option 1 — skills.sh (recommended, Claude Code)

```bash
npx skills@latest add elmerjacobo97/spec-flow-skills
```

To uninstall:

```bash
npx skills@latest remove elmerjacobo97/spec-flow-skills
```


### Option 2 — Other agents (Cursor, Codex, Antigravity, opencode)

```bash
git clone https://github.com/elmerjacobo97/spec-flow-skills ~/.spec-flow-skills
cd ~/your-project
~/.spec-flow-skills/scripts/install-to-agent.sh <agent>
```

`<agent>` can be `claude`, `cursor`, `codex`, `antigravity`, or `opencode`.

| Agent         | What gets written                                                                     |
| ------------- | ------------------------------------------------------------------------------------- |
| `claude`      | Symlinks each skill into `.claude/skills/` (project-scoped)                           |
| `cursor`      | Generates `.cursor/rules/<name>.mdc` files. Invoke with `@spec`, `@spec-impl`, etc.   |
| `codex`       | Adds a `## Skills` block to `AGENTS.md` and copies skill bodies into `.codex/skills/` |
| `antigravity` | Copies skill bodies into `.antigravity/skills/`                                       |
| `opencode`    | Generates `.opencode/commands/<name>.md` files. Invoke with `/spec`, `/spec-impl`, etc. |

> Cursor and opencode don't support Claude Code's `argument-hint`, `allowed-tools`, or `disable-model-invocation` frontmatter. The installer drops those fields and keeps the body — the workflow is the same, only the trigger and any tool-permission gating change.

### Option 3 — Manual

```bash
# Personal (all your projects)
mkdir -p ~/.claude/skills
cp -r skills/product/product-spec ~/.claude/skills/
cp -r skills/engineering/spec ~/.claude/skills/
cp -r skills/engineering/spec-impl ~/.claude/skills/

# Or per-project (versioned in git)
mkdir -p .claude/skills
cp -r skills/product/product-spec .claude/skills/
cp -r skills/engineering/spec .claude/skills/
cp -r skills/engineering/spec-impl .claude/skills/
```

For the method to work, you also need to create the `specs/` folder at the project root:

```bash
mkdir specs
```

Optionally, add a `specs/README.md` documenting the convention (see the example in this repo).

---

## Usage

### Full feature cycle

```bash
# 0. (Optional) Not sure how to approach it yet? Explore first — no files created
/spec-explore levels-and-highscores

# 0b. (Optional) Define the business case — evidence, metric, kill criterion
/product-spec levels-and-highscores

# Only worth doing if the feature's value or audience isn't already obvious.
# Saves specs/03-levels-and-highscores.brief.md with status: Draft.

# 1. Design the spec with clarifying questions
/spec levels-and-highscores

# Claude reads the project-memory file (CLAUDE.md, AGENTS.md, GEMINI.md, or README.md) and existing specs/
# (including the .brief.md from step 0, if it exists), asks questions
# in blocks, develops the spec section by section,
# and finally saves it as specs/03-levels-and-highscores.md
# with status: Draft.

# 2. Re-read the spec outside the chat and approve it manually
# (open the file in the editor, change Status: Draft → Approved)

# 3. Implement the approved spec
/spec-impl 03-levels-and-highscores

# Claude validates the status is Approved and resolves the branch:
# it reuses the active ticket work branch when there is one,
# otherwise creates spec-03-levels-and-highscores. It implements
# one group, ticks it off in the spec, and stops for your review
# and commit. Say "continue" for the next group. Add --one-shot
# to skip the pauses on small specs.

# 4. (Optional) Audit that the code matches the spec
/spec-verify 03-levels-and-highscores

# 5. Close it
/spec-close 03-levels-and-highscores

# Marks the spec Implementado, syncs CLAUDE.md/AGENTS.md when the change
# added dependencies, modules, or commands, and commits what is pending
# (asking first about changes it did not make). CloseMode controls the rest:
# local → merge into main (no push) + delete the branch; pr → keep the
# branch for the MR, then run /spec-close again after it is merged.

# Anytime: /spec-status shows every spec — state, progress, dependencies.
```

### What each skill does

#### `/spec-explore [topic]`

A no-stakes thinking partner for ideas that are not ready for a spec yet:

1. **Investigate** — reads the relevant code and cites what exists today.
2. **Options** — presents 2-3 concrete approaches with tradeoffs and one recommendation.
3. **Iterate** — digs deeper across turns while the idea sharpens.
4. **Handoff** — when the idea is clear enough to boundary it, it points to `/product-spec` (value still unclear) or `/spec` (ready). It creates nothing.

#### `/product-spec [short-topic]`

Defines the business case, in a single pass:

1. **Evidence** — who asked, how many, how strong is the signal.
2. **Metric** — baseline, target, and check-in date. No adjectives accepted.
3. **20% version** — the smallest cut that still tests the hypothesis.
4. **Kill criterion** — what result reverts this instead of iterating on it.
5. **Save** — `specs/NN-slug.brief.md` with status `Draft`. If there's no baseline to measure, the brief recommends instrumentation instead of scoping a feature.

#### `/spec [short-topic]`

Designs the feature document. Goes through four phases:

1. **Context** — reads the project-memory file (`CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, or `README.md`, whichever exists first) and previous specs.
2. **Clarification** — asks questions in blocks of 3-5 until the feature is clearly defined.
3. **Section by section development** — generates and confirms each spec section before moving on.
4. **Save** — writes the file in `specs/NN-slug.md` with status `Draft`.

#### `/spec-edit <NN-name> [what-to-change]`

Revises an existing spec in place — no regeneration, no new file. Goes through this flow:

1. **Identify** — locates the spec file (same flexible matching as `/spec-impl`).
2. **Read and index** — sections by meaning, current status, which steps are already checked.
3. **Understand the change** — if your request is already clear it skips straight ahead; otherwise it asks one block of 2–4 concrete questions. It also applies the update-vs-new-spec rule: same work refined → edit; intent changed or scope exploded → it recommends a new `/spec` instead.
4. **Impact analysis and preview** — shows the exact edits (`old → new`) per section, plus the cascades (scope change → plan and criteria) and any conflict, like an edit touching a step already marked `- [x]`.
5. **One confirmation → apply** — minimal diffs, spec language and format preserved, header date updated. The status line is never touched.
6. **Coherence check and summary** — reports anything left loose and warns when the spec needs human re-approval (it was `Approved`) or may now diverge from code (it was `Implemented`).

> **Status stays human-owned:** `/spec-edit` never edits the `Status:` / `Estado:` line. If the spec was `Approved` before the edit, re-approve it manually once you are happy — `/spec-impl` only works with an approved spec.

#### `/spec-impl <NN-name>`

Implements an approved spec. Goes through four phases:

1. **Identify** — locates the spec file.
2. **Validate** — verifies the status is `Approved`. If not, it stops.
3. **Resolve branch** — reuses the active ticket work branch (`feat/...`, `fix/...`) when one is present; otherwise creates and switches to `spec-NN-slug`.
4. **Implement** — group by group. Ticks each step `- [x]` inside the spec as it completes a group, then stops with a mini-summary so you can review and commit. Say "continue" for the next group. Verifies the acceptance criteria with real evidence at the end and closes with a summary. It only stops mid-group on an ambiguity or a step that breaks the project.

It never commits, merges, or closes the spec: the final summary hands off to `/spec-verify` and `/spec-close`.

> **Groups live in the spec:** the implementation plan is a grouped checkbox list (`### Group N — …` + `- [ ] N.M`). If an older spec is flat, `/spec-impl` proposes a grouping, writes it into the spec after your confirmation, and then implements group by group. An interrupted run resumes from the first unchecked box, and at the end only the criteria it can prove with evidence are marked — the rest wait for you.

> **`--one-shot`:** `/spec-impl 03-levels-and-highscores --one-shot` runs every group without the between-group pauses. Useful when the spec is small enough that a single review at the end suffices.

> **Branch control:** Phase 3 first checks the current branch. If it is neither the default branch nor `spec-NN-slug` — the case when a ticket tool (e.g. Forge) already created the work branch — Phase 3 keeps that branch and skips `AutoCreateBranch` entirely: one work branch per task, created once. Otherwise it reads the `AutoCreateBranch` flag from `specs/.spec-config.yml`. It defaults to `true` (creates the branch automatically). Set it to `false` to make `/spec-impl` ask `[y/N]` before creating any branch — useful if branch naming is part of your own Git workflow.
>
> ```yaml
> # specs/.spec-config.yml
> AutoCreateBranch: false
> ```

#### `/spec-close <NN-name>`

Closes an implemented spec. Runs only when invoked explicitly — it is the only skill allowed to write `Implementado`:

1. **Identify** — locates the spec (same flexible matching as the other skills; without an argument it infers from the active `spec-NN-slug` branch or asks).
2. **State gate** — `Aprobado` → `Implementado` (in the file's own label and language); already `Implementado` → no change; anything else → refuses.
3. **Separate changes** — splits `git status --short` into the agent's own changes and foreign ones (a dependency you added, a manual edit). Foreign files are never included or reverted without your explicit choice per file: include / review the diff / revert (tracked only) / leave out.
4. **Sync project memory** — checks the first of `CLAUDE.md` → `AGENTS.md` → `GEMINI.md` → `README.md` and proposes minimal `old → new` edits for facts this spec introduced (new dependencies, modules, commands), or says explicitly that no update is needed. Never a rewrite, never a changelog.
5. **One confirmation** with the full preview (including the memory edits), then a selective commit `feat(spec-NN-slug): <objective>` — never `git add -A`.
6. **Follow `CloseMode`** from `specs/.spec-config.yml`:
   - **`local`** — merges the work branch into the default branch and deletes the local branch. Nothing is pushed.
   - **`pr`** — commits and stops; the branch stays alive for the MR. You push and open it, and when it is merged you run `/spec-close` again: it verifies the merge landed in the default branch, runs `git pull`, and safe-deletes the local branch (`git branch -d`, never `-D`). The remote branch is yours to delete.

> **Close control:** `CloseMode` lives in `specs/.spec-config.yml`. When the flag is absent, `/spec-close` asks `[l] local merge + delete / [p] keep branch for the MR` in its confirmation — it never assumes. In both modes `git push` stays with you.
>
> ```yaml
> # specs/.spec-config.yml
> AutoCreateBranch: true
> CloseMode: pr
> ```

#### `/spec-status`

Read-only board of everything in `specs/`:

1. **Inventory** — specs, product briefs, and config, separated.
2. **Read** — state, dependencies, and plan/criteria checkbox counts per spec.
3. **Board** — one table sorted by number, with flags: `Implementado` with unchecked criteria, a dependency that is not `Implementado`, a header with a broken state.
4. **Next actions** — one suggested action per pending spec. It never executes them.

#### `/spec-verify <NN-name>`

Independent audit of an implementation against its spec — re-runnable at any time, including months later to catch drift:

1. **Index** — plan steps, criteria, decisions, and the data model.
2. **Evidence** — searches the code for each claim (a checkbox is a claim, not evidence) and runs tests/build/lint when the project defines them.
3. **Report** — completeness, correctness, coherence; each finding tagged CRITICAL / WARNING / SUGGESTION with a `file:line`.
4. **Verdict** — plus the mismatch direction: spec stale → `/spec-edit`; code drifted → fix the code. It is read-only: it never fixes anything itself. When the audit is clean, the next step is `/spec-close NN-slug`.

### Spec states

| State         | Meaning                                                                    |
| ------------- | -------------------------------------------------------------------------- |
| `Draft`       | The `/spec` skill generated it but the human hasn't re-read it.            |
| `In review`   | The human is reviewing or iterating with Claude.                           |
| `Approved`    | The human read and authorized it. `/spec-impl` only works with this state. |
| `Implemented` | The code exists and passes the acceptance criteria. `/spec-close` writes it when you close the spec. |
| `Obsolete`    | Replaced by another spec. Not deleted — referenced.                        |

**Changing the status to `Approved` is a deliberate human act.** It's the only signature on the contract — Claude can't approve its own work.

> The only state write the agent can make is `Aprobado` → `Implementado`, inside `/spec-close` — and only because you invoked it and confirmed the preview.

> Status labels are language-agnostic. `/spec-impl` only requires the status to mean **Approved** — `Approved`, `Aprobado`, or the equivalent in any language all work. Same goes for the other states. Pick the labels your team prefers and stay consistent.

---

## Why the two skills work as a pair

```
┌───────────────────────────────────────────────────────────┐
│                                                           │
│   /spec     Claude asks and designs                       │
│             ↓                                             │
│             specs/NN-slug.md  (Status: Draft)             │
│                                                           │
│   ──────── human re-reads and approves ────────           │
│             ↓                                             │
│             specs/NN-slug.md  (Status: Approved)          │
│                                                           │
│   /spec-impl  Claude validates and implements             │
│             ↓                                             │
│             branch spec-NN-slug (or ticket branch) + code │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

The gap between the two skills — re-reading and changing the status by hand — is deliberate. It's the only moment where **only you can do something**. Without that gap, the method degrades to "Claude writes pretty documentation and then writes whatever code occurs to it anyway".

If the spec needs changes before or after approval, `/spec-edit` revises it in place — and the human still re-approves it before implementation continues.

---

## Releases

This project uses [release-please](https://github.com/googleapis/release-please) for automated releases. Commit messages must follow [Conventional Commits](https://www.conventionalcommits.org/):

| Prefix | Effect |
| --- | --- |
| `feat:` | Bumps minor version |
| `fix:` | Bumps patch version |
| `feat!:` / `fix!:` | Bumps major version |
| `docs:`, `chore:`, `refactor:` | No version bump |

---

## License

MIT

---

_If you find a way to improve the method or the skills, open an issue or a PR. The most valuable part of a personal skill is that it evolves with use._
