# Artifact Block Format

Use `### <Type>: <API Name>` as the heading for every artifact. Include **every** attribute relevant to that artifact type — omit nothing. Every attribute in the templates below is mandatory unless explicitly marked optional. If an attribute does not apply to this artifact, write `N/A` — never leave it blank. Use the reference list below.

**Field**
```
### Field: <Object>.<API_Name__c>
- Label: 
- Type: (Text / Number / Currency / Picklist / Lookup / Master-Detail / Checkbox / Date / DateTime / Formula / Roll-Up Summary / etc.)
- Length / Precision / Scale: (where applicable)
- Default Value: 
- Help Text: 
- Required: Yes / No
- FLS: (which profiles/permission sets need Read / Edit)
- Picklist Values: (if applicable, list all values)
- Formula: (if Formula type, full formula expression)
```

**Custom Object**
```
### Custom Object: <API_Name__c>
- Label / Plural Label:
- Record Name: (field name and type — Text or Auto Number)
- Sharing Model: (Public Read/Write / Public Read Only / Private / Controlled by Parent)
- Description:
- Key fields: (list fields defined elsewhere in this spec or reused from artifacts.md)
```

**Apex Class**
```
### Apex Class: <ClassName>
- Sharing: (with sharing / without sharing / inherited sharing)
- Responsibility:
- Methods:
  - <methodName>(<paramType> <paramName>, ...): <returnType> — <what it does>
```

**Apex Trigger**
```
### Apex Trigger: <TriggerName>
- Object:
- Events: (before insert / after insert / before update / after update / before delete / after delete / after undelete)
- Responsibility: (one sentence — delegates to which handler class)
```

**Flow**
```
### Flow: <API_Name>
- Type: (Screen Flow / Record-Triggered Flow / Scheduled Flow / Platform Event-Triggered Flow / Autolaunched Flow)
- Trigger Object: (if Record-Triggered — object API name, trigger event: created / updated / created or updated / deleted)
- Trigger Condition: (if Record-Triggered — entry criteria, e.g. "Status__c changes to 'Active'")
- Scheduled: (if Scheduled — frequency and filter criteria)
- Platform Event: (if Platform Event-Triggered — event API name)
- Step-by-step logic: (number every step; be explicit about conditions, field values set, records queried or updated, subflows called, and screen components shown — no step may be summarised as "handle X" without specifying exactly how)
  1.
  2.
  3.
```

**LWC**
```
### LWC: <componentName>
- Placement: (record page / app page / utility bar / flow screen / etc.)
- Target Object: (if record page — object API name)
- @api Properties: (name, type, and purpose for each)
- Internal Properties: (tracked/reactive state variables — name, type, initial value)
- Events Fired: (event name, when fired, payload shape)
- Events Handled: (event name, source, handler behaviour)
- Data Retrieved: (full SOQL query or wire adapter name + parameters)
- Behaviour: (step-by-step — every user interaction, conditional render, and data mutation described explicitly)
```

**Lightning Record Page / App Page**
```
### Lightning Record Page: <API_Name>
- Object:
- Assigned To: (App / Record Type / Profile)
- Layout description:
```

**Permission Set**
```
### Permission Set: <API_Name>
- Label:
- Grants:
  - <Object or Field>: <access level>
```

**Platform Event**
```
### Platform Event: <API_Name__e>
- Fields:
  - <fieldName> (<Type>): <description>
- Published by:
- Subscribed by:
```

**Named Credential**
```
### Named Credential: <API_Name>
- URL:
- Authentication: (Named Principal / Per-User / JWT / etc.)
- Protocol:
```

**Custom Metadata Type**
```
### Custom Metadata Type: <API_Name__mdt>
- Purpose:
- Fields:
  - <fieldName> (<Type>): <description>
- Records to deploy:
```

**Custom Setting**
```
### Custom Setting: <API_Name__c>
- Type: (Hierarchy / List)
- Purpose:
- Fields:
  - <fieldName> (<Type>): <description>
```

**Custom Label**
```
### Custom Label: <Name>
- Value:
- Language:
- Purpose:
```

**Validation Rule**
```
### Validation Rule: <API_Name>
- Object:
- Error Condition Formula:
- Error Message:
- Error Location: (field-level field name / top of page)
```

**Email Template**
```
### Email Template: <API_Name>
- Subject:
- Type: (Text / HTML / Custom / Visualforce)
- Available Merge Fields:
- Body description:
```

**Sharing Rule**
```
### Sharing Rule: <API_Name>
- Object:
- Type: (Criteria-Based / Ownership-Based)
- Criteria / Owner: 
- Shared With:
- Access Level:
```

**Remote Site Setting**
```
### Remote Site Setting: <Name>
- URL:
- Purpose:
```

**Static Resource**
```
### Static Resource: <Name>
- Content type:
- Purpose:
```
