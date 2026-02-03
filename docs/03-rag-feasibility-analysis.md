# RAG Feasibility Analysis

> Technical assessment of Retrieval-Augmented Generation for domain knowledge access.

**Status:** Draft
**Audience:** Greg, Brianna, Technical Leads
**Related:** WW-RG-004, rg-ai-mobile-api architecture

---

## What is RAG?

**Retrieval-Augmented Generation** combines:
1. **Retrieval**: Search a knowledge base for relevant context
2. **Generation**: LLM generates response using retrieved context

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Query                               │
│            "How do timers work in Real Green?"                   │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    1. RETRIEVAL                                  │
│    Search vector DB for relevant code, docs, rules               │
│    Returns: ProductManager.cs snippets, Earl's rules, etc.       │
└─────────────────────────────┬───────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    2. GENERATION                                 │
│    LLM generates answer using retrieved context                  │
│    "Timers are managed by ProductManager. Key rules:..."         │
└─────────────────────────────────────────────────────────────────┘
```

---

## RAG for Domain Discovery: The Promise

| Use Case | How RAG Would Help |
|----------|-------------------|
| "How do timers work?" | Retrieve relevant classes, generate explanation |
| "What are the rules for stopping a timer?" | Retrieve Earl's documented rules + code |
| "What changes when I modify ServiceManager?" | Retrieve callers, dependencies, side effects |
| Adding new agent tool | Retrieve related domain context automatically |

**Potential benefit:** Instead of Brianna manually searching, RAG provides instant domain context.

---

## Technical Feasibility Assessment

### Codebase Characteristics

| Factor | Real-Green-Mobile | rg-ai-mobile-api | RAG Implication |
|--------|-------------------|------------------|-----------------|
| **Language** | C# (Xamarin) | Python (FastAPI) | Multi-language chunking needed |
| **Size** | Large (~100k+ LOC) | Medium | Index size manageable |
| **Structure** | Managers, Services | Tools, Guardrails | Clear module boundaries |
| **Business logic** | Heavy, scattered | Light (calls mobile) | Primary indexing target |
| **Documentation** | Sparse | Moderate | Limited doc corpus |

### What Would Be Indexed

```
Knowledge Base Contents:
├── Code (primary)
│   ├── Real-Green-Mobile/RG.Mobile.Core/Managers/*.cs
│   ├── Real-Green-Mobile/RG.Mobile.Core/Services/*.cs
│   └── rg-ai-mobile-api/rg_mobile_ai_api/tools/*.py
├── Documentation (secondary)
│   ├── Domain maps (once created)
│   ├── Confluence pages
│   └── Code comments
├── Expert Knowledge (high value, sparse)
│   ├── Earl's rules (captured in sessions)
│   └── Edge cases discovered
└── Database Schema (structured)
    └── Entity relationships
```

---

## RAG Challenges for This Use Case

### Challenge 1: Code Understanding vs. Retrieval

**Problem:** RAG retrieves text chunks, but domain logic requires understanding call chains.

```csharp
// RAG might retrieve this chunk:
public void StopTimer(int timerId) {
    DeleteProductEntries(timerId);  // ← Critical rule!
    UpdateServiceStatus(...);
}
```

**But:** Brianna's question "What happens when I stop a timer?" requires:
- Following `DeleteProductEntries()` to see what it does
- Understanding *why* it's called (business rule)
- Knowing edge cases (what if no entries exist?)

**RAG limitation:** Retrieval is shallow; tracing requires depth.

---

### Challenge 2: Tribal Knowledge Gap

**Problem:** The most valuable knowledge (Earl's rules) isn't in the codebase.

| Knowledge Type | In Code? | In Docs? | In RAG? |
|----------------|----------|----------|---------|
| Class structure | Yes | Partial | Yes |
| Method signatures | Yes | No | Yes |
| Business rules | Implicit | No | **No** |
| Edge cases | Sometimes | No | **Limited** |
| Why decisions were made | No | No | **No** |

**RAG limitation:** Can't retrieve what doesn't exist in written form.

---

### Challenge 3: Cross-Language Tracing

**Problem:** Agent tools (Python) call APIs that trigger mobile logic (C#).

```
Python tool                         C# mobile
─────────────                       ─────────
request_production_timer_toggle()
        │
        └──▶ API call ──▶ ??? ──▶ ProductManager.StopTimer()
                         (gap)
```

**RAG limitation:** Harder to connect chunks across language boundaries.

---

### Challenge 4: Freshness & Drift

**Problem:** Code changes; index becomes stale.

| Event | Impact on RAG |
|-------|---------------|
| Developer adds new rule in code | Not in index until re-indexed |
| Earl shares new edge case verbally | Never in index unless captured |
| Refactor renames class | Old references in index mislead |

**RAG limitation:** Requires maintenance pipeline to stay useful.

---

## Alternatives to Full RAG

### Option 1: Claude Code with Project Context (Current)

**How it works:** Claude Code reads files on-demand, no pre-indexing.

```
User: "How do timers work?"
Claude Code: [Reads ProductManager.cs, ServiceManager.cs, traces calls]
```

**Pros:**
- Always fresh (reads current code)
- Can trace call chains
- No infrastructure needed

**Cons:**
- Context window limits
- Slow for large codebases
- No persistence of findings

**Best for:** One-off discovery, small-medium codebases

---

### Option 2: Domain Maps + Targeted Retrieval

**How it works:** Create domain maps manually, use them as context.

```
User: "How do timers work?"
System: [Loads docs/domains/timers.md as context]
LLM: [Answers from domain map + can read code for details]
```

**Pros:**
- Expert-validated content
- Business rules captured
- Lightweight infrastructure

**Cons:**
- Manual creation effort
- Can become stale
- Requires Earl sessions

**Best for:** This use case - captures tribal knowledge RAG can't index

---

### Option 3: Hybrid (Domain Maps + Code RAG)

**How it works:** RAG for code search, domain maps for rules.

```
User: "What should I check before stopping a timer?"
System:
  1. [RAG retrieves relevant code chunks]
  2. [Loads timers domain map for rules]
  3. [LLM combines both]
```

**Pros:**
- Best of both approaches
- Code search + expert knowledge
- Can evolve toward full RAG later

**Cons:**
- More complex
- Two systems to maintain
- Integration effort

**Best for:** If team has bandwidth and code search is frequent need

---

## Recommendation

### Start Without RAG

**Rationale:**
1. **Biggest gap is tribal knowledge** - RAG can't help until Earl's rules are captured
2. **Domain maps are prerequisite** - Need them for RAG or any approach
3. **Claude Code is sufficient for discovery** - Can trace code without infrastructure
4. **Lower risk** - No new systems to build/maintain

### RAG Evaluation Criteria

Consider RAG later if:

| Trigger | Indicates |
|---------|-----------|
| Domain maps complete for 5+ entities | Enough indexed content |
| Team frequently asks same questions | Retrieval would save time |
| Codebase search is bottleneck | Code RAG adds value |
| Greg wants real-time rule enforcement | Agent needs instant access |

---

## If RAG is Pursued Later

### Recommended Stack

| Component | Option | Why |
|-----------|--------|-----|
| **Vector DB** | Pinecone / Weaviate / pgvector | Managed, low ops burden |
| **Embeddings** | OpenAI text-embedding-3-large | Best quality, already have API |
| **Chunking** | Tree-sitter (code-aware) | Respects code structure |
| **Framework** | LangChain / LlamaIndex | Mature, well-documented |

### Index Strategy

```
Priority 1: Domain maps (expert knowledge)
Priority 2: Manager/Service classes (business logic)
Priority 3: Database schema (entity relationships)
Priority 4: Tool definitions (agent capabilities)
```

### Maintenance Pipeline

```
On code commit:
  1. Detect changed files
  2. Re-chunk affected modules
  3. Update vector embeddings
  4. Validate retrieval quality (sample queries)
```

---

## Cost-Benefit Summary

| Approach | Setup Cost | Maintenance | Value for Domain Discovery |
|----------|------------|-------------|---------------------------|
| Claude Code only | None | None | Medium (limited by context) |
| Domain maps only | Medium | Low | **High** (captures tribal knowledge) |
| Code RAG only | High | Medium | Medium (misses business rules) |
| Hybrid (maps + RAG) | High | Medium | Highest (if bandwidth exists) |

**Recommendation:** Domain maps only, evaluate RAG in 3-6 months.

---

## Questions for Technical Discussion

1. **Current tooling**: What does Brianna use today? (Copilot, Gemini, other?)
2. **Search frequency**: How often does team search codebase for understanding?
3. **Infrastructure appetite**: Is there capacity to maintain a RAG system?
4. **Existing knowledge bases**: Any Confluence/Wiki content worth indexing?
5. **Real-time needs**: Does the agent need instant rule access, or is batch OK?

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