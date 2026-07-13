# arch

> Claude Code plugin for Salesforce projects — interview-driven ADR capture, open question resolution, and AI-ready technical specs from stories.

## Skills

### `/adr` — Architecture Decision Record

Interactive, interview-driven session. You provide a story key and requirements text. The skill captures your requirements verbatim, interviews you as a Salesforce Technical Architect (one question at a time, with recommendations), then writes a precise ADR entry.

```
/adr HF-534 As a sales rep I need to see account health scores on the Account page so I can prioritise my pipeline calls.
```

### `/resolve` — Resolve Open Questions

Interactive. Resolves open questions in ADR entries using the same one-question-at-a-time interview protocol as `/adr`. Each resolved question becomes a Decision in the ADR. Can be run standalone or is automatically invoked by `/tech-spec` when open questions are detected.

```
/resolve HF-07, HF-34
```

### `/tech-spec` — Technical Specification

Fully automated. Reads requirements and ADR decisions for one or more stories and generates a fully self-contained technical specification ready for consumption by a building AI agent. Cross-references existing artifacts to enforce DRY. If open questions are detected, transitions to `/resolve` before stopping.

```
/tech-spec HF-02, HF-07, HF-34
```

Multiple stories in one invocation are processed sequentially — use this when stories are closely related.

### `/reconcile` — Reconcile As-Built vs. Designed

Fully automated, with a single bulk approval gate. Diffs a story's shipped code against its ADR entry, domain model, and tech spec; classifies every disagreement (missing, changed, or built-but-never-specified); writes the corrected current truth back into the living docs as non-destructive `As-built` annotations; flags security/sharing-sensitive deltas for explicit ratification; and reports (without auto-running) any not-yet-built sibling stories whose specs now reference stale information.

```
/reconcile HF-48
```

## Installation

Install this plugin into your Salesforce project via Claude Code's plugin manager, or clone this repo and install locally.

## File layout (created in your project root)

```
arch/
├── requirements.md          verbatim requirements per story
├── ADR.md                   architecture decisions per story
├── domain-model.md          current-state snapshot of objects, fields, relationships, record types
├── artifacts.md             manifest of every Salesforce artifact built across all stories — each story section tagged (not-built) / (built)
├── action-log.md            one line per completed run of any skill — `YYYY-MM-DD hh:mm - {command} {story}`
├── tech-specs/
│   ├── tech-spec-HF-02-260624.md
│   └── tech-spec-HF-07-260624.md
└── learnings/
    ├── adr.md               noteworthy observations from /adr runs
    ├── resolve.md           noteworthy observations from /resolve runs
    ├── tech-spec.md          noteworthy observations from /tech-spec runs
    └── reconcile.md         noteworthy observations from /reconcile runs
```

## Workflow

1. `/adr HF-XX <requirements>` — capture requirements, interview, write ADR entry
2. `/resolve HF-XX` — if open questions remain, resolve them (or let `/tech-spec` trigger this automatically)
3. `/tech-spec HF-XX` — expand ADR into a fully self-contained technical spec
4. Hand `tech-spec-HF-XX-YYMMDD.md` to your building AI agent
5. After the story is built and merged, `/reconcile HF-XX` — audit as-built vs. designed, correct the living docs, flag stale siblings

## Rules

- `/tech-spec` detects open questions → transitions to `/resolve` → stops. Re-run `/tech-spec` after resolving.
- The whole `/tech-spec` batch fails together if any one story has open questions.
- `/tech-spec` also warns (but does not block) if a related prior story was speced but never reconciled — its ADR grounding may be stale.
- Re-running `/adr` or `/resolve` for the same story overwrites that story's entry. `/tech-spec` writes a new date-stamped file per run (`tech-spec-HF-XX-YYMMDD.md`), overwriting only same-day reruns; the most recent file is authoritative.
- ADR decisions superseded by a later story are marked `**Superseded by HF-XXX**` — never deleted. As-built corrections from `/reconcile` are marked `**As-built (HF-XXX):**` beside the original text — also never deleted.
- `artifacts.md` is the cross-reference index — lazy-loaded per artifact, enriched over time. `/reconcile` is the only skill that flips a story's status from `(not-built)` to `(built)`.
- Learnings are appended after each run when something noteworthy occurred. Clean runs write nothing.

## Version

2.2.0
