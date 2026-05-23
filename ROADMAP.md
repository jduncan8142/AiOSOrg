# AiOSOrg — Roadmap

The time-ordered delivery plan for the AiOS organizational widget. For vision,
architecture, the task data model, the consumer interface, and the
calendar/comms integration, see [PLAN.md](PLAN.md).

Phases use the **AiOS project's global phase numbers (P0–P7)** so they line up
with the parent `PLAN.md`, `DECISIONS.md`, and `TODO.md`. Milestones **M1 / M2 /
M3** are the AiOS project milestones — AiOSOrg does not set its own. Proposed
`O-*` codes reference AiOSOrg's own (not-yet-written) `FEATURES.md`; `F-*` codes
reference the parent `FEATURES.md`.

Dates are targets. The standing posture, inherited from the parent project, is
to **compress scope rather than miss a date** — narrow a phase's surface before
letting it slip. For AiOSOrg the sharpest compression lever is **deferring the
integration phases (calendar, then comms) and voice**, never weakening the local
core or the safety boundary on captured content.

## Where AiOSOrg sits relative to the milestones

AiOSOrg is the **fourth and last** of the canvas widgets and is **not on the
AiOS M1 critical path** — M1 is AiOSCanvas booting a canvas with one widget and
no AI (parent `DECISIONS.md`, M1 scope). AiOSOrg is later still than its widget
siblings: it presupposes **both** the agent runtime *and* two stable sibling
consumer interfaces (AiOSCalendar's four verbs, AiOSComms's `extract_actionable`)
*and* the agent's MCP mediation layer. Its useful early life is as a **headless,
dogfoodable task/habit core** that lands before any of the integration work, so
value arrives without waiting on the whole stack.

Three couplings shape the sequencing — and unusually for an AiOS component, the
defining ones point **outward** to sibling widgets, reached through the agent's
MCP mediation rather than in-process links (parent **F-MCP**):

- **The local task/habit/recurrence core depends on nothing else** and is built
  first (the AiOSTerminal/AiOSCalendar pattern: headless core before any
  integration).
- **The AiOSCalendar integration depends on AiOSCalendar's consumer interface**
  (its `K-IFACE`/`K-MCP`, designed in *its* P1/P2 and frozen at 1.0 in *its* P7)
  and on the agent's MCP mediation.
- **The AiOSComms integration depends on AiOSComms's consumer interface** (its
  `X-IF`/`X-AGENT` with `extract_actionable`, designed in *its* P2 and hardened
  for AiOSOrg in *its* P6) and on the agent's MCP mediation.
- **The canvas-widget frontend depends on AiOSCanvas** exposing a widget/surface
  contract, which does not exist yet (AiOSCanvas is at v0.2.0) — and on the
  AiOSCanvas **sticky-note** widget for the absorption path. That work tracks
  AiOSCanvas, exactly as AiOSTerminal's and AiOSCalendar's canvas frontends do —
  so it lands around **M2**, not before.

The deliberate ordering — **headless local task/habit core first, the consumer
interface stabilized early, the calendar integration, then the comms
integration, then the canvas widget + sticky-note absorption, then voice** —
falls directly out of the local-first principle and the dependency chain.

## Milestone overview

| Milestone | Date | Phases | What AiOSOrg delivers |
|-----------|------|--------|-----------------------|
| **M1 — Booting prototype** | Q3 2026 | — | Nothing required. AiOSOrg is post-M1 and the last widget; at most, Phase-0 scaffolding may begin opportunistically. |
| **(pre-M2 groundwork)** | 2026–2027 | P0–P2 | The headless `aiosorg-core`: the unified task model, the habit + recurrence engines, the encrypted local store, **the first stable cut of the consumer interface**, and the agent + MCP control plane with agent-assisted task **decomposition**. A local organizer usable headless / via the agent. Not gated by any sibling. |
| **M2 — Daily driver** | Q4 2027 | P3–P5 | The organizer integrated: **AiOSCalendar** (event↔task `external_refs`, free/busy-driven scheduling) and **AiOSComms** (`extract_actionable` → tasks) consumed via their MCP tooling, agent-mediated; the **canvas widget** (tasks/projects/habits/agenda) under touch; **sticky-note absorption**; voice capture. The unified task surface of the daily driver. |
| **M3 — Alpha release** | Q4 2028 | P6–P7 | Interop and richer integration depth; hardening, accessibility, the security review of the captured-content path; the stabilized 1.0 consumer interface. |

## P0 — Foundations & Scaffolding · *Org involvement: heavy*

**Goal:** turn the bare repository into a buildable project with a working
build-and-test loop, and lock the organizer foundations before modeling.

- Scaffold the Cargo workspace (`aiosorg-core` + `aiosorg`, plus a later
  `python/` client), `rust-toolchain.toml` (house: 1.94), `Makefile`, `NOTICE`,
  CI (`fmt` / `clippy -D warnings` / `test` + `cargo audit` / `cargo deny`);
  `.gitignore` already present.
- **Confirm the stack: Rust core** (PLAN — Key Decisions; proposed, matching
  every sibling), and resolve the **recurrence-engine** Open Question (reuse
  AiOSCalendar's RRULE engine as a shared crate vs. AiOSOrg's own — PLAN Open
  Question 2), since it gates the dependency survey.
- **Technology survey:** pick the recurrence/time crates (or wire the shared
  AiOSCalendar crate) and the encrypted-store crates (reusing the Vault-style
  envelope); confirm Apache-2.0/MIT/ISC and that they pass `cargo audit` /
  `cargo deny`.
- Write **`DESIGN.md`** — the canonical task model, the local-store schema, the
  **consumer interface** in full, the habit/recurrence design, and the *shapes*
  of the calendar/comms integration (which of their MCP tools AiOSOrg calls, and
  the `external_refs` link). Write **`FEATURES.md`** (`O-*` codes) and
  **`TODO.md`**.
- Establish the Linux build/test loop on a dev VM (house pattern).

**Exit:** `make build` / `make test` / `make lint` succeed locally and in CI; the
task model, consumer interface, and habit/recurrence design are written down in
`DESIGN.md`; dependencies are chosen and audited; the recurrence-engine question
is resolved with Jason.

## P1 — Local Organizer Core · *heavy* · pre-M2 groundwork

**Goal:** a correct, headless local organizer — the substrate everything else
rides on. **This is the most important phase**: it produces the task model and
the consumer interface future surfaces depend on.

- `O-MODEL` — the canonical `Task` / `Project` / `Habit` model with stable ids,
  user scope, provenance (incl. untrusted-origin), `parent_id` decomposition
  trees, and `external_refs` (the calendar/comms link slots). (PLAN: *The task
  data model*.)
- `O-STORE` — the encrypted local SQLite store: tasks, subtasks, projects,
  habits, recurrence rules, reminders, links, tombstones; forward-only schema
  migrations; the **Vault-style at-rest envelope**.
- `O-RECUR` — the recurrence/reminder engine: complex scheduling rules → concrete
  due/reminder instances (every weekday, last business day, every third Tuesday).
  Pure and exhaustively tested (property tests + golden fixtures). *(Built on the
  reused or own engine per the P0 resolution.)*
- `O-HABIT` — the habit tracker: cadences, streaks, completion history,
  skip-without-breaking semantics. Pure and tested.
- `O-IFACE` — **the first stable, versioned cut of the `TaskAccess` consumer
  interface** (read / subscribe / write / decompose), exercised by an in-repo
  test/headless consumer so it is real, not theoretical.

**Exit:** a headless local organizer that captures, stores, recurs, tracks
habits, and queries tasks correctly under test; the consumer interface is
callable and versioned. **No dependency on any sibling component** — this phase
lands on AiOSOrg's own schedule.

**Explicitly not in P1:** any canvas rendering, any calendar/comms integration,
voice, agent-driven decomposition (the *recording* of a decomposition tree is
here; the agent that *produces* it is P2).

## P2 — Agent Integration, MCP Control Plane & Decomposition · *moderate*

**Goal:** make the local organizer something the AiOS agent can drive — and add
the agent-assisted task decomposition that is the heart of F-TODO — once the
parent local-AI backbone (parent Phase 2) exists.

- `O-CTRL` — the Unix-socket NDJSON control plane + event stream mirroring the
  `TaskAccess` interface (house shape: AiOSFSS / AiOSVault / the sibling cores),
  plus the AiOS launch contract (`AIOS_*`) for a supervised headless mode.
- `O-MCP` — **the MCP tooling surface AiOSOrg exposes** (parent **F-MCP**,
  2026-05-23): the consumer-interface verbs (read / subscribe / write /
  decompose) exposed as MCP tools, backed by the control plane, and consumed
  identically by the agent and (future) other widgets.
- `O-AGENT` — the agent-facing decomposition + prioritization paths (via
  `O-MCP`) realizing **F-TODO**: the agent breaks a coarse to-do into completable
  subtasks and prioritizes, with **provenance on every agent write** and
  cross-domain-confirmation hooks (F-AGENT-POLICY). Decomposition aggressiveness
  follows the policy resolved from PLAN Open Question 5.
- `python/` — the thin `OrgClient` over the control socket (the agent seam),
  backing the MCP tooling; a wrapper *over* the core, never a direct link.

**Depends on:** the parent AiOS agent runtime existing. First phase coupled to a
sibling deliverable; sequence accordingly. **Not** dependent on AiOSCalendar or
AiOSComms — the *outbound* integrations are P3–P4.

**Exit:** the agent can capture, decompose, prioritize, and complete tasks
through the MCP surface; agent-created tasks/subtasks carry provenance.

## P3 — AiOSCalendar Integration · *moderate* · → M2

**Goal:** connect tasks to time — consume AiOSCalendar's MCP tooling
(agent-mediated) so tasks link to events and schedule into open time. The first
*outbound* integration.

- `O-CAL-LINK` — link a task to a calendar event via the calendar's typed
  **`external_refs`** (task → `calendar:event`; the calendar back-references the
  owning Org task), and subscribe to the calendar's change stream so a task
  pinned to an event stays current when the event moves.
- `O-CAL-SCHED` — **free/busy-driven scheduling**: "find me 90 minutes before
  Thursday" = a `free_busy` query (the calendar answers *when you're free*) plus
  AiOSOrg's task logic (it decides *what to put there*), setting `scheduled_for`.
- `O-CAL-RECUR` — read the calendar's recurrence rules + expansions to reason
  about habits/recurring tasks anchored to calendar patterns ("every weekday").
- `O-CAL-WRITE` *(scoped per PLAN Open Question 8)* — optionally write a
  scheduled task block back to the calendar as an event (`create_event` + the
  `external_refs` link), as a cross-domain, **confirmation-gated** action via the
  agent.

**Depends on:** AiOSCalendar's consumer interface (its `K-IFACE`/`K-MCP`, frozen
toward 1.0) and the agent's MCP mediation. AiOSOrg consumes the **four verbs**
(READ / SUBSCRIBE / WRITE / ASK `free_busy`) decided in AiOSCalendar's plan — it
does **not** reimplement any calendaring.

## P4 — AiOSComms Integration, Canvas Widget & Sticky-Note Absorption · *heavy* · **→ M2**

**Goal:** the AiOS milestone **M2** contribution — capture from communication,
the product form on the canvas, and folding in the sticky notes.

- `O-COMMS-CAPTURE` — consume AiOSComms's **`extract_actionable(thread|message)`**
  (agent-mediated MCP) and **subscribe** to new messages: turn structured
  candidate actions ("reply needed", "event proposed", "deadline mentioned",
  "task implied") — **inert data with provenance, never instructions** — into
  tasks/reminders. Where a task implies a reply, route AiOSComms's `send`/`reply`
  back through the agent, **confirmation-gated** (AiOSOrg sends nothing itself).
  This realizes F-TODO's "capture tasks from email/meetings".
- `O-VIEW` — the **canvas widget**: tasks / projects / habits / an agenda
  ("what's due / next") rendered into an AiOSCanvas-provided surface.
- `O-TOUCH` — touch-first capture, completion, reschedule, and habit check-off —
  keyboard/mouse supported, never required.
- `O-WIDGET` — the AiOSCanvas widget-contract integration (surface, input
  routing, user-scoped state), tracking whatever contract AiOSCanvas exposes.
- `O-STICKY` — **sticky-note absorption**: migrate an existing AiOSCanvas sticky
  note into a lightweight AiOSOrg task/note (path + timing per PLAN Open Question
  3), coordinated with AiOSCanvas.

**Depends on:** AiOSComms's consumer interface (its `X-IF`/`extract_actionable`,
hardened for AiOSOrg in its P6) and the agent's MCP mediation; AiOSCanvas
exposing a widget/surface contract **and** the sticky-note widget. Tracks
AiOSCanvas, like AiOSTerminal's / AiOSCalendar's canvas frontends.

**Exit (= M2 contribution):** with the agent runtime, AiOSCalendar, AiOSComms,
and AiOSCanvas, AiOSOrg captures tasks from communication, schedules them against
the calendar, renders tasks/habits/agenda on the canvas under touch, and has
absorbed the sticky notes. Daily-driver-usable as the unified task surface.

## P5 — Voice & Interaction · *moderate* · contributes to M2/M3

**Goal:** voice-driven organizing on the canvas, riding the parent voice stack.

- `O-VOICE` — capture and complete by voice ("remind me to call the dentist
  tomorrow", "mark the groceries done", "what's on my list today", "schedule an
  hour to write the report"); AiOSOrg surfaces intent feedback. The hard
  perception work is parent **F-VOICE**; AiOSOrg owns the task/habit intents and
  their confirmation surface (cross-domain intents stay confirmation-gated).

## P6 — Interop & Integration Depth · *moderate* · contributes to M3

**Goal:** round out the integrations against the *real* sibling widgets and
deepen the agent experience.

- `O-INTEROP` — validate and document the **calendar and comms integrations
  end-to-end** against the shipping widgets: the `external_refs` links, the
  free/busy-driven scheduling path, and the `extract_actionable` → task path,
  re-reviewing each sibling's consumer interface against AiOSOrg's actual needs
  before either side freezes.
- `O-AGENT+` — richer proactive surfacing and prioritization (F-TODO's "surfaces
  them proactively"): what to nudge, when, and how, under F-AGENT-POLICY.
- `O-EXPORT` *(if pursued)* — task/checklist export for non-AiOS interchange
  where it composes with parent **F-INTEROP** (e.g. emitting a to-do list into a
  document); a minor, optional surface.

**Depends on:** the shipping AiOSCalendar and AiOSComms widgets; parent
F-AGENT-POLICY maturity.

## P7 — Hardening & Polish · *moderate* · **→ M3**

**Goal:** alpha-quality organizer — the AiOS milestone **M3** contribution.

- **Security review of the captured-content path** against F-PROMPT-SAFETY: a
  task captured from a message is data, never an instruction; the agent never
  takes a cross-domain action from ingested content without confirmation and
  provenance. AiOSOrg participates in the parent P7 red-team exercise on the
  capture/decomposition path.
- Performance: fast recurrence expansion, habit/streak computation, and agenda
  rendering over large/long-lived task sets.
- Accessibility passes; recovery / session-restore behavior.
- **Stabilize the consumer interface at 1.0** — the documented, versioned
  contract any future consumer ships against, with compatibility guarantees
  fixed.
- Validation on physical touch hardware (dev VMs cannot surface real touch — the
  parent/AiOSCanvas posture).

## Cross-component dependencies

- **P0–P2 are self-contained on AiOSOrg's side** — the headless core, the
  consumer interface, and (with the parent agent runtime) the MCP control plane
  and decomposition depend only on AiOSOrg and the parent agent. No *sibling
  widget* gates them. (This is deliberate: the local organizer must not wait on
  the calendar or comms widgets.)
- **P2 depends on the parent AiOS agent runtime** (decomposition, MCP).
- **P3 (AiOSCalendar integration) depends on AiOSCalendar's consumer interface**
  (its four-verb `K-IFACE`/`K-MCP`) **and the agent's MCP mediation** — consumed,
  agent-mediated, not linked in-process (F-MCP).
- **P4 (AiOSComms integration) depends on AiOSComms's consumer interface**
  (`extract_actionable`/subscribe, `X-IF`) **and the agent's MCP mediation**;
  **P4 (canvas widget + sticky-note absorption) depends on AiOSCanvas** exposing a
  widget contract and the sticky-note widget — the same canvas dependency
  AiOSTerminal and AiOSCalendar carry.
- **AiOSVault / F-STORAGE** — the at-rest envelope the store reuses (ideally a
  shared crypto crate; coordinate with AiOSVault/AiOSComms). **AiOSFSS** is
  headless and imposes no dependency; it merely syncs the encrypted store between
  devices (the store is ciphertext at rest for exactly that reason), with runtime
  state kept outside the synced tree.
- **AiOSOrg depends on its siblings; they do not depend on AiOSOrg.**
  AiOSCalendar and AiOSComms each named AiOSOrg as their downstream consumer and
  freeze their interfaces at 1.0 (AiOSCalendar P7; AiOSComms P6) for exactly this
  reason. AiOSOrg is the **last** widget — the consumer at the end of the chain.

## Change Log

| Date | Change | Rationale |
|------|--------|-----------|
| 2026-05-22 | Initial roadmap created | Repository kickoff; phases aligned to the parent AiOS milestones M1/M2/M3 — local task/habit/recurrence core first (P0–P1), agent + MCP + decomposition (P2), then the *outbound* integrations consumed via MCP and agent-mediated: AiOSCalendar (P3) then AiOSComms + canvas widget + sticky-note absorption (P4 → M2), voice (P5), interop depth (P6), hardening + 1.0 consumer interface (P7 → M3). AiOSOrg is the last widget and not on the M1 critical path; its defining couplings point outward to two in-flight siblings whose interfaces are frozen for it. |
