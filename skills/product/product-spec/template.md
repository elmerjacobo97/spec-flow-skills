# Template for a useful product brief

This file is the reference the `/product-spec` skill consults when generating briefs. Each section includes its purpose and a minimal example. **It is not text to be copied verbatim** — it is the shape the skill must respect.

---

## Header

```markdown
# BRIEF NN — Short, descriptive title

> **Status:** Draft
> **Date:** YYYY-MM-DD
> **Objective:** A single sentence. If it needs two, the feature is too big — split it.
```

**Valid states:** `Draft`, `In review`, `Approved`, `Superseded`. Same state-machine convention as specs — labels are language-agnostic (`Borrador`/`Aprobado`/etc. all work), pick one set per repo and stay consistent.

---

## Section 1 — Evidence

Who asked for this and how strong is the evidence. Distinguish an anecdote from a pattern.

```markdown
## Evidence

- Source: 14 support tickets in the last 30 days, all citing the same missing confirmation step.
- Strength: pattern (not a single anecdote).
```

If the evidence is weak ("one person mentioned it once"), write that plainly. A brief with weak evidence is still useful — it tells the reader to treat the metric target as a guess.

---

## Section 2 — Current workaround

What users do today without this feature. If there is none and no visible pain, that is itself a finding.

```markdown
## Current workaround

Users manually re-check the payment page every few minutes. No in-app signal exists.
```

If there genuinely is no workaround and no pain signal: _"No workaround exists and no pain has been reported — this brief exists to test a hypothesis, not to fix an observed problem."_

---

## Section 3 — Metric

The one non-negotiable section. Every field is mandatory unless Section 6 (escape hatch) applies instead of this whole brief.

```markdown
## Metric

| Field | Value |
| --- | --- |
| Metric | % of users who retry a failed payment within 24h |
| Baseline today | 22% |
| Target | 40% |
| Check-in date | 2026-09-15 |
```

**Anti-patterns to avoid:**

- ❌ "Better user experience" — not a number.
- ❌ "Users will be happier" — not measurable.
- ❌ A target with no baseline — you can't tell movement from noise.
- ✅ A named metric, a real current value, a target value, a date to check both.

---

## Section 4 — The 20% version

The smallest version that still tests the hypothesis in Section 3, not the full feature as first requested.

```markdown
## 20% version

Send a single email 1 hour after a failed payment. No in-app banner, no retry
button in the UI yet — those are the 80% we're deferring until the email
version proves the hypothesis.
```

If the requester's original ask *is* already the smallest testable version, say so explicitly instead of inventing an artificial cut.

---

## Section 5 — Kill criterion

What result, checked on the date from Section 3, causes this to be reverted or deprioritized instead of iterated on.

```markdown
## Kill criterion

If the retry rate has not moved past 27% by the check-in date, revert the
email and do not build the in-app banner — the hypothesis that a nudge helps
would be falsified.
```

A brief without a kill criterion is a brief nobody will revisit — this section is what prevents a feature from becoming permanent by default.

---

## Section 6 — Instrumentation

Concrete event names and properties needed to measure Section 3. This section is where the brief ends if Section 3 had no baseline to begin with.

```markdown
## Instrumentation

- `payment_failed` — properties: `user_id`, `amount`, `reason`
- `payment_retry_started` — properties: `user_id`, `hours_since_failure`
- `payment_retry_succeeded` — properties: `user_id`, `hours_since_failure`
```

**Escape hatch case:** if Step 3 of the skill triggered because no baseline exists, this section is the *entire deliverable* of the brief. In that case, replace Sections 3–5 with a single note: _"No baseline exists for this behavior today. This brief recommends instrumenting the events below and waiting for at least one data point before scoping the feature itself."_

---

## Final section — Next step

Always the same, pointing at the paired skill:

```markdown
## Next step

`/spec NN-slug` — /spec will read this brief automatically for business context.
```

---

## Global rules about the whole document

- **One page.** If it doesn't fit on one scrolled screen, it's carrying scope that belongs in the technical spec instead.
- **No TODOs.** A TODO in the metric section means the decision was not made — resolve it or trigger the Section 6 escape hatch.
- **Concrete numbers, not adjectives.** "Significant improvement" is not a metric; "22% to 40%" is.
- **Standard markdown.** Must render on GitHub without surprises, same as specs.
