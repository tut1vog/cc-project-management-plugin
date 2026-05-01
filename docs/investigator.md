# investigator

A research specialist agent that investigates technical questions before action — bug root causes, feature design decisions, library or pattern choices, architectural alternatives — and reports a structured findings document. It does not implement; the fix or feature lands as a separate task on a different agent.

The procedure is owned by the `prior-art-research` skill, which the agent loads on every dispatch. Any agent (not just `investigator`) can load the same skill to follow the same procedure when full sub-dispatch is overkill.

## What it does

- Reads the dispatch prompt or user request to extract the question's shape — for bugs: observed/expected behavior, repro steps, affected files; for feature/design: desired outcome, decided constraints, area of code touched, what an acceptable answer looks like
- Reads the implicated source in full and walks the git history of those files (`git log`, `git blame`, `git show`) to correlate regressions with commits or to understand how the area has evolved
- For bugs: reproduces when feasible — may add temporary instrumentation (extra logs, `print` statements, scratch scripts) to gather runtime evidence
- Optionally researches prior art on GitHub and the web — `WebSearch`, `WebFetch`, and the `gh` CLI through `Bash` when the user has it authenticated — and cites every external finding by URL
- Synthesizes evidence into a single findings statement: most likely root cause for bugs, recommended approach with rationale for feature/design questions
- For bugs only: classifies the cause as **logic** (a localized defect) or **architectural** (a structural cause crossing module boundaries)
- States a confidence level and what would raise it
- Reverts every instrumentation edit before reporting (`git status` must show a clean tree)
- Emits a structured Investigation Report that director pastes verbatim into the `Task:` journal entry

## Usage

Invoke the investigator by name in Claude Code, or let director dispatch it for an investigation-shaped task in a plan.

### Direct invocation

Bug case:

```
Use investigator to find why /api/users returns an empty array when the database has 12 users
```

Feature/design case:

```
Use investigator to research which Python testing framework fits this project before we start writing tests
```

### Director-dispatched

When director is planning a phase that includes "diagnose the cause of X before fixing it" or "research approach Y before implementing", it auto-selects investigator from `.claude/agents/` based on the agent's `description`. Investigator runs, reports back, and director then plans a separate fix or implementation task on a different agent — for architectural causes or significant design choices, presenting the proposed phase to the user before proceeding.

## Modes

Three modes, run in order, plus a closeout. Mode 2 is optional. The detailed procedure lives in the `prior-art-research` skill.

| Mode | What it covers |
|------|----------------|
| 1. Understand | Extract the question's shape from the request; read implicated source; walk git history; for bugs: reproduce when feasible (instrumentation allowed, reverted later) |
| 2. Research (optional) | Look for prior art via `WebSearch`, `WebFetch`, `gh` CLI; cite every finding by URL. Skipped when Mode 1 already produces a confident answer |
| 3. Synthesize | State the finding in 1–3 sentences; for bugs: classify as `logic` or `architectural`; state confidence and what would raise it |

After Mode 3, the agent reverts every instrumentation edit and emits the report.

## Report format

The agent always emits this exact structure:

```
## Investigation Report

**Topic**: <what was investigated — bug, feature design, library choice, architectural question>
**Context**: <why this matters; the calling agent's goal in 1–2 sentences>
**Reproduction / Current state**: <for bugs: repro steps or "not reproduced — evidence from <source>". For features/design: brief snapshot of the relevant existing code or "greenfield">
**Evidence**:
- <fact with file:line ref>
- <fact with file:line ref or git sha>
- <external citation by URL — only when Mode 2 ran>
**Findings**: <synthesized conclusion in 1–3 sentences. For bugs: most likely root cause. For features/design: recommended approach with rationale.>
**Classification** (bugs only — omit for non-bug topics): logic | architectural
**Confidence**: high | medium | low — <one sentence on what would raise it>
**Recommended next step**: <concrete next action for the calling agent>
**Working tree**: clean (all instrumentation reverted)
```

## When to use this vs. other agents

| Situation | Agent |
|-----------|-------|
| **Diagnose a bug — find the root cause without fixing it** | investigator |
| **Research a feature or library choice before implementing** | investigator |
| **Plan and execute the fix or feature once the answer is known** | director (which dispatches an implementer subagent) |
| **Multi-step implementation work** | director |
| **Project setup, review, or goal change** | scaffolder → director |

## Tips

- **It does not implement.** Even when the cause is a one-line logic error or the recommended library is obvious, investigator stops at the report. Director plans the fix or feature as a separate task on a different agent — this preserves the same separation-of-concerns principle that keeps implementation and tests on different agents.
- **It does not commit.** Like every subagent in the pipeline, only director writes to git history.
- **The working tree is always clean when it finishes.** Instrumentation is a means, not an artifact. If you want runtime logging to stay in the codebase, that is a separate fix task with its own verification.
- **Architectural causes get escalated, not patched.** When investigator classifies a bug cause as architectural, the Recommended next step asks director to propose a fix phase to the user. This avoids landing a structural change without explicit review.
- **Significant design recommendations get user review.** For feature/design topics where the recommendation reshapes scope or commits to a non-trivial dependency, investigator's Recommended next step asks director to bring the choice back to the user before any implementation.
- **Citations are mandatory in Research mode.** External findings without a URL do not count as evidence.
- **The skill enforces a hard gate.** If the question is trivial — a typo, an obvious off-by-one, a feature with clear in-codebase precedent — the agent exits without running the procedure and reports the answer directly. Don't worry about over-invoking; the gate filters at runtime.
