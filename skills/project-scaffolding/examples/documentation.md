# Documentation

The `docs/` tree has mixed ownership. Each subdirectory has a defined owner; treat the rest as human-authored.

## Rules

- `docs/research/` — agent-generated research output. `docs/research/README.md` indexes it; director appends a one-line entry there for every research doc saved.
- `docs/reference/` — human-provided external context (specs, vendor docs, exported material). Consult it for additional information; never overwrite or restructure it.
- Any other content under `docs/` is human-authored project documentation — read it, never overwrite it.

## Examples

Good — new research doc saved, index updated in the same step:
```
docs/research/2026-05-14-rate-limiter-options.md   ← written
docs/research/README.md                            ← one line appended
```

Bad — consulting reference material, then rewriting it to "tidy up":
```
docs/reference/vendor-api-spec.md                  ← human-provided; leave it alone
```
