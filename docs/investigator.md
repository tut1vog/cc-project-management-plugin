# investigator

A debugging specialist agent that diagnoses bugs and reports a root-cause hypothesis — without applying the fix. Director dispatches it for diagnose-only tasks; the fix lands as a separate task on a different agent.

## What it does

- Reads the dispatch prompt or user request to extract observed behavior, expected behavior, reproduction steps, and affected files
- Reads the implicated source in full and walks the git history of those files (`git log`, `git blame`, `git show`) to correlate any regression with a commit
- Reproduces the bug when feasible — may add temporary instrumentation (extra logs, `print` statements, scratch scripts) to gather runtime evidence
- Optionally researches prior art on GitHub and the web — `WebSearch`, `WebFetch`, and the `gh` CLI through `Bash` when the user has it authenticated — and cites every external finding by URL
- Synthesizes evidence into a single most-likely cause and classifies it as **logic** (a localized defect) or **architectural** (a structural cause crossing module boundaries)
- States a confidence level and what would raise it
- Reverts every instrumentation edit before reporting (`git status` must show a clean tree)
- Emits a structured Investigation Report that director pastes verbatim into the `Task:` journal entry

## Usage

Invoke the investigator by name in Claude Code, or let director dispatch it for an investigation-shaped task in a plan.

### Direct invocation

```
Use investigator to find why /api/users returns an empty array when the database has 12 users
```

### Director-dispatched

When director is planning a phase that includes "diagnose the cause of X before fixing it", it auto-selects investigator from `.claude/agents/` based on the agent's `description`. Investigator runs, reports back, and director then plans a separate fix task on a fixer agent — for architectural causes, presenting the proposed fix phase to the user before proceeding.

## Modes

Three modes, run in order. Mode 2 is optional.

| Mode | What it covers |
|------|----------------|
| 1. Understand | Extract bug shape from request; read implicated source; walk git history; reproduce when feasible (instrumentation allowed, reverted later) |
| 2. Research (optional) | Look for prior reports of the same symptom — `WebSearch`, `WebFetch`, `gh` CLI; cite every finding by URL. Skipped when Mode 1 already produces a confident hypothesis |
| 3. Hypothesize | Synthesize the cause in 1–3 sentences; classify as `logic` or `architectural`; state confidence and what would raise it |

After Mode 3, the agent reverts every instrumentation edit and emits the report.

## Report format

The agent always emits this exact structure:

```
## Investigation Report

**Bug**: <one-sentence summary>
**Reproduction**: <steps that trigger it, or "not reproduced — evidence inferred from <source>">
**Evidence**:
- <fact with file:line ref>
- <fact with file:line ref or git sha>
- <external citation by URL — only when Mode 2 ran>
**Hypothesis**: <most likely cause in 1–3 sentences>
**Classification**: logic | architectural
**Confidence**: high | medium | low — <one sentence on what would raise it>
**Recommended next step**: <for logic: a one-paragraph fix description a fixer agent can act on. For architectural: "director should propose a separate fix phase to the user — this investigation does not commit to an approach.">
**Working tree**: clean (all instrumentation reverted)
```

## When to use this vs. other agents

| Situation | Agent |
|-----------|-------|
| **Diagnose a bug — find the root cause without fixing it** | investigator |
| **Plan and execute the fix once the cause is known** | director (which dispatches a fixer subagent) |
| **Multi-step implementation work** | director |
| **Project setup, review, or goal change** | advisor → director |

## Tips

- **It does not fix bugs.** Even when the cause is a one-line logic error, investigator stops at the report. Director plans the fix as a separate task on a different agent — this preserves the same separation-of-concerns principle that keeps implementation and tests on different agents.
- **It does not commit.** Like every subagent in the pipeline, only director writes to git history.
- **The working tree is always clean when it finishes.** Instrumentation is a means, not an artifact. If you want runtime logging to stay in the codebase, that is a separate fix task with its own verification.
- **Architectural causes get escalated, not patched.** When investigator classifies a cause as architectural, the Recommended next step asks director to propose a fix phase to the user. This avoids landing a structural change without explicit review.
- **Citations are mandatory in Research mode.** External findings without a URL do not count as evidence.
