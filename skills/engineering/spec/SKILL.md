---
name: spec
description: Designs and develops specs following the spec-driven method. Asks clarifying questions before proposing structure, and builds the spec section by section. Use it when starting a large feature, before writing code.
disable-model-invocation: true
argument-hint: "short feature description or requirement"
---

# /spec — Guided spec designer

This skill helps you produce a useful spec following the spec-driven method. **You don't write code here.** Your job is to help the user clarify what they want to build, ask questions when something is not well-defined enough, and develop the spec section by section until it is ready to be saved into `specs/`.

## Philosophy

A spec is not decorative documentation. It is the contract that drives later execution. If the spec is vague, the code will improvise. That is why this flow is **deliberately slow during the definition phase** and **fast during the writing phase**.

Read `template.md` (in the same directory as this skill) to see the full structure the spec will follow. Lean on it at every step.

## Command flow

- Follow the four phases in order. **Do not skip phases.** If the user wants to go faster, remind them that the cost of a bad spec gets paid later in code.
- Your replies must be in the same language as the initial prompt. E.g.: if the initial prompt is in Spanish, your replies must be in Spanish; if it is in English, your replies must be in English.

## Question protocol

This skill must work in any agent (Claude Code, opencode, Codex, Cursor, Gemini CLI…). Questions are plain chat; a selection tool is an optional extra, never a requirement.

**Decisions** (choosing between concrete options):

- Show a numbered list in chat, one line per option, the recommendation first and marked `(Recommended)`, then close with "or tell me something else".
- Accept any natural reply: the number, the label, a paraphrase, or free text, in any language.
- If the agent has a selection tool (Claude Code `AskUserQuestion`, opencode `question`), you may use it with the same content. Never depend on it.

**Confirmations** (approving a drafted section, a file name, an edit): one short plain question in the user's language. Any affirmative answer proceeds. No magic words and no `[y/N]` / `[Y/n]` prompts. When there is a default, say it in words ("I'll save it as specs/03-slug.md unless you prefer another name").

Rules:

- One question per decision, with 2–4 concrete options.
- **Make each question self-explanatory.** The question text carries, in this order and in the user's words (not the implementer's): (a) what you already verified in the code or the conversation, if anything; (b) the decision itself; (c) one sentence saying what the answer defines ("This defines whether we need to send emails"). Never ask something the code can answer — read it and cite it instead.
- Labels are short and understandable on their own. Each option adds one line with the concrete consequence. If the label is already clear, keep that line minimal — no filler.
- Ask only about **material ambiguity** (scope, observable behavior, acceptance criteria). For minor details, assume a reasonable default and record it in the spec's decisions section.
- Follow dependencies: scope → behavior → data → details. Group questions in one message only when they are independent of each other (up to 4); if an answer can change the next question, ask them one at a time. Wait for all answers before continuing.

Example of a well-formed decision:

> Today there is no user onboarding flow. Who creates the password of a new user? This defines whether we need to send emails.
>
> 1. Email link (Recommended): the user picks the password from a one-time link; the admin never sees it.
> 2. Temporary password: the admin sees a generated password and passes it on.
>
> Or tell me something else.

### Phase 1 — Understand the context

Before asking questions about the feature, make sure you have project context:

1. Read the project-memory file, if one exists. Try in order and stop at the first hit: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`. This adapts the skill to whichever agent is running it (Claude Code, Codex, Gemini CLI, etc.).
2. List the contents of `specs/` to see which specs already exist and how they are numbered.
3. If previous specs exist, read at least the two most recent ones to pick up the project's conventions.
4. Check whether a `specs/NN-slug.brief.md` exists whose slug matches the feature being described. If it does, read it — it carries the business case (evidence, metric, kill criterion) produced by `/product-spec`, and this technical spec must reuse its exact `NN-slug` (see Phase 4, step 1) rather than being assigned a new number. If no matching brief exists and the feature's value or audience is not already obvious from the conversation, mention that `/product-spec` exists for this — but do not require it or block on it.
5. If the feature is still a fuzzy idea rather than a description (several approaches, unclear tradeoffs), suggest exploring it first with `/spec-explore` — it reads the code and creates nothing.

If the `$ARGUMENTS` argument comes in empty, ask the user for an initial **single-sentence** description of what they want to build. If the description does not fit in one sentence, that is the first signal that the feature is too big — suggest splitting it before continuing.

### Phase 2 — Clarify through questions

This is the most important phase of the command. Your job here is to **detect ambiguities and ask**, not to assume.

Ask using the Question protocol: group independent questions (up to 4 per call), but go one at a time when an answer conditions the next. After each call, wait for all the answers before continuing. Stop as soon as there is enough clarity — do not exhaust every branch.

**Question categories you should always consider:**

- **Scope:** What is in and what is NOT? Which parts of the feature are deferred to another spec?
- **Data:** What new structures are introduced? How are they named? Where do they live?
- **Integration:** Does this feature depend on previous specs? Does it modify something existing or only add?
- **Persistence:** Is anything saved between sessions? Where? With what versioning?
- **UX and states:** What does it look like when it works? What does it look like when it fails? Are there intermediate states?
- **Risks:** What can break this? What happens in the degraded case?
- **Tracking:** Does this spec belong to a tracker or sprint task, and what is its id? Ask only when the project-memory file shows an id convention (e.g. branch `feature/{id}-...`) or the user already mentioned an id; otherwise skip silently. The answer becomes the `Ticket` header line.
- **Closed decisions:** Is there any decision the user has already made and does not want to reopen?

**How to phrase the questions:**

- Use concrete questions, not open-ended ones. ❌ "How do you imagine persistence?" → ✅ "Is persistence localStorage, IndexedDB, or a JSON file on disk?"
- When you offer options, give 2–4, mark which one is your recommendation and why.
- If you spot an answer that would open Pandora's box (e.g. "and we also want multiplayer"), point out that it deserves its own spec and ask whether we leave it out of this one's scope.

**When to stop asking:**

Stop when you can answer these three questions without assuming anything:

1. Which files will appear or change?
2. What is the first executable step and what is the last one?
3. How do I verify the feature is finished?

If you still cannot answer one of them, keep asking.

### Phase 3 — Develop the spec section by section

Once you have clarity, **do not generate the full spec in one shot**. You will develop the template's sections **one by one**, showing each section to the user and waiting for confirmation before moving to the next one.

Strict order:

1. **Header** (state, dependencies, ticket id when known, date, one-sentence objective). The one-sentence objective is critical — if it does not fit in one sentence, go back to Phase 2.
2. **Scope** (what is in and what is NOT). The "not in" must be explicit.
3. **Data model** (concrete structures with real names). If the feature introduces no new data, skip this section and say so explicitly.
4. **Implementation plan** (groups of checkbox steps: `### Group N — <name>` + `- [ ] N.M ...`; each step leaves the system functional and each group is a reviewable, commitable chunk — `/spec-impl` ticks them off as it implements).
5. **Acceptance criteria** (boolean checklist, not aspirational). One observable behavior per criterion. Include at least one error or edge case when it applies, not only the happy path, and ask the user which case would hurt most if it broke.
6. **Decisions taken and discarded** (with brief justification).
7. **Identified risks** (only if applicable — if no relevant risks exist, skip it).

**After each section:**

- Show the draft inside a ```` ```markdown ```` fence so it displays verbatim even in terminals that do not render markdown, and is copyable. Keep your comments and the approval question outside the fence, as plain text.
- Draw any flow or diagram in plain ASCII (`+ - | -->`); Unicode box glyphs misalign across terminals.
- Ask one short confirmation in plain chat, in the user's language (e.g. "Does it look right, or should I adjust something?").
- If the user requests changes, apply them and show again.
- Only move to the next section once the user confirms. A short "sí" / "dale" / "ok" counts; silence does not.

**Common mistakes to avoid:**

- Generating acceptance criteria that are not verifiable ("that it works well").
- Putting things into the implementation plan that are not in the scope.
- Assuming file names or structures the user did not confirm.
- Skipping the decisions section — that section is the one with the most long-term value.

### Phase 4 — Save the spec

When all sections are confirmed:

1. Determine the file's `NN-slug`. If a matching `specs/NN-slug.brief.md` was found in Phase 1, reuse its exact `NN-slug` — the technical spec and the brief share the same number, they are not two entries in the sequence. Otherwise, determine the next sequential number by looking at `specs/`: if the last one is `02-powerups.md`, this one will be `03-`.
2. Generate a short slug from the objective (e.g. `levels-and-highscores`), unless step 1 already fixed it from an existing brief.
3. Run a silent self-check before asking: (a) the objective fits in one sentence; (b) each plan group serves something in scope, and each criterion has a step that makes it possible; (c) no vague criterion; (d) "out of scope" is explicit. Report only failures, one line each, and fix them with the user. If all pass, say nothing.
4. Ask in plain chat, in the user's language, whether the proposed file name works (e.g. "I'll save it as specs/03-slug.md, okay?"). Do not write the file before this confirmation.
5. Create the file at `specs/NN-slug.md` with all approved sections.
6. Mark the state as `Draft` by default. **Do not mark it as `Approved` automatically** — the user does that once they have re-read it.
7. **Seed the config file if it does not exist.** Check for `specs/.spec-config.yml`. If it is **missing**, create it with the default content below. If it **already exists, leave it untouched** — never overwrite the user's settings.

   ```yaml
   # spec workflow configuration
   #
   # AutoCreateBranch — controls whether /spec-impl creates the git branch automatically.
   #   true  (default) → /spec-impl creates and switches to spec-NN-slug without asking
   #   false           → /spec-impl asks you in chat before creating the branch
   AutoCreateBranch: true
   ```

8. Confirm to the user:
   - Path of the created file.
   - Reminder: the spec is in `Draft` state. Change it to `Approved` once you have re-read it.
   - If you just created `specs/.spec-config.yml`, mention it exists and that `AutoCreateBranch` defaults to `true` (set it to `false` to control branch creation yourself).
   - Next step: once reviewed and approved, run `/spec-impl NN-slug` to implement it.
   - To change the spec later, use `/spec-edit NN-slug` — it edits in place, keeps the state human-owned, and warns when re-approval is needed.
   - Optional before implementing: split the spec into tracker tickets (e.g. Forge, skill `forge-tickets`), one per implementation-plan group, with `Spec: specs/NN-slug.md` in each description. Then either work each ticket — `/spec-impl` reuses the ticket's work branch — or run `/spec-impl NN-slug` directly.
   - **Stop here.** Do not propose implementing the spec, writing code, or taking any further action beyond this confirmation.

## Hard rules

- **Never write code during this command.** Only the spec's `.md` file at the end.
- **Never propose implementing the spec after saving it.** Your job ends when the file is written. The user runs `/spec-impl` when they are ready.
- **Never assume decisions the user did not confirm.** If you are missing information, ask.
- **Never generate the full spec in a single response.** Section by section, with confirmation.
- **If the user wants to speed up and skip Phase 2**, remind them: "Questions now save hours later. Are you sure you want to skip them?". If they insist, respect their decision but record it in the spec's decisions section ("Quick definition without detailed clarification").
- **If the feature is too big** (does not fit in one sentence, touches more than three areas of the system, requires decisions in four or more domains), propose splitting it into two or more specs before continuing.

## Tone when asking questions

Be direct and specific. Do not apologize for asking. Do not use phrases like "if you don't mind..." or "could you maybe...". The user invoked this skill precisely because they want you to ask questions. Use the Question protocol when a decision needs concrete options; for everything else — including section approvals — ask in plain chat, one question per line.

Example of a well-formed block of independent questions:

> Before writing the data model I need to clarify two things:
>
> 1. **Persistence.** The game has no saved state today. Where do high scores live? This defines whether we need a storage layer. Options: (a) localStorage, (b) IndexedDB, (c) JSON file. Recommendation: (a), the data fits in <5MB and needs no queries.
> 2. **Schema versioning.** This defines how we migrate later. Options: (a) version prefix in the key, (b) ignore and rebuild, (c) migrate on load. Recommendation: (a).

## Arguments

If the user invoked `/spec levels-and-highscores`, use `levels-and-highscores` as the initial slug suggestion, but confirm with the user before writing the file.

If they invoked `/spec` without arguments, start by asking for the one-sentence description.
