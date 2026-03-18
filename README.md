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
├── rules/
└── skills/
```

Each folder contains `*.team.*` files on `main`, with `*.group.*` files added on group branches.

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

## What's included (scaffold only)

This repo intentionally contains **structure and documentation only**:

- `.claude/*/.gitkeep` to establish folders in git
- One `example.team.md` per folder with a header explaining the file type
- Group branches add `CLAUDE.group.md` with group-specific context

No real rules/commands/agents/skills have been authored yet.
