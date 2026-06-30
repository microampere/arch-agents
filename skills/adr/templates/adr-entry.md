## <STORY-KEY> — <short title derived from requirements>

### Context

<2–4 sentences: what problem this story solves, why it is being built, and any key business constraints.>

### Decisions

- **Decision:** <what was decided, stated precisely>  
  **Rationale:** <why — reference Salesforce platform constraints, governor limits, or prior ADR decisions by story key where relevant>

- **Decision:** ...  
  **Rationale:** ...

### Constraints & Assumptions

- <one bullet per constraint or assumption that bounds the decisions above>

### User Journey

<!-- Include only when the story has a UI component. Omit this section entirely otherwise. -->

**Entry point: <how the user arrives — e.g., "Quick Action on Opportunity record page">**

1. <Step 1 — what the user sees or does>
2. <Step 2 — ...>

<!-- Repeat the block above for each distinct entry point or mode -->

### Method Contracts

<!-- Include only when the story introduces or modifies Apex methods, @wire adapters, or consumes an external/platform API. Omit this section entirely otherwise. -->

#### @AuraEnabled Methods

- `methodName(ParamType paramName, ...): ReturnType` — <one-line responsibility>
  - Wrapper: `ClassName { FieldType field; ... }` _(include only if the return type is a custom wrapper class)_

#### @wire Adapters

- `@wire(AdapterName, { param: value }) propertyName: AdapterType`

#### External / Platform APIs Consumed

- `ClassName.method(inputShape): returnShape` — <purpose>

### Open Questions

- <one bullet per item where the user explicitly said "I don't know" or "TBD" during the interview — omit this section entirely if none>
