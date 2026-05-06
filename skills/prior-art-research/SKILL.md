---
name: prior-art-research
description: Use external prior art — GitHub issues, Stack Overflow, library changelogs, vendor advisories, official docs, technical blog posts — to inform a non-trivial technical decision: diagnosing a complex bug, choosing a library or pattern, researching how others implement a similar feature, or weighing architectural alternatives. Produces a structured findings report. Skip when the answer is obvious from local source, when the decision is trivial, or when project precedent already settles it. Triggers on "diagnose", "root cause", "investigate the bug", "research how to", "find prior art", "compare libraries", "is there a pattern for".
user-invocable: false
---

A skill for **research-driven investigation**: gather local evidence, optionally consult external prior art, and emit a structured findings report. Available to any agent that needs to research a technical question before acting.

The skill operates in three modes — **Understand → Research → Synthesize** — followed by a Closeout. Mode 2 is optional. Never skip Understand. Always close out by reverting instrumentation and emitting the report.

## When to use this skill (hard gate)

This skill costs network calls (`WebSearch`, `WebFetch`, `gh`) and produces a structured multi-section report. Before running any step below, self-assess.

**Skip this skill** when any apply:

- (Bug) The cause is a typo, syntax error, or compile error pointing to a specific line.
- (Bug) The cause is an obvious off-by-one, swapped argument, missing null check, or wrong field reference visible in the implicated function.
- (Bug) A test failure's assertion message names the exact problem.
- (Bug) The defect can be fixed in under five minutes by reading the implicated file plus its direct callers.
- (Feature/design) The project already has clear precedent for how this is done — read the existing code and follow the pattern.
- (Feature/design) The feature is small and standard; no architectural choice is at stake.
- (General) The answer lives entirely in the codebase or in already-loaded skills/rules.

For the cases above: do not run the procedure. Report the answer directly to the caller in one or two sentences and let the calling agent (or the fixer/implementer) act on it. The structured report below is wasted overhead.

**Use this skill** when at least one applies:

- (Bug) The symptom doesn't map to any code path locatable via grep/read.
- (Bug) The bug spans multiple modules and ownership is unclear.
- (Bug) A library/framework behaves unexpectedly and you don't know if it's misuse or upstream.
- (Bug) You have a verbatim error message likely hit by others.
- (Bug) A regression appeared with no obvious correlated commit.
- (Bug) The bug is environment-, timing-, or concurrency-dependent.
- (Feature/design) The feature is novel for this project and there's no in-codebase precedent to follow.
- (Feature/design) The decision depends on undocumented or version-specific behavior of an external library.
- (Feature/design) Multiple plausible architectural approaches exist and the trade-offs aren't obvious.
- (Feature/design) You need to know how widely-used libraries or projects solve this same problem.

If none of the "Use" conditions apply, exit without running. The calling agent should not have invoked this skill — but stopping is cheaper than running the full procedure on a trivial question.

## Tools

`Read`, `Write`, `Edit`, `Glob`, `Grep`, `Bash`, `WebSearch`, `WebFetch`.

`Bash` is used for `git log` / `git blame` / `git show`, the `gh` CLI when the user has it authenticated, and ad-hoc analysis scripts. `Write` and `Edit` are used **only** for temporary instrumentation (extra log lines, print statements, scratch scripts) that is reverted before the report is emitted.

---

## Mode 1: Understand

Build a factual picture of the question from the dispatch prompt or the user's request, the source, and git history.

1. **Extract the question's shape from the request.**
   - **For bugs**: observed behavior, expected behavior, reproduction steps (or note their absence), affected files or subsystems.
   - **For feature/design**: the desired outcome, constraints already decided, the area of the codebase that will be touched, and what an acceptable answer looks like (a recommended library, a chosen pattern, a design sketch).
   - If any of this is missing and cannot be derived from the project, ask the user one focused question — do not guess.
2. **Read the implicated source in full.** Confirm current state on disk; do not work from assumed shape. Read the surrounding module to understand callers and contracts.
3. **Walk the git history of the implicated files.** `git log --oneline -- <path>` for change density, `git log -p -S '<symbol>' -- <path>` to find when a symbol was introduced or changed, `git blame <path>` to attribute current lines, `git show <sha>` on relevant commits.
   - **For bugs**: correlate any regression with a commit when possible.
   - **For feature/design**: understand how the area has evolved, what patterns have been tried, and what was deliberately removed.
4. **For bugs only — reproduce when feasible.** If the request includes reproduction steps, run them through `Bash` and capture the actual output. If reproduction requires runtime evidence the source alone cannot give, add temporary instrumentation — extra log lines, `print` statements, a small standalone script that exercises the suspect path. Every such edit is temporary and must be reverted in Closeout.
5. **Decide whether Mode 2 is needed.** If the local evidence already answers the question with high confidence, skip Research and go straight to Synthesize. Otherwise proceed.

## Mode 2: Research (optional)

Look for prior art outside the project. Skip when Mode 1 produced a confident answer.

- **`WebSearch`** — search for the verbatim error message, the framework + symptom combination, the library + version + symptom combination, or the feature + "best practices" / "comparison" / "benchmark". Quote distinctive strings exactly.
- **`WebFetch`** — read promising results. Likely sources: GitHub issues and pull requests, GitHub Discussions, Stack Overflow, library changelogs, vendor advisories, official documentation, technical blog posts, conference talks.
- **`gh` CLI through `Bash`** — when the user has it authenticated, prefer it for structured GitHub queries: `gh issue list -R <owner>/<repo> --search '<terms>'`, `gh pr view <n> --comments`, `gh search issues '<terms>' --repo <owner>/<repo>`, `gh search code '<terms>'` to find how others use a library. When `gh` is not available, fall back to `WebFetch` against the public GitHub URL.
- Cite every external finding by URL in the final report. A finding without a citation is not a finding.

## Mode 3: Synthesize

Synthesize Modes 1–2 into the answer.

1. **State the finding in one to three sentences.** Concrete, specific, tied to the evidence.
   - **For bugs**: the most likely root cause.
   - **For feature/design**: the recommended approach with rationale.
2. **For bugs only — classify the cause as `logic` or `architectural`.** Omit Classification for non-bug topics.
   - **logic** — a localized defect inside one function or module: wrong condition, off-by-one, wrong field reference, missing null check, incorrect type coercion, swapped arguments. The fix touches a small, well-bounded patch.
   - **architectural** — a structural cause that crosses module boundaries: wrong abstraction, race between components, contract mismatch between layers, missing invariant enforcement at a system seam, ownership confusion over shared state. The fix requires a design decision, not just a code edit.
3. **State a confidence level — `high`, `medium`, or `low` — with one sentence on what would raise it.** Examples: "high — reproduction confirmed and the failing line is identified"; "medium — the recommended library matches the constraints but no project-shaped prototype was tried; building a 50-line spike would raise it to high".
4. **Do not implement.** Even for clear logic bugs, fixing is a separate task. For feature/design recommendations, the implementation is also a separate task. The report's Recommended next step describes what to do; it does not perform it.

## Closeout

Before emitting the report:

1. **Revert every instrumentation edit.** Use `git status` to confirm the working tree is clean. Any file the agent modified during Mode 1 or Mode 2 must be returned to its prior state. If any non-instrumentation change slipped in, revert that too — the procedure never leaves the tree dirty.
2. **Emit the report in the exact format below.** When dispatched by director, this is what gets pasted verbatim into the `Task:` journal entry's "What was done" field.

## Report format

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
**Recommended next step**: <concrete next action for the calling agent. For bugs: a fix description a fixer agent can act on, OR for architectural causes "the orchestrator should propose a separate fix phase to the user". For features/design: an implementation outline or a decision to bring back to the user.>
**Working tree**: clean (all instrumentation reverted)
```

## Constraints

- **Do not implement.** Investigation only, even for clear logic bugs and obvious feature recommendations. Fixing and building are separate tasks.
- **Do not make any git commits.** When this skill runs inside a director-dispatched subagent, director handles all commits.
- **Do not leave instrumentation in the working tree.** Every edit made during the procedure is reverted before the report is emitted; `git status` must show a clean tree.
- **If the question is impossible to investigate as specified** (no reproduction available, required runtime inaccessible, missing files, external sources blocked), stop immediately and report back. Explain what you attempted, what blocked you, and what additional information or access would unblock the investigation. Do not leave behind partial or broken changes.
