---
name: tech-spec-chain
description: Run /tech-spec across a queue of stories, one at a time, pausing for confirmation between each — for regenerating stale siblings flagged by /reconcile without committing to the whole batch
argument-hint: <story-key> <story-key> ...
disable-model-invocation: true
---

`$ARGUMENTS` is an ordered, space- or comma-separated list of story keys — the queue.

Take the first key. Run `/tech-spec` on it in full.

Then ask:

> `<KEY>` done. Continue with `/tech-spec-chain <REMAINING-KEYS>`? (y/n)

- Yes → repeat with the remaining queue.
- No → stop: "Resume anytime with `/tech-spec-chain <REMAINING-KEYS>`."
- Queue empty after dropping the head → stop, nothing to ask.

Never ask upfront to run the whole queue — one story, one confirmation, every time.
