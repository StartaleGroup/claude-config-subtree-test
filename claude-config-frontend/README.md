# Startale Front-End Team Claude Config (Team Tier)

This repo provides the **front-end team shared** Claude configuration layer for Startale projects.

## The four-tier system

We use a simple naming convention to separate config by ownership and scope:

| Tier    | Managed in                     | File pattern     | Scope                              |
|---------|--------------------------------|------------------|------------------------------------|
| team    | subtree `main`                 | `*.team.*`       | All projects across the team       |
| group   | subtree `group/*` branches     | `*.group.*`      | App-category-specific config       |
| project | each project repo              | `*.project.*`    | Per-project config                 |
| local   | each engineer's machine        | `*.local.*`      | Per-engineer, never committed      |

### Tier hierarchy

Config is applied in order from broadest to most specific:

```
team → group → project → local
```

Later tiers can override or extend earlier ones.

### Available groups

Groups represent categories of apps:

- **`strium`** — covers the Strium homepage and DEX application
- **`startale`** — covers the Startale super app
- **`sdk`** — covers SDK-related projects

Each group has a dedicated branch: `group/strium`, `group/startale`, `group/sdk`

## Where files live

This repo defines the **team tier** (on `main`) under:

```
.claude/
├── agents/
├── commands/
│   └── design-and-ux/    # UX review commands
├── hooks/
│   └── design-and-ux/    # UX-related hooks
├── rules/
│   └── design-and-ux/    # UX, a11y, motion, interaction rules
└── skills/
    └── design-and-ux/    # Design system context skills
```

Files are organized by domain (e.g., `ux/`) within each folder. This allows adding other domains later (e.g., `api/`, `testing/`).

Each folder contains `*.team.*` files on `main`, with `*.group.*` files added on group branches.

## Branch rules

### Never merge group branches into main

Group branches (`group/strium`, `group/startale`, `group/sdk`) contain group-specific config that must not leak into `main`. The merge direction is always:

```
main → group branches    ✅  (merge main INTO group)
group branches → main    ❌  (NEVER merge group INTO main)
```

When team-level changes are made on `main`, merge main into each group branch to pick them up:

```bash
git checkout group/strium
git merge main
```

### Contributing changes back to this repo

When making changes from a consumer project via `git subtree push`, always push to a **feature branch** — never directly to `main` or a `group/*` branch. Open a PR for review.

Branch naming convention for subtree PRs:

```
from/<consumer-project>/<short-description>
```

Examples:
- `from/dex-app/add-trading-rules`
- `from/super-app/update-motion-tokens`
- `from/sdk-docs/fix-a11y-rule`

## Design and UX

The `design-and-ux/` domain provides comprehensive frontend quality tooling built on [Impeccable](https://github.com/pbakaus/impeccable) by Paul Bakaus.

### Why Impeccable?

Without guidance, AI coding assistants produce generic output: Inter font, purple-to-blue gradients, cards nested inside cards, gray text on colored backgrounds, and hero sections with glassmorphism. Every LLM trained on the same templates produces the same generic output.

Impeccable is an open-source Claude Code skill set (Apache 2.0) that addresses this "AI slop" problem by:

- **Telling AI what NOT to do** — Negative constraints break the default trajectory
- **Providing domain-specific guidance** — 7 reference docs covering typography, color, spatial design, motion, interaction, responsive, and UX writing
- **Offering targeted slash commands** — 20 commands for specific design tasks

### Impeccable Skills

| Skill | Description |
|-------|-------------|
| `/audit` | Comprehensive quality audit (a11y, performance, theming, responsive) |
| `/critique` | UX review with actionable feedback |
| `/polish` | Final pre-ship quality pass |
| `/normalize` | Align with design system standards |
| `/colorize` | Color palette and theming |
| `/typeset` | Typography improvements |
| `/arrange` | Layout and spatial design |
| `/animate` | Motion and transitions |
| `/bolder` | Increase visual impact |
| `/quieter` | Reduce visual noise |
| `/delight` | Add personality and moments of joy |
| `/overdrive` | Maximum creative expression |
| `/clarify` | Improve comprehension and scannability |
| `/distill` | Simplify and reduce complexity |
| `/extract` | Pull reusable components |
| `/adapt` | Responsive and cross-platform adaptations |
| `/harden` | Edge cases, error states, defensive design |
| `/onboard` | First-run and onboarding flows |
| `/optimize` | Performance optimization |
| `/teach-impeccable` | One-time setup — gathers project design context |

### How It Connects

1. **Skills reference each other** — `/audit` suggests follow-up commands like `/normalize` or `/optimize`
2. **Hub skill** — `frontend-design` provides design principles and anti-patterns that other skills reference
3. **Project context** — Run `/teach-impeccable` once per project to capture brand, audience, and aesthetic direction in `.impeccable.md`

## Available slash commands

| Command | Description |
|---------|-------------|
| `/review-ux` | Full review across all four UX rule groups (a11y, motion, interaction, UX patterns) |
| `/review-motion` | Motion-only review — subset of `/review-ux` for animation/transition work |

### /review-ux

Runs a comprehensive UX review against all rule groups:

1. **Pre-check** — Scans for duplicate components that already exist in the design system
2. **Accessibility** — WCAG compliance, ARIA, keyboard navigation, focus management
3. **Animation & Motion** — Transitions, reduced-motion support, performance
4. **Interaction Design** — States, touch targets, dialogs, optimistic UI
5. **UX Patterns** — Design tokens, empty/loading states, forms, dark/light mode

Output is a structured report with severity levels: `error`, `warning`, `prompt`.

### /review-motion

Quick motion-only review for when you're working exclusively on animations or transitions.
Runs only the pre-check and motion steps from `/review-ux`.

## Hooks

| Hook | Trigger | Description |
|------|---------|-------------|
| `pre-commit-ux-prompt` | pre-commit | Prompts engineer to run `/review-ux` when UI files are staged |

### pre-commit-ux-prompt

Fires when staged files include `*.tsx`, `*.css`, or `*.module.css`.
Prompts the engineer to optionally run `/review-ux` before committing.
Does not block — responds to `n` or timeout by allowing the commit to proceed.

## Rules

Team-tier rules provide passive context that Claude uses when working on matching files:

| Rule | Applies to | Description |
|------|------------|-------------|
| `ux.team.md` | `**/*.tsx`, `**/*.css` | UX patterns, component conventions, layout rules |
| `a11y.team.md` | `**/*.tsx` | Accessibility — WCAG, ARIA, keyboard nav, focus |
| `motion.team.md` | `**/*.tsx`, `**/*.css` | Animation, transitions, reduced-motion, performance |
| `interaction.team.md` | `**/*.tsx` | States, touch targets, dialogs, optimistic UI |

## On agents

The current commands are self-contained. If review logic grows complex enough
to warrant separation, the recommended upgrade path is:

- `@a11y-agent` — dedicated accessibility reviewer, callable from `/review-ux` or independently
- `@motion-agent` — dedicated motion reviewer

This keeps commands thin and agents focused. Promote to agents when you find
yourself duplicating review logic across multiple commands.

## Naming conventions

- Use the tier marker as the middle segment: `name.<tier>.ext`
  - Examples: `frontend.team.md`, `dex.group.md`, `lint.project.md`, `personal.local.md`
- Tier markers are:
  - `team`
  - `group`
  - `project`
  - `local`

## Adding this repo to a project (git subtree)

This repo is added to consumer projects as a **git subtree**. The subtree is placed at a neutral path, then symlinked to `.claude/` so Claude Code picks it up.

### Step 1: Add the subtree

```bash
# Add the remote (one-time)
git remote add claude-config https://github.com/StartaleGroup/claude-config-frontend.git

# Add the subtree
git subtree add --prefix=claude-config-frontend claude-config main --squash
```

For a group-specific branch:

```bash
git subtree add --prefix=claude-config-frontend claude-config group/strium --squash
```

### Step 2: Create the symlink

Claude Code looks for `.claude/` at the project root. Create a symlink so it finds the config:

```bash
ln -s claude-config-frontend/.claude .claude
git add .claude
git commit -m "Add .claude symlink to subtree config"
```

> **Important:** The symlink must point to `claude-config-frontend/.claude`, not to `claude-config-frontend/` itself. The subtree root also contains `.gitignore` and `README.md` which should not live inside `.claude/`.

### Step 3: Set up git aliases (recommended)

Add these to your project's git config for shorter commands:

```bash
git config alias.cc-pull '!git subtree pull --prefix=claude-config-frontend claude-config main --squash'
git config alias.cc-push '!git subtree push --prefix=claude-config-frontend claude-config'
```

Then use:

```bash
# Pull latest config
git cc-pull

# Push changes back (always to a feature branch, then open a PR)
git cc-push from/my-project/description-of-change
```

### Step 4: Add to .gitignore (optional)

The subtree is fully committed, so no `.gitignore` changes are strictly needed. However, you may want to ignore local config files:

```gitignore
# Local/personal Claude config (never committed)
.claude/*.local.*
.claude/**/*.local.*
settings.local.*
```

## Pulling updates

```bash
git cc-pull
# Or without alias:
git subtree pull --prefix=claude-config-frontend claude-config main --squash
```

## Pushing changes back

Changes made to config files inside a consumer project can be pushed back to this repo. Always push to a feature branch and open a PR — never push directly to `main` or `group/*`.

```bash
git cc-push from/my-project/add-api-rules
# Or without alias:
git subtree push --prefix=claude-config-frontend claude-config from/my-project/add-api-rules
```

Then open a PR from `from/my-project/add-api-rules` → `main` (or the appropriate `group/*` branch).

## Updating to latest team/group config

```bash
git cc-pull
# Or for a specific group branch:
git subtree pull --prefix=claude-config-frontend claude-config group/strium --squash
```

## What's included

### Rules (passive context)
- `rules/design-and-ux/ux.team.md` — UX patterns, component conventions, layout
- `rules/design-and-ux/a11y.team.md` — Accessibility standards
- `rules/design-and-ux/motion.team.md` — Animation and transition rules
- `rules/design-and-ux/interaction.team.md` — Interaction design patterns

### Commands (slash commands)
- `commands/design-and-ux/review-ux.team.md` — Full UX review command
- `commands/design-and-ux/review-motion.team.md` — Motion-only review command

### Hooks
- `hooks/design-and-ux/pre-commit-ux-prompt.team.md` — Pre-commit prompt for UI file changes

### Skills
- 20+ Impeccable design skills for frontend quality
- `skills/design-dev-tools.md` — Design dev tooling skill

### Structure
- `.claude/*/.gitkeep` to establish folders in git
- Group branches add `CLAUDE.group.md` with group-specific context
