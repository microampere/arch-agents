# Domain Checklist

Cover all relevant domains below — only those in scope for the story:

- **User Journey** — Ordered steps a user takes end-to-end for each entry point and mode; screen states, loading/error states, confirmations, post-completion navigation. _(Cover only for stories with a UI component.)_
- **Data model** — Objects, fields, relationships, record types. _Before interviewing this domain, read `arch/domain-model.md` as the authoritative current-state snapshot. Base recommendations on what already exists there — not just `arch/ADR.md` history — and state which relevant objects / fields / record types already exist before proposing any addition or change._ When a story proposes a related list of object X on parent page Y and X is not a direct child of Y (e.g., a grandchild via an intermediate object), surface the missing direct lookup field requirement immediately — Salesforce related lists require a direct parent-child relationship. When a story asks to capture configuration attributes (type, amount, duration, etc.) on a child record added via a configurator or product-picker flow, ask explicitly whether entry happens through configurator attribute mapping (e.g., a CPQ attribute-to-field mapping layer) or through direct field editing after the record is added — do not default to recommending direct field edits without asking which mechanism the project uses.
- **Automation** — Flow vs Apex trigger vs scheduled job; sync vs async. When requirements say "[Role] approval is required" for a specific condition, disambiguate whether that means only that step or the full approval chain up to and including it — ask explicitly rather than assuming.
- **Integration** — REST/SOAP/Platform Events/CDC/Outbound Messaging; auth pattern; for each external or platform API consumed: the exact input shape (parameters or fieldValues map keys) and the return/response shape the caller depends on.
- **UI** — LWC component design, navigation, data binding; component mode/context determination (how the component identifies which surface it is on and which mode to enter); @api properties and internal state; conditional rendering rules; user-facing loading and error states.
- **Sharing & Visibility** — OWD, sharing rules, manual shares, with sharing / without sharing.
- **Governor limits** — Bulkification strategy, async offload, limit exposure points.
- **Error handling** — Retry strategy, dead-letter logging, user-facing messages; for multi-step transactions without rollback: what happens to partial state if a later step fails, and what does the user see.
- **Deployment** — Metadata dependencies, order of operations, rollback plan.
- **Testing** — Unit test scope, mock strategy, integration test triggers.
- **Method Contracts** — Signature for every @AuraEnabled method (name, typed parameters, return type), every @wire adapter (adapter name, parameters, reactive property type), and every wrapper/inner class returned (all fields with types). _(Cover only methods introduced or materially changed by this story; address this domain last, after all upstream decisions are settled.)_

Walk depth-first: resolve dependencies between decisions before moving to unrelated domains (e.g., confirm the data model before asking about automation that references it).
