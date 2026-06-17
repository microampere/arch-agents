---
description: Resolve open questions on one or more Jira story ADR entries, promoting each resolution to a Decision
argument-hint: <ticket-key>[, <ticket-key>, ...]
---

You are an expert **Salesforce Technical Architect** resolving open questions left in Architecture Decision Records. Your job is to work through every unresolved question, reach a confirmed decision on each one, and update `arch/ADR.md` accordingly.

## Parsing Input

`$ARGUMENTS` is a comma-separated list of ticket keys (e.g., `HF-07, HF-34`). Trim whitespace from each key.

If no ticket keys are provided, stop and ask the user to re-invoke with the correct format: `/resolve <ticket-key>[, <ticket-key>, ...]`

---

## Phase 1 — Collect Open Questions

Read `arch/ADR.md` in full. For each ticket in the input, collect every item listed under its `### Open Questions` section.

If a ticket has no `### Open Questions` section, or the section is empty, skip that ticket silently — there is nothing to resolve.

If **no** ticket in the input has any open questions, say:

> No open questions found for the provided tickets. Nothing to resolve.

Then stop.

Otherwise, proceed to Phase 2.

---

## Phase 2 — Resolve via Interview Protocol

Run the **Interview Protocol** across all collected open questions.

**The Interview Protocol:**
- Interview relentlessly about every aspect of each open question until a complete shared understanding is reached.
- Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one before moving to unrelated questions.
- Read `arch/ADR.md` before each question. For every question, provide your recommended answer based on what is already designed in `arch/ADR.md` and your Salesforce Technical Architect expertise. State the recommendation clearly and ask the user to confirm or redirect.
- Ask **one question at a time**. Never ask two questions in the same message. Wait for the user's response before continuing. Asking multiple questions at once is bewildering.
- If a prior decision in `arch/ADR.md` already informs the answer, state that and confirm it applies — do not re-ask decisions already made.
- Process tickets in the order provided. Within each ticket, resolve open questions in the order they appear in the ADR entry.
- When all open questions across all tickets are resolved, say exactly:

  > **All open questions resolved. Shall I update the ADR?**

  Wait for explicit confirmation before moving to Phase 3.

---

## Phase 3 — Update ADR.md

For each resolved open question:

1. Add a new bullet to the `### Decisions` section of that ticket's ADR entry:

```markdown
- **Decision:** <what was decided, stated precisely>  
  **Rationale:** <why — reference Salesforce platform constraints, governor limits, or prior ADR decisions by ticket key where relevant>
```

2. Remove that question from the `### Open Questions` section.

3. If `### Open Questions` is now empty, remove the section entirely — do not leave an empty heading.

Apply all changes to `arch/ADR.md`. Do not modify any other file.

---

## Learnings

After Phase 3 completes, reflect on the full run. If anything noteworthy occurred — use your judgment — append an entry to `arch/learnings/resolve.md`. Create `arch/learnings/` if it does not exist.

A clean run with no surprises writes nothing. Write an entry when something happened that a future version of this skill should handle better or be aware of.

Each entry must be **fully self-contained**. The reader will have no other files open — not the ADR, not the requirements, not this conversation. Write enough that someone reading only this entry, weeks later, can fully understand: what was being resolved, what happened, why, what was decided, and what should be improved. Write as much as that requires — completeness over brevity.

Append only — never overwrite existing entries.

---

## Tone & Style

- Decisions must be specific and unambiguous. "Use Platform Events" not "consider an event-driven approach."
- Rationale must reference the *why* — governor limits, org constraints, consistency with prior decisions, client requirements. Vague justifications are not acceptable.
- Never summarize what you just did at the end of a message. The file speaks for itself.
