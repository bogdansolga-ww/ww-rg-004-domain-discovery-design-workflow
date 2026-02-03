# Mobile ↔ Jerry Interaction Flows

Sequence diagrams showing how the mobile app and AI assistant work together.

---

## Core Pattern

Jerry **signals intent**, mobile app **executes logic**.

```mermaid
sequenceDiagram
    participant User
    participant Mobile as Mobile App
    participant Jerry as Jerry API
    participant GPT as GPT-4o
    participant Backend as Backend API

    User->>Mobile: Voice/Text command
    Mobile->>Jerry: Context + message
    Jerry->>GPT: Reason about intent
    GPT-->>Jerry: Tool call decision
    Jerry-->>Mobile: Signal (<discard>)
    Mobile->>Mobile: Manager executes logic
    Mobile->>Backend: API call
    Backend-->>Mobile: Result
    Mobile-->>User: UI updated
```

---

## Workflow 1: Start Timer

**User says:** "Start my timer"

```mermaid
sequenceDiagram
    participant Tech as Technician
    participant App as Mobile App
    participant Jerry
    participant SM as ServiceTimersManager
    participant API as Backend API

    Tech->>App: "Start my timer"
    App->>Jerry: {screen: "JobDetails", job_id: 123, is_clocked_in: true}

    Note over Jerry: GPT-4o reasons:<br/>User wants timer started<br/>Context shows job selected<br/>User is clocked in ✓

    Jerry-->>App: tool: request_production_timer_toggle<br/>returns: "<discard>"

    App->>SM: SetTimeInAsync(service)
    SM->>SM: Create ProdEntr record
    SM->>SM: Set Start timestamp
    SM->>SM: Create DoneByEn (crew)
    SM->>API: QuickProductionProxy.StartTimer()
    API-->>SM: OK
    SM-->>App: Timer started
    App-->>Tech: UI shows "In Progress"
```

---

## Workflow 2: Clock In

**User says:** "Clock me in"

```mermaid
sequenceDiagram
    participant Tech as Technician
    participant App as Mobile App
    participant Jerry
    participant TC as TimeClockManager
    participant API as Backend API

    Tech->>App: "Clock me in"
    App->>Jerry: {screen: "Home", is_clocked_in: false}

    Note over Jerry: GPT-4o reasons:<br/>User wants to clock in<br/>Currently clocked out ✓

    Jerry-->>App: tool: request_time_clock_toggle_tool<br/>returns: "<discard>"

    App->>TC: ClockInAsync()
    TC->>TC: Validate not already clocked in
    TC->>TC: Record clock-in time
    TC->>API: TimeClockProxy.ClockIn()
    API-->>TC: OK
    TC-->>App: Clocked in
    App-->>Tech: UI shows "Clocked In: 8:00 AM"
```

---

## Workflow 3: Add Job Note

**User says:** "Add a note: customer not home"

```mermaid
sequenceDiagram
    participant Tech as Technician
    participant App as Mobile App
    participant Jerry
    participant NM as NotesManager
    participant API as Backend API

    Tech->>App: "Add a note: customer not home"
    App->>Jerry: {screen: "JobDetails", job_id: 456}

    Note over Jerry: GPT-4o reasons:<br/>User wants to add note<br/>Extract text: "customer not home"<br/>Job context available ✓

    Jerry-->>App: tool: add_job_note<br/>note: "customer not home"<br/>returns: "<discard>"

    App->>NM: AddNoteAsync(job_id, text)
    NM->>NM: Create Note entity
    NM->>NM: Link to current job
    NM->>API: NotesProxy.SaveNote()
    API-->>NM: OK
    NM-->>App: Note saved
    App-->>Tech: UI shows note added
```

---

## Workflow 4: Stop Timer (with pricing)

**User says:** "Stop the timer"

```mermaid
sequenceDiagram
    participant Tech as Technician
    participant App as Mobile App
    participant Jerry
    participant SM as ServiceTimersManager
    participant PM as ProductManager
    participant API as Backend API

    Tech->>App: "Stop the timer"
    App->>Jerry: {screen: "JobDetails", job_id: 123, timer_running: true}

    Jerry-->>App: tool: request_production_timer_toggle<br/>returns: "<discard>"

    App->>SM: SetTimeOutAsync(service)
    SM->>SM: Set End timestamp
    SM->>SM: Calculate duration
    SM->>PM: Get crew size
    PM-->>SM: crew_size: 2

    Note over SM: Business Logic:<br/>duration = 45 min<br/>man_hours = 45 × 2 = 90<br/>price = man_hours × rate

    SM->>SM: Calculate service price
    SM->>SM: Apply discounts
    SM->>SM: Calculate taxes
    SM->>API: QuickProductionProxy.StopTimer()
    API-->>SM: OK
    SM-->>App: Timer stopped, price calculated
    App-->>Tech: UI shows "Complete: $125.00"
```

---

## Key Takeaways

1. **Jerry never touches business logic** — only decides which tool to call
2. **Managers contain the rules** — pricing, validation, state transitions
3. **Context is critical** — Jerry needs screen, entity ID, user state to decide correctly
4. **`<discard>` is a signal** — tells mobile app "execute this action" without displaying text
