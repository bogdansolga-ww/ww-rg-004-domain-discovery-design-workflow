# Validate Domain

Interactive session to validate a domain map with a domain expert.

## Arguments

- `$ARGUMENTS` - Entity name (e.g., "timers", "services", "customers")

## Prerequisites

- Domain map exists at `docs/domains/[entity].md`
- Domain expert available (default: Earl)

## Execution Steps

### 1. Load Domain Map

Read `docs/domains/[entity].md` and extract:
- All items marked with `[?]` (uncertainties)
- All items marked with `[inferred]` (needs confirmation)
- Edge cases without source

### 2. Present Validation Checklist

For each uncertainty, present to user:

```
## Validation Session: [Entity]

### Business Rules to Validate

1. **RULE_001: [Description]**
   Status: [?] Needs confirmation
   Code suggests: [what code implies]

   → Is this correct? (yes/no/clarify)

2. **RULE_002: [Description]**
   Status: [inferred] From code pattern

   → Confirm or correct?
```

### 3. Capture Expert Input

For each item:
- Record confirmation or correction
- Add any additional context provided
- Note edge cases mentioned

### 4. Update Domain Map

After validation session:
- Change `[?]` → `[validated]` for confirmed items
- Update incorrect rules with expert's correction
- Add new edge cases discovered
- Add expert's name and date to changelog

### 5. Report

```
## Validation Complete

- Rules validated: X
- Rules corrected: Y
- Edge cases added: Z
- Remaining uncertainties: N

Domain map updated: docs/domains/[entity].md
Validated by: [Expert Name] on [Date]
```

## Quality Gate

Session complete when:
- [ ] All `[?]` items resolved
- [ ] All `[inferred]` items confirmed or corrected
- [ ] Expert sign-off obtained
- [ ] Changelog updated
