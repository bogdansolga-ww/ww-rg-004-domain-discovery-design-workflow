# Check Domain Staleness

Detect outdated domain maps by comparing against code changes.

**Reuses:** `/check-docs-staleness` pattern from **ww-mi-004-brownfield-documentation-generation-workflow**

## Execution Steps

### 1. Find Domain Maps

List all files in `docs/domains/`:
```bash
ls docs/domains/*.md
```

### 2. For Each Domain Map

Extract metadata:
- Last Updated date
- Primary Manager file
- Related files mentioned

### 3. Check Code Changes

For each domain map, check if source files changed since last update:

```bash
git log --since="[last-updated]" --oneline -- [manager-file]
```

### 4. Analyze Changes

Categorize staleness:

| Level | Criteria |
|-------|----------|
| **Fresh** | No changes to source files |
| **Minor** | Comments, formatting, minor refactors |
| **Moderate** | New methods added, parameters changed |
| **Stale** | Business logic changed, rules modified |
| **Critical** | Major refactor, file renamed/moved |

### 5. Report

```
## Domain Map Staleness Report

| Domain | Last Updated | Staleness | Changes |
|--------|--------------|-----------|---------|
| timers | 2026-01-15 | Fresh | None |
| services | 2025-12-01 | Stale | 5 commits, pricing logic changed |
| customers | 2025-11-20 | Critical | Manager split into 2 files |

### Recommended Actions

1. **services** - Re-run `/discover-domain services`, validate pricing rules
2. **customers** - Full re-discovery needed, major refactor detected
```

## Staleness Triggers

Flag for re-validation when:
- Manager file has commits with keywords: `fix`, `change`, `update`, `refactor`
- New public methods added
- Method signatures changed
- Business rule comments modified
- Related AI tools changed in `rg-ai-mobile-api`
