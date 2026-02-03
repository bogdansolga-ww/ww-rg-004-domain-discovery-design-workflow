# CLAUDE.md - Domain Discovery Design Workflow

This repository contains the design and implementation of a Claude Code workflow for domain logic discovery, targeting the RealGreen Mobile team.

## Project Context

**Workflow ID:** WW-RG-004
**Champion:** Brianna Krisher
**Team:** RG Mobile
**Stakeholders:** Greg (Chief Architect), Earl (Domain Expert)

## The Problem

Developers spend 1-2 days discovering domain logic when adding new AI agent capabilities:
- Business logic is scattered in mobile C# code (not server-side)
- Tribal knowledge lives in Earl's head
- AI tools (Copilot, Gemini) miss edge cases
- No domain documentation exists

**Greg's summary:** "In general, how we are approaching the agent in the mobile architecture is a problem. We have soo much business logic in the mobile app, and very little logic server side."

## Session Summary (Feb 3, 2026)

### What We Did

1. **Analyzed the current state** via `/git:catchup` - found APEX commands work on `apex-commands` branch
2. **Located correlation document** linking the domain discovery workflow to `rg-ai-mobile-api`
3. **Created this project folder** with 3 discussion documents for Brianna/Greg meeting

### Documents Created

| Document | Purpose |
|----------|---------|
| `docs/01-domain-discovery-tooling.md` | Claude Code `/discover-domain` command design |
| `docs/02-architecture-recommendation.md` | Analysis of "business logic in mobile" with 3 paths forward |
| `docs/03-rag-feasibility-analysis.md` | Technical assessment of RAG vs alternatives |
| `docs/04-system-overview.md` | End-to-end architecture with Mermaid diagrams |

### Key Recommendations

1. **Start with domain maps** (Path A) - document without moving code
2. **Skip RAG initially** - tribal knowledge gap is the blocker, not retrieval
3. **Plan for extraction** (Path B) - use maps to identify candidates for shared layer
4. **Domain maps are prerequisite** for any architecture path

### Reference Files Copied

- `reference/04-domain-logic-discovery-flow.md` - Source workflow from CAPTURE phase
- `reference/rg-mobile-guardrails-domain-logic.md` - Guardrails analysis connecting to rg-ai-mobile-api
- `reference/ww-rg-004-active-workflow.md` - Active workflow README

## Target Projects

| Project | Path | Purpose |
|---------|------|---------|
| `Real-Green-Mobile` | `../Real-Green-Mobile` | C# mobile app with heavy business logic |
| `rg-ai-mobile-api` | `../rg-ai-mobile-api` | Python AI agent API (Jerry) |

## Next Steps

1. **Discussion with Brianna & Greg** - Review 3 documents, fill action points
2. **Manual pilot** - Generate domain map for "Timers" using Claude Code
3. **Validate with Earl** - Capture missed business rules
4. **Command development** - Create `/discover-domain` command
5. **Workflow documentation** - Full methodology like brownfield-documentation-generation-workflow

## Commands to Implement

| Command | Purpose | Status |
|---------|---------|--------|
| `/discover-domain [entity]` | Generate domain map draft from codebase | Planned |
| `/validate-domain [entity]` | Interactive expert validation session | Planned |
| `/check-domain-staleness` | Detect outdated domain maps | Planned |

## Related Projects

- `../brownfield-documentation-generation-workflow` - Pattern reference for methodology repos
- `../workwave-claude-config` - APEX commands repo (apex-commands branch)
- `../workwave-artifacts` - Pilot artifacts and session notes

## File Structure

```
ww-rg-004-domain-discovery-design/
├── CLAUDE.md                 # This file
├── README.md                 # Project overview (to create)
├── docs/
│   ├── 01-domain-discovery-tooling.md
│   ├── 02-architecture-recommendation.md
│   ├── 03-rag-feasibility-analysis.md
│   └── 04-system-overview.md
├── reference/                # Source documents
│   ├── 04-domain-logic-discovery-flow.md
│   ├── rg-mobile-guardrails-domain-logic.md
│   └── ww-rg-004-active-workflow.md
├── templates/                # Domain map templates (to create)
└── .claude/
    └── commands/             # Claude Code commands (to create)
```

## Working with This Project

### To Continue Design Work

```bash
cd /Volumes/NVMe/Development/IdeaProjects/n-ix/Accounts/WorkWave/ww-rg-004-domain-discovery-design
claude
```

### To Run Manual Pilot

1. Open Claude Code in `Real-Green-Mobile` project
2. Ask: "Generate a domain map for Timers starting from ProductManager.cs"
3. Review output, note gaps for Earl validation

### To Create Commands

Use `brownfield-documentation-generation-workflow` as pattern:
- Commands go in `.claude/commands/`
- Templates go in `templates/`
- Methodology docs go in `docs/`