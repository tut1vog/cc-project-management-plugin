---
name: web-research
description: Researches a topic online using web sources, GitHub, and official docs. Synthesizes findings into a structured answer. Saves to disk when the user's prompt signals persistence intent; otherwise answers inline. Use for any online research request.
user-invocable: true
---

Research a topic online and synthesize findings into a structured answer. Save to disk when the user's prompt signals persistence intent (e.g. "save", "I'll need this again"); otherwise answer inline.

## Tools

`WebSearch`, `Bash` (primary URL fetcher via `curl -sL --max-time 10`; `gh` CLI when authenticated), `WebFetch` (fallback when `Bash` is unavailable).

## Phase 1: Plan

Parse the topic/question. Identify 3–5 distinct research angles (official docs, GitHub issues/PRs, changelogs, comparisons, blog posts). Write out the plan before issuing any searches — this prevents redundant fetches.

## Phase 2: Research

Execute searches iteratively. Follow promising threads; skip low-signal results.

**Fetching**: use `curl -sL --max-time 10 <url>` via `Bash` for each URL. If the request times out (no response within 10 s), skip that URL, add it to the unreachable list, and do **not** count it against the budget. Fall back to `WebFetch` only when `Bash` is unavailable.

**Fetch budget**: soft limit of **20 fetches** (successful fetches only — timed-out URLs are excluded).
- Track the running count as you go.
- At 20: stop. Report your current confidence level and a one-paragraph summary of what you know. Ask: *"I've used 20 fetches. Confidence is [level]. Continue researching? (y/n)"*
- If the user approves, continue in increments of 10, asking again at each increment.

Cite every finding by URL. A fact without a URL is not a finding.

**Source priority** (highest to lowest): official docs and changelogs → GitHub issues/PRs/Discussions → Stack Overflow → technical blog posts → conference talks.

## Phase 3: Synthesize

Produce the following components:
1. **Summary** — 2–4 sentences covering the core answer to the topic/question.
2. **Key facts** — bulleted list, each anchored to a citation URL.
3. **Recommendations** (if applicable) — concrete, opinionated guidance derived from the evidence.
4. **Reference links** — curated list of the most valuable sources, each with a one-line description.
5. **Unreachable sources** (only if any URLs timed out) — list of URLs that did not respond, so the user can retry them manually.

## Phase 4: Output

- **Inline**: render the components directly in the chat response.
- **Saved**: write the components as plain markdown to a location that fits the project's conventions. Use your judgment on filename and path.

State which path you're taking at the start of Phase 4.

## Constraints

- Do not implement, fix, or build anything — research and synthesize only.
- Do not commit generated files.
- If external sources are blocked or the topic cannot be researched as specified, stop and report what blocked you and what additional access would unblock it.
