# WW-RG-004: Domain Logic Discovery Workflow

A Claude Code methodology for extracting domain knowledge from legacy codebases, specifically targeting the RealGreen Mobile architecture.

## Quick Links

| Document | Description |
|----------|-------------|
| [Domain Discovery Tooling](docs/01-domain-discovery-tooling.md) | `/discover-domain` command design |
| [Architecture Recommendation](docs/02-architecture-recommendation.md) | Business logic placement analysis |
| [RAG Feasibility](docs/03-rag-feasibility-analysis.md) | RAG vs alternatives assessment |

## The Problem

When adding new AI agent capabilities, developers spend 1-2 days discovering how domain logic works:

```
Current Workflow (Pain):
1. Use Copilot → incomplete answers
2. Explore code manually → time sink
3. Ask Gemini → misses edge cases
4. Consult expert (Earl) → bottleneck
5. Rework after discovering missed rules
```

**Target:** Reduce domain discovery from 1-2 days to 2-4 hours.

## The Solution

A Claude Code workflow that generates **Domain Maps** - structured documentation combining:
- Code entry points (from codebase analysis)
- Business rules (from expert validation)
- Edge cases (from implementation history)

## Target Projects

- **Real-Green-Mobile** - C# Xamarin app with heavy business logic
- **rg-ai-mobile-api** - Python FastAPI AI agent (Jerry)

## Status

**Phase:** Design / Discussion Prep
**Next:** Meeting with Brianna & Greg to validate approach

## Repository Structure

```
├── docs/               # Design documents
├── reference/          # Source materials
├── templates/          # Domain map templates (TBD)
└── .claude/commands/   # Claude Code commands (TBD)
```

## Getting Started

1. Read the [CLAUDE.md](CLAUDE.md) for full context
2. Review the 3 docs in `docs/` folder
3. Fill action points after Brianna/Greg discussion