# RG Mobile AI API: Guardrails Implementation & Domain Logic Discovery

> Analysis connecting the current guardrails implementation in `rg-ai-mobile-api` to the pain points identified in WW-RG-004 (Domain Logic Discovery workflow).

---

## Executive Summary

The RG Mobile AI API has a robust **safety guardrails** implementation using NeMo Guardrails + OpenAI Moderation API. However, the same architectural pattern could address Brianna's core pain point: **domain business logic is scattered and undocumented**.

**Key Insight:** The guardrails system is designed to enforce rules and validate AI behavior. Extending this pattern to include **domain logic rules** (timers, services, customer data) could:
1. Codify tribal knowledge from Earl
2. Prevent AI from making domain-specific mistakes
3. Create a living, executable domain map

---

## Current Guardrails Implementation

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    User Input / AI Output                        │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │   NeMo Guardrails (LLMRails)   │
                    │   ┌─────────────────────┐     │
                    │   │  Input Rails Flow   │     │
                    │   └─────────────────────┘     │
                    │   ┌─────────────────────┐     │
                    │   │  Output Rails Flow  │     │
                    │   └─────────────────────┘     │
                    └─────────────┬─────────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Pattern Matching │  │ OpenAI Moderation│  │   PII Detection  │
│  (jailbreak,     │  │       API        │  │  (SSN, CC, email)│
│   sensitive)     │  │                  │  │                  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

### What's Currently Protected

| Guardrail Type | Implementation | Location |
|----------------|----------------|----------|
| **Jailbreak Detection** | Pattern matching + perplexity thresholds | `guardrails.py:163-185` |
| **Sensitive Info Requests** | Pattern matching (credit card, SSN, password, API key) | `guardrails.py:188-209` |
| **PII in Output** | Regex patterns (SSN, CC, API keys) | `guardrails.py:235-257` |
| **Harmful Content** | Pattern matching (explosives, hacking) | `guardrails.py:260-279` |
| **PII in Input** | Regex detection (SSN, CC, email, phone) | `guardrails.py:305-315` |
| **Content Moderation** | OpenAI Moderation API | `content_moderation.py:69-110` |

### Key Code Patterns

**Colang Flow Definition:**
```colang
define flow check input
  user ...
  $allowed = execute check_input_guardrails
  if not $allowed
    bot refuse request
    stop
```

**Custom Action Registration:**
```python
rails.register_action(check_input_guardrails, "check_input_guardrails")
rails.register_action(check_output_guardrails, "check_output_guardrails")
rails.register_action(detect_pii_in_input, "detect_pii_in_input")
```

---

## Pain Points from WW-RG-004

From Brianna's Domain Logic Discovery workflow:

| Pain Point | Quote / Evidence |
|------------|------------------|
| **Expert bottleneck** | "A lot of it, unfortunately, is in our real green expert's head" |
| **Scattered knowledge** | Logic is in app code, database structure, API, and tribal knowledge |
| **AI misses edge cases** | "Me and Gemini missed" rules like product entry deletion on timer stop |
| **No domain documentation** | "We used to create documentation... during Skunk Works we haven't" |
| **Time sink** | "When creating a new tool, that's most of my day — getting all that information" |

### The Core Problem

When adding new agent capabilities (like "stop timers"), Brianna must:
1. Search code with incomplete AI answers
2. Consult Earl for edge cases AI missed
3. Discover rules like: *"When stopping a timer, product entries must be deleted"*
4. Repeat for every new domain area

**This is exactly what guardrails are designed to solve — but for safety, not domain logic.**

---

## Opportunity: Domain Logic Guardrails

### Concept

Extend the guardrails pattern to include **domain business rules**:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Agent Tool Call                               │
│    request_production_timer_toggle(cust_no, service_ids)        │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │   Domain Logic Rails      │
                    │   (NEW - extends NeMo)    │
                    └─────────────┬─────────────┘
                                  │
          ┌───────────────────────┼───────────────────────┐
          ▼                       ▼                       ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  Timer Rules     │  │  Service Rules   │  │  Customer Rules  │
│  (from Earl)     │  │  (from codebase) │  │  (from DB schema)│
└──────────────────┘  └──────────────────┘  └──────────────────┘
```

### Example: Timer Domain Rules

Based on the pain point "product entry deletion on timer stop", a domain guardrail could look like:

```python
# domain_guardrails.py

TIMER_DOMAIN_RULES = """
# Timer Business Rules (Source: Earl, validated 2026-01)

define tool request_production_timer_toggle
  # Rule 1: Service must be startable
  "Timer can only start for services in STARTABLE state"

  # Rule 2: Product entries on stop
  "When timer stops, all associated product entries MUST be deleted"

  # Rule 3: Single active timer
  "Only one timer can be active per technician at a time"

  # Rule 4: Clock-in requirement
  "Technician must be clocked in before starting any production timer"

define flow validate_timer_action
  tool request_production_timer_toggle ...

  $is_clocked_in = execute check_technician_clocked_in
  if not $is_clocked_in
    bot "You need to clock in before starting a timer. Would you like me to clock you in first?"
    stop

  $service_startable = execute check_service_startable
  if not $service_startable
    bot "That service isn't available to start right now. Let me check what services are ready."
    stop
"""

@action(name="check_service_startable")
async def check_service_startable(context: dict) -> bool:
    """
    Domain Rule: Service must be in STARTABLE state
    Source: Earl (RealGreen expert), validated 2026-01-15

    Edge Cases:
    - Service already completed today → not startable
    - Service has active timer elsewhere → not startable
    - Service missing required products → not startable
    """
    service_ids = context.get("service_ids", [])
    cust_no = context.get("cust_no")

    # Implementation would check against app state
    # For now, this documents the rule
    return True
```

### Benefits

| Current State | With Domain Guardrails |
|---------------|------------------------|
| Rules in Earl's head | Rules codified in Python/Colang |
| AI discovers edge cases by failing | AI prevented from invalid actions |
| Documentation separate from code | Rules ARE the documentation |
| Manual validation with Earl | Automated validation on every call |
| Knowledge lost when Earl leaves | Knowledge persists in codebase |

---

## Proposed Domain Map Structure

For each major entity, create a domain guardrail module:

```
rg_mobile_ai_api/
├── guardrails/
│   ├── guardrails.py           # Safety guardrails (existing)
│   ├── content_moderation.py   # OpenAI moderation (existing)
│   └── domain/                  # NEW: Domain logic guardrails
│       ├── __init__.py
│       ├── timers.py           # Timer business rules
│       ├── services.py         # Service/job rules
│       ├── customers.py        # Customer data rules
│       ├── products.py         # Product/inventory rules
│       └── clock.py            # Time clock rules
```

### Timer Domain Module Example

```python
# rg_mobile_ai_api/guardrails/domain/timers.py
"""
Timer Domain Logic
==================
Entity: Production Timer
Source: Earl (RealGreen Expert) + ProductManager.cs analysis
Last Validated: 2026-01-15

Key Classes:
- ProductManager (app/managers/ProductManager.cs)
- ServiceManager (app/managers/ServiceManager.cs)

Database Tables:
- production_timers
- product_entries
- services
"""

from nemoguardrails.actions import action

# Business Rules (from Earl)
TIMER_RULES = {
    "RULE_001": {
        "name": "Single Active Timer",
        "description": "Only one timer can be active per technician at a time",
        "source": "Earl, 2026-01-10",
        "code_reference": "ProductManager.cs:StartTimer()",
    },
    "RULE_002": {
        "name": "Product Entry Cleanup",
        "description": "When timer stops, delete all associated product entries",
        "source": "Earl, 2026-01-10 - CRITICAL: AI originally missed this",
        "code_reference": "ProductManager.cs:StopTimer():L245-260",
    },
    "RULE_003": {
        "name": "Clock-In Prerequisite",
        "description": "Technician must be clocked in to start production timer",
        "source": "Business requirement, validated with Earl",
        "code_reference": "App enforces in UI, agent should too",
    },
}

# Edge Cases (discovered during implementation)
EDGE_CASES = [
    {
        "scenario": "Timer started but service already completed",
        "expected": "Reject with message about service status",
        "discovered": "2026-01-12 during testing",
    },
    {
        "scenario": "Multiple services selected, only some startable",
        "expected": "Start only startable services, report others",
        "discovered": "Earl review session, 2026-01-15",
    },
]

@action(name="validate_timer_start")
async def validate_timer_start(context: dict) -> dict:
    """
    Validate timer start against domain rules

    Returns:
        {
            "valid": bool,
            "violations": [{"rule": str, "message": str}],
            "warnings": [str]
        }
    """
    violations = []
    warnings = []

    # Check RULE_003: Clock-in prerequisite
    if not context.get("user_clocked_in", False):
        violations.append({
            "rule": "RULE_003",
            "message": "Must clock in before starting production timer"
        })

    # Additional validations would go here

    return {
        "valid": len(violations) == 0,
        "violations": violations,
        "warnings": warnings
    }
```

---

## Implementation Roadmap

### Phase 1: Document Existing Rules (Week 1)

1. **Extract timer rules** from codebase + Earl session
2. **Create domain/timers.py** with documented rules
3. **Add validation actions** (can be no-op initially, just documenting)

### Phase 2: Wire into Agent (Week 2)

1. **Register domain actions** in guardrails initialization
2. **Add domain flows** to Colang config
3. **Test with existing tools** (request_production_timer_toggle)

### Phase 3: Expand to Other Domains (Weeks 3-4)

1. **Services domain** (service states, job relationships)
2. **Customer domain** (history, geolocation, data access)
3. **Products domain** (inventory, usage tracking)

### Phase 4: MCP Integration (Future)

1. **Expose domain rules** via MCP server
2. **Real-time queries** for rule validation
3. **AI-assisted rule discovery** from codebase

---

## Connection to APEX Workflow

This analysis supports the **APEX PILOT phase** for WW-RG-004:

| APEX Step | Action |
|-----------|--------|
| **CAPTURE** | Documented domain logic pain points (this analysis) |
| **PREPARE** | Design domain guardrails architecture |
| **SPOT** | Review with Brianna + Earl, select first domain (Timers) |
| **CO-IMPLEMENT** | Build `domain/timers.py` together |
| **DEMO** | Show rule validation preventing AI mistakes |
| **PACKAGE** | Reusable pattern for other domains |

---

## Metrics (Before → After)

| Metric | Before | After (Target) |
|--------|--------|----------------|
| Domain discovery time per feature | 1-2 days | 2-4 hours |
| Expert consultation frequency | 3-5 questions/feature | 0-1 |
| Rework from missed edge cases | 1-2/feature | 0 |
| Time to onboard new team member | Weeks | Days |

---

## Files Referenced

| File | Purpose |
|------|---------|
| `rg_mobile_ai_api/guardrails/guardrails.py` | Current safety guardrails implementation |
| `rg_mobile_ai_api/guardrails/content_moderation.py` | OpenAI Moderation API integration |
| `rg_mobile_ai_api/tools/tools.py` | Agent tools (timer, clock, notes) |
| `WW-RG-004` workflow | Domain Logic Discovery pain points |

---

*Document Version: 1.0*
*Created: February 2026*
*Author: GenAI Value Lab*
