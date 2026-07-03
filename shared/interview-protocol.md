# Interview Protocol

Whenever a skill document says to "run the Interview Protocol," follow these rules exactly — no exceptions:

- Interview relentlessly about every aspect of the topic in scope until a complete shared understanding is reached.
- Walk depth-first: resolve dependencies between decisions one-by-one before moving to unrelated branches.
- Ask **one question at a time**. Never ask two questions in the same message. Wait for the user's response before continuing — asking multiple questions at once is bewildering.
- For every question, provide a recommended answer grounded in the project's existing design (`arch/domain-model.md` and `arch/ADR.md`) and your Salesforce Technical Architect expertise. State it clearly and ask the user to confirm or redirect.
- If a prior decision in `arch/domain-model.md` or `arch/ADR.md` already answers the question, say so and confirm it still applies — do not re-ask settled decisions.
- **Detect contradictions inline.** If an answer (or an accepted recommendation) contradicts a current-state fact in `arch/domain-model.md` or a decision in `arch/ADR.md`, stop and surface it immediately — cite the conflicting prior story key — and resolve before moving on. Never bank a contradictory answer to defer.
- Never treat uncertainty or complexity as grounds to defer. Keep probing — sub-questions, concrete recommendations, narrowed options — until the user gives a concrete answer **or** explicitly says "I don't know" / "TBD". Only an explicit deferral is left as an Open Question.
- **Press on vague instructions.** When an answer contains both a concrete decision and a vague behavioral instruction (e.g., "display," "notify," "show," "inform," "alert"), the vague half is not answered. Immediately re-ask it with a concrete follow-up: "You said X — where and how does that appear to the user?" This applies especially when there is no UI component in scope, since "display" then implies a surface that must be explicitly named.
- **Probe edge cases before mechanism.** Before discussing the *implementation mechanism* for a hide/show, gating, or lock rule, confirm the rule's behavior in its edge or terminal state — e.g., "if the blocking condition is cleared or becomes invalid, should the hidden/locked affordance become available again?" The answer can dissolve the need for the mechanism question entirely; don't design the mechanism first and discover the edge case later.
- **Verify collection membership for specially-designated members.** When a hierarchy or set has one specially-designated member (e.g., a "primary" or "default" record among peers), do not assume an operation scoped to "all members" excludes it because of its special role. Ask explicitly: "Does the specially-designated member also need to be covered, or is its designation purely structural (e.g., anchoring a relationship) with no bearing on collection-wide operations?"

## Handling Mid-Interview External Corrections

When the user interrupts an in-progress interview with new information from outside the current story — signaled by phrases like "I got new information" or "I was given new direction" — treat it as a scoped, ad-hoc contradiction check rather than folding it silently into the current story's decisions:

1. Pause the current interview thread.
2. Grep `arch/ADR.md` and `arch/domain-model.md` for every occurrence of the changed value.
3. Classify each occurrence as a **hard reference** (API name, DeveloperName, field name, or any other code-level identifier — including implicit dependencies, such as a SOQL query that filtered on the old value without an explicit reference to it) or a **soft reference** (prose description only, no code impact).
4. List only the hard references as requiring supersession. For each, also check whether it changes any implicit assumption downstream (e.g., a query that could now silently return unintended records).
5. Confirm the scope of impact with the user before proceeding.
6. Resume the paused interview.
7. Apply the correction as a supersession when the ADR is written (see each skill's write phase) — never edit `arch/ADR.md` mid-interview. The prior story's original decision must remain readable alongside the correction via a `**Superseded by <STORY-KEY>**` marker.
