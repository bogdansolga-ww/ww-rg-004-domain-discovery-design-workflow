# Discover Domain

Generate a domain map for a business entity in RealGreen Mobile.

**Reuses:** `/discover-operations` and `/trace-operation` patterns from **ww-mi-004-brownfield-documentation-generation-workflow**

## Arguments

- `$ARGUMENTS` - Entity name (e.g., "timers", "services", "customers")

## Phase 0: Goals (Interactive)

Before discovering, ask the user:

1. **What AI capability needs this domain knowledge?**
   - e.g., "agent should stop timers", "agent should add job notes"

2. **What's already known vs. uncertain?**
   - Known constraints, unclear business rules

3. **Who will validate the domain map?**
   - Default: Earl (domain expert)

## Phase 1: Discover

### 1.1 Find Primary Manager

Search `Real-Green-Mobile` for Manager classes related to the entity:

```
Managers/[Entity]Manager.cs
Managers/[Related]Manager.cs
```

### 1.2 Find Entry Points

In the primary manager, identify:
- Public async methods (user-triggered actions)
- Event handlers (reactive logic)
- Methods called by other managers

### 1.3 Map AI Tool Connection

Search `rg-ai-mobile-api` tools for related functionality:
- Tool name matching entity
- Tools that send signals to mobile app for this domain

### 1.4 List Related Components

- **Entities:** Data models used by the manager
- **Proxies:** API clients called by the manager
- **Other Managers:** Cross-domain dependencies

## Phase 2: Trace

For each major entry point:

1. **Follow call chain** - What methods does it call?
2. **Extract business rules** - Validation, calculations, state changes
3. **Document side effects** - DB writes, API calls, events published
4. **Flag uncertainties** - Mark with `[?]` for expert validation

## Phase 3: Generate

Create domain map at `docs/domains/[entity].md` using template from `templates/domain-map.md`:

1. Fill Overview section
2. List Code Entry Points table
3. Document Business Rules with status (`[inferred]` / `[?]`)
4. Create State Machine diagram (if applicable)
5. Create Data Flow diagram
6. List Side Effects
7. Document Edge Cases (flag uncertain ones)
8. List Dependencies
9. Add Pitfalls/Warnings
10. Generate Acceptance Criteria template

## Output

Report:
- Primary manager found
- Entry points discovered
- AI tool mapping (if exists)
- Business rules extracted (count)
- Uncertainties flagged (count)
- Domain map created at: `docs/domains/[entity].md`

Prompt user to run `/validate-domain [entity]` with domain expert.
