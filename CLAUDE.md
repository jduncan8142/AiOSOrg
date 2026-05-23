# CLAUDE.md — AiOSOrg

**Status: planning / not started.** No code exists yet; this repo is a placeholder for a future AiOS widget.

## What this is
AiOSOrg is a planned **canvas widget** for AiOS — an AI-first, Linux-based OS whose desktop is a free-form canvas of widgets rendered by AiOSCanvas (a Smithay/Wayland compositor). Parent project: https://github.com/jduncan8142/AiOS .

AiOSOrg is the **organizational** widget: a complete calendar + to-do + task-management system. It records to-dos and lets the AI help break them down into smaller, completable tasks. It includes habit tracking and recurring reminders/tasks with complex scheduling. It is tightly coupled with the Calendar (AiOSCalendar) and Communication (AiOSComms) widgets, and the existing canvas sticky notes fold into it over time.

## Scope (from the AiOS PARKING_LOT — "Planned Applications & Widgets")
- Complete organizational system: calendar + to-do + task management in one canvas widget.
- Records to-dos; the AI helps break them into smaller, completable tasks.
- Habit tracking plus recurring reminders/tasks with complex scheduling.
- Tightly coupled with AiOSCalendar and AiOSComms; existing canvas sticky notes fold into this over time.

## Conventions inherited from AiOS
- Integrates as a canvas widget hosted by AiOSCanvas; authoritative cross-project planning docs live in the parent AiOS repo.
- License: Apache-2.0 (AiOS-wide).
- Language/stack: **TBD at kickoff**.
- Security posture: all external/inbound content (emails, messages) is untrusted per AiOS F-PROMPT-SAFETY; design accordingly later.

## Next steps (when we start)
- Decide stack + architecture; write PLAN/ROADMAP; scaffold; wire in as an AiOS component. For AiOSComms, do a scoping pass on the integration matrix first.
