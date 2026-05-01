---
name: discovery-grilling
description: Refuse vague answers and probe cross-phase contradictions during scaffolder's Discovery interview, until each substantive answer is precise enough that director can plan against it without asking back. Loaded explicitly at the start of Discovery Phases 1, 2, 3, and 6 — the phases where a sloppy answer corrupts the Requirements Summary. Triggers on "discovery grilling", "grill discovery", "grill the user", "interrogate the answer".
user-invocable: false
---

A grilling procedure for scaffolder's Discovery interview. The bar is a Requirements Summary precise enough that director can decompose each item into a phase plan without asking the user back. Discovery is the one shot to spend research budget — searches done here amortize across every task director executes.

## When grilling applies

Scaffolder loads this skill at the start of Phases 1, 2, 3, and 6 — the phases where vague answers propagate downstream (a sloppy Goal, a wrong stack, a generic rule file). Phases 4, 5, and 7 are procedural and skip grilling.

In a grilled phase, grilling applies to **substantive answers**:

- Content the user introduces or modifies of their own accord.
- A hedged confirmation of a Phase 0 finding ("more or less", "I think so", "sounds right").

Grilling does **not** apply to a bare "yes" that ratifies a Phase 0-evidenced fact — Phase 0 already verified it, and re-asking punishes the user for scaffolder having done its homework.

In Phase 6 grilling runs in two directions:
- **Refuse vague user requests** ("just add a testing rule" → "what test framework, what's the trigger to read the rule, what project-specific content fills it?").
- **Refuse vague scaffolder proposals** — every rule file you propose must have an exact path, a one-line trigger, and project-specific content drawn from Phase 0 / earlier answers. If you can't fill it, don't propose it.

## What counts as "vague enough to re-ask"

Use the five diagnostic types to spot vagueness, and the director-plan bar to judge whether to advance.

**Diagnostic types**

1. **Quantitative gap** — "fast" / "scalable" / "secure" without numbers, thresholds, or threat model.
2. **Scope gap** — "add some features" / "improve UX" without naming which.
3. **Stakeholder gap** — "the team" / "users" without roles, headcount, or who decides.
4. **Operational gap** — "deploy to cloud" / "use a database" without saying which / how.
5. **Hand-waved rationale** — "it's the standard choice" / "we always do it this way" with no source.

**Director-plan bar (the test for advancing)**

> Could director write a phase plan from this answer without asking the user back?

If yes, confirm understanding aloud and advance. If no, follow up.

## Termination

For each substantive answer:

1. Clears the director-plan bar → confirm understanding aloud, advance.
2. Vague (matches a diagnostic type, or fails the bar) → ask one sharper follow-up.
3. Still vague after the follow-up:
   - **Public-knowable gap** (library currency, standard wording, ecosystem norm) → search (WebSearch / WebFetch) to resolve before re-asking the user. Verify any concrete claim the user makes about a library, framework version, standard, or ecosystem norm; required before accepting it as Requirements Summary content.
   - **Private gap** (the user's own intent, team structure, internal SLA) → continue follow-ups.
4. **Loop-breaker.** After each follow-up, ask yourself silently: *did this produce new precision?* If two consecutive follow-ups produce no new precision, stop pushing and offer park-it:

   > "I can't sharpen this without more info — want to park it as a known unknown, or keep digging?"

   Park-it lands in Phase 2's known-unknowns block in the Requirements Summary, scoped to which phase the unknown belongs to. Park-it is offered only after at least one follow-up has run; the user must actively accept the park.

## Cross-phase contradiction probing

While grilling Phase 2, 3, or 6, actively check incoming answers against earlier-phase answers. When a later answer contradicts a Phase 1 / 2 / 3 answer, surface the contradiction explicitly and ask the user which side wins:

> "You said in Phase 1 that the primary users are non-technical end consumers, but you just described a CLI for engineers. Which is the real user — and should I revisit Phase 1?"

Whichever side the user picks, patch the *losing* side in the running Requirements Summary. Do not silently flag and do not silently re-walk full phases.

## Worked example

> **Phase 2, Goal item:** "Add a billing dashboard."
>
> **Vague (scope gap).** Follow-up: "Which views does the dashboard need to expose, and which actions can a user take from it? What's the smallest version that's useful in production?"
>
> **User:** "Show monthly revenue and let admins issue refunds."
>
> **Director-plan check.** Can director plan against "monthly revenue + admin refund flow"? Almost — but "admin" is a stakeholder gap.
>
> **Follow-up:** "Who counts as an admin — anyone in the team workspace, or a specific role? Does the refund need an approval step?"
>
> **User:** "Just users with the `billing-admin` role. No approval flow yet."
>
> **Advance.** The Goal item is now precise enough for director: "Build dashboard with a monthly-revenue panel and a refund action gated on the `billing-admin` role; no approval workflow."
