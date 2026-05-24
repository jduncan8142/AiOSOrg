# ADHD-Informed Habit-Tracking & Task-Decomposition — Design Notes

**Status:** Research / design-input reference. **Not** a PLAN or ROADMAP change.
**Audience:** input for AiOSOrg's eventual `DESIGN.md` (P0/P1 habit + P2 decomposition work).
**Scope:** the two AiOSOrg surfaces the plan explicitly marks *ADHD-informed* — **habit
tracking** (`O-HABIT`, PLAN Q4) and **agent-assisted task decomposition** (`O-AGENT`,
PLAN Q5). Everything here fits *inside* the already-decided architecture
([PLAN.md](../PLAN.md), [ROADMAP.md](../ROADMAP.md)); nothing here reopens a decision.

> **This is informational design research, not medical advice.** It summarizes public
> ADHD literature (CHADD, ADDitude/ADDA, peer-reviewed sources) to inform product
> design. It is not clinical guidance, not a diagnosis aid, and not a treatment
> recommendation. AiOSOrg is a personal organizer, not a medical device. Jason reviews;
> **do not merge as a PLAN change.**

---

## 0. Why ADHD is the right design lens for these two features

AiOSOrg's plan names habit-tracking and decomposition as *ADHD-informed* deliberately.
The reason is that the executive-function profile associated with ADHD is the
**stress-test** for an organizer: a design that works for someone with task-initiation
difficulty, time-blindness, and weak working-memory externalization will work for
everyone, while the inverse is not true. The goal is **universal-design-via-the-hardest-
case**, not a separate "ADHD mode." Concretely, three findings drive every
recommendation below:

1. Difficulty *starting* tasks is neurological, not motivational
   ([Child Mind Institute][cmi], [Grow Therapy][grow]).
2. The future is hard to *feel* — "now vs. not now" ([ADDitude][add-warp],
   [ADDA][adda-tb]) — so anything not surfaced *now* effectively does not exist
   ([Simply Psychology][sp-op]).
3. All-or-nothing streaks actively *backfire* — a broken streak triggers abandonment
   ([Klarity][klarity]).

The plan's existing choices already lean the right way (uhabits as the habit model,
forgiving streaks as an open DESIGN.md question, a *conservative-by-default* sliding
scale for decomposition). This memo gives those choices an evidence base and turns them
into concrete recommendations.

---

## Part A — The ADHD research base (with sources)

### A.1 Task initiation & "task paralysis"

Task initiation is itself an executive function, and it is one of the functions most
affected in ADHD. The distinction that matters for design: **task paralysis is not
procrastination.** Procrastination is a *choice* to delay; ADHD task paralysis is an
involuntary inability to begin despite *wanting* to, driven by executive dysfunction
around planning, prioritizing, and starting ([Grow Therapy][grow],
[Child Mind Institute][cmi]). Executive dysfunction is "one of the strongest predictors
of functional impairment in adults with ADHD" — more so than inattention or
hyperactivity alone ([Grow Therapy][grow]).

A frequent *trigger* of the freeze is a task that is too large or too vaguely defined to
have an obvious first move. The literature's consistent answer is to shrink the unit of
work until the first action is trivially startable (Part A.6).

**Design consequence:** the organizer's job is to lower *activation energy* — always
present a single, tiny, concrete **next action**, never a wall of obligations.

### A.2 Time-blindness ("now vs. not now")

ADHD is associated with difficulty sensing elapsed time and estimating how long tasks
take ([ADDA][adda-tb]). Time tends to "collapse into two categories: now and not now" —
if something isn't happening *right now* it lives in "a vague future that feels equally
distant whether it's in 20 minutes or two weeks" ([ADDitude][add-warp]). This is
compounded by **temporal discounting**: the future is steeply devalued, so planning
toward it is hard "when it's nowhere to be seen" ([ADDitude][add-warp]).

CHADD frames the closely related failure as **prospective memory** — "remembering to
remember." It distinguishes **time-based** prospective memory ("do X at 3pm") from
**event-based** ("do X when Y happens"), and notes time-based is *harder* for ADHD
brains because it depends on monitoring the clock ([CHADD prospective memory][chadd-pm]).

**Design consequences:**
- Make time **visible/concrete**, not abstract (countdowns, "due in 2h", relative
  phrasing) — externalize time ([ADDitude][add-warp], [ADDA][adda-tb]).
- **Prefer event/context cues over bare clock times** where possible ("when you start
  your day", "after the standup") — convert time-based reminders into event-based ones
  ([CHADD prospective memory][chadd-pm]).
- Buffer estimates; people under-estimate duration ([ADDA][adda-tb]).

### A.3 Object permanence / "out of sight, out of mind"

Colloquial "ADHD object permanence" is not a literal Piagetian deficit; it's that the
ADHD brain tends to **stop thinking about** things that aren't visible or immediately
rewarding, rooted in working-memory and dopamine-signaling differences
([Simply Psychology][sp-op], [Talkiatry][talk-op]). A task that scrolls off-screen is, in
effect, a task that no longer exists.

**Design consequence:** **surfacing is the feature.** A task or habit that the user has
not yet acted on must be *re-presented*, not silently filed. This is the strongest
argument for AiOSOrg's proactive-agenda/"what's next" surface and for the agent's
proactive-surfacing role (`O-AGENT+`, F-ORG "surfaces them proactively"). "Out of sight"
must be engineered *out* of the default experience.

### A.4 Dopamine, reward timing & delay aversion

ADHD reward processing is characterized by **delay aversion** and steep **temporal
discounting**: a strong, affectively-driven pull toward *smaller-sooner* rewards over
*larger-later* ones, grounded in fronto-striatal dopamine circuitry
([Marx et al. 2021 meta-analysis][marx], [Sonuga-Barke delay-aversion][sb-da]). Building
a habit whose payoff is weeks away is therefore *intrinsically* hard — the reward signal
arrives too late to reinforce the behavior.

The practical countermeasure from the habit literature: deliver an **immediate** reward
on completion; for an ADHD brain the reinforcement "should be just as stimulating and
accessible" as competing options or it won't stick ([ADDA break-habits][adda-habits]).
Breaking a task into smaller steps also helps precisely because each completed micro-step
yields a *more frequent burst* of dopamine ([NeuroLaunch][nl-break]).

**Design consequences:**
- **Immediate, visible positive feedback** on every check-off and micro-step completion
  (the satisfying tick, an *advancing* consistency score) — reward must be *now*.
- Smaller units → more frequent wins (ties directly to decomposition, Part A.6).
- Lean on *progress/consistency* framing rather than a far-off goal.

### A.5 Working-memory externalization, implementation intentions & body-doubling

- **Externalization.** The recurring prescription across the literature is to *offload*
  working memory to the environment: "substitute it with external reminders instead of
  relying on 'I'll remember'" ([Medical News Today / CHADD synthesis][mnt-bd],
  [CHADD prospective memory][chadd-pm]). The organizer *is* this external store.

- **Implementation intentions ("if-then").** Pre-committing a cue→action plan ("*if* it's
  8am, *then* I meditate") gives the brain "a script to follow" at the trigger moment
  ([ADDA break-habits][adda-habits]); if-then plans have evidence for improving
  inhibitory control in children with ADHD ([ADDA break-habits][adda-habits]). **Important
  nuance:** a controlled study (van Timmeren & de Wit, 2022) found if-then plans *aid
  initiation* but can *reduce behavioral flexibility* and outcome-learning, cautioning
  against rigid if-then scripting "in complex, dynamic situations" ([PMC10585941][vt]).
  → Use if-then framing to help *start*, but keep it **flexible**, not a rigid rule whose
  violation reads as failure. This is independent evidence *for* forgiving designs.

- **Body-doubling.** Working alongside another presence helps ADHD users start and
  sustain a task; it "externaliz[es] structure", makes the task "feel smaller", and
  anchors shared pacing that helps with time-blindness ([CHADD body-doubling][chadd-bd],
  [Cleveland Clinic][cc-bd]). The supportive AI agent is a natural **software
  body-double** — present, co-pacing, non-judgmental (Part C).

### A.6 ADHD-informed task decomposition (micro-steps, next-action, the 2-minute rule)

Breaking work down is the single most-recommended ADHD task strategy. Make each step
"ridiculously small (e.g., 'open laptop' not 'write essay')" — small enough to "bypass
our brain's resistance to getting started", yet substantial enough to feel like progress
([Shimmer][shimmer], [NeuroLaunch][nl-break]). Surface **the next action, not the whole
task** (the GTD "next action" idea). Decomposition helps because it gives "more frequent
bursts of dopamine", reduces working-memory load, and reduces "information overload …
and processing bottlenecks" ([NeuroLaunch][nl-break]).

The **2-minute rule** (David Allen, *Getting Things Done*): if a step takes under ~2
minutes, do it now ([myPatientAdvice][mpa-2min], [Inflow][inflow-2min]). Useful as both a
*sizing target* for micro-steps and a nudge heuristic.

**Crucial caveat — over-decomposition is its own failure mode.** GTD in full is "a lot to
remember" and can itself overwhelm ADHD users ([WorkBrighter][wb-gtd]); shattering every
to-do into noise is worse than none (already flagged in PLAN *Risks*). The design lever
is *tunable aggressiveness* with a conservative default (Part C).

### A.7 Broken streaks demotivate — design for forgiveness

This is the load-bearing finding for the habit UI. Traditional streaks assume "missing
one day means you've 'broken the streak'" — an all-or-nothing model that "ignores the
executive function challenges" of ADHD. A broken streak becomes "a visible record of
failure" that can trigger Rejection Sensitive Dysphoria, "self-blame, avoidance, and often
permanent app abandonment" ([Klarity][klarity]). Because a miss is "practically inevitable
given … executive functioning challenges", a brittle streak is *engineered* to eventually
fail the user ([Klarity][klarity]).

The recommended alternatives, consistently: **flexible scheduling** ("aim for 4 of 7
days"), **cumulative** rather than strictly-consecutive tracking, **optional** streak
visibility, **easy restart**, "celebrate partial progress", and explicitly "fram[e]
missed days as data rather than failure" ([Klarity][klarity], [AFFiNE][affine]). Some
ADHD-oriented trackers drop streaks entirely in favor of weekly targets with "no streaks,
no resets, and no guilt" ([Klarity][klarity]).

**Design consequence:** the primary habit signal must be forgiving (a smoothed
consistency score, not a brittle chain); a miss must read as *neutral data*; skips must be
first-class. uhabits already embodies this (Part B), which is why the plan picked it.

---

## Part B — uhabits (Loop Habit Tracker), specifically

AiOSOrg's plan names [uhabits](https://github.com/iSoron/uhabits) as the habit-model
inspiration (PLAN Q4). Three things make it the right reference, and all three are
ADHD-relevant.

### B.1 "Habit strength" / score — a forgiving, non-binary signal

uhabits does **not** show a brittle streak count as its primary metric; it computes a
**habit strength score (0–100%)** via **exponential smoothing** — a weighted average over
*all* repetitions since the habit began, weighting recent ones more heavily
([uhabits README][uh-readme], [discussion #580][uh-580]).

- The score's defining property, in the project's own words: *"Every repetition makes your
  habit stronger and every missed day makes it weaker"*, **but** *"a few missed days after
  a long streak … will not completely destroy your progress"* ([uhabits README][uh-readme]).
  This is the antithesis of don't-break-the-chain.
- **Frequency** for non-daily habits = repetitions ÷ interval length (e.g. 3 reps in 8
  days → 0.375); the score is normalized against the *target* frequency so over-doing a
  habit doesn't inflate it ("completing a weekly habit twice per week doesn't make your
  score improve any faster") ([discussion #580][uh-580], [FAQ #689][uh-689]).
- Progression for a perfectly-kept **daily** habit: ~80% after one month, 96% after two,
  99% after three; non-daily habits climb proportionally slower (every-other-day ~2
  months to 80%, weekly ~7 months) ([uhabits README][uh-readme], [FAQ #689][uh-689]).

> *Implementation note:* the exact smoothing constant lives in uhabits' `Score`
> source and shifts across releases; the **documented behavior** above (exponential
> smoothing, frequency = reps/interval, normalized to target) is what AiOSOrg should
> reproduce, not a hard-copied magic number. The deterministic score computation is a
> prime `proptest` target (PLAN — `O-HABIT` "pure and tested").

### B.2 Flexible scheduling ("3×/week" vs daily)

uhabits supports "habits with more complex schedules, such as 3 times per week or every
other day" ([uhabits README][uh-readme]) — you specify *how many times per interval*,
not a rigid daily checkbox ([FAQ #689][uh-689]). This maps **exactly** onto AiOSOrg's
decided `aios-recurrence` crate, which the plan already says must handle *both* RRULE
*and* "habit-style cadences RRULE can't express ('3×/week', 'every other day')" (PLAN Q2).
No new mechanism is needed — the cadence vocabulary is already specified.

### B.3 The non-punitive check / **skip** model

uhabits has three states, not two: **completed**, **not-completed**, and an explicit
**skip** (`-`). A skip *"leaves the score unchanged and doesn't break the streak"* — it's
for when completing was impossible "due to circumstances beyond your control"
([FAQ #689][uh-689]). This single primitive is the most important ADHD-friendly mechanic
to carry over: it converts a "miss" from a *failure event* into *neutral data*, which is
exactly the reframing the ADHD habit-tracker literature calls for (A.7, [Klarity][klarity]).

### B.4 Why uhabits is the right reference for ADHD

The three properties above line up one-to-one with the ADHD findings: the smoothed score
answers delay-aversion/immediate-reward (A.4) and broken-streak demotivation (A.7);
flexible "X per interval" cadences answer cumulative-not-consecutive tracking (A.7) and
map onto the decided `aios-recurrence` vocabulary (PLAN Q2); and the explicit skip turns a
miss into neutral data (A.7). The plan's choice of uhabits (PLAN Q4) is therefore
well-founded — Part C carries these into AiOSOrg's decided model.

---

## Part C — Concrete AiOSOrg recommendations

These translate Parts A/B into the decided architecture. Each is tagged with the ROADMAP
item it informs. **All are DESIGN.md inputs — none change PLAN/ROADMAP.**

### C.1 Habit tracking → `O-HABIT`, `O-STORE`, `O-RECUR` (P1)

**(R1) What the completion-tracking store records.** Per the plan, a habit is a `Task`
flagged for habit-tracking plus an underlying completion store (PLAN Q4). Recommended
`HabitLog`-style record, per habit-flagged task:
- one row per *occurrence* with a tri-state outcome — **`completed` / `missed` /
  `skipped`** (the uhabits model, B.3) — never just a boolean. `skipped` is first-class.
- `timestamp` of the check-off (for "when", and for honest streak math across timezones),
- the resolved **target cadence** at that time (from `aios-recurrence`, so historical
  scoring is stable even if the cadence later changes),
- optional lightweight `value` slot for *measurable* habits (e.g. "8 glasses",
  "20 min") — uhabits supports numeric habits; cheap to allow now, avoids a migration.
- provenance (user check-off vs. agent-confirmed), per the model's provenance tenet.

This is enough to compute everything in R2/R3 and nothing more — keep it minimal.

**(R2) A uhabits-style consistency/strength score — not a raw streak.** Make the
*primary* habit signal an **exponential-smoothing "consistency" score (0–100%)** computed
from the completion log, reproducing uhabits' *documented behavior* (B.1): every
completion strengthens, every miss weakens gently, a few misses after a long run don't
"destroy progress", frequency = completions ÷ interval normalized to the target cadence.
Rationale: directly answers the delay-aversion/immediate-reward finding (A.4) — the score
*moves now*, every day — and the broken-streak finding (A.7). Computation is pure →
`proptest` + golden fixtures (matches `O-HABIT` "pure and tested"; mirrors how
`aios-recurrence` is vetted).

**(R3) Forgiving streaks via `aios-recurrence` cadences + skips.** If a streak/count *is*
shown, make it forgiving by construction:
- **Flexible cadence is the unit.** "3×/week" is satisfied by *any* 3 days that week — a
  Tuesday miss is irrelevant if Mon/Wed/Fri land. This is the cumulative-not-consecutive
  recommendation (A.7) and it's already expressible in `aios-recurrence` (B.2, PLAN Q2);
  **no new recurrence mechanism is required.**
- **Skip = neutral data.** A `skipped` occurrence leaves the score unchanged and does not
  break a streak (B.3, [uhabits FAQ][uh-689]). Surface skip as a *first-class, low-
  friction* action (one tap / "skip today, life happens"), not a buried option.
- **Default to consistency %, make raw streaks opt-in.** Per the "optional streak
  visibility" guidance (A.7). The headline number should be the forgiving score; an
  all-or-nothing consecutive count, if present, is secondary and dismissible.
- **A missed occurrence is "data," never a scolding.** Copy/visual frames a miss
  neutrally and offers the next occurrence — never a "you broke your 30-day streak!"
  ([Klarity][klarity]). *(Streak edge cases — skip-without-breaking, missed-occurrence
  handling — are explicitly a DESIGN.md item per PLAN; these are the recommended
  semantics.)*

**(R4) ADHD-tuned surfacing & reminders (time-blindness + object-permanence).** This is
where object-permanence (A.3) and time-blindness (A.2) bite hardest:
- **Surface, don't file.** A due/missed habit must re-appear on the agenda/"what's next"
  surface, not silently slip past (A.3). Out-of-sight = out-of-mind is the default to
  *defeat*. This is the agent's proactive-surfacing role (`O-AGENT+`).
- **Prefer event/context cues over bare clock times.** "After your morning standup" beats
  "9:15am" for time-based-prospective-memory reasons ([CHADD prospective memory][chadd-pm],
  A.2). Where scheduling *is* clock-anchored, that block is a **calendar event** (PLAN Q8)
  — Org asks the agent to read it back, it doesn't store the instant.
- **Make time concrete.** Relative, shrinking phrasing ("due in 90 min", "2 left this
  week") over abstract timestamps (A.2, [ADDitude][add-warp]).
- **Support if-then framing for habit cues** ("*when* X, *then* this habit") — helps
  initiation (A.5) — but keep it a *flexible prompt*, not a rigid rule whose miss = failure
  (the van Timmeren flexibility caveat, [PMC10585941][vt]).

**(R5) The agent's supportive, *non-nagging* role.** The literature on body-doubling
(A.5) and on shame-driven abandonment (A.7) converges on one tone requirement: **present
and encouraging, never punitive.** Recommendations:
- Frame the agent as a **software body-double** — co-pacing and available, not a
  taskmaster ([CHADD body-doubling][chadd-bd]).
- **Celebrate completions immediately** (the immediate-reward finding, A.4); **respond to
  misses neutrally** ("want to skip today, or do a 2-minute version?").
- **Rate-limit nudges** and respect quiet hours — repeated nagging feeds avoidance.
  Proactivity is governed by **F-AGENT-POLICY** (`O-AGENT+`), so this is a *policy dial*,
  not hardcoded.
- Offer a **shrink-it escape hatch** on a stalled habit/task ("just do the 2-minute
  version") rather than letting it rot off-screen — combines A.6 and A.3.

### C.2 Decomposition sliding scale → `O-AGENT` (P2), under F-AGENT-POLICY

The plan fixes the *mechanism*: decomposition aggressiveness is a **user-controlled
sliding scale** with a **global default + per-task overrides**, governed by
F-AGENT-POLICY, bundled with habit-tracking by default (PLAN Q5). This memo recommends
**what the levels concretely mean** and how the ADHD findings shape each.

**(R6) Concrete low→high semantics.** Recommended three-or-four-stop scale (DESIGN.md
finalizes the exact stops):

| Level | Agent behavior | When it acts | ADHD rationale |
|------|----------------|--------------|----------------|
| **Off** | Never decomposes; user breaks tasks down by hand. | — | Full user control; some users find *any* auto-suggestion overwhelming ([WorkBrighter][wb-gtd]). |
| **Low (on-request)** | Decomposes **only when the user explicitly asks** ("break this down"). | User-initiated only | Respects autonomy; zero unsolicited noise. The user's request is *trusted direct input* (PLAN Q7). |
| **Mid (propose-then-confirm)** — *recommended default* | When a task looks coarse/stalled, the agent **proposes** a breakdown and **waits for confirmation** before writing subtasks. | Agent-suggested, user-approved | Lowers activation energy (A.1) without shattering everything into noise; user stays in command. Conservative default matches PLAN *Risks*. |
| **High (proactive next-step nudges)** | Agent proactively decomposes coarse tasks **and** surfaces the **single next action** at the right moment. | Agent-initiated | Maximal scaffolding for task-paralysis (A.1) + object-permanence (A.3); only for users who *want* a strong body-double. |

Defaulting to **Mid** honors the plan's "ship the default conservative and let the user
dial it up" posture (PLAN *Risks*).

**(R7) Per-task override.** A given task can pin its own level regardless of the global
default — e.g. a fuzzy dread-task ("do taxes") set to **High** while the global stays
**Mid**; a simple errand forced to **Off**. The plan already routes this through the
interface: decompose drafts "carry the requested aggressiveness; the resolved policy is
read from the agent-policy layer" (PLAN — consumer-interface notes). No new plumbing.

**(R8) ADHD-informed micro-stepping + next-action surfacing.** *How* the agent breaks a
task down, whenever it does:
- **Micro-steps sized to start, not to impress.** Aim each leaf at the **2-minute rule**
  where feasible ("open the form", not "do the application") — small enough to defeat
  initiation resistance (A.1, A.6, [Shimmer][shimmer], [Inflow][inflow-2min]).
- **Surface ONE next action, not the whole tree.** The agenda highlights the single next
  leaf; the full subtask tree is available but not in the user's face (GTD next-action;
  combats object-permanence overwhelm, A.3/A.6, [NeuroLaunch][nl-break]).
- **Each completed micro-step is an immediate win** — frequent dopamine bursts are *why*
  decomposition helps the ADHD brain (A.4, A.6, [NeuroLaunch][nl-break]); make the
  completion feedback satisfying and instant.
- **Don't over-shatter.** More steps isn't better past a point; respect the scale and
  don't decompose a task that's already a single concrete action (A.6 caveat,
  [WorkBrighter][wb-gtd]).
- **Decomposition is structure, not a new object** — subtasks under a `parent_id`, the
  same `Task` type with provenance "decomposed by the agent from task X" (PLAN data-model
  tenet 3). Provenance lets the user *see and undo* agent-created subtasks (PLAN *Risks*).

**(R9) F-AGENT-POLICY confirmation + undo.** Agent-created subtasks carry provenance;
writing them is a policy-gated agent action with the standard **confirmation on
irreversible/cross-domain actions + an undo window** (PLAN principle 8, `O-AGENT`).
Proactive *nudge* frequency (the High level) is itself a F-AGENT-POLICY dial so "proactive"
never becomes "nagging" (ties to R5).

**(R10) F-PROMPT-SAFETY guardrails for tasks captured from untrusted content.** This is
the **non-negotiable boundary**, and it intersects decomposition directly:
- A task whose text came from an **email / AiOSComms message is untrusted data, never an
  instruction** (PLAN principle 7, Security). Decomposing such a task = an LLM operating
  on hostile text → it **must** route through the agent's **planner/reader split**
  (F-PROMPT-SAFETY); AiOSOrg does not invoke a model directly on untrusted content
  (PLAN Q7, Security).
- By contrast, **direct keyboard/voice input is trusted** — a to-do the user types or a
  decomposition they explicitly request on their own task **bypasses** the split (PLAN Q7).
  So "Low (on-request)" on a *user-authored* task is trusted; the *same* request on a
  *captured* task still goes through the reader/planner split.
- The classic attack ("create a task to wire money and mark it done" embedded in an
  email) must degrade to *an inert candidate the user sees and dismisses* — never an agent
  action (PLAN Security). Any cross-domain follow-through a decomposed subtask implies
  (send a reply, create a calendar event) stays **confirmation-gated** and routed through
  the owning widget (PLAN Q8, Security).
- **ADHD-relevant tension, called out for DESIGN.md:** the same users who benefit most
  from aggressive proactivity (High) are the ones for whom an un-vetted captured-content
  action is most dangerous (act-now pull, A.4). Recommendation: **aggressiveness scales
  proactivity, never trust** — a High setting may *propose* more, but the
  trusted/untrusted boundary and confirmation gates are *independent* of the slider and
  never relaxed by it.

---

## Part D — Open items to resolve in DESIGN.md (not here)

These are flagged by the plan as DESIGN.md/P0 work; this memo supplies inputs, not
verdicts:

1. **Exact consistency-score formula & constant** — reproduce uhabits' *behavior*
   (B.1); pick/justify the smoothing constant; pin it with `proptest` + golden fixtures.
2. **Precise streak/skip semantics** — the tri-state log, what "forgiving" means
   numerically, missed-occurrence handling (PLAN explicitly defers these; R1/R3 are the
   recommendation).
3. **The exact sliding-scale stops** and the global default value (R6 recommends Mid).
4. **Nudge-frequency policy** under F-AGENT-POLICY (R5/R9) — rate limits, quiet hours.
5. **Reminder cue vocabulary** — how event/context cues (R4) are expressed and where the
   clock-anchored ones live (calendar events, PLAN Q8).
6. **Measurable-habit `value` slot** — include now (cheap, R1) or defer.

---

## Sources

ADHD executive function, task initiation, time-blindness, object permanence:
- [Child Mind Institute — What Is ADHD Paralysis?][cmi]
- [Grow Therapy — Task freeze: understanding ADHD paralysis][grow]
- [ADDitude — How ADHD Warps Time Perception][add-warp]
- [ADDA — ADHD Time Blindness][adda-tb]
- [CHADD — Remembering the Future: How ADHD Affects Prospective Memory][chadd-pm]
- [Simply Psychology — Object Permanence & ADHD][sp-op]
- [Talkiatry — Object Permanence & ADHD: A Psychiatrist Explains][talk-op]

Dopamine, reward timing, delay aversion (peer-reviewed):
- [Marx et al., 2021 — ADHD and the Choice of Small Immediate Over Larger Delayed Rewards: A Comparative Meta-Analysis (*J. Atten. Disord.*)][marx]
- [Sonuga-Barke et al. — Delay Aversion in ADHD: an empirical investigation of the broader phenotype (*Neuropsychologia*, PubMed)][sb-da]

Habit formation, implementation intentions, body-doubling:
- [ADDA — How to Break Bad Habits (if-then / reward timing)][adda-habits]
- [van Timmeren & de Wit, 2022 — Do implementation intentions accelerate habit formation? (*PMC10585941*)][vt]
- [CHADD — ADHD Body Doubling][chadd-bd]
- [Cleveland Clinic — How Body Doubling Helps With ADHD][cc-bd]
- [Medical News Today — Body doubling for ADHD][mnt-bd]

Task decomposition, micro-steps, GTD / 2-minute rule:
- [Shimmer ADHD Coaching — How to Break Down Projects Into Tasks][shimmer]
- [NeuroLaunch — ADHD Breaking Down Tasks][nl-break]
- [myPatientAdvice — The Two-Minute Rule][mpa-2min]
- [Inflow — Get things done with ADHD (2-minute rule)][inflow-2min]
- [Work Brighter — Is the GTD System ADHD-Friendly?][wb-gtd]

uhabits / Loop Habit Tracker:
- [iSoron/uhabits — repository & README][uh-readme]
- [uhabits Discussion #580 — where strength is computed][uh-580]
- [uhabits Discussion #689 — FAQ (score progression, skips, X/week)][uh-689]

Forgiving-streak design for ADHD:
- [Klarity — Breaking the Chain: Why Streak Features Fail ADHD Users][klarity]
- [AFFiNE — ADHD-Friendly Habit Tracker Ideas][affine]

[cmi]: https://childmind.org/article/what-is-adhd-paralysis/
[grow]: https://growtherapy.com/blog/adhd-paralysis/
[add-warp]: https://www.additudemag.com/wasting-time-adhd-and-time-perception/
[adda-tb]: https://add.org/adhd-time-blindness/
[chadd-pm]: https://chadd.org/attention-article/remembering-the-future-how-adhd-affects-prospective-memory-and-how-to-work-with-it/
[sp-op]: https://www.simplypsychology.org/object-permanence-and-adhd.html
[talk-op]: https://www.talkiatry.com/blog/object-permanence-adhd
[marx]: https://journals.sagepub.com/doi/abs/10.1177/1087054718772138
[sb-da]: https://pubmed.ncbi.nlm.nih.gov/18929587/
[adda-habits]: https://add.org/break-bad-habits/
[vt]: https://www.ncbi.nlm.nih.gov/pmc/articles/PMC10585941/
[chadd-bd]: https://chadd.org/adhd-in-the-news/adhd-body-doubling/
[cc-bd]: https://health.clevelandclinic.org/body-doubling-for-adhd
[mnt-bd]: https://www.medicalnewstoday.com/articles/body-doubling-adhd
[shimmer]: https://www.shimmer.care/blog/breaking-down-tasks
[nl-break]: https://neurolaunch.com/adhd-breaking-down-tasks/
[mpa-2min]: https://mypatientadvice.co.uk/knowledge-base/adhd/recognising-symptoms/executive-dysfunction/starting-but-not-finishing-tasks/what-is-the-two-minute-rule-and-how-can-it-help/
[inflow-2min]: https://www.getinflow.io/post/get-things-done-with-adhd
[wb-gtd]: https://workbrighter.co/gtd-adhd/
[uh-readme]: https://github.com/iSoron/uhabits
[uh-580]: https://github.com/iSoron/uhabits/discussions/580
[uh-689]: https://github.com/iSoron/uhabits/discussions/689
[klarity]: https://www.helloklarity.com/post/breaking-the-chain-why-streak-features-fail-adhd-users-and-how-to-design-better-alternatives/
[affine]: https://affine.pro/blog/adhd-friendly-habit-tracker-ideas

---

*Prepared as design-input research for AiOSOrg. Informational, not medical advice.
Not a PLAN/ROADMAP change — for incorporation into a future `DESIGN.md`. Jason reviews.*
