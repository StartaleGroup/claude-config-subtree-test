# Startale Front-End Team Claude Config

Shared Claude Code configuration for Startale front-end projects, distributed as a **git subtree**.

## Table of contents

- [How it works](#how-it-works)
  - [The four-tier system](#the-four-tier-system)
  - [Available groups](#available-groups)
  - [Naming conventions](#naming-conventions)
  - [Repo structure](#repo-structure)
- [Setup](#setup)
  - [Step 1: Add the subtree](#step-1-add-the-subtree)
  - [Step 2: Create the symlink](#step-2-create-the-symlink)
  - [Step 3: Set up git aliases](#step-3-set-up-git-aliases-recommended)
  - [Step 4: Add to .gitignore](#step-4-add-to-gitignore-optional)
- [Pulling updates](#pulling-updates)
- [Pushing changes back](#pushing-changes-back)
  - [Branch naming convention](#branch-naming-convention)
  - [How to push](#how-to-push)
  - [Merge direction](#merge-direction)
- [What's included](#whats-included)
  - [Rules](#rules)
  - [Commands](#commands)
  - [Hooks](#hooks)
  - [Design and UX (Impeccable)](#design-and-ux-impeccable)
  - [On agents](#on-agents)

---

## How it works

This repo contains a `.claude/` directory with shared rules, skills, commands, and hooks. Consumer projects pull it in as a **git subtree** at the path `claude-config-frontend/`, then create a **symlink** from `.claude` to `claude-config-frontend/.claude` so Claude Code discovers the config automatically.

```
your-project/
├── .claude → claude-config-frontend/.claude   # symlink
├── claude-config-frontend/                    # git subtree (this repo)
│   ├── .claude/
│   │   ├── rules/
│   │   ├── skills/
│   │   ├── commands/
│   │   └── hooks/
│   ├── .gitignore
│   └── README.md
└── src/                                       # your project code
```

The subtree is fully committed in the consumer project's git history. You can pull updates from this repo and push changes back via PRs.

### The four-tier system

We use a naming convention to separate config by ownership and scope:

| Tier    | Managed in                     | File pattern     | Scope                              |
|---------|--------------------------------|------------------|------------------------------------|
| team    | subtree `main`                 | `*.team.*`       | All projects across the team       |
| group   | subtree `group/*` branches     | `*.group.*`      | App-category-specific config       |
| project | each project repo              | `*.project.*`    | Per-project config                 |
| local   | each engineer's machine        | `*.local.*`      | Per-engineer, never committed      |

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

### Naming conventions

Use the tier marker as the middle segment: `name.<tier>.ext`

- Examples: `frontend.team.md`, `dex.group.md`, `lint.project.md`, `personal.local.md`
- Tier markers: `team`, `group`, `project`, `local`

### Repo structure

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

Files are organized by domain (e.g., `design-and-ux/`) within each folder. This allows adding other domains later (e.g., `api/`, `testing/`).

Each folder contains `*.team.*` files on `main`, with `*.group.*` files added on group branches.

---

## Setup

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
git config alias.cc-pull 'subtree pull --prefix=claude-config-frontend claude-config main --squash'
git config alias.cc-push 'subtree push --prefix=claude-config-frontend claude-config'
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

---

## Pulling updates

```bash
git cc-pull
# Or without alias:
git subtree pull --prefix=claude-config-frontend claude-config main --squash
```

For a specific group branch:

```bash
git subtree pull --prefix=claude-config-frontend claude-config group/strium --squash
```

---

## Pushing changes back

Changes made to config files inside a consumer project can be pushed back to this repo. Always push to a **feature branch** and open a PR — never push directly to `main` or `group/*`.

### Branch naming convention

```
from/<consumer-project>/<short-description>
```

Examples:
- `from/dex-app/add-trading-rules`
- `from/super-app/update-motion-tokens`
- `from/sdk-docs/fix-a11y-rule`

### How to push

```bash
git cc-push from/my-project/add-api-rules
# Or without alias:
git subtree push --prefix=claude-config-frontend claude-config from/my-project/add-api-rules
```

Then open a PR from `from/my-project/add-api-rules` → `main` (or the appropriate `group/*` branch).

### Merge direction

Group branches (`group/strium`, `group/startale`, `group/sdk`) contain group-specific config that must not leak into `main`. The merge direction is strictly one-way:

```
main → group branches    ✅  (merge main INTO group)
group branches → main    ❌  (NEVER merge group INTO main)
```

When team-level changes are made on `main`, merge main into each group branch to pick them up:

```bash
git checkout group/strium
git merge main
```

---

## What's included

### Rules

Team-tier rules provide passive context that Claude uses when working on matching files:

| Rule | Applies to | Description |
|------|------------|-------------|
| `ux.team.md` | `**/*.tsx`, `**/*.css` | UX patterns, component conventions, layout rules |
| `a11y.team.md` | `**/*.tsx` | Accessibility — WCAG, ARIA, keyboard nav, focus |
| `motion.team.md` | `**/*.tsx`, `**/*.css` | Animation, transitions, reduced-motion, performance |
| `interaction.team.md` | `**/*.tsx` | States, touch targets, dialogs, optimistic UI |
| `contributing.team.md` | all files | Branch naming and merge direction rules |

### Commands

| Command | Description |
|---------|-------------|
| `/review-ux` | Full review across all four UX rule groups (a11y, motion, interaction, UX patterns) |
| `/review-motion` | Motion-only review — subset of `/review-ux` for animation/transition work |

**`/review-ux`** runs a comprehensive UX review:

1. **Pre-check** — Scans for duplicate components that already exist in the design system
2. **Accessibility** — WCAG compliance, ARIA, keyboard navigation, focus management
3. **Animation & Motion** — Transitions, reduced-motion support, performance
4. **Interaction Design** — States, touch targets, dialogs, optimistic UI
5. **UX Patterns** — Design tokens, empty/loading states, forms, dark/light mode

Output is a structured report with severity levels: `error`, `warning`, `prompt`.

**`/review-motion`** is a quick motion-only subset of `/review-ux` for animation/transition work.

### Hooks

| Hook | Trigger | Description |
|------|---------|-------------|
| `pre-commit-ux-prompt` | pre-commit | Prompts engineer to run `/review-ux` when UI files are staged |

Fires when staged files include `*.tsx`, `*.css`, or `*.module.css`. Does not block — responds to `n` or timeout by allowing the commit to proceed.

### Design and UX (Impeccable)

The `design-and-ux/` domain provides frontend quality tooling built on [Impeccable](https://github.com/pbakaus/impeccable) by Paul Bakaus.

**Why Impeccable?** Without guidance, AI coding assistants produce generic output — Inter font, purple-to-blue gradients, cards nested inside cards, gray text on colored backgrounds. Impeccable is an open-source Claude Code skill set (Apache 2.0) that breaks these defaults with negative constraints, domain-specific reference docs, and targeted slash commands.

| Skill | Description |
|-------|-------------|
| `/frontend-design` | Hub skill — design principles and anti-patterns referenced by other skills |
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

Skills reference each other — `/audit` suggests follow-ups like `/normalize` or `/optimize`. Run `/teach-impeccable` once per project to capture brand, audience, and aesthetic direction in `.impeccable.md`.

### On agents

The current commands are self-contained. If review logic grows complex enough to warrant separation, the recommended upgrade path is:

- `@a11y-agent` — dedicated accessibility reviewer
- `@motion-agent` — dedicated motion reviewer

Promote to agents when you find yourself duplicating review logic across multiple commands.
