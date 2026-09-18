---
name: spec-close
description: Closes an approved or already-implemented spec — state gate to Implementado, a minimal sync of the project memory (CLAUDE.md/AGENTS.md) when the change added dependencies, modules, or commands, commit of the pending changes (asking first about changes the agent did not make), then CloseMode follows — local merge plus branch deletion, or commit only for the MR with a later post-merge cleanup. It never pushes.
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
- Missing or unrecognized → ask in the confirmation step: `Mode?  [l] local merge + delete   [p] keep branch for the MR`. Never assume a mode.

### Phase 4 — Classify the pending changes

Run `git status --short` and split every pending file into two buckets:

1. **Files the agent touched in this conversation** — if the implementation or the fixes happened here, its running `Files:` list. These enter the proposed commit.
2. **Files it did not touch** — a dependency the human added, a manual edit, an untracked file. These are **foreign changes**. Never include or revert one without explicit approval.

If you have no reliable run list (a different session, context compaction, manual work), treat **every** pending file as foreign. When in doubt, ask.

### Phase 5 — Sync project memory

The close is the last moment the implementation facts are fresh. Check whether the project-memory file still describes the project accurately, and propose the minimum needed.

1. **Find the memory file**, first hit in this order: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`. If none exists, report `Memory: no memory file found — skipped` and move on.
2. **Extract the facts this spec introduced** from the spec itself (`## Data model`, `## Decisions`) and from the pending diff: new dependencies, new modules or directories, new commands or scripts, changed conventions. Only facts introduced by **this** spec — no general cleanup, no rewriting for style.
3. **Evaluate always** and be explicit about the outcome:
   - Nothing relevant: `Memory: no update needed — this change adds no dependencies, modules, or commands.` Continue with no edit.
   - Relevant facts: prepare **minimal edits** in the memory file's own language and format, and show them in the confirmation as `old → new` pairs, one line per edit. Never regenerate the file, never turn it into a changelog.
4. **Section placement:** use an existing section that fits. If no section fits, ask before creating one; never invent structure without approval.
5. Approved memory edits become part of the close commit (Phase 7), staged with the other approved paths.

### Phase 6 — One confirmation, with everything visible

Resolve foreign changes and memory edits first, then present:

```
Close specs/NN-slug.md?

State:   <current> → <new>            (or "already Implemented — no change")
Mode:    local  — commit, merge into <default branch>, delete the branch
         (or: pr — commit only, keep the branch for the MR)
Branch:  <current branch> → <default branch>
Commit:  feat(spec-NN-slug): <objective from the spec>

Memory:  <CLAUDE.md — N edits proposed, listed below>
         (or: no update needed / no memory file found — skipped)
  + Dependencies: add `zustand`
  ~ Structure: features/payments/ now owns the checkout flow

⚠ Changes I did not make:
  - <path>  <modified|untracked>  → include / diff / revert / leave out

Then: <mode-dependent>
  local:  merge into <default branch> (NO push), delete local branch <branch>.
  pr:     stop after the commit — you push and open the MR.

Proceed? [y/N]
```

Per foreign change, offer: `include` (enters the commit), `diff` (show `git diff <path>` and ask again), `revert` (`git checkout -- <path>`, tracked files only — **never delete an untracked file yourself**, tell the human to remove it), or `leave out` (stays uncommitted). Wait for `[y/N]` before touching git.

### Phase 7 — Commit

Stage only the approved paths with `git add <path> …` — never `git add -A`. Commit with the proposed message (`feat(spec-NN-slug): <objective>`), including the state change and the approved memory edits when they apply. If the tree is clean, skip the commit and say so.

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
- **Never revert or delete a file the human did not explicitly approve.**
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
  Phase 4  →  Splits pending changes into mine vs foreign; foreign ones need approval
  Phase 5  →  Memory sync: checks CLAUDE.md/AGENTS.md → no update needed, or minimal old → new edits
  Phase 6  →  One confirmation with the full preview
  Phase 7  →  Commit feat(spec-03-levels-and-highscores): <objective>
  Phase 8  →  Merge into main (no push), delete local branch spec-03-levels-and-highscores

/spec-close 04-payments   (state: Aprobado, CloseMode: pr)

  Phase 8  →  Commit only; branch kept for the MR
              Output: git push -u origin spec-04-payments + open the MR (human does both)
              Later: /spec-close 04-payments again → git pull + git branch -d

/spec-close 05-draft-feature   (state: Borrador)

  Phase 2  →  Refuses: only approved or already-implemented specs can be closed
```
