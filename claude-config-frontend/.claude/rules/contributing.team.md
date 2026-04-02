# Contributing to Shared Config

## Submitting changes from consumer projects

When pushing config changes back to this repo from a consumer project, always push to a **feature branch** and open a PR. Never push directly to `main` or a `group/*` branch.

### Branch naming convention

```
from/<consumer-project>/<short-description>
```

- `from/` prefix indicates the change originates from a consumer project
- `<consumer-project>` is the name of the app or project (e.g., `dex-app`, `super-app`, `sdk-docs`)
- `<short-description>` briefly describes the change (e.g., `add-trading-rules`, `fix-a11y-rule`)

### Merge direction

- `main → group branches` — merge main INTO group branches to pick up team-level changes
- `group branches → main` — NEVER merge group branches into main (group-specific config must not leak into team tier)

### What belongs in this repo

Only team-level (`*.team.*`) and group-level (`*.group.*`) config belongs here. Project-specific and local config stays in the consumer project.
