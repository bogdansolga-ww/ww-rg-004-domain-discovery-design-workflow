---
command: onboarding
description: Walk through the Domain Discovery workflow step by step
creation-date: 2026-02-11
last-update: 2026-02-11
---

# Domain Discovery — Onboarding

Welcome the user to the Domain Discovery workflow and walk them through the entire process interactively.

## Instructions

### Step 1: Welcome & Explain

Present this overview:

"**Welcome to the Domain Discovery Workflow!**

This tool helps you extract and document domain knowledge from the RealGreen Mobile codebase so that AI agent developers (and anyone else) can understand business logic without spending days in the code or bottlenecking on domain experts.

It works by:

1. **Discovering** the managers, entry points, and AI tool connections for a business entity
2. **Tracing** call chains to extract business rules, side effects, and edge cases
3. **Generating** a structured Domain Map with diagrams, rules, and acceptance criteria
4. **Validating** the map with a domain expert (Earl) to confirm or correct inferred rules

The goal: reduce domain discovery from **1-2 days to 2-4 hours**.

**How it works:**

```
Business entity (e.g., 'timers')
      │
      ▼
  Discover managers & entry points
      │
      ▼
  Trace call chains & extract business rules
      │
      ▼
  Generate Domain Map
  ┌─────────────────────────────────────────┐
  │ Business rules table (with [?] flags)   │
  │ State machine diagram                   │
  │ Data flow diagram                       │
  │ Side effects, edge cases, pitfalls      │
  │ Acceptance criteria template            │
  └─────────────────────────────────────────┘
      │
      ▼
  Validate with domain expert
  (confirm rules, resolve [?] flags)
```

Every business rule extracted from code is marked with a validation status:
- `[?]` — Needs expert confirmation
- `[inferred]` — Inferred from code patterns, needs validation
- `[validated]` — Confirmed by domain expert with date"

### Step 2: Check Prerequisites

Ask the user these questions one at a time. Wait for each answer before proceeding.

**Question 1:** "Which **business entity/domain** do you want to map?

For example:
- `timers` — Production timer tracking (start/stop/pause for services)
- `time-clock` — Employee clock in/out with break lockout
- `job-route-listing` — Daily job routes and scheduling
- `customers` — Customer/property management
- `services` — Service execution and completion

What domain are you working with?"

If the user isn't sure, explain that a domain typically maps to a Manager class in `RG.Mobile.Core/Managers/` (there are 39 of them), and offer to help identify the right one.

**Question 2:** "Do you have the **Real-Green-Mobile** repo cloned locally?

This is the Xamarin C# app where the business logic lives — specifically in the Manager classes under `RG.Mobile.Core/Managers/`.

If yes, what is its local path?"

If the user doesn't have it, explain they'll need it for discovery and offer to continue with a dry walkthrough using the existing example domain maps.

**Question 3:** "Do you also have the **rg-ai-mobile-api** repo (Jerry's Python backend)?

This is optional but useful — it lets the workflow map how the AI agent connects to the business logic you're documenting (signals, tool dispatcher, etc.).

If yes, what is its local path?"

### Step 3: Explain Key Concepts

Present this framework:

"The workflow is built around understanding how business logic flows in the RealGreen Mobile architecture:

**The Architecture:**
```
Jerry (AI Agent, Python)
    │ sends signals like <discard>
    ▼
rg-ai-mobile-api (FastAPI)
    │ routes to tool
    ▼
Real-Green-Mobile (Xamarin C#)
    │ ToolDispatcherService → Manager → Proxy → API
    ▼
Business logic executes on device
```

**Key insight:** Jerry doesn't execute business logic — it signals the mobile app, which does the work. So the business rules live in the C# Manager classes, not in Python.

**What the workflow discovers:**

| # | What | Where to find it | Example |
|---|------|-----------------|---------|
| 1 | **Entry points** | Manager public methods, event handlers | `SetTimeInAsync()`, `SetTimeOutAsync()` |
| 2 | **Business rules** | Validation, calculations, state transitions | 'Cannot time out if not timed in' |
| 3 | **State machine** | Status enums, state transition logic | NotStarted → InProgress → Completed |
| 4 | **Side effects** | DB writes, API calls, events published | Posts production entry, queues sync |
| 5 | **Edge cases** | Error handling, boundary conditions | Timer already running, offline mode |
| 6 | **AI tool mapping** | ToolDispatcherService, Jerry signals | `request_production_timer_toggle` |

Everything uncertain gets flagged with `[?]` for expert validation."

### Step 4: Present Available Commands

"Here's how to run the workflow:

---

**Option A: Full discovery (recommended)**

Run the discovery command that handles phases 1-3:

```
/discover-domain <entity>
```

For example:
```
/discover-domain timers
```

This will:
1. Find the primary Manager class and related managers
2. List all entry points (public methods, event handlers)
3. Map the AI tool connection (if rg-ai-mobile-api is available)
4. Trace call chains and extract business rules
5. Generate a Domain Map at `docs/domains/<entity>.md`

---

**Option B: Validate with expert**

After discovery, review the domain map with your domain expert:

```
/validate-domain <entity>
```

This starts an interactive session where you:
1. Walk through each `[?]` flagged item
2. Confirm or correct inferred business rules
3. Add missing edge cases or rules
4. Mark validated items with `[validated]` + date

---

**Option C: Check for staleness**

If domain maps already exist and you want to check if code has changed:

```
/check-domain-staleness
```

This compares domain map dates against git history to find maps that may need updating.

---

**Typical flow:**
```
/discover-domain timers          # Generate the map
# ... review the output ...
/validate-domain timers          # Validate with Earl
# ... later, after code changes ...
/check-domain-staleness          # Check if maps need updating
```

Which would you like to start with?"

### Step 5: Guide Through Execution

Based on the user's choice:

**If discovery:**
- Confirm the entity name and repo paths
- Explain what to look for in the output:
  - "Check that the primary Manager is correctly identified"
  - "Verify the entry points list covers all the methods you'd expect"
  - "Review business rules — items marked `[?]` are uncertain and need expert input"
  - "Check that diagrams render correctly (state machine, data flow)"

**If validation:**
- Confirm they have a domain map already generated
- Explain the validation session:
  - "We'll go through each `[?]` item and you'll mark it as confirmed, corrected, or needing more info"
  - "Bring your domain expert (Earl) or have their input ready"
  - "The goal is to resolve every `[?]` flag"

Then ask: "Ready to start? I'll run the command now."

If the user says yes and has provided the entity name and paths, execute the appropriate command.

If the user chose a dry walkthrough, walk through the existing domain maps in `docs/domains/` (production-timers, time-clock, job-route-listing) as examples.

### Step 6: Explain the Domain Map Output

"The Domain Map is a single markdown file with 12 sections:

| # | Section | What it captures |
|---|---------|-----------------|
| 1 | **Overview** | Entity name, status (Draft/Validated), primary manager |
| 2 | **Entry Points** | Table of file, method, trigger, purpose |
| 3 | **Business Rules** | Table with ID, rule description, source, validation status |
| 4 | **Rule Details** | Expanded descriptions with code references |
| 5 | **State Machine** | Mermaid diagram of status transitions |
| 6 | **Data Flow** | Mermaid diagram of data movement |
| 7 | **Side Effects** | Actions triggered (DB writes, API calls, events) |
| 8 | **Edge Cases** | Scenario + expected behavior + source |
| 9 | **Dependencies** | Internal managers, proxies, entities used |
| 10 | **Pitfalls** | Gotchas and warnings for implementers |
| 11 | **Acceptance Criteria** | Checkbox template for testing |
| 12 | **Changelog** | Date, change, author tracking |

The map is designed to be self-contained — an AI agent developer should be able to implement a tool correctly by reading just this document."

### Step 7: Post-Generation Guidance

After the domain map is generated (or if doing a dry walkthrough):

"Your domain map is ready! Here's what to do next:

1. **Review the map** — Open `docs/domains/<entity>.md` and read through all sections
2. **Check `[?]` items** — These are uncertain and need domain expert input
3. **Schedule validation** — Run `/validate-domain <entity>` with Earl or your domain expert
4. **Resolve all flags** — Every `[?]` should become `[validated]` or be removed
5. **Update status** — Change 'Status: Draft' to 'Status: Validated' once confirmed
6. **Use the map** — AI agent developers can now implement tools using the domain map as reference

**Priority domains for validation** (if you're unsure where to start):
1. **Production Timers** — Most complex business logic, multiple edge cases
2. **Time Clock** — Clear rules with break lockout logic
3. **Job/Route Listing** — Foundation for other tools

**Tip:** Domain maps are living documents. When code changes, run `/check-domain-staleness` to find maps that need updating."

### Step 8: Offer Next Steps

"Anything else I can help with?

- **Discover a domain now** — if you have an entity in mind
- **List available managers** — I can scan `RG.Mobile.Core/Managers/` to show all 39 domains
- **Review an existing domain map** — if you want to discuss one in `docs/domains/`
- **Explain the RealGreen architecture** — see `docs/06-system-primer.md` for a quick overview
- **Check the glossary** — `docs/glossary.md` has all entity definitions and relationships"

Wait for the user's response and help accordingly.

## Tone

- Be welcoming and conversational, not robotic
- Use concrete examples over abstract descriptions
- If the user seems experienced, skip the basics and move quickly
- If the user seems new, take time to explain each concept
- Always confirm before running commands that scan external repos
