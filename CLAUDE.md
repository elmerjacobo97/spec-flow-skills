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

Use `allowed-tools` in frontmatter to restrict what Bash commands a skill may run (see `skills/engineering/spec-impl/SKILL.md` for an example that limits to read-only git/fs).

Use `!`command`` shell snippets inside `SKILL.md` to inject live repo state at skill-load time (e.g., current branch, available specs). Embed them at the top of the skill body before the instructions.

Companion files (e.g., `template.md`) sit next to `SKILL.md` and are referenced by relative path within the skill body.

## The spec workflow

This repo encodes a three-skill chain: `/product-spec` (optional) → `/spec` → `/spec-impl`.

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

**Estado** after saving is always `Borrador`. State transitions are human-driven — Claude must never change `**Estado:**` automatically.

Valid states: `Borrador` → `En revisión` → `Aprobado` → `Implementado` · `Obsoleto`

### `/spec-impl`

Accepts `<NN-slug>` as argument. Phases:

1. Locate `specs/<NN-slug>.md`.
2. Read `**Estado:**` — abort with a standard error message if it is not exactly `Aprobado`.
3. Resolve the branch: if the current branch is neither the default branch nor `spec-NN-slug`, treat it as an existing work branch (the ticket flow created it) and stay on it. Otherwise apply the `AutoCreateBranch` logic.
4. Implement the spec's plan step-by-step, pausing after each step for diff review.

Branch creation in step 3 is gated by the `AutoCreateBranch` flag, read at skill-load time from `specs/.spec-config.yml` via a `!`cat`` snippet. Default (file or value absent) is `true` → branch is created automatically when starting from the default branch. An explicit `false` makes the skill ask `[y/N]` before creating the branch; on decline it implements on the current branch. The existing-work-branch rule takes precedence over the flag: in the ticket flow no branch is created and no question is asked. There is still no runtime config infra — the flag is just a value injected into the prompt and interpreted by the model.

At completion, remind the user to verify acceptance criteria and mark the spec `Implementado` manually before merging.

### Tickets bridge (optional)

A spec can be split into tracker tickets — one per implementation-plan section — before implementation. The ticket description carries `Spec: specs/NN-slug.md`, and the board tracks state and time while the spec remains the source of truth for scope. Working a ticket runs on a ticket work branch (`feat/<slug>`, `fix/<slug>`, …); when `/spec-impl` finds that branch active, Phase 3 reuses it instead of creating `spec-NN-slug`. The order is: approve the spec → create the tickets → work each ticket with the ticket skill → merge per ticket. This keeps the board honest without duplicating the plan; for projects using OpenSpec the same bridge works with `Change: openspec/changes/<name>` and one ticket per `tasks.md` section.

## Distribution

The repo is consumed by users in two ways:

1. **skills.sh** (`npx skills@latest add elmerjacobo97/spec-flow-skills`) — auto-discovers public GitHub repos with `skills/**/SKILL.md`. Just push to GitHub.
2. **Multi-agent installer** (`scripts/install-to-agent.sh <agent>`) — translates skills for Cursor (`.cursor/rules/*.mdc`), Codex (`AGENTS.md` block + `.codex/skills/`), Antigravity (`.antigravity/skills/`), and opencode (`.opencode/commands/*.md`). Run from the _target_ repo, not this one.

`scripts/link-skills.sh` symlinks every skill into `~/.claude/skills` for local development.

## No build or test commands

There is no package manager, build step, or test suite. All skills are plain Markdown files.
