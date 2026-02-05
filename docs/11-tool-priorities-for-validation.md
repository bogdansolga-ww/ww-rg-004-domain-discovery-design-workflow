# AI Tool Priorities for Domain Discovery

**Status:** Pending Brianna validation
**Date:** 2026-02-05
**Source:** `Real-Green-Mobile` branch `MOB-12830-AI-assistant-NEW`
**File:** `RG.Mobile.Core/Managers/AIService/ToolDispatcherService.cs`

---

## Top 3 for Workflow Validation

| # | Journey | Key Tool | Key Managers | Brianna Priority |
|---|---------|----------|--------------|------------------|
| 1 | Production Timers | `ExecuteProductionTimerToggle` | `ServiceTimersManager`, `ServiceManager` | ___ |
| 2 | Time Clock | `ExecuteTimeClockToggle` | `TimeClockManager`, `EmployeeManager` | ___ |
| 3 | Job/Route Listing | `ExecuteGetJobsForDate` | `RouteProxy`, `CustomerInfoManager`, `ServiceManager` | ___ |

---

## Full Tool Inventory

| Tool | Purpose | Managers Involved | Implemented |
|------|---------|-------------------|-------------|
| `ExecuteProductionTimerToggle` | Start/stop service timers | `ServiceTimersManager`, `ServiceManager` | Yes |
| `ExecuteGetStartableServicesForJob` | Services ready to start | `ServiceManager` | Yes |
| `ExecuteGetStoppableServicesForJob` | Services ready to stop (with readiness check) | `ServiceManager` | Yes |
| `ExecuteGetAllRunningTimers` | All active timers across jobs | `ServiceManager` | Yes |
| `ExecuteTimeClockToggle` | Employee clock in/out | `TimeClockManager`, `EmployeeManager`, `ConfigurationManager` | Yes |
| `ExecuteGetJobsForDate` | View route/jobs for day | `RouteProxy`, `CustomerInfoManager`, `ServiceManager` | Yes |
| `ExecuteGetJobDetails` | Detailed job info | `RouteProxy`, `EmployeeManager` | Yes |
| `ExecuteGetSummary` | Daily service/product aggregation | `ServiceManager`, `ProductManager` | Yes |
| `ExecuteGetCustomerHistory` | Customer history (services, invoices) | `ProductManager`, `ServiceManager` | Yes |
| `ExecuteGetServiceInfo` | Service details by ID | `ServiceManager` | Yes |
| `ExecuteGetWeather` | Current weather | `WeatherManager`, `LocationReader` | Yes |
| `ExecuteAddJobNote` | Add note to job | `NotesManager` | No |
| `ExecuteUpdateGeolocation` | Update geolocation | — | No |

---

## Notes for Brianna

- **Priority column:** Fill with 1-5 (1 = highest) based on upcoming roadmap
- **Top 3 rationale:**
  - Production Timers: Most complex BL, multiple edge cases
  - Time Clock: Clear BL with break lockout logic
  - Job/Route: Foundation for other tools, data aggregation
- **Questions:**
  - Are there tools not yet in `ToolDispatcherService` that are planned soon?
  - Any of the top 3 already well-documented elsewhere?

---

## Next Steps

1. Brianna reviews and adjusts priorities
2. Run `/discover-domain` on top priority journey
3. Validate domain map output with Brianna
4. Refine workflow based on feedback
