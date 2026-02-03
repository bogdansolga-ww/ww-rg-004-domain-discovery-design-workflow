# Architecture Recommendation

> Analysis of the "business logic in mobile" problem with paths forward.

**Status:** Draft
**Audience:** Greg (Chief Architect), Brianna, RG Mobile Team
**Related:** WW-RG-004, Greg's architectural concerns

---

## The Architectural Problem

**Greg's summary (verbatim):**
> "In general, how we are approaching the agent in the mobile architecture is a problem. We have soo much business logic in the mobile app, and very little logic server side."

This creates a fundamental tension:

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI Agent (rg-ai-mobile-api)                   │
│                    Python / FastAPI / Strands                    │
└─────────────────────────────┬───────────────────────────────────┘
                              │ Calls tools like:
                              │ request_production_timer_toggle()
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RealGreen API (Backend)                       │
│                    Thin layer - minimal logic                    │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Real-Green-Mobile App                         │
│                    C# / Xamarin                                  │
│    ┌─────────────────────────────────────────────────────────┐  │
│    │ ProductManager, ServiceManager, CustomerManager...       │  │
│    │ HEAVY BUSINESS LOGIC HERE                                │  │
│    │ - Timer rules, service states, product entries           │  │
│    │ - Validation, edge cases, domain rules                   │  │
│    └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

**The problem:** The AI agent needs to understand and enforce business rules that live in mobile C# code, not in the API it calls.

---

## Why This Matters for AI Agents

| Issue | Impact |
|-------|--------|
| **Agent can't see mobile logic** | Tools call APIs, but real validation happens client-side |
| **Duplicate validation needed** | Agent must re-implement rules that mobile already enforces |
| **Drift risk** | Mobile logic changes, agent doesn't know |
| **Earl bottleneck** | Only expert knows the mobile business rules |
| **Testing gap** | Agent tests don't cover mobile-side validation |

**Example:** The `request_production_timer_toggle` tool calls an API, but:
- "Product entries must be deleted on timer stop" is enforced in `ProductManager.cs`
- "Technician must be clocked in" is checked in mobile UI
- Agent has no visibility into these rules

---

## Three Paths Forward

### Path A: Document in Place (Short-term)

**What:** Create comprehensive domain maps without moving code.

```
Real-Green-Mobile                    rg-ai-mobile-api
┌────────────────┐                  ┌────────────────┐
│ Business Logic │◄─────────────────│ Domain Maps    │
│ (stays here)   │   documents      │ (reference)    │
└────────────────┘                  └────────────────┘
```

**Pros:**
- Lowest risk, fastest to implement
- No code changes required
- Domain maps useful regardless of future direction
- Can start immediately

**Cons:**
- Doesn't solve drift problem
- Still requires manual sync
- Business logic remains scattered

**Effort:** Low (weeks)

---

### Path B: Extract to Shared Layer (Medium-term)

**What:** Extract business rules into a shared library/service accessible to both mobile and agent.

```
                    ┌────────────────────────┐
                    │  Domain Rules Service  │
                    │  (new microservice or  │
                    │   shared library)      │
                    └───────────┬────────────┘
                                │
              ┌─────────────────┼─────────────────┐
              ▼                 ▼                 ▼
     Real-Green-Mobile    rg-ai-mobile-api    Future clients
```

**Pros:**
- Single source of truth for rules
- Agent and mobile use same validation
- Rules testable independently
- Enables future clients

**Cons:**
- Significant refactoring
- Risk of breaking mobile app
- Needs careful migration strategy

**Effort:** High (months)

---

### Path C: Move Logic Server-Side (Long-term)

**What:** Progressively move business logic from mobile to backend API.

```
┌─────────────────────────────────────────────────────────────────┐
│                    RealGreen API (Enhanced)                      │
│    ┌─────────────────────────────────────────────────────────┐  │
│    │ Business Logic Layer (migrated from mobile)              │  │
│    │ ProductManager, ServiceManager, etc.                     │  │
│    └─────────────────────────────────────────────────────────┘  │
└─────────────────────────────┬───────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        Mobile App      AI Agent       Other Clients
        (thin client)
```

**Pros:**
- Clean architecture
- Agent has native access to logic
- Mobile becomes thin client
- Standard industry pattern

**Cons:**
- Major architectural change
- Multi-year effort
- Requires offline strategy for mobile
- High risk without careful planning

**Effort:** Very High (years)

---

## Recommendation: Layered Approach

**Start with Path A, plan for Path B:**

| Phase | Action | Outcome |
|-------|--------|---------|
| **Now** | Domain mapping (Path A) | Documented rules, reduced Earl dependency |
| **Q1** | Identify extraction candidates | Prioritized list of rules to extract |
| **Q2** | Pilot extraction (Path B) | One domain (Timers) in shared layer |
| **Future** | Evaluate Path C | Based on pilot learnings |

**Why this order:**
1. Domain maps are valuable regardless of architecture direction
2. Mapping reveals which rules are most critical to extract
3. Low-risk start builds confidence for bigger changes
4. Greg gets visibility into the problem scope

---

## Domain Mapping as Step 1

Regardless of architecture path, domain mapping provides:

| Benefit | For Brianna | For Greg | For AI Agent |
|---------|-------------|----------|--------------|
| **Visibility** | Knows where logic lives | Sees scope of problem | Rules to enforce |
| **Documentation** | Reduces Earl dependency | Architecture audit trail | Training data |
| **Prioritization** | Focuses effort | Informs refactor order | Risk assessment |

**Key insight:** You can't extract or move what you haven't mapped.

---

## Questions for Greg

1. **Strategic direction**: Is moving logic server-side on the roadmap? Timeline?
2. **Extraction appetite**: Is a shared rules layer feasible given team capacity?
3. **Mobile constraints**: What offline requirements prevent server-side logic?
4. **Priority domains**: Which business logic areas cause the most pain?
5. **Agent scope**: Should agent enforce all mobile rules, or just critical ones?

---

## Risk Matrix

| Path | Risk | Mitigation |
|------|------|------------|
| A (Document) | Drift between docs and code | Automated staleness detection |
| A (Document) | Incomplete coverage | Expert validation sessions |
| B (Extract) | Breaking mobile | Feature flags, gradual rollout |
| B (Extract) | Team bandwidth | Start with one domain |
| C (Move) | Offline mobile support | Evaluate after Path B pilot |
| C (Move) | Multi-year commitment | Decision gate after Phase 1 |

---

## Connection to Agent Architecture

The current agent (Jerry) uses NeMo Guardrails for safety. The same pattern could enforce domain rules:

```python
# Current: Safety guardrails
rails.register_action(check_input_guardrails, "check_input_guardrails")

# Future: Domain guardrails (from domain maps)
rails.register_action(validate_timer_start, "validate_timer_start")
```

This means domain maps aren't just documentation - they become executable validation if we reach Path B.

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