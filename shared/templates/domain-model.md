# Domain Model

> Current-state snapshot of the data model — what is true **right now**. No rationale, no
> history, no `Superseded by` markers (those live in `arch/ADR.md`). One section per object.
> Each field / relationship / record type cites the story key that last introduced or changed
> it as a trailing `<!-- STORY-KEY -->` comment, for traceability back to the ADR entry.

## <Object API Name>

- **Type:** Standard | Custom (`__c`)
- **Fields:**
  - `<Field_API_Name__c>` (<Type>[, length/precision]) <!-- STORY-KEY, STORY-KEY, ... -->
- **Relationships:**
  - <Lookup | Master-Detail> to `<Object>` via `<Field__c>` <!-- STORY-KEY, STORY-KEY, ... -->
- **Record Types:**
  - `<RecordTypeDeveloperName>` <!-- STORY-KEY, STORY-KEY, ... -->

<!-- Omit any sub-section (Fields / Relationships / Record Types) that has no entries. -->
