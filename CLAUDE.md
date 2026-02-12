# WW-RG-004: Domain Discovery Workflow

Claude Code workflow for discovering and documenting business logic in RealGreen Mobile.

## Quick Start

```bash
# In Real-Green-Mobile project:
/discover-domain timers

# Validate with domain expert:
/validate-domain timers
```

## Problem

Developers spend 1-2 days discovering domain logic when adding AI agent capabilities:
- Business logic scattered in mobile C# (not server-side)
- Tribal knowledge in Earl's head
- No domain documentation exists

## Target Projects

| Project | Role |
|---------|------|
| `Real-Green-Mobile` | C# mobile app with 39 domain managers |
| `rg-ai-mobile-api` | Python AI assistant "Jerry" |

## Commands

| Command | Purpose |
|---------|---------|
| `/onboarding` | Interactive walkthrough of the entire workflow |
| `/discover-domain [entity]` | Generate domain map from codebase |
| `/validate-domain [entity]` | Interactive expert validation |
| `/check-domain-staleness` | Find outdated domain maps |

## Workflow

See `docs/playbook.md` for step-by-step execution.

```
1. Discover → Find managers, entry points, call chains
2. Trace → Follow business logic, capture rules
3. Generate → Create domain map draft
4. Validate → Review with domain expert (Earl)
```

## Documentation

### Getting Started
| Document | Purpose |
|----------|---------|
| [playbook.md](docs/playbook.md) | Step-by-step workflow with quality gates |
| [06-system-primer.md](docs/06-system-primer.md) | Quick onboarding guide |
| [07-interaction-flows.md](docs/07-interaction-flows.md) | Mobile ↔ Jerry sequence diagrams |

### Architecture
| Document | Purpose |
|----------|---------|
| [04-system-overview.md](docs/04-system-overview.md) | End-to-end architecture with Mermaid diagrams |
| [01-domain-discovery-tooling.md](docs/01-domain-discovery-tooling.md) | `/discover-domain` command design |

### Analysis & Decisions
| Document | Purpose |
|----------|---------|
| [02-architecture-recommendation.md](docs/02-architecture-recommendation.md) | Paths forward for business logic |
| [03-rag-feasibility-analysis.md](docs/03-rag-feasibility-analysis.md) | RAG vs alternatives assessment |
| [05-workflow-reuse-analysis.md](docs/05-workflow-reuse-analysis.md) | Reuse from brownfield workflow (~75%) |

### Templates
| Document | Purpose |
|----------|---------|
| [templates/domain-map.md](templates/domain-map.md) | Domain map template |

## Reuses

Built on **ww-mi-004-brownfield-documentation-generation-workflow**:
- Entry-point anchored discovery
- Interrogative Phase 0
- Quality gates and HITL review

See [05-workflow-reuse-analysis.md](docs/05-workflow-reuse-analysis.md) for details.
