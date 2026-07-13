# Protecting an already-built story's as-built work while a sibling is being implemented on a stale spec

**Date:** 2026-07-13
**Question:** US-01 is implemented but not yet reconciled. US-02 and US-03 have tech specs but are unimplemented — and someone has already started implementing US-02, off a spec/ADR that predates US-01's reconciliation. How do we make sure US-02's implementation doesn't inadvertently override what US-01 actually built?

---

## The risk

If US-02's implementation touches an artifact (field, object, permission set, etc.) that US-01 changed during its build, and US-02's branch/code was written against the *originally specified* (possibly wrong) version of that artifact rather than the *as-built* one, merging US-02 could silently clobber what US-01 actually shipped — either via a literal metadata overwrite, or via git auto-merging two non-conflicting-but-logically-incompatible edits.

## The gap in today's pipeline

Confirmed by grep across all four `skills/*/SKILL.md` files: there is no branch/in-progress/conflict awareness anywhere in the plugin.

- `/tech-spec`'s Phase 1b ("Unreconciled-Prior Warning") only fires on ADR `Superseded by` relationships — not general artifact overlap — and only runs at spec-*generation* time, which for US-02 was likely before this became an issue.
- `/reconcile`'s Phase 7 (sibling staleness) only checks `artifacts.md` Cross-References for not-`(built)` siblings, and only recommends "re-run `/tech-spec`?" — it has no idea whether that sibling already has live, in-progress code.

---

## Part 1 — Immediate steps (no code changes)

1. **Run `/reconcile US-01` now.** It's safe to run even with US-02 mid-flight — it only diffs git history and annotates docs (`ADR.md`, `domain-model.md`, `artifacts.md`); it never touches code. This is also the *only* way to get the as-built truth about US-01 written down, which is what US-02 needs to be checked against.
2. **Read Phase 7's output carefully.** It will very likely flag US-02 (and possibly US-03) as referencing an artifact that changed. Don't just answer the "re-run `/tech-spec`?" prompt reflexively — for US-02 specifically, treat a "yes" as a signal to *also* go look at the in-progress branch, not just regenerate a doc nobody's reading anymore.
3. **Diff the live US-02 branch against the corrected docs, by hand, for the flagged artifact(s) only.** You're looking for: does US-02's code assume the old (pre-as-built) shape of that artifact? E.g., if US-01's reconciliation revealed a field ended up as a Lookup instead of Master-Detail, does US-02's SOQL/DML/metadata assume Master-Detail semantics?
4. **Rebase (don't just merge) the US-02 branch onto the post-US-01 base branch before it merges.** This is the actual safety net for the "git auto-merges two incompatible edits without conflict" failure mode — rebasing forces US-02's changes to replay on top of US-01's real shipped state, so a genuine textual conflict on a shared metadata file surfaces instead of silently resolving wrong. A plain `git merge` of a long-diverged branch is what lets this slip through quietly.
5. **Only after US-02 is merged, run `/reconcile US-02`.** Don't reconcile a story that's still mid-implementation — Phase 1 diffs the final branch tip / merge commit, so reconciling early would just measure against incomplete work.
6. **Going forward, tighten the window:** reconcile each story immediately after it merges, *before* the next story's implementation starts. The whole class of risk here exists only in the gap between "story merged" and "story reconciled" — the shorter that gap, the less exposure.

---

## Part 2 — Skill enhancement: detect in-progress siblings, not just stale docs

**Problem with the status quo:** Phase 7 treats every not-`(built)` sibling the same, whether it's an untouched backlog item or a story with live commits on a branch right now. The recommendation ("re-run `/tech-spec`?") is right for the former and insufficient for the latter — regenerating a spec doesn't help code that's already written against the old one.

**Fix — extend `skills/reconcile/SKILL.md` Phase 7:**

For each sibling flagged structural (per the existing logic), reuse the *same* branch/commit detection Phase 1 already does (`git branch -a --list "*<SIBLING-KEY>*"`, falling back to `git log --all --grep="<SIBLING-KEY>"`) to check whether that sibling already has in-progress work.

- **No in-progress work found** → keep the existing behavior/wording unchanged (recommend `/tech-spec` regeneration).
- **In-progress work found** → escalate the message to name the live branch and push a merge-safety recommendation instead of (or in addition to) the spec-regen question:

  > ⚠ `<SIBLING-KEY>` references `<artifact>`, which changed in this reconciliation — **and `<SIBLING-KEY>` already has in-progress work on branch `<branch-name>` (N commits).** Regenerating the tech-spec may not be enough on its own: review that branch's changes to `<artifact>` against the as-built correction before it merges, and rebase it onto the current base branch first so a real conflict surfaces if the two are incompatible.

This is a same-shape addition to an existing phase (reusing Phase 1's detection code, not new machinery), so it's low-cost and consistent with how the skill already talks to the user (name the exact artifact, never "some things may need updating" — per the skill's existing Tone & Style rules).

**Target files (if/when this gets implemented):**
- `skills/reconcile/SKILL.md` — extend Phase 7 with the in-progress detection + escalated message, as above.
- `README.md` — add one line to the Rules section: reconcile each story immediately after merge, before the next story's build starts, to minimize the in-progress-sibling exposure window (documents the Part 1 discipline as a stated rule, not just tribal knowledge).
- `.claude-plugin/plugin.json`, `README.md` Version section, `showcase.html` (both spots) — version bump. This is new *behavior* in an existing skill (Phase 7 gains a new branch), so per this repo's own versioning rule in `CLAUDE.md`, it's a MINOR bump.

**Not in scope for this change:**
- No new skill, no new phase number, no change to `/tech-spec`'s Phase 1b (that check is about ADR `Superseded by` relationships and is a different, legitimate mechanism — not the same gap).
- No automated rebase/merge action — the skill stops at recommending it, consistent with `/reconcile` never taking git-mutating actions anywhere else in its design (Phase 1 only reads; Phase 4 requires human approval before any write).
