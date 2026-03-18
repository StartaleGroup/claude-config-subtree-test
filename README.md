# Startale Front-End Team Claude Config (Team Tier)

This repo provides the **front-end team shared** Claude configuration layer for Startale projects.

## The four-tier system

We use a simple naming convention to separate config by ownership and scope:

| Tier    | Managed in                     | File pattern     | Scope                              |
|---------|--------------------------------|------------------|------------------------------------|
| team    | submodule `main`               | `*.team.*`       | All projects across the team       |
| group   | submodule `group/*` branches   | `*.group.*`      | App-category-specific config       |
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

## Adding this repo to a project (submodule)

The recommended setup is to add this repo as a **submodule** inside your project, and keep project/local files in the project repo alongside it.

One practical layout looks like:

```
<project-root>/
  .claude/
    team/                # git submodule (this repo)
    agents/              # project/local live here (tracked by the project repo)
    commands/
    hooks/
    rules/
    skills/
```

### Step 1: Add the submodule

```bash
git submodule add git@github.com:startale/claude-config.git .claude/team
git submodule update --init --recursive
```

### Step 2: Switch to your group branch

Depending on which app category your project belongs to, switch the submodule to the appropriate group branch:

```bash
cd .claude/team
git checkout group/strium    # For Strium homepage/DEX projects
# OR
git checkout group/startale  # For Startale super app projects
# OR
git checkout group/sdk       # For SDK-related projects
cd ../..
```

To lock your project to a specific group branch:

```bash
git add .claude/team
git commit -m "Pin claude-config submodule to group/strium branch"
```

### Step 3: Set up your project files

Then, in the project repo:

- Put team-shared files in `.claude/team/.claude/**` (from this repo, `main` branch)
- Put group-shared files in `.claude/team/.claude/**` (from this repo, `group/*` branch)
- Put project files in `.claude/**` using `*.project.*`
- Put personal files in `.claude/**` using `*.local.*`

## Project repo `.gitignore` snippet (copy/paste)

Add this to **each project repo**'s `.gitignore` to ensure group and personal files never get committed to the project repo, while allowing project-tier files:

```gitignore
# Claude config tiers
# - *.team.* lives in the .claude/team submodule (main branch)
# - *.group.* lives in the .claude/team submodule (group/* branch)
# - *.project.* is committed in this project repo
# - *.local.* is personal-only (never committed)

# Group-tier config (managed via submodule branch, not committed per-project)
.claude/*.group.*
.claude/**/*.group.*
settings.group.*

# Local/personal config (never committed)
.claude/*.local.*
.claude/**/*.local.*
settings.local.*
```

If you want to ensure the **submodule directory itself** is not mistaken for project config, do not add ignore rules for `.claude/team/` (it's tracked via the gitlink created by the submodule).

## Updating to latest team/group config

To pull the latest changes from the config repo:

```bash
cd .claude/team
git pull origin <your-group-branch>  # e.g., group/strium
cd ../..
git add .claude/team
git commit -m "Update claude-config submodule"
```

## What's included

This repo contains:

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
- `skills/design-and-ux/design-system.team.md` — Design system context (tokens, components, patterns)

### Structure
- `.claude/*/.gitkeep` to establish folders in git
- Group branches add `CLAUDE.group.md` with group-specific context
