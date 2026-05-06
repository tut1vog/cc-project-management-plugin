# Code Comments

## Rules
- Default to no comments. One line maximum when a comment is warranted.
- Add a comment only when the **why** is non-obvious: a hidden constraint, a subtle invariant, a workaround, or behavior that would surprise a reader.
- Never explain what the code does — well-named identifiers do that.
- Never embed planning decisions, phase history, or task rationale in comments or docstrings. That context belongs in git history.

## Examples

Bad — phase-decision narration:
```python
def sync(items):
    """Phase 2 switched from bulk insert to upsert after duplicate-key
    errors in staging. Phase 4 added retry on deadlock per ops feedback."""
```

Good — one-line WHY:
```python
# upstream drops connections silently on idle timeout — retry before raising
def sync(items):
```
