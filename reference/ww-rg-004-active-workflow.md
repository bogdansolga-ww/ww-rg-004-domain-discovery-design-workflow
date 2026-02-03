# WW-RG-004: Domain Logic Discovery

**Champion:** Brianna Krisher
**Team:** RG Mobile
**Priority:** P3
**Complexity:** L-XL
**SDLC Stage:** Discovery/Design

---

## Description

### Problem

Domain knowledge lives in expert heads, scattered code, and tribal knowledge. Brianna noted: "A lot of it, unfortunately, is in our Real Green expert's head" — when adding new agent capabilities, most of the day is spent discovering how things work rather than building.

### Current State

1. Receive request to add agent capability (e.g., "agent should stop timers")
2. **Unknown territory**: Don't know how timers work (came from different product)
3. Use Copilot in repo: "How do timers work?" — incomplete answers
4. Manually explore app code: find ProductManager, ServiceManager classes
5. Ask Gemini: paste code + "What's the logic for stopping a timer?"
6. Gemini provides partial answer (misses edge cases)
7. Ask Real Green expert (Earl): "What about product entries? What if service isn't startable?"
8. Earl provides critical rules AI missed
9. Update implementation with complete logic
10. Repeat for each new domain area (notes, customer history, jobs, etc.)

**Key friction points:**
- Expert bottleneck — Earl is "doing 100 other things"
- Scattered knowledge — logic in app code, database, API, and tribal knowledge
- AI misses edge cases — "Me and Gemini missed" critical business rules
- Cross-product learning curve — Real Green, PesPac, PestPac all different
- No domain documentation — "We used to create documentation... during Skunk Works we haven't"
- Time sink — "When creating a new tool, that's most of my day"

### Proposed Solution

AI-assisted **Domain Map** generation from codebase + expert validation:

- **Input**: Entity name (e.g., "Timers") + codebase access
- **Output**: Domain Map with entry points, business rules, edge cases, pitfalls
- Build **domain context pack** for Claude/Copilot (schema + code + rules)
- Use Claude Code to **draft domain map from code**, then validate with Earl
- Store maps in Confluence, reference in Jira stories
- Future: MCP server with database schema + app architecture for real-time queries

Brianna's goal: "Get all that information in 2-4 hours instead of 1-2 days"

---

## Acceptance Criteria

- [ ] Domain Map template created (entity overview, code entry points, business rules, pitfalls)
- [ ] Pilot with one domain (Timers — already partially explored)
- [ ] Claude Code generates draft domain map from codebase
- [ ] Expert (Earl) validates and adds missed rules
- [ ] Baseline vs AI-assisted discovery time measured
- [ ] Edge cases AI found vs missed documented
- [ ] Domain map stored in accessible location (Confluence)
- [ ] Workflow documented for team replication

---

## Technical Approach

### Collect Artifacts

- Real Green database schema for target entity
- Relevant app code (ProductManager, ServiceManager classes)
- Known business rules from previous Earl conversations
- Edge cases discovered during past implementations

### Build Domain Map Template

Structure:
```
## Entity: [Name]
### Overview
What it is, where data lives

### Code Entry Points
Files, classes, functions

### Business Rules
From expert (Earl)

### Edge Cases / Pitfalls
Discovered during implementation

### Acceptance Criteria Format
How to validate new features against this domain
```

### Pilot Execution

1. Select domain: **Timers** (already partially explored)
2. Generate draft with Claude Code from codebase
3. Review with Earl, capture missed rules
4. Compare AI draft vs final validated version

### Document & Package

- Capture before/after metrics
- Create reusable domain map template
- Document expert validation process

---

## Success Metrics

| Metric | Baseline | Target | Measurement Method |
|--------|----------|--------|-------------------|
| Domain discovery time per feature | 1-2 days | 2-4 hours | Stopwatch from request to complete understanding |
| Expert consultation frequency | 3-5 questions per feature | 0-1 questions | Count Earl interruptions |
| Rework from missed edge cases | 1-2 per feature | 0 | Count post-implementation fixes |
| Domain map completeness | None | 80%+ from AI draft | Compare AI draft to final validated version |

---

## Implementation Tasks

### Preparation (GenAI Adopter)

- [ ] Document domain map template structure
- [ ] Identify code entry points for Timers domain
- [ ] Collect known business rules from past conversations
- [ ] Create initial prompt for domain extraction
- [ ] Test prompt with Timers codebase

### Pilot Execution (with Brianna + Earl)

- [ ] Generate draft domain map with Claude Code
- [ ] Review draft with Brianna (first pass)
- [ ] Validate with Earl (capture missed rules)
- [ ] Measure time and completeness metrics

### Post-Pilot

- [ ] Finalize domain map template
- [ ] Document AI vs expert contribution ratio
- [ ] Create breakthrough story
- [ ] Identify next domain to map (Notes, Customer History, Jobs)

---

## Dependencies

| Dependency | Status | Notes |
|------------|--------|-------|
| Real Green codebase access | ✅ Available | Brianna has access |
| Database schema | ⏳ Needed | Request from Earl or DBA |
| Earl's availability | ⚠️ Constrained | "Doing 100 other things" — schedule validation session |
| Previous edge case notes | ⏳ Needed | Collect from Brianna's implementation history |
| Domain map template | ⏳ Needed | Create during prep |
