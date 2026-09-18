# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This repo is a workspace for authoring Claude Code slash-command skills, specifically for a spec-driven development workflow. Skills are written in **Spanish**.

`reference/` is a vendored copy of `mattpocock/skills` kept as a structural/style reference — do not edit it during normal work. Its own `reference/CLAUDE.md` governs that subtree.

## Skill authoring conventions

Each skill lives under `skills/<bucket>/<name>/` (e.g. `skills/engineering/spec/`) as a directory containing at minimum a `SKILL.md`. Buckets group skills by domain (`engineering/` for skills that execute code or git, `product/` for skills that define business intent ahead of a technical spec). The YAML frontmatter of `SKILL.md` must declare:

```yaml
---
name: skill-name
description: One-line description (in Spanish)
disable-model-invocation: true
argument-hint: [hint]
---
```

Use `allowed-tools` in frontmatter to restrict what Bash commands a skill may run (see `skills/engineering/spec-verify/SKILL.md` for an example that limits to read-only git/fs).

Use `!`command`` shell snippets inside `SKILL.md` to inject live repo state at skill-load time (e.g., current branch, available specs). Embed them at the top of the skill body before the instructions.

Companion files (e.g., `template.md`) sit next to `SKILL.md` and are referenced by relative path within the skill body.

## The spec workflow

This repo encodes a skill chain: `/product-spec` (optional) → `/spec` → `/spec-impl`, plus `/spec-edit` to revise an existing spec in place at any point. Three auxiliary skills round it out: `/spec-explore` (optional pre-spec thinking pass, creates nothing), `/spec-status` (read-only board of `specs/`), and `/spec-verify` (read-only audit of spec ↔ code, re-runnable).

### `/product-spec`

Optional step before `/spec`. Single pass, not phased: asks for evidence, current workaround, a metric with baseline/target/check-in date, a 20% version, and a kill criterion. Saves `specs/NN-slug.brief.md` with status `Draft`.

If no baseline exists for the metric, the skill does not fabricate one — it outputs a brief whose only content is what to instrument (event names/properties) and stops there. This is a correct terminal output, not a failure state.

`/spec` Phase 1 checks for a matching `specs/NN-slug.brief.md` and, if found, reuses its exact `NN-slug` for the technical spec rather than assigning a new number — the brief and the spec are two files sharing one number, not two entries in the sequence.

This skill pair (below) is unchanged by the addition:

### `/spec`

Guides the user through 4 phases: read project context → clarify with questions (blocks of 3–5) → draft each spec section one at a time with user confirmation → save to `specs/NN-slug.md`.

On save, it also seeds `specs/.spec-config.yml` (default `AutoCreateBranch: true`) **only if the file is missing** — an existing config is never overwritten. This is how the `AutoCreateBranch` flag that `/spec-impl` reads gets created without the user knowing the file exists.

Output file format: `specs/NN-slug.md` with a header block:

```
> **Estado:** Borrador · **Depende de:** ... · **Fecha:** YYYY-MM-DD
> **Objetivo:** One sentence.
```

**Estado** after saving is always `Borrador`. State transitions are human-driven — Claude must never change `**Estado:**` automatically. The single exception is the close flow in `/spec-impl` Phase 5: it runs only on an explicit close request ("cierra la spec", "close the spec", any language) plus one confirmation preview, and it is the only place allowed to write `Implementado`.

Valid states: `Borrador` → `En revisión` → `Aprobado` → `Implementado` · `Obsoleto`

### `/spec-edit`

Accepts `<NN-slug>` plus an optional free-text change request (e.g. `/spec-edit 03-levels "remove the combo system"`). Locates the spec with the same flexible matching as `/spec-impl`, reads and indexes it (sections by meaning, current state, which steps are checked), then:

- If the request is vague: asks one block of 2–4 concrete questions.
- Applies the update-vs-new-spec rule: same work refined → edit; intent fundamentally changed or scope exploded → recommends a new `/spec` instead.
- Impact analysis and preview: exact edits (`old → new`) per section, cascades (scope change → plan and criteria, reverted decision → decisions section), and conflicts (an edit touching a step already marked `- [x]`).
- One confirmation → applies minimal diffs, preserving language, heading and checkbox format; updates the header date.

Hard rules: edits only `specs/NN-slug.md` (never the `.brief.md`), never writes code, never regenerates the document, never unchecks `- [x]` without explicit permission, and **never edits `**Estado:**`**. If the spec was `Aprobado`, it tells the human to re-approve; if it was `Implementado`, it warns about code drift.

### `/spec-impl`

Accepts `<NN-slug>` as argument. Phases:

1. Locate `specs/<NN-slug>.md`.
2. Read `**Estado:**` — abort with a standard error message if it is not exactly `Aprobado`.
3. Resolve the branch: if the current branch is neither the default branch nor `spec-NN-slug`, treat it as an existing work branch (the ticket flow created it) and stay on it. Otherwise apply the `AutoCreateBranch` logic.
4. Implement the spec's plan group by group: tick each step off (`- [ ]` → `- [x]`) inside the spec file as it completes a group, then stop with a mini-summary so the human can review and commit before saying "continue". At the end it verifies the acceptance criteria with real evidence and ticks only the ones it proved. `--one-shot` runs every group without the between-group pauses. It stops early only on a real ambiguity or a step that breaks the project.
5. On request only — the human asks to close the spec ("cierra la spec", "close the spec", any language) — run the close flow: state gate (`Aprobado` → `Implementado`; already `Implementado` → no change; anything else → refuse), split the pending changes into the agent's own vs foreign and ask before including any foreign one, commit `feat(spec-NN-slug): <objetivo>`, then follow `CloseMode` from `specs/.spec-config.yml`: `local` merges into the default branch and deletes the local branch; `pr` stops after the commit and keeps the branch for the MR (missing flag → asks). A later "cleanup the spec branch" request runs the post-merge phase: verify the merge landed in the default branch, `git pull`, safe-delete the local branch. It never pushes, in any mode.

Branch creation in step 3 is gated by the `AutoCreateBranch` flag, read at skill-load time from `specs/.spec-config.yml` via a `!`cat`` snippet. Default (file or value absent) is `true` → branch is created automatically when starting from the default branch. An explicit `false` makes the skill ask `[y/N]` before creating the branch; on decline it implements on the current branch. The existing-work-branch rule takes precedence over the flag: in the ticket flow no branch is created and no question is asked. There is still no runtime config infra — the flag is just a value injected into the prompt and interpreted by the model.

At completion it prints a chat summary (steps, files, why, verified/pending criteria) and suggests `/spec-verify` followed by the close phrase. On a close request in any language it handles the commit and then follows `CloseMode` — local merge + branch deletion, or commit only for the MR with a later pull + cleanup — and is the only skill allowed to write `Implementado`; it never pushes: publishing stays manual.

Changes to an approved spec go through `/spec-edit` and require the human to re-approve before the next `/spec-impl` run.

### Auxiliary skills

`/spec-explore [topic]` — optional thinking pass before a spec exists. Reads the code, presents 2–3 concrete options with tradeoffs, creates nothing, and ends by recommending `/product-spec` or `/spec`.

`/spec-status` — read-only board of `specs/`: state, dependencies, plan/criteria progress per spec, flags for stale or blocked specs, and one suggested next action each. It never executes actions.

`/spec-verify <NN-slug>` — read-only audit of an implementation against its spec, re-runnable at any time (before merging, after refactors, or later for drift). Reports completeness/correctness/coherence with CRITICAL/WARNING/SUGGESTION severities and a mismatch-direction verdict: spec stale → `/spec-edit`; code drifted → fix the code. Never edits code, checkboxes, or `Estado`.

### Tickets bridge (optional)

A spec can be split into tracker tickets — one per implementation-plan group — before implementation. The ticket description carries `Spec: specs/NN-slug.md`, and the board tracks state and time while the spec remains the source of truth for scope. Working a ticket runs on a ticket work branch (`feat/<slug>`, `fix/<slug>`, …); when `/spec-impl` finds that branch active, Phase 3 reuses it instead of creating `spec-NN-slug`. The order is: approve the spec → create the tickets → work each ticket with the ticket skill → merge per ticket. This keeps the board honest without duplicating the plan; for projects using OpenSpec the same bridge works with `Change: openspec/changes/<name>` and one ticket per `tasks.md` section.

## Distribution

The repo is consumed by users in two ways:

1. **skills.sh** (`npx skills@latest add elmerjacobo97/spec-flow-skills`) — auto-discovers public GitHub repos with `skills/**/SKILL.md`. Just push to GitHub.
2. **Multi-agent installer** (`scripts/install-to-agent.sh <agent>`) — translates skills for Cursor (`.cursor/rules/*.mdc`), Codex (`AGENTS.md` block + `.codex/skills/`), Antigravity (`.antigravity/skills/`), and opencode (`.opencode/commands/*.md`). Run from the _target_ repo, not this one.

`scripts/link-skills.sh` symlinks every skill into `~/.claude/skills` for local development.

## No build or test commands

There is no package manager, build step, or test suite. All skills are plain Markdown files.
