# Workflow Reuse Analysis

How **ww-rg-004-domain-discovery-design** (WIP) reuses **ww-mi-004-brownfield-documentation-generation-workflow** (validated).

---

## Executive Summary

Domain Discovery is a **specialization** of Brownfield Documentation, not a replacement. It reuses 70% of the methodology and adapts 30% for capturing tribal knowledge from domain experts.

---

## Core Concept Comparison

| Dimension | Brownfield Documentation | Domain Discovery |
|-----------|-------------------------|------------------|
| **Question answered** | "How does this code work?" | "What business rules govern this?" |
| **Starting point** | Technical entry point | Business entity |
| **Knowledge gap** | Missing docs | Missing tribal knowledge |
| **Validator** | Code reviewer | Domain expert |
| **Output consumer** | Developers | AI agent builders |

---

## Methodology Reuse

### Fully Reused (No Changes)

| Component | Description |
|-----------|-------------|
| **Phase structure** | 5 phases: Goals → Discover → Trace → Generate → Validate |
| **Interrogative Phase 0** | Ask questions before generating output |
| **Quality gates** | Checkboxes that must pass before next phase |
| **Staleness detection** | Compare docs against code changes |
| **Template-driven output** | Structured markdown with consistent sections |
| **Mermaid diagrams** | Visual documentation of flows and relationships |

### Adapted (Same Pattern, Different Focus)

| Brownfield | Domain Discovery | Adaptation |
|------------|------------------|------------|
| Find HTTP routes, jobs, queues | Find Manager classes | Different entry point patterns |
| Trace execution path | Trace business logic | Focus on rules, not just calls |
| Document dependencies | Document dependencies | Same |
| Flag code uncertainties | Flag business rule uncertainties | `[?]` for expert validation |
| PR-style review | Expert validation session | Non-technical reviewer |

### New (Domain-Specific)

| Component | Purpose |
|-----------|---------|
| **Validation status tracking** | `[validated]` / `[inferred]` / `[?]` per rule |
| **State machine diagrams** | Entity lifecycle visualization |
| **Acceptance criteria template** | Ready-to-use test scenarios |
| **AI tool mapping** | Link domain to Jerry's tools |

---

## Command Reuse Matrix

| Brownfield Command | Reuse Level | Domain Command |
|--------------------|-------------|----------------|
| `/discover-operations` | **Pattern reused** | `/discover-domain` |
| `/trace-operation` | **Embedded** | Part of `/discover-domain` |
| `/generate-tests-legacy` | **Future** | `/generate-domain-tests` (planned) |
| `/check-docs-staleness` | **Direct reuse** | `/check-domain-staleness` |
| `/render-architecture` | **Future** | C4 diagrams for domains (planned) |
| — | **New** | `/validate-domain` |

---

## Phase-by-Phase Reuse

### Phase 0: Goals (100% reused)

**Brownfield asks:**
1. Who will read this?
2. What's driving this effort?
3. Which diagrams help?

**Domain Discovery asks:**
1. What AI capability needs this?
2. What's already known vs. uncertain?
3. Who will validate?

*Same interrogative pattern, different questions.*

### Phase 1: Discover (Pattern reused, targets adapted)

**Brownfield finds:**
- HTTP routes (`@Get`, `app.MapGet`)
- Background jobs (`@Cron`, `IJob`)
- Message consumers (`IConsumer<T>`)

**Domain Discovery finds:**
- Manager classes (`*Manager.cs`)
- Public async methods (entry points)
- AI tool connections (`tools.py`)

*Same "find entry points" methodology, different file patterns.*

### Phase 2: Trace (80% reused)

**Both trace:**
- Call chains from entry points
- Dependencies (internal + external)
- Side effects (DB, API, events)

**Domain Discovery adds:**
- Business rule extraction
- State transitions
- Validation logic
- Expert uncertainty flags `[?]`

### Phase 3: Generate (Template adapted)

**Brownfield template sections:**
- Endpoint details
- Request/Response schemas
- Execution flow
- Error handling

**Domain map template sections:**
- Business rules table
- State machine diagram
- Acceptance criteria
- Validation status

*Same template-driven approach, different content structure.*

### Phase 4: Validate (Adapted)

**Brownfield:** PR-style code review with developers

**Domain Discovery:** Interactive session with domain expert
- Present uncertainties one by one
- Capture confirmations and corrections
- Update validation status

*Same HITL principle, different reviewer type.*

---

## What NOT to Reuse

| Brownfield Feature | Why Not Reused |
|--------------------|----------------|
| Structurizr DSL proposals | Overkill for domain maps |
| ADR inference from git | Domain rules predate git history |
| Test generation from code | Need scenario-based tests instead |
| Multi-framework detection | Single codebase (C# only) |

---

## Implementation Recommendation

### Phase 1: Current Sprint (Done)
- [x] Reuse Phase 0 interrogative pattern
- [x] Adapt `/discover-operations` → `/discover-domain`
- [x] Reuse `/check-docs-staleness` pattern
- [x] Create domain map template
- [x] Create playbook with quality gates

### Phase 2: Pilot
- [ ] Run `/discover-domain timers` manually
- [ ] Validate with Earl
- [ ] Refine template based on feedback

### Phase 3: Enhancement
- [ ] Add `/generate-domain-tests` (scenario-based)
- [ ] Add C4 diagram generation for domains
- [ ] Cross-project tracing (C# ↔ Python)

---

## Reuse Metrics

| Category | Brownfield Components | Reused in Domain Discovery |
|----------|----------------------|---------------------------|
| Methodology | 5 phases | 5/5 (100%) |
| Commands | 6 commands | 4/6 (67%) |
| Template sections | 10 sections | 7/10 (70%) |
| Quality gates | 4 gates | 4/4 (100%) |
| **Overall** | — | **~75% reuse** |

---

## Key Insight

> Domain Discovery doesn't replace Brownfield Documentation—it **extends** it for a specific use case: capturing tribal knowledge that isn't in code.

The brownfield workflow answers: *"What does the code do?"*
The domain workflow answers: *"What should the code do, and why?"*

Both are needed. Domain maps reference code locations discovered via brownfield patterns, then add the business context that only experts know.
