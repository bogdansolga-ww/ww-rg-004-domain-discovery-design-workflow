# RealGreen System Primer

Quick reference for understanding the RealGreen mobile ecosystem.

---

## What It Does

**RealGreen Mobile** enables lawn care/pest control technicians to:
- View daily job routes
- Track time on services (clock in/out, production timers)
- Record work completed (products used, conditions found)
- Capture signatures and payments
- Get AI assistance via "Jerry"

---

## The Two Apps

```
┌─────────────────────────┐     ┌─────────────────────────┐
│   Real-Green-Mobile     │     │    rg-ai-mobile-api     │
│   (C# / .NET MAUI)      │◄───►│    (Python / FastAPI)   │
├─────────────────────────┤     ├─────────────────────────┤
│ • Field technician UI   │     │ • "Jerry" AI assistant  │
│ • 39 domain managers    │     │ • 10 text tools         │
│ • Business logic (70%)  │     │ • Voice support         │
│ • Offline-capable       │     │ • GPT-4o powered        │
└─────────────────────────┘     └─────────────────────────┘
         │                                │
         ▼                                ▼
┌─────────────────────────┐     ┌─────────────────────────┐
│   RealGreen Backend     │     │      OpenAI API         │
│   (mlapi.realgreen.com) │     │                         │
└─────────────────────────┘     └─────────────────────────┘
```

---

## How They Work Together

**User says:** "Hey Jerry, start my timer"

```
1. Mobile App → sends context (screen, job, user state) → Jerry API
2. Jerry API → reasons with GPT-4o → decides to call tool
3. Jerry API → returns signal "<discard>" → Mobile App
4. Mobile App → ServiceTimersManager executes business logic
5. Mobile App → Proxy calls backend API → Database updated
```

**Key insight:** Jerry doesn't execute business logic. It sends signals. The mobile app does the work.

---

## Where Business Logic Lives

| Location | What's There | Example |
|----------|--------------|---------|
| **Mobile App (70%)** | Pricing, validation, workflows | Timer duration × crew size × rate |
| **Backend API (20%)** | Data persistence, auth | Save production entry to DB |
| **Jerry API (10%)** | Intent parsing, tool selection | "start timer" → `request_production_timer_toggle` |

**This is the problem:** AI developers need to understand C# mobile code to build correct tools.

---

## Key Components in Mobile App

| Component | Count | Purpose |
|-----------|-------|---------|
| **Managers** | 39 | Business logic orchestration |
| **Proxies** | 27 | API client wrappers |
| **Entities** | 50+ | Data models (mirror backend DB) |
| **ViewModels** | 18 | UI presentation logic |

**For domain discovery, focus on Managers** — that's where business rules live.

---

## Jerry's Tools (AI Capabilities)

| Tool | What It Does | Triggers In Mobile |
|------|--------------|-------------------|
| `request_production_timer_toggle` | Start/stop service timer | ServiceTimersManager |
| `request_time_clock_toggle_tool` | Clock in/out | TimeClockManager |
| `get_job_program_details_tool` | Fetch job info | ServiceManager |
| `add_job_note` | Add note to job | NotesManager |
| `get_customer_history_tool` | Past work history | CustomerInfoManager |

---

## WorkWave Product Context

RealGreen is one of four WorkWave products with similar architecture:

| Product | Industry | Mobile Tech |
|---------|----------|-------------|
| **RealGreen** | Lawn care | C# (.NET MAUI) |
| WinTeam | Cleaning services | C# |
| PestPac | Pest control | C# |
| TimeGate+ | Security | React Native |

**Shared services** across products:
- SMS Shared Service (TypeScript)
- Rescheduling Shared Service (C# with AI) ← potential pattern for logic extraction

---

## Starting Points for Domain Discovery

| Domain | Start Here | AI Tool |
|--------|------------|---------|
| **Timers** | `ServiceTimersManager.cs` | `request_production_timer_toggle` |
| **Time Clock** | `TimeClockManager.cs` | `request_time_clock_toggle_tool` |
| **Jobs/Services** | `ServiceManager.cs` | `get_job_program_details_tool` |
| **Customers** | `CustomerInfoManager.cs` | `get_customer_history_tool` |
| **Products** | `ProductManager.cs` | — |
| **Payments** | `PaymentManager.cs` | — |

---

## Quick Reference

**To explore mobile business logic:**
```bash
# In Real-Green-Mobile project
/discover-domain timers
```

**To see AI tool definitions:**
```bash
# In rg-ai-mobile-api project
# Look at: rg_mobile_ai_api/tools/tools.py
```

**To understand the connection:**
- Jerry tool returns `"<discard>"` signal
- Mobile app receives signal, calls Manager method
- Manager executes business logic, calls Proxy
- Proxy calls backend API
