# AiOSOrg — Project Plan

The organizational system for **AiOS**: a local-first, encrypted to-do / task /
habit / scheduling widget that lives on the AiOS canvas, where the agent captures
work from across the system, helps break it into completable tasks, and surfaces
it proactively. It is the **unified task model** of AiOS — the place a "thing the
user needs to do" lives, no matter where it came from.

**Status:** Planning — pre-Phase 0. The repository holds this document set, a
`CLAUDE.md` stub, the Apache-2.0 `LICENSE`, and a `README.md`; there is no code
yet.

## Vision

AiOSOrg is the part of AiOS that knows *what needs doing*. AiOS discards the
conventional desktop — no file manager, no app launcher, no office suite, no
dedicated calendar, mail, or to-do client (parent `DECISIONS.md`, "Explicit
non-inclusions"). Work, like everything else, surfaces on the canvas: an
organizational widget the user places and reads at a glance, and that the
proactive AI agent reads from and writes to on the user's behalf.

It is the natural complement to its two siblings. **AiOSCalendar knows *when*.
AiOSComms knows *what was said*. AiOSOrg knows *what to do about it.*** A meeting
proposed in an email (AiOSComms), a deadline that lands on a date
(AiOSCalendar), a habit the user wants to keep — all converge here as **tasks**:
captured, decomposed, prioritized, scheduled into open time, and surfaced when
they matter. This realizes the parent's **F-TODO** feature: *a unified task
model; the agent captures tasks from email, voice, meetings, and notes,
prioritizes them, and surfaces them proactively.*

It is two things at once, and the plan keeps them distinct:

1. **An organizer the user sees** — a canvas widget showing tasks, projects,
   habits, and what's due/next, editable by touch and voice; the surface the
   existing canvas **sticky notes** grow into over time.
2. **An organizer the system reasons over** — a headless core library that owns
   the **task model**, decomposes to-dos with the agent's help, tracks habits and
   recurrences, and *consumes the calendar and communication widgets through
   their agent-facing interfaces* to schedule work and capture it from messages.

The defining design challenge is the **coupling**. The parent `PARKING_LOT.md`
(#5, Organizational) specifies AiOSOrg as *tightly coupled with the Calendar (#3)
and Communication (#4) widgets*. Crucially, that coupling is **not a set of
bespoke in-process links**: per the AiOS-wide **F-MCP** decision (parent
`DECISIONS.md`, 2026-05-23), every component exposes its capabilities to the
agent as **MCP tooling**, and *the agent typically mediates between widgets*.
AiOSOrg therefore consumes AiOSCalendar's and AiOSComms's **already-decided
consumer interfaces** as MCP tools — the very interfaces those two plans designed,
deliberately, with AiOSOrg named as the consumer. AiOSOrg does not reimplement
calendaring or communication; it owns the task layer and reaches the others the
same way the agent does.

**Near-term:** a headless task / habit / recurrence core with an encrypted local
store and a stable consumer interface — something Jason can dogfood for personal
to-dos independent of the other widgets. **Long-term:** the unified
organizational surface of AiOS — agent-assisted task decomposition, calendar-
and-comms-fed capture, habit tracking, complex recurring reminders, the
sticky-note absorption, and full voice control.

## Guiding Principles

Inherited from the parent project (parent `DECISIONS.md`, parent `FEATURES.md`)
and specialized for an organizer:

1. **Local-first.** The organizer is fully functional with no other widget
   connected and no network. The local task store is the source of truth; the
   calendar and comms integrations *enrich* it but are never a prerequisite for
   capturing, decomposing, or completing a task. Cloud is never a default
   (parent local-first posture).
2. **Confidentiality by default.** A task list is a map of someone's life —
   obligations, health, relationships, finances. The task store is encrypted at
   rest like all AiOS user data (parent `FEATURES.md` F-STORAGE), using the
   **AiOSVault-style envelope** (per-object keys under a wrapping key) composed
   with full-disk encryption — the AiOS-wide data-at-rest answer.
3. **No app chrome.** AiOSOrg is a widget on the canvas, not a windowed to-do
   application with menu bars. A feature framed in terms of window chrome is
   fighting the product (cf. AiOSCanvas, AiOSCalendar, AiOSComms principle 1).
4. **Touch-first, never touch-blocked.** Every interaction — capture, complete,
   reschedule, decompose, check off a habit — must be completable by touch and
   voice. Keyboard and mouse are supported but never *required* (parent
   physical-input decision).
5. **The task model is the product.** The unified task data model and the
   consumer interface other AiOS surfaces (and future widgets) build on are the
   most important external API in this repo. They are versioned, documented, and
   changed deliberately — mirroring how AiOSCalendar treats its event model and
   AiOSComms its unified message model.
6. **Don't reimplement the siblings — consume them.** AiOSCalendar owns events
   and recurrence-on-the-calendar; AiOSComms owns messages and
   `extract_actionable`. AiOSOrg consumes both through their decided MCP
   interfaces, agent-mediated, rather than re-deriving a calendar or a mail
   client. The novelty budget goes to the *task model*, *decomposition*,
   *habits*, and the *integration*.
7. **Ingested content is untrusted data, never instructions.** A task captured
   from an email or a meeting note arrives as **data, not a command** (parent
   F-PROMPT-SAFETY). An "actionable item" handed over by AiOSComms is an inert
   candidate the user/agent act on — never an instruction the agent obeys. This
   is the single most important boundary in AiOS, applied to task capture — see
   *Security*.
8. **The agent helps; the user is in command.** The agent decomposes a to-do
   into smaller completable subtasks, prioritizes, schedules into open time, and
   surfaces proactively (F-TODO, F-AGENT-POLICY) — but task creation,
   destructive edits, and cross-domain actions (creating a calendar event,
   sending a reply) follow the agent-policy gates: provenance on everything,
   confirmation on irreversible or cross-domain actions, an undo window.
9. **Multi-user-ready from day one.** Every task, project, habit, and link
   carries a user scope from the first schema, even though AiOS is single-user
   through M2 (parent multi-user trajectory). Retrofitting a user scope onto a
   task store later is the trap the parent project chose to avoid.

## Scope

### In scope

- A **unified local task model**: tasks/items with status, priority,
  scheduled/due times, notes, provenance, and **subtasks** (the structure
  agent-assisted decomposition produces); **projects/lists** to group them; an
  encrypted local store as the source of truth.
- **Agent-assisted decomposition**: the agent breaks a coarse to-do ("plan the
  offsite") into smaller, completable tasks, under policy and with provenance —
  the defining F-TODO capability.
- **Habit tracking**: recurring intentions with cadence, streaks, and
  completion history ("meditate every weekday", "gym 3×/week").
- **Recurring tasks & reminders with complex scheduling**: rules richer than a
  single due date (every weekday, the last business day of the month, every
  third Tuesday), with reminders/alarms.
- A stable, versioned **consumer interface** (the task model + read / subscribe /
  write / decompose verbs), exposed to the agent and future consumers as **MCP
  tooling** — designed as a contract from the first release.
- **AiOSCalendar integration** (consume its MCP tooling, agent-mediated): link a
  task to a calendar event via the calendar's typed `external_refs`; use
  `free_busy` to schedule a task into open time; read recurrence rules and
  expansions to reason about habits.
- **AiOSComms integration** (consume its MCP tooling, agent-mediated): subscribe
  to new messages and turn `extract_actionable` results ("reply needed",
  "deadline mentioned", "task implied") into tasks/reminders with provenance.
- The **canvas widget** (the product form): tasks, projects, habits, and an
  agenda/"what's next" view, touch-first.
- **Sticky-note absorption**: the existing AiOSCanvas sticky-note widget folds
  into AiOSOrg over time, migrating a sticky note into a lightweight task/note in
  the unified model.
- **Voice** capture and completion ("remind me to call the dentist", "mark the
  groceries done"), riding the parent voice stack.

### Explicitly out of scope

- **Being a calendar.** AiOSOrg owns *tasks*, not *events*. It does not store
  events, render day/week/month grids, or sync calendar providers — that is
  **AiOSCalendar** (parent `PARKING_LOT.md` #3, F-CAL). AiOSOrg *references*
  calendar events and *queries* free/busy through AiOSCalendar's interface.
  AiOSCalendar deliberately never grows a task model; AiOSOrg is where it lives
  (AiOSCalendar `PLAN.md`, confirmed boundary).
- **Being a communication client.** AiOSOrg does not connect to Gmail, IMAP,
  Telegram, or any provider, parse MIME, or send mail/messages directly — that is
  **AiOSComms** (parent `PARKING_LOT.md` #4, F-COMMS). AiOSOrg consumes
  AiOSComms's `extract_actionable` and subscribe interfaces; where a task implies
  sending a reply, AiOSOrg asks the agent to drive AiOSComms's `send`/`reply`
  (confirmation-gated), rather than sending anything itself.
- **A standalone task / project-management product** (shared team boards,
  assignees across people, Gantt charts, time-tracking billing). AiOSOrg is a
  personal organizer for the AiOS user (multi-user-*ready* in schema, but not a
  collaboration suite).
- **Owning the user's identity or credentials.** Unlock and at-rest encryption
  ride on AiOS F-IDENTITY / F-STORAGE; any secret lives in AiOSVault. AiOSOrg is
  a consumer of those, never an owner. (AiOSOrg holds *no* provider credentials —
  it never talks to a provider; the calendar and comms widgets do.)
- **Re-deriving a recurrence engine if a shared one is the right call.** Whether
  AiOSOrg reuses an extracted AiOSCalendar RRULE engine or grows its own is an
  Open Question (below), not a settled in-scope build.

## How it fits into AiOS

AiOSOrg is a **canvas widget**, hosted by AiOSCanvas, and a **sibling component
repository** of the AiOS project — the **fourth and last** of the
`PARKING_LOT`-staged canvas widgets to graduate into its own repo, alongside
AiOSMedia, AiOSCalendar, and AiOSComms. Like AiOSFSS, AiOSPac, AiOSTerminal,
AiOSVault, and its widget siblings it is independently buildable, has its own
toolchain pin and CI, and (when wired) is composed into the parent
[AiOS](https://github.com/jduncan8142/AiOS) repo at a pinned revision. The parent
repo holds the authoritative cross-cutting docs; this repo's docs defer to them
on cross-cutting matters.

It follows the AiOS-wide **headless core library + thin frontends** pattern that
AiOSTerminal, AiOSCalendar, and AiOSComms use:

- **`aiosorg-core`** — a headless library with no UI: the unified task model, the
  decomposition logic (driven by the agent), the habit and recurrence engines,
  the encrypted local store, and the **consumer interface** the agent and other
  surfaces call. Fully testable headless. **This is the component's most
  important deliverable.**
- **Canvas-widget frontend** — the product form: tasks / projects / habits /
  agenda rendered into an AiOSCanvas-provided surface, input routed by the
  compositor. Thin; it renders the core's data and forwards user intent back.
- **Headless / agent frontend** — the core's interface as consumed by the AiOS
  agent (Python) and exposed as MCP tooling. For cross-process callers this is a
  control-plane surface in the AiOS house shape (Unix-socket NDJSON + event
  stream, as in AiOSFSS / AiOSVault / the calendar and comms cores); the MCP
  surface fronts it.

AiOSOrg implements the parent's **F-TODO** feature (the managed, unified task
model). It is a consumer of **F-VAULT** (indirectly — its store rides the
Vault-style envelope), **F-SYNC**, **F-CANVAS**, and the parent agent runtime,
and an enforcement point for **F-PROMPT-SAFETY** on captured content. It
*depends on*:

- **AiOSCalendar (F-CAL)** — AiOSOrg consumes the calendar's **four-verb consumer
  interface** (READ `events_in_range`/`event`/`calendars`; SUBSCRIBE to a change
  stream; WRITE `create`/`update`/`delete_event` with provenance; ASK
  `free_busy`) as **MCP tooling**, agent-mediated. The calendar **owns events**;
  AiOSOrg **owns tasks**; the two connect via the calendar's typed
  **`external_refs`** back-reference (a task references a calendar event; an event
  back-references the owning Org task). AiOSOrg uses `free_busy` to schedule a
  task into open time and reads recurrence rules + expansions to reason about
  habits. (Interface decided 2026-05-23 in AiOSCalendar's plan.)
- **AiOSComms (F-COMMS)** — AiOSOrg consumes the comms **consumer interface**
  (read, subscribe, send/reply, mark/flag, and crucially
  **`extract_actionable(thread|message)`**) as **MCP tooling**, agent-mediated.
  `extract_actionable` returns **structured candidate actions** ("reply needed",
  "event proposed: …", "deadline mentioned: …", "task implied: …") as **inert
  data with provenance — never executable instructions**; that is the explicit
  hook AiOSOrg turns into tasks and reminders. AiOSComms sits behind a strict
  untrusted-content boundary, but AiOSOrg reaches it via its MCP interface like
  any other consumer. (Interface decided 2026-05-23 in AiOSComms's plan.)
- **The AiOS agent runtime** (Python, parent repo) as the privileged operator
  that drives decomposition, prioritization, proactive surfacing (F-TODO,
  F-AGENT-POLICY), and — per F-MCP — *mediates* AiOSOrg's reads/writes to the
  calendar and comms widgets.
- **AiOSCanvas (F-CANVAS)** for the widget surface, input routing, and the
  canvas-widget contract — which does not exist yet (AiOSCanvas is at v0.2.0,
  pre-widget-SDK), plus the **sticky-note widget** AiOSOrg absorbs. The canvas
  frontend tracks AiOSCanvas, exactly as AiOSTerminal's / AiOSCalendar's canvas
  frontends do.
- **AiOSVault (F-VAULT) / F-STORAGE**, for the at-rest envelope the store reuses;
  and **AiOSFSS (F-SYNC)**, indirectly — the encrypted store is ciphertext at
  rest, so AiOSFSS keeps it in agreement across the user's devices with no cloud.
  Any runtime/socket state lives **outside** the synced directory (the AiOSVault
  rule).

And it is **the consumer at the end of the chain**: AiOSCalendar and AiOSComms
were each designed with AiOSOrg named as their primary downstream consumer.
AiOSOrg depends on them; they do not depend on AiOSOrg. This is the coupling the
plan is built around.

## Architecture

AiOSOrg is layered from the local task store up to the canvas views, with the
**consumer interface** across the top of the core (the seam the agent and future
consumers attach to), and the **integration adapters** reaching *outward* to the
calendar and comms widgets through the agent's MCP mediation:

```
        AiOS agent (Python)        canvas widget        future consumers
                 │                       │                     │
                 ▼ consumer interface (read/subscribe · write · decompose)
┌──────────────────────────────────────────────────────────────────────┐
│  Canvas-widget frontend (thin)                                         │
│    tasks · projects · habits · agenda / "what's next" ·                │
│    touch & voice input · rendered into an AiOSCanvas surface           │
├──────────────────────────────────────────────────────────────────────┤
│  Consumer interface (the load-bearing external API → MCP tooling)      │
│    task/project/habit read + change subscription · create/update ·     │
│    complete · decompose · schedule — versioned & documented            │
├──────────────────────────────────────────────────────────────────────┤
│  Org engine                                                            │
│    unified task model · decomposition (agent-driven) · habit tracking ·│
│    recurrence/reminder scheduling · prioritization · external links    │
├──────────────────────────────────────────────────────────────────────┤
│  Integration adapters — OUTBOUND, via the agent's MCP mediation        │
│    ┌────────────────────────────┐   ┌────────────────────────────────┐ │
│    │ AiOSCalendar (consume MCP) │   │ AiOSComms (consume MCP)        │ │
│    │  events_in_range · event · │   │  subscribe(new msgs) ·         │ │
│    │  free_busy · create/update │   │  extract_actionable ·          │ │
│    │  · external_refs link      │   │  (send/reply via agent, gated) │ │
│    └────────────────────────────┘   └────────────────────────────────┘ │
├──────────────────────────────────────────────────────────────────────┤
│  Local store (source of truth for tasks)                               │
│    encrypted at rest · SQLite · tasks, subtasks, projects, habits,     │
│    recurrence rules, reminders, external_refs, provenance, tombstones  │
├──────────────────────────────────────────────────────────────────────┤
│  AiOS services                                                         │
│    AiOSVault-style envelope (at-rest enc) · F-STORAGE · AiOSFSS (syncs │
│    the ciphertext store) · F-IDENTITY (unlock) · agent (mediation)     │
└──────────────────────────────────────────────────────────────────────┘
```

- **Local store** — the source of truth for tasks, subtasks, projects, habits,
  recurrence rules, reminders, cross-widget links, and provenance. SQLite (house
  style: `rusqlite` bundled, as in AiOSVault/AiOSFSS/the sibling cores),
  encrypted at rest. Holds tombstones so deletions reconcile across AiOSFSS
  rather than resurrecting, and a user scope on every row.
- **Org engine** — the canonical task model; the decomposition flow (the agent
  proposes a subtask breakdown, the engine stores it with provenance); the habit
  tracker (cadence, streaks, completion history); the recurrence/reminder
  scheduler (complex rules → concrete due/reminder instances); and
  prioritization. Pure and deterministic where it can be (recurrence expansion,
  streak computation are heavily testable); the agent-driven parts are I/O at the
  edges.
- **Integration adapters** — the *outbound* seams. Unlike its siblings (whose
  hard couplings are inward — providers, the canvas), AiOSOrg's defining couplings
  point **outward** to two other widgets, and they go **through the agent's MCP
  mediation**, not bespoke in-process links (F-MCP). The calendar adapter calls
  the calendar's four verbs (read events, `free_busy`, write events with
  provenance, maintain the `external_refs` link); the comms adapter subscribes to
  new messages and calls `extract_actionable`, and routes any outbound
  send/reply back through the agent (gated). Both treat everything they receive
  as untrusted data (see *Security*).
- **Consumer interface** — the documented, versioned API across the top of the
  core that the agent and future consumers use, surfaced as MCP tooling. Detailed
  in its own section.
- **Canvas-widget frontend** — renders tasks / projects / habits / agenda into an
  AiOSCanvas surface and forwards touch/voice intent back into the core. Thin by
  design; the surface the sticky notes migrate into.

## The task data model & the consumer interface

> **This is the section to read.** AiOSOrg owns the **unified task model** of
> AiOS (F-TODO). Like the calendar's event model and the comms unified model, it
> is treated as a **stable, documented, multi-consumer contract** from the first
> release — additive evolution, explicit versioning, no breaking change without a
> migration — and exposed to the agent (and future consumers) as **MCP tooling**.
> The local canvas views are *a* consumer of this interface, on equal footing
> with the agent, which keeps the interface honest.

### Design tenets for the contract

1. **One unified task, many origins.** A to-do typed by the user, a task the
   agent decomposed out of a bigger one, a task captured from an email, a habit
   occurrence, a reminder pinned to a calendar event — all are the same
   `Task`/item type, distinguished by **provenance** and links, not by separate
   schemas. This is the "unified task model" F-TODO calls for.
2. **Provenance is first-class.** Every task records its origin — user, agent
   (and *why*: "decomposed from task X", "captured from message Y"), or a habit
   /recurrence expansion. Provenance feeds F-PROMPT-SAFETY (a task the agent
   created from external content is distinguishable and its source is shown),
   F-AGENT-POLICY, and F-AUDIT, and it lets the UI explain *why* a task exists.
3. **Decomposition is structure, not a separate object.** "Break this down"
   produces **subtasks** under a parent — the same `Task` type, linked by
   parentage. A task is completable in itself or via its subtasks; the model
   represents both without a distinct "project vs. task" dichotomy at the leaf
   (projects/lists are a grouping layer above).
4. **Links out are typed and inert.** A task may reference a calendar event
   (`external_refs` → AiOSCalendar) and/or a message/thread (`external_refs` →
   AiOSComms). These are typed, namespaced references to *another widget's*
   objects — never copies of that widget's data, and never executable. The
   calendar/comms data is fetched live through their interfaces when needed.
5. **Read, subscribe, write, and decompose.** The interface is deliberately
   small: **read** tasks/projects/habits (by filter, by id, "what's due/next");
   **subscribe** to a live change stream so the views and the agent stay current
   without polling; **write** (create / update / complete / reschedule, every
   write carrying provenance); and **decompose** (the agent-assisted breakdown of
   a task into subtasks, gated by policy). These cover what the agent and the
   views need without exposing internals.
6. **Untrusted-origin tasks are marked.** A task whose content came from an
   email or a meeting note carries that provenance and an untrusted-origin flag,
   so the agent never treats its text as an instruction and the UI can show where
   it came from (F-PROMPT-SAFETY).

### The canonical task model (sketch — to be finalized in DESIGN.md)

A `Task` carries, at minimum:

- `id` — AiOS-internal stable identifier (UUID); the contract's notion of
  identity.
- `user_id` — the owning user scope (present from the first schema);
  `project_id?` — optional grouping (a project/list).
- `parent_id?` — for a subtask, the task it was decomposed from (the
  decomposition tree).
- `title`, `notes` — human text. **Treated as untrusted data when it originates
  from a captured message or external content** (F-PROMPT-SAFETY).
- `status` — e.g. `todo` / `in_progress` / `done` / `cancelled` /
  `waiting`/blocked (final set in DESIGN.md).
- `priority` — a small ordered set the agent and user both set; feeds proactive
  surfacing.
- `scheduled_for?` / `due?` — timezone-aware instants or date values; `due`
  drives reminders and "what's overdue", `scheduled_for` is when the user
  intends to *work* on it (often the result of a `free_busy`-driven placement).
- `recurrence?` — an optional rule for a recurring task/reminder (see *Recurrence
  & habits* and the Open Question on the engine).
- `reminders?` — relative or absolute trigger times.
- `external_refs` — typed, namespaced back-references to other widgets' objects:
  `calendar:event` (AiOSCalendar) and `comms:message`/`comms:thread`
  (AiOSComms). Open-ended so new consumers don't force a schema change — the same
  pattern the calendar uses.
- `provenance` — origin of the current state (user / agent — with the motivating
  task or message — / recurrence expansion), and an untrusted-origin flag.
- `created`, `modified`, completion metadata, and crypto-erase-friendly deletion
  metadata (tombstones, so deletions reconcile across AiOSFSS rather than
  resurrecting).

A `Project`/`List` groups tasks: `id`, `user_id`, `name`, optional color/icon,
and ordering. A `Habit` is a recurring intention with a cadence (e.g. "every
weekday", "3× per week"), a streak/`history` of completions, and its own
recurrence rule; whether a habit is *a specialization of* a recurring task or a
sibling type is a DESIGN.md call flagged in the Open Questions.

### Recurrence & habits

Complex scheduling is core to F-TODO ("recurring reminders/tasks with complex
scheduling") and to habit tracking. Two notable design points, both surfaced as
Open Questions for Jason rather than settled here:

- **The recurrence engine.** AiOSCalendar already builds a vetted RFC-5545 RRULE
  engine (its `K-RECUR`). AiOSOrg needs comparable expressiveness ("every
  weekday", "last business day of the month", "every third Tuesday"). Whether to
  **reuse/extract AiOSCalendar's RRULE engine as a shared crate** or grow
  AiOSOrg's own is an explicit Open Question — reuse avoids duplicating a
  correctness-minefield, but couples release cycles and may be a poor fit for
  habit cadences ("3× per week" is not an RRULE).
- **The habit model.** Streaks, cadences that aren't calendar-anchored, "skip
  without breaking the streak", and how a missed occurrence is treated are
  habit-tracker specifics that need their own small design — flagged for Jason.

### The consumer interface (sketch)

Conceptually, the core exposes a `TaskStore`-style interface with four
capability groups. (Rust trait sketch below — the *internal* core shape.) **The
agent consumes this contract as MCP tooling** (per F-MCP, decided 2026-05-23);
future widgets would consume the same MCP surface. A house-style cross-process
control plane (NDJSON over a Unix socket, AiOSFSS/AiOSVault shape) may back that
MCP surface; the exact layering is a Phase-0 detail.

```rust
/// The stable, versioned interface the agent and future consumers build on,
/// surfaced as MCP tooling. The canvas views are a consumer too, which keeps
/// the contract honest.
trait TaskAccess {
    // READ — tasks/projects/habits by filter or id; "what's due / next".
    fn tasks(&self, q: &TaskQuery) -> Result<Vec<Task>>;
    fn task(&self, id: &TaskId) -> Result<Option<Task>>;
    fn projects(&self, user: &UserId) -> Result<Vec<Project>>;
    fn habits(&self, user: &UserId) -> Result<Vec<Habit>>;
    fn agenda(&self, q: &AgendaQuery) -> Result<Agenda>; // due/overdue/scheduled

    // SUBSCRIBE — a live change stream (created / updated / completed / due),
    // so the views and the agent stay current without polling.
    fn subscribe(&self, filter: &ChangeFilter) -> Result<ChangeStream>;

    // WRITE — create / update / complete / reschedule; every write carries
    // provenance and returns the stable id.
    fn create_task(&self, draft: &TaskDraft, by: &Provenance) -> Result<TaskId>;
    fn update_task(&self, edit: &TaskEdit, by: &Provenance) -> Result<()>;
    fn complete_task(&self, id: &TaskId, by: &Provenance) -> Result<()>;
    fn log_habit(&self, id: &HabitId, when: &Occurrence, by: &Provenance) -> Result<()>;

    // DECOMPOSE — record an agent-proposed breakdown of a task into subtasks,
    // gated by policy; the agent supplies the proposal, the core stores the tree.
    fn decompose(&self, parent: &TaskId, into: &[SubtaskDraft], by: &Provenance)
        -> Result<Vec<TaskId>>;
}
```

- The **change stream** is how the canvas views redraw and how the agent learns a
  task became due or a habit's streak is at risk — one mechanism, several
  consumers (the pattern the siblings use).
- `decompose` is called out explicitly because it is *the* F-TODO primitive: the
  agent breaks a coarse to-do into completable subtasks. The agent does the
  reasoning; the core records the resulting tree with provenance ("decomposed by
  the agent from task X"). *How aggressively the agent auto-decomposes* — versus
  only on request — is a policy Open Question for Jason.
- The interface is **not** where the calendar/comms integration lives: that is
  *outbound* (AiOSOrg calls *their* MCP tools, agent-mediated). This interface is
  what AiOSOrg *exposes inward* to the agent and future consumers.

### Versioning & stability

The interface and the on-the-wire model carry an explicit version. Evolution is
additive (new optional fields, new methods); a breaking change requires a version
bump and a migration path. This is written down before any consumer beyond the
local views exists — the same discipline AiOSCalendar applied to its event model
and AiOSComms to its consumer interface.

## Key Decisions

### Inherited from the AiOS project (confirmed — see parent `DECISIONS.md`)

- **Apache 2.0** license across all code (the `LICENSE` in this repo is already
  Apache-2.0).
- **Canvas, not windows.** AiOSOrg is a widget hosted by AiOSCanvas; there is no
  standalone organizer application in the product.
- **Touch primary**, keyboard/mouse supported but never required.
- **Multi-user-ready** schema from day one; AiOS is single-user through M2.
- **Local-first; cloud is a fallback.** No other widget or network is a
  prerequisite for using the organizer.
- **F-PROMPT-SAFETY.** Content captured from messages/notes is data, never
  instructions; agent-created tasks carry provenance; cross-domain actions are
  confirmation-gated.
- **F-MCP (2026-05-23).** AiOSOrg **exposes** its capabilities to the agent as
  MCP tooling **and consumes** AiOSCalendar's and AiOSComms's MCP tooling; the
  agent typically mediates between widgets. MCP is a uniform access layer under
  the full safety model, not a bypass.
- **Credentials live in AiOSVault** — moot for AiOSOrg, which holds none (it
  never talks to a provider), but the store reuses the **Vault-style at-rest
  envelope**.

### Component-specific — proposed, to confirm in Phase 0

- **Stack: Rust headless core.** *Proposed (matches every sibling).* The parent
  mandates Rust for "system/security-critical components" and Python for the agent
  and tooling (parent `DECISIONS.md`, Languages). Like AiOSCalendar, an organizer
  is a data-and-UI component rather than obviously either — and the recommendation
  is the same: **Rust for `aiosorg-core`**, for parallel reasons. (1) It is a
  **stable library and a canvas frontend** in an all-Rust widget family; the
  canvas frontend must be Rust to render through the AiOSCanvas compositor, so the
  core sharing that language removes a boundary. (2) It **ingests untrusted
  content** — text captured from emails and notes flows in; memory-safe handling
  is the house posture for hostile input (parent `THREAT_MODEL.md` §7.5), even
  though AiOSComms does the heavy MIME/HTML parsing upstream. (3) **House-style
  consistency** — `rusqlite`, `tokio`, `serde`, the Unix-socket NDJSON control
  plane, the Makefile + CI conventions, edition 2024 / toolchain pin — all already
  established across the siblings; reusing them is cheaper than a parallel Python
  toolchain for one component. A thin Python client over the core's control plane
  (as AiOSVault/AiOSFSS/the sibling cores ship) gives the agent ergonomic access
  without the core being Python. **Considered alternative — Python core**
  (the agent's language, rich date/recurrence libraries like `dateutil`); rejected
  for the same reason as the siblings: it pushes a boundary onto the Rust canvas
  frontend and diverges from house style, and the agent gets ergonomic access via
  the thin client anyway.
- **Crate layout** *(Phase 0)* — a Cargo workspace (`aiosorg-core` library +
  `aiosorg` binary/frontends, plus a `python/` client), mirroring the siblings so
  the core that future consumers depend on is independently auditable and
  versionable. Final call in Phase 0. Rust edition 2024; toolchain pinned to match
  the sibling repos (currently 1.94).
- **Store** *(Phase 0)* — SQLite via `rusqlite` (bundled), encrypted payloads,
  matching the house pattern; the **AiOSVault-style envelope** (per-object keys
  under a wrapping key) composed with F-STORAGE FDE. Coordinate with AiOSVault /
  AiOSComms on a shared crypto crate rather than duplicating it.
- **Recurrence engine** *(Open Question)* — reuse/extract AiOSCalendar's RRULE
  engine as a shared crate vs. AiOSOrg's own. Flagged for Jason; affects whether a
  shared crate is extracted from AiOSCalendar.
- **Consumer-interface schema + MCP surface** *(Phase 0)* — transport is the house
  Unix-socket NDJSON + event stream, fronted by MCP tooling; the *schema* and its
  versioning policy are designed in Phase 0 because it is a contract from the
  first release.
- **Calendar/comms integration is via the agent's MCP mediation** *(decided
  upstream, 2026-05-23)* — AiOSOrg consumes AiOSCalendar's four-verb interface and
  AiOSComms's `extract_actionable`/subscribe interface as MCP tools, agent-
  mediated, not as bespoke in-process links. This is the F-MCP pattern; AiOSOrg is
  the canonical example of "the agent mediates between widgets".

## Repository layout (planned)

Mirrors the sibling repos (AiOSCalendar / AiOSComms / AiOSVault):

```
AiOSOrg/
├─ Cargo.toml              # cargo workspace (recommended)
├─ Cargo.lock              # committed
├─ rust-toolchain.toml     # pinned channel (house: 1.94)
├─ Makefile                # build / release / test / lint / fmt / clean
├─ LICENSE                 # Apache-2.0 (present)
├─ NOTICE
├─ README.md               # present (stub)
├─ CLAUDE.md               # present (stub) — points at this doc set
├─ PLAN.md ROADMAP.md      # this document set
├─ DESIGN.md               # task model + consumer interface + store schema +
│                          #   the calendar/comms integration shapes
├─ FEATURES.md             # the AiOSOrg feature catalog (proposed `O-*` codes)
├─ TODO.md                 # actionable, phase-by-phase checklist
├─ .gitignore              # present
├─ aiosorg.example.toml    # widget/daemon tunables (no secrets)
├─ .github/workflows/      # CI: fmt, clippy, test, cargo audit/deny
├─ crates/
│  ├─ aiosorg-core/        # task model, decomposition, habits, recurrence,
│  │                       #   store, consumer interface           (lib)
│  └─ aiosorg/             # frontend(s) — canvas widget; headless/agent surface
└─ python/                 # thin client over the control plane (agent; later)
```

`DESIGN.md`, `FEATURES.md`, and `TODO.md` are written as the work begins; this
PR establishes `PLAN.md` and `ROADMAP.md` only (docs-first, code-free — the same
way AiOSCanvas, AiOSVault, AiOSTerminal, AiOSCalendar, and AiOSComms each began).

## Proposed dependencies

- **House style, shared with the sibling repos:** `anyhow`, `thiserror`, `clap`
  (derive), `tokio`, `serde`/`serde_json`, `rusqlite` (bundled), `toml`,
  `tracing`/`tracing-subscriber`, `uuid`.
- **Crypto / store** (reusing AiOSVault's choices for the encrypted store, ideally
  a shared crate): `chacha20poly1305`, `zeroize`, `secrecy`, `getrandom`,
  `blake3`.
- **Recurrence / time:** an RRULE engine (`rrule`) and time-zone-aware date/time
  (`chrono` + `chrono-tz`, or `time` + a tzdb) — *or* a shared crate extracted
  from AiOSCalendar (the Open Question). Finalized in the Phase-0 survey.
- **Dev:** `tempfile`, `proptest` (recurrence-expansion and streak-computation
  property tests), `insta` (snapshot tests for decomposition/agenda shapes).

All dependencies must be permissively licensed (Apache-2.0 / MIT / ISC) and
compatible with Apache-2.0, and pass the parent's `cargo audit` / `cargo deny`
supply-chain gate (parent `THREAT_MODEL.md` §7.4).

## Security

AiOSOrg inherits the AiOS threat model (parent `THREAT_MODEL.md`); the
organizer-specific points:

- **Captured content is untrusted data (F-PROMPT-SAFETY).** A task's title/notes
  may originate from an email or meeting note that AiOSComms surfaced. That text
  is **data, never an instruction**: it is stored with untrusted-origin
  provenance, and the agent that reads the task to act on it sees structured task
  fields, not a channel for injected commands (the command–data boundary, parent
  `THREAT_MODEL.md` §5, applied to task capture). A malicious email that says
  "create a task to wire money and mark it done" becomes, at most, an inert
  candidate task the user sees and dismisses — never an action the agent takes.
- **`extract_actionable` output is inert by construction.** AiOSOrg consumes
  AiOSComms's candidate actions as *data with provenance*. Turning a candidate
  into a real task is an explicit step; turning a task into a cross-domain action
  (send a reply, create a calendar event) is **confirmation-gated** and routed
  through the agent under F-AGENT-POLICY. AiOSOrg never sends a message or writes
  a calendar event silently in response to ingested content.
- **Cross-domain actions are gated and provenance-stamped.** When a task implies
  a calendar write (`create_event`) or a comms send (`reply`), AiOSOrg asks the
  agent to perform it through the *owning* widget's MCP interface, with provenance
  naming AiOSOrg and the motivating task, and with confirmation on irreversible or
  cross-domain actions (F-AGENT-POLICY, F-AUDIT). AiOSOrg holds no provider
  credentials and performs no provider I/O — the credential and egress surface
  lives entirely in the calendar/comms widgets.
- **Confidentiality at rest (F-STORAGE).** The task store is encrypted at rest
  with the Vault-style envelope; a task list is highly sensitive behavioral data.
  Deletion is crypto-erase-friendly (tombstones that reconcile across AiOSFSS,
  keys destroyable).
- **MCP is under the safety model, both directions.** The MCP surface AiOSOrg
  *exposes* is subject to per-context tool allowlists, provenance, and audit; the
  MCP tools AiOSOrg *consumes* (calendar, comms) are reached through the agent's
  mediation under the same model. MCP is a uniform access layer, not a privilege
  escalation (parent F-MCP).
- **Multi-user scope** is carried on every task/project/habit/link row from the
  first schema, so a second user on the device (M3+) cannot see another's tasks.

## Risks

- **The defining couplings are to two in-flight siblings.** AiOSOrg's value
  depends on AiOSCalendar's four-verb interface and AiOSComms's
  `extract_actionable` existing and being stable — and on the agent's MCP
  mediation layer. *Mitigation:* both sibling interfaces are **designed early and
  frozen at 1.0** in their own roadmaps (AiOSCalendar P1/P7; AiOSComms P2/P6),
  explicitly *because* AiOSOrg is their consumer; AiOSOrg's local task/habit core
  is independent of both and is built **first**, so a delay in either sibling
  delays only the integration phases, not the whole component.
- **AiOSOrg is last and not on any critical path.** It presupposes the agent
  runtime *and* two stable sibling interfaces, so it competes for the same
  evenings as everything upstream of it and starts later. *Mitigation:* inherit
  the parent scope-compression posture — the local organizer is the floor; the
  calendar integration, then the comms integration, then voice are the things to
  defer; narrow a phase before slipping a date.
- **The task model is committed-to as a contract before consumers beyond the
  views exist.** *Mitigation:* make the local canvas views a real consumer of the
  same interface from day one so it is exercised; keep it small (four capability
  groups); version it additively.
- **Recurrence + habit scheduling is a correctness minefield.** Complex rules,
  DST, "skip without breaking a streak", non-calendar cadences ("3×/week").
  *Mitigation:* build on a vetted RRULE engine (reused or own — the Open
  Question) rather than hand-rolling calendar math; treat recurrence expansion
  and streak logic as pure and test them exhaustively (property tests) before any
  UI or integration rides on them.
- **Over-eager agent decomposition is a UX risk.** An agent that shatters every
  to-do into noise is worse than none. *Mitigation:* decomposition is a policy
  surface (the Open Question) — default conservative (decompose on request, or
  propose-then-confirm), tune with use; provenance lets the user see and undo
  agent-created subtasks.
- **Sticky-note absorption touches another component's data.** Folding the
  AiOSCanvas sticky notes in means a migration path and a timing decision (the
  Open Question), and coordination with AiOSCanvas. *Mitigation:* treat it as a
  later, well-scoped phase with an explicit one-way migration; do not block the
  core organizer on it.

## Open Questions

Decisions that need Jason's (or the parent project's) input. **Resolve nothing
here** — these are flagged for review, mirroring how the sibling plans opened
their kickoff questions.

1. **Parent feature — F-TODO as-is, or a new F-ORG?** AiOSOrg implements the
   parent **F-TODO** ("Managed Tasks — unified task model") *at minimum*. But its
   actual scope — calendar + to-do + task-management + **habit tracking** +
   **complex recurring scheduling** + sticky-note absorption — is broader than
   F-TODO's one-line "unified task model; capture, prioritize, surface". Does
   AiOSOrg (a) implement F-TODO as written, decomposing into `O-*` sub-tags under
   it (mirroring `K-*` under F-CAL, `MD-*` under F-MEDIA), or (b) warrant a new
   parent feature **F-ORG** with F-TODO subsumed under it — the way **F-COMMS**
   was minted to replace the narrower **F-EMAIL** when AiOSComms's scope outgrew
   it? It implements F-TODO at least; flagged for Jason. *(The parent
   `FEATURES.md` edit + submodule wiring happen when AiOSOrg graduates from
   `PARKING_LOT` into a sprint — which has not happened yet.)*
2. **Recurrence engine — reuse AiOSCalendar's, or grow our own?** AiOSCalendar
   builds a vetted RFC-5545 RRULE engine (`K-RECUR`). Should AiOSOrg **reuse it as
   an extracted shared crate** (avoid duplicating a correctness-minefield, but
   couple release cycles, and RRULE may not fit habit cadences like "3×/week"), or
   **build its own** recurrence/cadence engine tuned for tasks and habits? This
   also decides whether a shared crate is extracted from AiOSCalendar.
3. **Sticky-note absorption — migration path and timing.** *How* does an existing
   AiOSCanvas sticky note become an AiOSOrg task/note (one-way migration? live
   bridge? does the sticky-note widget persist as a thin AiOSOrg view?), and *when*
   (which phase)? Needs coordination with AiOSCanvas and a decision on whether
   sticky notes survive as a concept or are fully replaced.
4. **Habit-tracking model specifics.** Is a `Habit` a specialization of a
   recurring `Task` or a sibling type? How are streaks defined, how is a missed or
   deliberately-skipped occurrence treated, and what cadences must be expressible
   ("every weekday", "3×/week", "twice a day")? This shapes both the schema and
   the recurrence-engine choice (Q2).
5. **How aggressively the agent auto-decomposes tasks (policy).** Does the agent
   decompose **only on explicit request**, **propose-then-confirm**, or
   **proactively** break down to-dos it judges too coarse? This is an
   F-AGENT-POLICY surface with a real UX cost if wrong; the safe default is
   conservative, but it's Jason's call.
6. **Standalone (non-canvas) frontend — none, confirming the Calendar
   precedent?** AiOSCalendar resolved to **Linux AiOS canvas-only — no standalone
   window, not even for dev dogfooding** — with the headless core + agent (MCP)
   access carrying early use. AiOSOrg proposes to follow that precedent (canvas
   widget + headless/agent surface, no standalone dev frontend). Confirm — or do
   you want a standalone dev frontend for AiOSOrg to dogfood before the AiOSCanvas
   widget contract lands?
7. **Where the decomposition / capture model runs.** Decomposition and any
   summarization of captured content are *agent* work. Confirm AiOSOrg routes
   these through the parent agent's planner/reader (F-PROMPT-SAFETY's
   privileged-planner / unprivileged-reader split), rather than AiOSOrg invoking a
   model directly — so untrusted captured content never reaches a privileged
   context as instructions. (Likely yes by the parent safety model; flagged to be
   explicit.)
8. **Calendar write-back scope.** AiOSOrg uses `free_busy` to *find* open time
   for a task. Should it also **write the scheduled block back** to AiOSCalendar
   as an event (via the calendar's `create_event` + an `external_refs` link), so
   a scheduled task shows on the calendar — or only hold `scheduled_for`
   internally and leave the calendar untouched? Writing back is the richer UX but
   a cross-domain, confirmation-gated action; flagged for Jason.

## Change Log

| Date | Change | Rationale |
|------|--------|-----------|
| 2026-05-22 | Initial AiOSOrg planning document set created (PLAN, ROADMAP) | Repository kickoff; the Organizational item (parent `PARKING_LOT.md` #5) — the 4th and last canvas widget — gets its initial plan. Derived from the parent AiOS planning docs (F-TODO, F-MCP, F-PROMPT-SAFETY, F-AGENT-POLICY) and the merged sibling-widget house style (AiOSCalendar, AiOSComms, AiOSMedia). Implements **F-TODO**; consumes AiOSCalendar's and AiOSComms's decided MCP consumer interfaces, agent-mediated (F-MCP). Eight Open Questions raised for Jason; nothing resolved here. |
