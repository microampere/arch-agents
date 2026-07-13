# arch-agents Pipeline Evaluation — US-01 Spec vs. As-Built

**Author:** architecture review · **Date:** 2026-07-09/10
**Question:** why did the `/adr → /resolve → /tech-spec` pipeline produce a US-01 spec that needed ~20 corrective commits, and what concrete plugin changes would make the spec track the real implementation?

---

## 0. Methodology & honesty note

Grounded in three exhaustive read passes (plugin source, docs artifacts, as-built code + commit forensics), my own spot-checks of the code, and an evidence-fed persona/fact-check fleet run in batches.

- **Plugin source** `C:\wsjj\arch-agents` (v2.1.0; the installed cache that ran was v2.0.1 — the `/tech-spec` skill is byte-identical in both, so drift is not a version artifact).
- **Docs repo** `C:\ws\heartflow-arch\arch\` — `requirements.md`, `ADR.md`, `domain-model.md`, `artifacts.md`, `learnings/*`, `tech-specs/tech-spec-US-0{1,2,3,5}-260703.md`.
- **Implementation repo** `C:\ws\heartflow2` (git; US-01 = **HF-48**). First spec-driven build = commit `0408828` (2026-07-07, +58,133 lines). ~20 corrective commits followed through 2026-07-09 21:12.

**Fleet status (transparent):** the first 81-agent attempt was killed by the session rate limit. On your instruction I re-ran it in **value-ordered batches of ~10, writing after each**. **Batches 1–3 complete: 31/31 agents, 0 errors** — 3 Salesforce-docs fact-checks + 28 personas (sibling-spec staleness, reconciliation/regeneration design, the two honesty guards, the process-core lenses, and the Salesforce-domain lenses: feasibility, sharing, permissions, picklists, existing-automation, test-data, test-coverage, deploy-manifest, packaging, hierarchy-lifecycle). Batches 4–6 (remaining domain + artifact-fitness lenses) remain; each batch's deltas are logged in §10. Batch 1 corrected three draft claims; batch 2 added two recommendations + three honesty corrections; batch 3 upgraded the permissions and test-data recommendations, added a security finding, and dropped the packaging round from the preventable set — all integrated below.

**Three facts confirmed against Salesforce documentation** (were hypotheses in the draft):
- **Master-Detail with `QuoteLineItem` as master is platform-infeasible.** Salesforce's Object Reference lists the standard objects that cannot be a master-detail primary — BusinessHours, Idea, Lead, OrderItem, PriceBook2, Product2, **QuoteLineItem**, User. The spec's `QuoteLineSite__c.QuoteLineItem__c = Master-Detail` was not buildable. (`tech-spec-US-01-260703.md:119`; as-built `field-meta.xml:13` = `Lookup`.)
- **Controlled-by-Parent OWD requires Master-Detail.** A required Lookup cannot have it — so the spec's junction was *doubly* infeasible. As-built OWD is `ReadWrite` (`object-meta.xml:24`).
- **`PlaceSalesTransactionExecutor` is real (v66), but the ADR locked false *behavior* for it.** It executes **synchronously** and returns a `PlaceSalesTransactionResponse`. The ADR/spec justified an entire three-transaction design on the false premise that it "is a callout" throwing `CalloutException` and "returns a Quote Id" with QLIs auto-created by CPQ (`ADR.md:83,113-114`; spec `:57,98,185-191`). The as-built abandoned it for a plain `insert Quote` (`SitePickerController.cls:299-321`; the class name appears **zero** times in Apex). *(Fact-check also notes, as strong guidance, that a plain-DML Quote insert bypasses the RLM pricing engine and that `SyncedQuoteId` is normally set via Flow/@future — see the deferring note in §4.)*

---

## 1. Verdict (brutal)

**The pipeline manufactures confident, precise specifications without ever checking two kinds of reality — the state of the org, and the documented behavior of the platform APIs it names — and it has no pass that verifies its output against its own stated standard.** What it decides at the architecture level mostly survived into the correct build. But **~8–10 of the ~20 corrective commits (fair denominator ~15–16, once two other-ticket and two CI commits are removed) trace to org facts the pipeline couldn't know**, and a further class — infeasible metadata, an abandoned locked API, a required field left unvalued — slipped through as **silent day-one deviations with no commit at all**, because nothing re-reads the spec, the docs, or the shipped code.

Five verifiable indictments:

1. **No org-metadata channel.** `/tech-spec` reads only `requirements.md`, `ADR.md`, `artifacts.md` — not the org, the repo, or even `domain-model.md` (grep-verified). Picklist values, existing VRs, RLM prerequisites, packaging behavior: all unverified. This axis owns the ~8–10 preventable commits.
2. **No authoritative-docs channel, plus a "do not hedge" instruction that punishes uncertainty.** `contradiction-heuristics.md` checks internal consistency only; `interview-protocol.md:8` grounds recommendations in "your Salesforce Technical Architect expertise" (model memory); `adr/SKILL.md:134` says "do not hedge." That is how a fabricated API behavior and an infeasible relationship get locked and copied verbatim into the spec.
3. **The headline standard is asserted but unenforced.** The spec demands "implement… without making a single judgment call" and "every field has a value" (`skills/tech-spec/SKILL.md:10`), yet there is **no verification pass after the spec is drafted** (Phase 1 guards only Open Questions; Phases 2–4 generate and write). The judgment-calls lens counted **18 distinct judgment calls** the initial build was forced into. The spec even ships its own forbidden hedge — "Verify at deployment whether `PlaceSalesTransactionExecutor` accepts…" (`spec:186,190`) — which violates its own tone rules (`:156-157`) and nothing catches.
4. **A locked decision died silently, and nothing can notice.** The `PlaceSalesTransactionExecutor` mandate is absent from code; the docs still assert it. No corrective commit — so no commit metric can see it.
5. **The living docs are already wrong and actively poisoning future work.** `domain-model.md:65,70,72` still describes the junction as Master-Detail / cascade / Controlled-by-Parent — the infeasible design. That file is the fast-path source for the contradiction checks in `/adr` Phase 2 (`adr/SKILL.md:65`) and `/resolve` Phase 3 (`resolve/SKILL.md:62`). Every future story touching the junction is validated against a false model.

**The fair counterweight (the brutal-honesty guard):** "almost 20 commits" over-charges the spec. Drop 4 non-chargeable (HF-135: `74c3709`, `9115d4c`; CI: `fff46e0`, `d61b9b3`) → fair base ~15–16. Subtract the irreducible floor of ~4–6 (UI polish `602e6a4`/`5161746`/part of `997eb10`; one requirement-evolution `cc25916`; building-agent noise `b6c0277`, the ContextDefinition dump in `1fcb56d`, the packaging experiment `8badb67`→`4c8c663`). **Honest spec-preventable ceiling: ~8–10 commits — all in the org-grounding cluster** — plus a separate class of *silent day-one deviations* the commit metric can't see, addressed by a cheap self-audit (§6).

---

## 2. The altitude question — is each file at the right level?

**`requirements.md` — right altitude, one defect.** Pure business level. Defect: US-01/US-02 functional requirements are **unnumbered prose** while US-03+ carry stable IDs. No IDs = no join key between a requirement, its artifact, and its test. (Fix = R-IDs, Tier 3.)

**`ADR.md` — wrong altitude, and it's a root cause. The fix is to *split*, not strip.** The ADR locks method signatures, full SOQL, field API names, even an executor class name inside a method body (`ADR.md:45-64`) — forced by `adr-entry.md:30-45` (the Method Contracts template) and `adr/SKILL.md:106` ("detailed enough… to produce a spec from it alone"). The survival data proves the level is wrong:
- **Architecture-altitude content survived:** the junction, two-level hierarchy, `with`/`without sharing` split, transaction-commit ordering (`ADR.md:82-83`), the callout boundary, bulkification.
- **Identifier-altitude locks drifted, died, or churned:** `PlaceSalesTransactionExecutor` (dead — on a fabricated rationale), Master-Detail (infeasible → Lookup), `@wire` (→ imperative `getQuoteContext`), and **two method contracts superseded before code existed** (`ADR.md:60,64` — a mere add-a-parameter edit forced to manifest as an *architecture* supersession).

The tell that identifier-locking is false authority: the build **freely overrode every contract with no corrective commit**. Over-specification that is silently ignored is not a safety rail.
- **Fix:** re-scope `adr-entry.md:30-45` Method Contracts to *architectural properties only* (name + one-line responsibility, transaction/commit boundary, sharing context, callout-or-not); explicitly exclude typed params/returns, wrapper field lists, and verbatim SOQL. Weaken `adr/SKILL.md:106` to "produce the spec's Decisions and Constraints from it alone" — identifiers are materialized by `/tech-spec` (which already invents correct ones, e.g. `getExistingQuoteSiteIds`). Keep the *architectural* API constraint at the Constraints altitude (it already exists at `ADR.md:123`) — don't duplicate it into a method body where, when wrong, it becomes a permanent lie. **This fix prevents zero of the 20 commits; its payoff is doc-integrity and reserving the supersession mechanism for real architecture changes — and it makes reconciliation (D1) cheap.**

**`tech-spec-US-01` — right altitude, wrong coverage + unverified + unaudited** (see §6).

**`domain-model.md` — orphaned *and* wrong** (`:65,70,72`), yet still consumed by the contradiction checks. Strongest argument that *something* must write as-built truth back (§7).

**`artifacts.md` — right idea (a real reverse-dependency graph), under-exploited** — write-only, never reconciled, so it still lists the abandoned record page (`:20`) and the pre-rename PSG (`:22`) as live.

---

## 3. The 20 corrective commits, classified

| # | Commit | What changed | Class | Preventable & how |
|---|--------|--------------|-------|-------------------|
| 1 | `74c3709` | HF-135 data-load byproducts (reverted) | OUT_OF_SCOPE | No |
| 2 | `dfa1b18` | +5 RLM Settings, +`package-setup.xml`; PSG renames; `rca_Sales_Rep` gains object CRUD | ORG_BLIND + PREFERENCE | Partly — org-facts + perm audit |
| 3 | `fff46e0` | CI Node pin | OUT_OF_SCOPE | No |
| 4 | `d61b9b3` | CI Node pin #2 | OUT_OF_SCOPE | No |
| 5 | `7dec198` | PSG member renames | PREFERENCE | Marginal |
| 6 | `cc934be` | `StageName` → org-valid value | ORG_BLIND | **Yes** — org-facts |
| 7 | `d20f4ec` | PricebookEntry needs RLM ProductSellingModel | ORG_BLIND | **Yes** — org-facts |
| 8 | `1fcb56d` | Deleted 30k-line ContextDefinition dump; +bypass perm/test user | AGENT_DEVIATION + SCOPE_GAP | Partly |
| 9 | `21ffc2c` | add modified `Account.Create_Restriction` to manifest | ORG_BLIND | **Yes** — org-facts |
| 10 | `e9efcce` | VR gains `NOT($Permission.Bypass_Validation_Rules)` | ORG_BLIND | **Yes** — org-facts |
| 11 | `5d96bff` | Test data: `insert` PSM not query (SeeAllData) | ORG_BLIND | **Yes** — org-facts |
| 12 | `ce103e4` | PSM needs `PricingTerm`/`PricingTermUnit` | ORG_BLIND | **Yes** — org-facts |
| 13 | `9115d4c` | HF-135 tooling + `Product2.Netsuite_Id__c` | OUT_OF_SCOPE | No |
| 14 | `cc25916` | **NEW:** `Opportunity_Products__c` + picker column | REQ_EVOLUTION | No |
| 15 | `8badb67` | manifest object-member swap | ORG_BLIND (packaging) | **Yes** — org-facts |
| 16 | `b6c0277` | Deleted stray agent-invented `Quote.QuoteAccountId` | AGENT_DEVIATION | Marginal |
| 17 | `4c8c663` | Reverted #15 — object members "explode to 700+" | ORG_BLIND (packaging) | **Yes** — org-facts |
| 18 | `602e6a4` | datatable `wrapText` | POLISH | No |
| 19 | `5161746` | datatable `column-widths-mode` | POLISH | No |
| 20 | `997eb10` | Null-parent → blank picker (+ min-width) | SPEC_ERROR + POLISH | Partly — Spec Risks |

**Spec-preventable set (~6–8, refined by batch 3 — no single fix covers it):**
- **Test-data cluster** `cc934be, d20f4ec, 5d96bff, ce103e4` — needs **both** org-facts (RLM prerequisites, valid picklist values, SeeAllData rule) **and** a Test Data Strategy section to land the recipe. Org-facts alone is *necessary but not sufficient* — there is nowhere in the spec today for the fact to go (test-data-strategy + picklist lenses). Reclassified from pure ORG_BLIND to **SCOPE_GAP + platform-knowledge**, since RLM was explicit in the ADR (`PlaceSalesTransactionExecutor`), not a hidden org fact.
- **Existing-automation chain** `1fcb56d` (bypass-perm portion)`, 21ffc2c, e9efcce` — the pre-existing `Account.Create_Restriction` VR blocked test setup. Preventable (contingent on user recall) by an existing-automation interview clause (P4).
- **Permission derivation** `dfa1b18` (the `rca_Sales_Rep` object-CRUD portion) — **100% mechanically derivable** from the spec's own SOQL/DML, **not** org-blindness (permission-completeness lens). Fixed by P3.

**Dropped from preventable (batch 3):** the packaging round `8badb67→4c8c663` nets to zero on `package.xml` and self-corrected in 41 min by a dev whose commit message ("explodes to 700+ components") proves he *knew* the rule — it was a deliberate experiment triggered by out-of-spec object-config work, not spec ignorance a doc would cure (packaging-semantics lens beats deploy-manifest here). **Anti-double-count:** the test-data commits are counted **once** — org-facts + a Test Data Strategy section are one coordinated fix, not two additive ones.

---

## 4. Worse than the commits: silent day-one deviations (no commit trail)

The commit metric is structurally blind to these — the builder deviated *inside the initial build*. This is the class the org-free self-audit (P2) targets.

| Spec said | As-built | Status |
|---|---|---|
| `QuoteLineItem__c` = Master-Detail; OWD Controlled-by-Parent | required **Lookup**; OWD **ReadWrite** | **Confirmed infeasible** (both the relationship and the sharing model) |
| `PlaceSalesTransactionExecutor` is *the* Quote API (locked, callout/return-Id rationale) | plain `insert Quote`; executor absent from Apex | **Confirmed false rationale**; docs still assert it |
| `@wire(getRecord)` for gate fields | imperative `getQuoteContext` (`cls:513`) | LDS getRecord-coalescing quirk; unrecorded |
| 6 controller methods | +4 (`unlink…`, `demote…`, `getQuoteContext`, `buildAuraHandledException`) + `QuoteContext` | primary-change lifecycle never designed |
| no value for required `Quote.Name`; no `BillToContactId` | invented both (`cls:303-306`) | violated the spec's own "every field has a value" |
| no enforcement artifact for "two-level hierarchy only" | new VR `Primary_Opp_Cant_Be_Child_Opp` | ADR invariant had no materialized guard |
| new `Quote_Multi_Site_Record_Page` hosts edit mode | page abandoned; `Default_*` + new `Quote.Manage_Sites` | US-02/US-05 specs still reference the abandoned page |

> **On the RLM pricing path — deferring to your judgment.** The fact-check indicates (as strong guidance) that a plain-DML `insert Quote` bypasses the RLM pricing/configuration engine `PlaceSalesTransactionExecutor` integrates. You've said the as-built is correct, so I'm **not** flagging a bug — you know whether this path is pre-pricing or priced elsewhere. The *process* point stands: the pipeline locked a decision on a false rationale, the build silently replaced it, and nothing records whether that was a deliberate correct call or a latent gap. Surfacing exactly that for an expert to ratify is what D1 (§7) is for.

---

## 5. Root causes

1. **No org-metadata channel** (picklists, VRs, RLM prerequisites, packaging) — lookup-able facts nothing looks up. The sandbox org (`heartflow-full`) existed at spec time; grounding had a source.
2. **No authoritative-docs channel**, plus a "do not hedge" rule that converts platform unknowns into confident wrong assertions — *originating in the interview* (`interview-protocol.md:8,11`), one phase before `/tech-spec`.
3. **An unenforced standard**: no self-audit re-reads the drafted spec against its own "zero judgment calls / every field valued" promise.
4. **False-certainty by design**: the only two epistemic states are Decision or user-deferred Open Question — no channel for "concrete answer resting on an unverified fact."
5. **Nothing reads implementation back**, so locked decisions die silently and the living docs go wrong-and-consumed.

---

## 6. Recommendations

**Preventive** = reduces the ~8–10 build-time commits or the silent day-one deviations. **Detective/hygiene** = prevents *zero* commits; value is stopping future rework and doc-rot. Keeping these separate is the correction batch 1 forced; conflating them over-credits the reconciliation work.

### Tier 1 — preventive, highest leverage

**P1 — One human-maintained `arch/org-facts.md`, wired into the existing consumption pattern.** *(Cost lens's recommendation — more defensible than automated org introspection + four new sections.)*
- **Target:** new `arch/org-facts.md`; edit `shared/domain-checklist.md:6,13,14` ("before this domain, read `arch/org-facts.md`…"); edit `skills/tech-spec/SKILL.md:47-53` (Phase 2) to load it. Mirrors the pattern that already works for `domain-model.md`.
- **Contents:** valid picklist values per object; installed managed features + insert prerequisites (RLM: PricebookEntry ⇒ ProductSellingModel + Option + PricingTerm; SeeAllData rules); existing VRs/automation on touched objects + the `NOT($Permission…)` bypass pattern; packaging gotchas (whole-object members explode — deploy field-level members).
- **Prevents:** the ~6–8 org-blind commits (`cc934be, d20f4ec, ce103e4, 5d96bff, 21ffc2c, e9efcce`, RLM-settings part of `dfa1b18`, packaging round `8badb67→4c8c663`). **The single highest-ROI change.**
- **Cost:** near-zero recurring — maintained once per org, amortized across stories.

**P2 — Make the zero-judgment standard honest: a pre-write, org-free self-audit + a Spec Risks channel.** *(This is batch 2's biggest contribution. It catches the silent day-one deviations — infeasible MD, unvalued `Quote.Name`, abandoned `PlaceSalesTransactionExecutor` — that **neither P1 nor D1 catches**: P1 reads the org but these are platform/spec-internal facts; D1 is post-build. Only a pre-write audit stops them entering the spec.)*
- **Target:** new "Phase 3c — Self-Audit" in `skills/tech-spec/SKILL.md` (between draft and write); a new mandatory `## Spec Risks / Unverified Assumptions` section in the spec structure (`:74-96`); a third answer disposition in `shared/interview-protocol.md:11`.
- **The four org-free checks (block the write, bounce to `/resolve` on failure):**
  1. **Forbidden-hedge scan** — grep the drafted spec for "verify at deployment", "if accepted", "as appropriate", "as needed", "etc.", "TBD". Any hit is an unresolved judgment call. *(Catches the `spec:186,190` self-hedge → the silent `PlaceSalesTransactionExecutor` abandonment.)*
  2. **Required-field completeness** — every `insert`/`create` in specified code must value every required field of that sObject, **including standard required fields**. *(Catches unvalued `Quote.Name`.)*
  3. **Relationship/API feasibility** — a Master-Detail field must name a permitted master object; a named API's asserted behavior (sync/async, return type, side effects) must be doc-verified or flagged. *(Catches the QuoteLineItem MD infeasibility and the executor's false behavior.)* The relationship-feasibility lens notes the **primary home is `/adr`**: add the master-eligibility rule to `domain-checklist.md:6` (Data-model domain), right next to the related-list rule that *already exists there and already worked* (it drove the correct `Quote__c`-as-Lookup decision at `ADR.md:147`). The self-audit here is the belt-and-suspenders backstop. Also flag the downstream poison: `spec:126` falsely claims MD fields aren't independently FLS-governed — a wrong premise that a Lookup field *is* FLS-governed, which corrupted the permission reasoning.
  4. **Invariant-materialization** — every ADR invariant ("two-level hierarchy") must map to a named enforcement artifact (VR/trigger) or be declared out of scope. *(Catches the missing `Primary_Opp_Cant_Be_Child_Opp` VR.)*
- **Spec Risks channel + third interview disposition:** add a "concrete-but-unverified" disposition to `interview-protocol.md:11` (keep it a Decision, don't defer — but tag it), and a mandatory `## Spec Risks` section where each bullet states the assumption, the artifact it underpins, the check to run before building it, and the fallback. This gives architect known-unknowns a home instead of forcing them into confident assertions. **Note:** the MD-infeasibility is a *general platform fact* you only learn on deploy failure — reading the org does **not** reveal it; only this assumptions-audit does. So P2 is independent of P1, not a duplicate.
- **Cost:** low — all four checks operate on spec text + static platform facts, no org access.

### Tier 2 — preventive, mechanical or cheap-interview

**P3 — Permission Matrix Derivation (org-free, mechanical).** *(Upgraded by the permission-completeness lens: the permission gap is the single most mechanically derivable delta in the story, and it's 100% on the page — not org-blindness.)* Add a generation step to `skills/tech-spec/SKILL.md` (after artifacts are emitted, before writing the Permission Set) that: walks every specified method's SOQL (SELECT field lists, WHERE/relationship fields), DML (insert/update/delete target + assigned fields), and every LWC `@wire(getRecord)` field list; maps each to a required access level (SELECT→Read, insert→Create, update→Edit, delete→Delete; field→FLS Read/Edit); attributes it to the persona fronting the entry point; and **fails generation (like the open-questions guard) if any object/field the code touches lacks a matching grant.** Do *not* key FLS off Field artifact blocks — standard fields never get one; key it off the code. *Prevents the `rca_Sales_Rep` object-CRUD portion of `dfa1b18` (the spec's own code touches Opportunity/Quote/QuoteLineItem with no grant).* Cost: low, no org access.

**P4 — A Test Data Strategy section (the landing spot org-facts needs).** *(Reconciles a real tension: the cost lens argued org-facts.md alone suffices; the test-data and picklist lenses showed org-facts is necessary-but-not-sufficient because the spec has nowhere to put the recipe, so it evaporates like the orphaned Testing interview domain.)* Add a mandatory `## Test Data Strategy` section to `skills/tech-spec/SKILL.md:74-96` and a Test-Data block to `artifact-blocks.md`, carrying a **standing RLM landmine note**: when the story creates/reads RLM Quote/pricing data, test setup must **insert** the pricing scaffold (ProductSellingModel + Option + `PricebookEntry.ProductSellingModelId` + `PricingTerm`/`PricingTermUnit`) and must not rely on `SeeAllData`. org-facts.md (P1) supplies the *values*; this section is where they become a recipe the builder follows. *Prevents `d20f4ec, 5d96bff, ce103e4` (with P1: `cc934be`).* Cost: low — static platform knowledge, no org access.

**P5 — Two cheap interview clauses (mirror the Data-model current-state precedent).**
- **Existing-automation enumeration** — add to `domain-checklist.md:7` (Automation): before designing new automation, enumerate every DML write-path the story's code performs and ask the user which *existing* VRs/triggers/flows fire on those objects (this state cannot live in `domain-model.md` — its template has no automation slot — so it must be a user-sourced question). *Prevents (contingent on recall) the `Account.Create_Restriction` chain `1fcb56d`/`21ffc2c`/`e9efcce`.*
- **Mutable-role lifecycle probe** — add to `interview-protocol.md` after the existing line 14: when one record among peers carries a mutable designation (a self-lookup or flag like "primary"/"synced"), ask whether it can be *reassigned* and what happens to the former holder and its dependents on transition, and name the invariant. *Would have surfaced the entire primary promote/demote/unlink lifecycle (`SitePickerController.cls:245,319,358`) and the `Primary_Opp_Cant_Be_Child_Opp` VR that the builder invented unspecified.* (Prevents zero commits — it lands in `0408828` — but closes a real spec-fidelity gap.)

### Tier 3 — detective / hygiene (prevents **zero** commits; value is doc-truth + future rework)

**D1 — Write as-built truth back into the living docs after each story.** *Something* must correct `domain-model.md`, `ADR.md`, `artifacts.md` from shipped code — they are wrong now and are consumed by every contradiction check. Minimal-viable design (reconciliation-design lens), reusing existing machinery: read spec + ADR + docs + `package.xml`/git-diff; emit a three-bucket diff (specified-but-absent / specified-but-changed / built-but-unspecified); **human classification gate** (mirrors `resolve/SKILL.md:22-26` — mandatory, because the build mixes canon with noise like the stray `QuoteAccountId`); write-back never-deleting (`**As-built (<KEY>):**` markers, same construct as `**Superseded by**`); emit a stale-reference report for sibling specs.

**D2 — Selective regeneration for sibling-spec *staleness* (not contamination).** A sibling spec S regenerates after story T lands iff S is not-yet-implemented, T's as-built delta touches an artifact S references, and the delta is structural. Requires a per-story/per-artifact status field in `artifacts.md` (so US-05's already-shipped changes aren't re-emitted) and treating a combined run as one unit.

> **Honest disagreement preserved:** the **cost lens argued to KILL a standalone `/as-built` skill** (post-build, prevents zero commits, strains "usable by one person per story") and make write-back a manual note; the **reconciliation/regeneration lenses argued to build it** (the existing Phase-4 update writes from design intent, not code, and produced the stale sibling refs). **Adjudication:** the need is real, you asked for it, and `domain-model.md` being wrong-and-consumed makes "do nothing" untenable — so **start D1 as a manual discipline; harden to a skill only if drift recurs.** Do not sell D1/D2 as commit-reducing.

### Evaluated and NOT recommended (killed with reasons)
- **Spec-freeze / sequencing / snapshot-semantics machinery.** Three independent lenses (sequencing-policy, scope-contamination, supersession-semantics) confirmed later-story content leaked into the US-01 spec (US-05 gates folded in unmarked) **but caused zero commits and shipped correctly.** All candidate rules (freeze-at-impl-start, ADR-state-hash, forbid-in-flight regeneration) are provable no-ops on this timeline. An optional spec-hygiene tweak exists (a forward `[Added by <STORY>]` marker for additive later-story scope, symmetric with the existing `Superseded by` marker, at `tech-spec/SKILL.md:83`) — adopt only if you value traceability for its own sake; it prevents nothing.
- **Heavy full-test-code block** — still not recommended. The *Test Data Strategy* section (P4) and an AC→scenario + run-as-persona note are worth it (the spec feeds a separate testing agent, and half the build by volume is currently unspecified test code), but pedantically re-specifying 1,000+ lines of test bodies would balloon the spec for content the builder generates competently anyway. Note the test-coverage lens's honesty caveat: even a full Test block prevents **zero** of the 20 commits by itself — the test commits are org-fact/scope-gap fixes, so the value is AC-traceability and reproducibility, not rework reduction.
- **Heavy Deployment section — the two domain lenses disagree; I lean against.** The "metadata dependencies / order of operations" half **already lands** via the Constraints escape valve (`ADR.md:122`→`spec:97`), so the blanket "deploy scope gap" was overstated. The deploy-manifest lens argues for emitting a feature `package.xml` (cheap; would prevent `8badb67`/`4c8c663`); the packaging-semantics lens argues the packaging round is net-zero self-corrected trial-and-error by a dev who *knew* the rule, so it's not spec-preventable and a manifest block is altitude-wrong scope creep. **The commit message proves the dev knew the rule, so I side with packaging-semantics: don't add a manifest section.** Fold the packaging gotcha into `org-facts.md`/`CLAUDE.md` (which already grew a deploy note at `5d96bff`) as a convention, not a per-story spec artifact.
- **UI-pixel specification** (`wrapText`, widths) — honest iteration.
- **Mandatory "read all learnings every run"** — append-only files grow unbounded; cap or curate if adopted. Maps to zero commits.

---

## 7. Cross-spec drift: how US-02/03/05 must adjust to each successive implementation

Your second question. **Distinguish two things the fleet proved are different:**
- **Sibling-spec *staleness*** (US-02/03/05 reference US-01 artifacts that changed as-built) — **real future rework**, fixed by D1 + D2. Detailed below.
- **Intra-spec *contamination*** (US-05 content folded into the US-01 spec) — **benign; zero commits; shipped correctly.** Do not build machinery for it (see §6 kills).

**Principle: never hand-patch stale specs — write as-built truth into the docs (D1), then selectively regenerate not-yet-implemented siblings (D2).** All sibling staleness is *latent* (US-02/03/05 unbuilt) — it caused **none** of the 20 commits.

**US-02 — 3 verified stale refs (2 high-severity):**
- `:174/:178/:188` add a related list to `Quote_Multi_Site_Record_Page` — **abandoned**. → Retarget `Default_Quote_Record_Page`.
- `:38-39` justify the auto-link guard on "createQuote calls associateForQuote before commit" — as-built `createQuote` plain-inserts with no QLIs, so that race doesn't occur. → Rewrite rationale to the as-built flow.
- `:41` calls the QLI leg "master-detail" — it's a Lookup with Cascade. → "lookup leg (Cascade)." *(Conclusion still holds.)*
- **Refuted:** the `associateForQuote` signature ref is correct; the PSG rename is a US-01-spec problem, not US-02's.

**US-03 — a hard, compile-breaking staleness:**
- Its lock/unlock depends on `QuoteLineSite__c.Quote__c` (`:70,:88,:202,:208,:483,:484`) — **doesn't exist** as-built; the junction relates via `QuoteLineItem__r.QuoteId` (`cls:426,490`). Every `WHERE Quote__c = :quoteId` fails to compile. → Use the traversal; drop the `Quote__c` prerequisite.
- `rca_Sales_Rep` grants (`:466,470-475`) assume field-level FLS; as-built is object-CRUD + viewAllFields → restate.
- **Refuted:** the sharing delta does *not* break US-03's *locking* (`Approval.lock()` is per-record and overrides OWD). **But a latent security regression hides here** (sharing-model lens): the forced MD→Lookup flip turned the junction OWD from Controlled-by-Parent into **ReadWrite/Public** (`object-meta.xml:24-25`), and `rca_Sales_Rep` holds full CRED on it — so *outside* the approval-lock window, any Sales Rep can read/edit/delete **any** territory's site assignments. US-03's `without sharing` rationale (`:90`) was written against a Controlled-by-Parent model that no longer exists. This is exactly the kind of forced-deviation consequence D1 must surface for a human to ratify (make the junction's Sharing Model a *derived* attribute, not a free-text assertion). **New gap:** US-03's approval-lock is also uncoordinated with the contract-lock guard already in as-built `syncQuoteLineSites`.

**US-05 — 100% stale, zero residual:** all three mandates already shipped in `0408828` (`cls:402-412`; `sitePicker.js:74-172`; `.html:10-129`). → Regeneration reduces US-05 to an "already delivered" list, leaving only the `@wire`→imperative divergence to record.

**Edge cases D2 must handle (all live here):** partially-implemented (US-05), combined-run coupling (US-01+US-02), abandoned artifacts (delete from `artifacts.md`, don't repoint).

---

## 8. The agile boundary — the honest floor

Fair denominator ~15–16. Irreducible floor even with every fix: ~4–6 (UI polish; one requirement-evolution, which should round-trip via D1; building-agent noise). **Realistic target: ~4–6 residual commits per story, down from ~20**, by killing the org-grounding cluster and the silent-deviation class. Not zero.

---

## 9. Prioritized action list

1. **`arch/org-facts.md`** + `domain-checklist.md:6,13,14` + `tech-spec/SKILL.md:47-53` (P1). *~6–8 commits, near-zero cost.*
2. **Pre-write Self-Audit (Phase 3c) + Spec Risks section + third interview disposition** (P2). *Kills the silent day-one deviations org-grounding and reconciliation both miss.*
3. **Object-CRUD in Permission Set block + existing-automation enumeration** (P3).
4. **Split ADR Method Contracts by altitude** — `adr-entry.md:30-45` + `adr/SKILL.md:106` (§2). *Doc-integrity; makes D1 cheap.*
5. **As-built write-back discipline** (D1) — start manual; `domain-model.md:65,70,72` is wrong *now*.
6. **Selective-regeneration policy + `artifacts.md` status field** (D2) — before US-02/03/05 are built.
7. **Immediate fixes:** correct `domain-model.md:65,70,72`; fix the three US-02 / six US-03 stale lines in §7.

---

## 10. Self-critique & fleet corroboration log

**Batch 1 corrections (adopted):** split "no org grounding" into two axes (org-metadata vs authoritative-docs); cut the count from ~13–15 to ~8–10; separated preventive from detective; adopted `org-facts.md` over automated introspection; upgraded two hedges to confirmed facts; replaced §7 with concrete cited findings incl. the US-03 compile-break.

**Batch 2 additions & corrections (adopted):**
- **Added P2** — the pipeline manufactures false certainty by design (only two epistemic states; a "do not hedge" rule; no post-draft self-audit), and the fix is a cheap **org-free pre-write audit + Spec Risks channel**. This closes the *silent day-one deviation* class (MD, `Quote.Name`, `PlaceSalesTransactionExecutor`) that neither P1 nor D1 reaches. Biggest gap in my v2.
- **Sharpened the ADR altitude fix to "split, don't strip"** with concrete targets, and noted it prevents zero commits (its harm is doc-integrity + false authority).
- **Corrected the "deploy scope gap"** — dependency-ordering already lands via Constraints; only manifest + rollback are homeless.
- **Anti-double-count** — the ~6 test/deploy commits are org-blind at root; they are the *same* commits whether framed as scope-gap or org-grounding, counted once.
- **Killed spec-freeze/sequencing/snapshot machinery** — three lenses independently showed the leakage was benign and the fixes are no-ops. Kept sibling-*staleness* reconciliation (D1/D2) as future-rework only; separated it from intra-spec *contamination*.
- **Pushed org-grounding upstream** into the interview (third disposition + existing-automation enumeration) — the damage originates one phase before `/tech-spec`.

**Preserved dissent:** the cost lens would not build a `/as-built` skill; I kept D1 but recommended starting it manual.

**Fleet status:** batches 1–2 complete (21 agents, 0 errors), integrated. Batches 3–6 (Salesforce-domain + artifact-fitness lenses) pending; expected to mostly corroborate §5–§7. Any contradiction will be integrated here with the same discipline.
