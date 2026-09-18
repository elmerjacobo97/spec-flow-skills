---
name: spec-close
description: Closes an approved or already-implemented spec — state gate to Implementado, a minimal sync of the project memory (CLAUDE.md/AGENTS.md) when the change added dependencies, modules, or commands, a commit of the pending changes attributable to the spec, then CloseMode follows — local merge plus branch deletion, or commit only for the MR with a later post-merge cleanup. It never pushes.
disable-model-invocation: true
argument-hint: <NN-spec-name>
allowed-tools: Bash(git status:*), Bash(git branch:*), Bash(git checkout:*), Bash(git symbolic-ref:*), Bash(git add:*), Bash(git commit:*), Bash(git merge:*), Bash(git pull:*), Bash(git log:*), Bash(git diff:*), Bash(cat:*), Bash(ls:*)
---

# /spec-close — Close an implemented spec

Closes the loop after implementation and audit: marks the spec `Implementado`, commits what is still pending, and follows `CloseMode` — a local merge with branch deletion, or a commit that keeps the branch alive for the MR. Publishing is never part of this skill.

## Session context

Working tree:
!`git status --short`

Current branch:
!`git branch --show-current`

Default branch:
!`git symbolic-ref --short refs/remotes/origin/HEAD 2>/dev/null || echo "unknown"`

Specs available in this folder:
!`ls specs/ 2>/dev/null || echo "The specs/ folder does not exist"`

Close-mode config:
!`cat specs/.spec-config.yml 2>/dev/null || echo "No config file: CloseMode missing (ask in the confirmation)"`

---

## Instructions

### Phase 1 — Identify the spec

The received argument is: `$ARGUMENTS`

- If it has a value, find the file with the same flexible matching the other skills use: full name (`03-levels-and-highscores`), only the number (`03`), or only the slug (`levels-and-highscores`). If you do not find it, show the available specs and ask for the correct name.
- If it is empty and the current branch is `spec-NN-slug`, propose the matching `specs/NN-slug.md` and ask for one confirmation before continuing.
- If it is empty and there is no obvious match, list the specs available above and ask which one to close. Stop and wait.

### Phase 2 — State gate

Read the spec's state line, matching by meaning (any language):

| State found                                                   | Action                                                                                                                          |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Already means "Implemented" (`Implementado`, `Implemented`, …) | Do not rewrite it. Note "already Implemented" and continue.                                                                     |
| Means "Approved" (`Aprobado`, `Approved`, …)                   | Change it to the Implemented equivalent **using the file's own label and language** (`**Estado:** Implementado` / `**Status:** Implemented`) as part of the close commit. |
| Anything else (Draft, In review, Obsolete, unrecognized)      | **Refuse to close.** Show the state found; only approved or already-implemented specs can be closed. Do not commit, merge, or delete. |

### Phase 3 — Resolve the close mode

Read `CloseMode` from the config shown in the session context:

- `local` → commit, merge into the default branch, delete the local branch. The personal-project flow.
- `pr` (accept `mr`, `MR`, `PR`) → commit and stop: the branch stays alive for a pull/merge request; no merge, no deletion.
- Missing or unrecognized → the mode is a **decision**, not a confirmation: ask it as the single question of Phase 6, with the interactive question tool when available. Never assume a mode.

### Phase 4 — Classify the pending changes

Run `git status --short`, then read the diff of every pending file and sort them:

1. **Files the agent touched in this conversation** — the implementation or fixes that happened here. They enter the commit.
2. **Files of the spec being closed** — `specs/NN-slug.md` (the file read in Phase 2) and its companion `specs/NN-slug.brief.md` when it exists. They always enter the commit without asking: the spec file carries the state change from Phase 2 and the brief shares its number. Files of another spec do not fall in this bucket.
3. **Attributable changes** — pending files whose path or diff matches this spec's plan, its decisions, or their fallout (migration churn included). They enter the commit; report only their count, never a per-file question.
4. **Unattributable changes** — a dependency the human added, a manual edit, anything the diff does not connect to this spec. They stay **out of the commit** by default and appear under `No incluidos:` in the confirmation. Never revert or delete one.

If you have no reliable run list (a different session, context compaction, manual work), bucket 1 is empty: attribute each pending file by reviewing its diff against the spec. A file whose diff does not clearly match is unattributable — never guess it in. If the human wants to approve file by file, they will say so; only then ask per file.

### Phase 5 — Sync project memory

The close is the last moment the implementation facts are fresh. Check whether the project-memory file still describes the project accurately, and propose the minimum needed.

1. **Find the memory file**, first hit in this order: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`. If none exists, report `Memory: no memory file found — skipped` and move on.
2. **Extract the facts this spec introduced** from the spec itself (`## Data model`, `## Decisions`) and from the pending diff: new dependencies, new modules or directories, new commands or scripts, changed conventions. Only facts introduced by **this** spec — no general cleanup, no rewriting for style.
3. **Evaluate always** and be explicit about the outcome:
   - Nothing relevant: `Memory: no update needed — this change adds no dependencies, modules, or commands.` Continue with no edit.
   - Relevant facts: prepare **minimal edits** in the memory file's own language and format, and show them in the confirmation as `old → new` pairs, one line per edit. Never regenerate the file, never turn it into a changelog.
4. **Section placement:** use an existing section that fits. If no section fits, ask before creating one; never invent structure without approval.
5. Approved memory edits become part of the close commit (Phase 7), staged with the other approved paths.

### Phase 6 — One confirmation, one question

No phase narration, no progress prelude: the first visible output is the block below, written in the user's language and kept this compact. Resolve the memory edits first, then present:

```
Close specs/NN-slug.md — <current state> → <new state>

Branch:  <current branch> → <default branch>
Commit:  feat(spec-NN-slug): <objective from the spec>
Files:   <N> pending — spec + code attributable to this spec
Memory:  <CLAUDE.md — N edits>   (or: no changes / no memory file found)
  ~ <edit 1, one line>
  ~ <edit 2>
Not included: <path, path>       (or: none)
```

Then the single question. Use the interactive question tool when available — Claude Code `AskUserQuestion`, opencode `question` — so the human selects instead of typing; the plain-chat fallback keeps the same A/B/C letters.

- `CloseMode` missing or unrecognized: `A) local — commit, merge into <default branch>, delete the branch (no push)`, `B) pr — commit only, keep the branch for the MR`, `C) cancel`.
- `CloseMode` set: `A)` the configured mode, labeled `(Recommended)`, `B)` the other mode, `C) cancel`.

The answer is the approval: A or B runs Phases 7–8; C or anything else stops. No `[y/N]`, no `y l` combinations, no second confirmation.

Unattributable files stay out and are listed on the `No incluidos:` line; the human can ask for `diff <path>` or to include one in chat before answering.

### Phase 7 — Commit

Stage the paths selected in Phase 4 (buckets 1–3) plus the approved memory edits with `git add <path> …` — never `git add -A`. Commit with the proposed message (`feat(spec-NN-slug): <objective>`), including the state change when it applies. If the tree is clean, skip the commit and say so.

### Phase 8 — Finish according to the mode

- **`local`:** if the current branch **is** the default branch, there is nothing to merge — skip both merge and deletion. Otherwise `git checkout <default branch>` then `git merge <branch>` (fast-forward when possible). On any conflict: stop, report the conflicting files, leave the repository on the default branch, and never resolve the conflict or force anything by yourself. Then delete the local branch: `git branch -d <branch>` (safe delete; it refuses unmerged branches). If the branch is a ticket branch (`feat/…`, `fix/…` — anything that is not `spec-NN-slug`), ask before deleting it. If `-d` fails, report why and stop. **Never use `-D`.**
- **`pr`:** do not merge, do not delete. Close with the exact next commands for the human:
  ```
  ✅ Committed on <branch>. Branch kept for the MR.

  Next (yours — I never push):
    git push -u origin <branch>
    open the MR → <default branch>

  When the MR is merged, run /spec-close again (or say "cleanup the spec branch")
  and I will: git checkout <default branch> → git pull → git branch -d <branch>.
  ```

State plainly which of the two happened: in `local` mode, the branch was merged locally and deleted and **nothing was pushed**; in `pr` mode, only the commit happened and the branch is waiting for the MR. In both modes, publishing is the human's (`git push`), and the spec is now marked Implemented when that change was applied.

### Post-merge cleanup (pr mode)

If the spec is already implemented and its branch still exists after the MR was merged, the same command performs the cleanup instead of a new close:

1. **Verify the merge landed in the default branch:** `git branch --merged <default>` lists the branch, or the branch's commits appear in `git log <default> --oneline`. If it is not merged, refuse: say so and stop. Never delete a branch whose work is not in the default branch yet.
2. `git checkout <default branch>` then `git pull` (this brings the merged MR into the local default branch). If the pull fails or produces conflicts, stop and report.
3. `git branch -d <branch>` — ask first if it is a ticket branch; never `-D`.
4. The **remote** branch is not touched: remind the human to delete it from the MR interface or by hand. `git push origin --delete` is theirs to run, never yours.

---

## Hard rules

- **Never `git push` or touch a remote** — including remote branch deletion. Publishing always stays with the human.
- **Never `-D`.** Safe delete only; if it refuses, report why.
- **Never revert or delete a file.** Unattributable changes stay out of the commit by default; nothing is ever reverted or removed.
- **No phase narration.** Never open with "Phase 1–5 done", "spec found", "gate passed", or a checklist of what was just reviewed. The first visible output is the Phase 6 block.
- **The mode question is the only question.** A/B/C is the whole approval; unattributable files stay out unless the human asks otherwise in chat.
- **Memory edits are minimal and factual.** Only facts this spec introduced, in the file's existing language and format; never a rewrite, never a changelog, never a new section without approval.
- **Never edit the state line outside the gate in Phase 2.**
- **Checkboxes are claims; this skill does not re-audit.** If the human wants verification first, that is `/spec-verify`.
- **No push, no remote, no conflict resolution on your own.**

## Summary of expected behavior

```
/spec-close 03-levels-and-highscores   (state: Aprobado, CloseMode: local)

  Phase 1  →  Finds specs/03-levels-and-highscores.md
  Phase 2  →  State gate: Aprobado → Implementado
  Phase 3  →  CloseMode: local
  Phase 4  →  Attributes every pending change: spec files + attributable code go in; unattributable stays out
  Phase 5  →  Memory sync: checks CLAUDE.md/AGENTS.md → no update needed, or minimal old → new edits
  Phase 6  →  One compact block + one question: A) local  B) pr  C) cancel
  Phase 7  →  Commit feat(spec-03-levels-and-highscores): <objective>
  Phase 8  →  Merge into main (no push), delete local branch spec-03-levels-and-highscores

/spec-close 06-new-thing   (state: Aprobado, CloseMode: missing)

  Phase 6  →  Same block, then A) local  B) pr  C) cancel — no mode assumed

/spec-close 04-payments   (state: Aprobado, CloseMode: pr)

  Phase 8  →  Commit only; branch kept for the MR
              Output: git push -u origin spec-04-payments + open the MR (human does both)
              Later: /spec-close 04-payments again → git pull + git branch -d

/spec-close 05-draft-feature   (state: Borrador)

  Phase 2  →  Refuses: only approved or already-implemented specs can be closed
```
