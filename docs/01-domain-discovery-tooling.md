a# Domain Discovery Tooling

> A Claude Code workflow to help developers extract domain knowledge from legacy codebases faster.

**Status:** Draft
**Audience:** Brianna, Greg, RG Mobile Team
**Related:** WW-RG-004, brownfield-documentation-generation-workflow

---

## Problem Statement

When adding new agent capabilities (e.g., "agent should stop timers"), developers spend 1-2 days discovering how things work instead of building:

| Step | Current State | Pain |
|------|---------------|------|
| 1 | Use Copilot: "How do timers work?" | Incomplete answers |
| 2 | Manually explore app code | Time-consuming |
| 3 | Ask Gemini with pasted code | Misses edge cases |
| 4 | Consult domain expert (Earl) | Expert bottleneck |
| 5 | Update implementation | Rework from missed rules |

**Brianna's quote:** "When creating a new tool, that's most of my day - getting all that information"

---

## Proposed Solution

A Claude Code `/discover-domain` command that generates **Domain Maps** - structured documentation of business entities with:

- Entity overview (what it is, where data lives)
- Code entry points (files, classes, functions)
- Business rules (from expert validation)
- Edge cases and pitfalls
- Acceptance criteria templates

### Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                    /discover-domain timers                       │
└─────────────────────────────┬───────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────────────────┐
        ▼                     ▼                                 ▼
┌──────────────┐    ┌──────────────────┐    ┌──────────────────────┐
│ 1. DISCOVER  │    │ 2. TRACE         │    │ 3. GENERATE          │
│ Find classes,│    │ Follow call      │    │ Draft domain map     │
│ methods,     │───▶│ chains, data     │───▶│ with business        │
│ entry points │    │ flow, side       │    │ rules template       │
│              │    │ effects          │    │                      │
└──────────────┘    └──────────────────┘    └──────────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │ 4. VALIDATE      │
                    │ Review with      │
                    │ domain expert    │
                    │ (Earl)           │
                    └──────────────────┘
```

---

## Target Projects

| Project | Language | Role |
|---------|----------|------|
| `Real-Green-Mobile` | C# (Xamarin) | Mobile app with heavy business logic |
| `rg-ai-mobile-api` | Python (FastAPI) | AI agent API (tools call mobile logic) |

The workflow must support **cross-project tracing**: agent tools in Python call business logic in C# mobile app.

---

## Command Design

### `/discover-domain [entity]`

**Input:**
- Entity name (e.g., "timers", "services", "customers")
- Optional: specific class or file to start from

**Output:**
- Domain map markdown file
- Mermaid diagrams (class relationships, call flow)
- Business rules template (blanks for expert to fill)

### Example Session

```bash
> /discover-domain timers

Discovering "timers" domain...

Found entry points:
- Real-Green-Mobile/RG.Mobile.Core/Managers/ProductManager.cs
- Real-Green-Mobile/RG.Mobile.Core/Managers/ServiceManager.cs
- rg-ai-mobile-api/rg_mobile_ai_api/tools/tools.py (request_production_timer_toggle)

Tracing call chains...
- StartTimer() → ValidateService() → CreateProductEntry()
- StopTimer() → DeleteProductEntries() → UpdateServiceStatus()

Generating domain map draft...
Created: docs/domains/timers.md

⚠️  Business rules need expert validation:
- [ ] RULE_001: Single active timer per technician?
- [ ] RULE_002: Product entries deleted on timer stop?
- [ ] RULE_003: Clock-in required before timer start?

Next: Review with domain expert (Earl) to validate rules
```

---

## Domain Map Template

```markdown
# Domain: [Entity Name]

## Overview
[What this entity is, where data lives]

## Code Entry Points

| Project | File | Class/Function | Purpose |
|---------|------|----------------|---------|
| Real-Green-Mobile | ... | ... | ... |
| rg-ai-mobile-api | ... | ... | ... |

## Business Rules

| ID | Rule | Source | Validated |
|----|------|--------|-----------|
| RULE_001 | [Description] | [Earl/Code/Inferred] | [ ] |

## Edge Cases

| Scenario | Expected Behavior | Discovered |
|----------|-------------------|------------|
| ... | ... | ... |

## Call Flow

[Mermaid sequence diagram]

## Data Model

[Entity relationships, database tables]

## Pitfalls / Warnings

- [Things that caught us off guard]
```

---

## Success Metrics

| Metric | Baseline | Target |
|--------|----------|--------|
| Domain discovery time | 1-2 days | 2-4 hours |
| Expert consultations per feature | 3-5 | 0-1 |
| Rework from missed edge cases | 1-2 | 0 |
| AI draft completeness | N/A | 80%+ |

---

## Implementation Phases

### Phase 1: Manual Pilot (This Week)
- Use Claude Code manually to generate domain map for "Timers"
- Validate with Earl
- Measure time and completeness

### Phase 2: Command Development
- Create `/discover-domain` command
- Add C# and Python discovery patterns
- Build domain map template

### Phase 3: Cross-Project Support
- Link Real-Green-Mobile and rg-ai-mobile-api
- Trace from agent tools to mobile business logic
- Generate unified domain maps

### Phase 4: Team Rollout
- Install in Real-Green-Mobile repo
- Train Brianna and team
- Document best practices

---

## Open Questions

1. **Storage location**: Where should domain maps live? (Confluence, repo docs, both?)
2. **Update trigger**: How do we detect when domain maps become stale?
3. **Expert availability**: How to structure Earl review sessions efficiently?
4. **Cross-repo linking**: How to reference code across Real-Green-Mobile and rg-ai-mobile-api?

---

## Action Points

> *To be filled by Bogdan after Brianna/Greg discussion*

| # | Action | Owner | Due | Notes |
|---|--------|-------|-----|-------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |