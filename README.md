# Startale Front-End Team Claude Config (Team Tier)

This repo provides the **front-end team shared** Claude configuration layer for Startale projects.

## The three-tier system

We use a simple naming convention to separate config by ownership and scope:

- **`*.team.*`**: Team-wide defaults shared across projects (tracked in this repo)
- **`*.project.*`**: Project-specific config (tracked in each project repo, ignored here)
- **`*.local.*`**: Personal config (never committed; ignored everywhere)

### Where files live

This repo defines the **team tier** under:

```
.claude/
├── agents/
├── commands/
├── rules/
└── skills/
```

Each folder contains `*.team.*` files only.

## Naming conventions

- Use the tier marker as the middle segment: `name.<tier>.ext`
  - Examples: `frontend.team.md`, `lint.project.md`, `personal.local.md`
- Tier markers are:
  - `team`
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

Add the submodule:

```bash
git submodule add git@github.com:startale/claude-config.git .claude/team
git submodule update --init --recursive
```

Then, in the project repo:

- Put team-shared files in `.claude/team/.claude/**` (from this repo)
- Put project files in `.claude/**` using `*.project.*`
- Put personal files in `.claude/**` using `*.local.*`

### Project repo `.gitignore` snippet (copy/paste)

Add this to **each project repo**’s `.gitignore` to ensure personal files never get committed, while allowing project-tier files to live in the project repo:

```gitignore
# Claude config tiers
# - *.team.* lives in the .claude/team submodule
# - *.project.* is committed in this project repo
# - *.local.* is personal-only (never committed)

.claude/**/*.local.*
settings.local.*
```

If you want to ensure the **submodule directory itself** is not mistaken for project config, do not add ignore rules for `.claude/team/` (it’s tracked via the gitlink created by the submodule).

## What’s included (scaffold only)

This repo intentionally contains **structure and documentation only**:

- `.claude/*/.gitkeep` to establish folders in git
- One `example.team.md` per folder with a header explaining the file type

No real rules/commands/agents/skills have been authored yet.
