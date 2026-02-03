---
workflow_id: WW-RG-004
priority: P3
pilot_readiness: needs_prep
champion: Brianna Krisher
team: RG Mobile
---

# WW-RG-004: Domain Logic Discovery (Timers, Services, Customer Data)

## Workflow Title
Domain Logic Discovery: AI + Expert → Reusable Domain Map

## Team / Domain
RG Mobile Team / Cross-functional (Agent Tools + App Logic)

## SDLC Stage
Requirements / Design

## Workflow Description (Process Today)
1. Receive request to add agent capability (e.g., "agent should be able to stop timers")
2. **Unknown territory**: Brianna doesn't know how timers work in Real Green (came from PesPac)
3. Use Copilot in repo: "How do timers work in Real Green?" — some luck, incomplete answers
4. Manually explore app code: find ProductManager, ServiceManager classes
5. Ask Gemini: paste code + "What's the logic for stopping a timer?"
6. Gemini provides partial answer (misses edge cases)
7. Ask Real Green expert (Earl): "What about product entries? What if service isn't startable?"
8. Earl provides critical rules that AI missed
9. Update tool implementation with complete logic
10. Repeat for each new domain area (notes, customer history, jobs, etc.)

## Greg's summary
[4:00 PM] in general, how we are approaching the agent in the mobile architecture is a problem
[4:00 PM] we have soo much business logic in the mobile app
[4:01 PM] and very little logic server side
[4:02 PM] this is the reason I asked for you to reach out to me when we run into these issues

## Friction (Where Pain Is)
- **Expert bottleneck**: "A lot of it, unfortunately, is in our real green expert's head"
- **Scattered knowledge**: Logic is in app code, database structure, API, and tribal knowledge
- **AI misses edge cases**: "Me and Gemini missed" rules like product entry deletion on timer stop
- **Cross-product learning curve**: Each product (Real Green, PestPac) has different patterns
- **No domain documentation**: "We used to create documentation... during Skunk Works we haven't"
- **Time sink**: "When creating a new tool, that's most of my day — getting all that information"

## Metric (Baseline → Target)
- **Domain discovery time per feature**: ~1-2 days → ~2-4 hours (with domain map)
- **Expert consultation frequency**: 3-5 questions per feature → 0-1 (with captured rules)
- **Rework from missed edge cases**: 1-2 per feature → 0 (with validation rules)

## AI Solution Idea (How to Improve)
- Create **Domain Map** for each major entity:
  - Entity overview (what it is, where data lives)
  - Key code entry points (files/classes/functions)
  - Business rules / edge cases (from Earl)
  - Known pitfalls
  - Acceptance criteria format
- Build **domain context pack** that can be fed to Claude/Copilot:
  - Database schema for entity
  - Relevant app code snippets
  - Business rules as comments/annotations
- Use Claude Code to **generate domain map draft** from codebase, then validate with Earl
- Store domain maps in Confluence, reference in Jira stories
- Future: MCP server with database schema + app architecture for real-time queries

## Dependencies / Impediments
- Earl's availability (he's "doing 100 other things")
- Access to Real Green database schema
- App code is complex with heavy app-side logic (not just API calls)
- No single source of truth for business rules

## Complexity
**L** (Large) — Requires extraction from multiple sources + expert validation; high value

## Impact
**High** — "That probably takes a couple days, honestly" for each new domain; multiplies across every new feature

## Next Steps
1. **Pick one domain**: Start with "Timers" (already partially explored)
2. **Extract**: Code entry points, Earl's rules, edge cases discovered during implementation
3. **Draft domain map**: Generate with Claude Code from codebase, annotate with business rules
4. **Validate**: Review with Earl, capture missed rules
