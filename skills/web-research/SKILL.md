---
name: web-research
description: Researches a topic online using web sources, GitHub, official docs, and media transcripts. Synthesizes findings into a structured answer. Saves to disk when the user's prompt signals persistence intent; otherwise answers inline. Use for any online research request.
user-invocable: true
---

Research a topic online and synthesize findings into a structured answer. Save to disk when the user's prompt signals persistence intent (e.g. "save", "I'll need this again"); otherwise answer inline.

## Tools

`WebSearch`, `Bash` (primary URL fetcher via `curl -sL --max-time 10`; `gh` CLI when authenticated; `yt-dlp` when available, for media transcripts), `WebFetch` (fallback when `Bash` is unavailable).

## Phase 1: Plan

Parse the topic/question. Identify 3–5 distinct research angles (official docs, GitHub issues/PRs, changelogs, comparisons, blog posts). Write out the plan before issuing any searches — this prevents redundant fetches.

## Phase 2: Research

Execute searches iteratively. Follow promising threads; skip low-signal results.

**Fetching**: use `curl -sL --max-time 10 <url>` via `Bash` for each URL. If the request times out (no response within 10 s), skip that URL, add it to the unreachable list, and do **not** count it against the budget. Fall back to `WebFetch` only when `Bash` is unavailable.

**Media transcripts**: for conference talks, podcasts, and recorded interviews, fetch captions with `yt-dlp` into a temp dir to avoid repo pollution:

```
DIR=$(mktemp -d) && yt-dlp --write-subs --write-auto-subs \
  --sub-langs en --skip-download --convert-subs srt \
  -o "$DIR/%(id)s.%(ext)s" <url>
```

Prefer the manually-uploaded subtitle file (e.g. `<id>.en.srt`) over the auto-caption file (e.g. `<id>.en.auto.srt`) when both are present. Flag findings drawn from auto-captions as potentially noisy — auto-captions mistranscribe technical terms. If neither file is produced, add the URL to the unreachable list and move on. For long transcripts (>30 min of content), grep for query-relevant terms (`grep -B5 -A10 <keyword> <file>`) before full-reading; short clips can be read in full.

**Fetch budget**: soft limit of **20 fetches** (successful fetches only — timed-out URLs are excluded), plus a separate cap of **5 media transcripts**.
- Track both counts as you go.
- At 20 fetches or 5 transcripts: stop. Report your current confidence level and a one-paragraph summary of what you know. Ask: *"I've used [N] fetches and [M] transcripts. Confidence is [level]. Continue researching? (y/n)"*
- If the user approves, continue in increments of 10 fetches / 3 transcripts, asking again at each increment.

Cite every finding by URL. A fact without a URL is not a finding. For findings drawn from media transcripts, include both an `[mm:ss]` timestamp in the finding line and a deep-link URL (e.g. `https://youtube.com/watch?v=ABC123&t=2538`).

**Source priority** (highest to lowest): official docs and changelogs → GitHub issues/PRs/Discussions → Stack Overflow → technical blog posts → conference talks and media transcripts.

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
