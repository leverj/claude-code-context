---
name: code-context
description: Generate and maintain a comprehensive code-context.md file that captures deep codebase understanding — models, routes, components, stores, architecture, data flows, and key patterns. Commands - init, update, uninstall.
allowed-tools: Agent Bash(ls *) Bash(git diff *) Bash(git log *) Bash(git rev-parse *) Bash(rm -rf *) Glob Grep Read Write Edit
argument-hint: "<init|update|uninstall> [optional focus area or description]"
effort: high
---

# Code Context Manager

You manage the `code-context.md` file for this project. This file serves as a persistent codebase understanding cache — it eliminates the need for every Claude session to re-explore the codebase from scratch.

## User instructions

$ARGUMENTS

## Command routing

Parse the first word of the user's arguments to determine the command:

- **`init`** (or no arguments, or arguments that don't start with `update`/`uninstall`/`global-uninstall`) → Go to [Init Command](#init-command)
- **`update`** → Go to [Update Command](#update-command)
- **`uninstall`** → Go to [Uninstall Command](#uninstall-command)
- **`global-uninstall`** → Go to [Global Uninstall Command](#global-uninstall-command)

---

## Init Command

Generate a comprehensive `code-context.md` from scratch. If a `code-context.md` already exists, read it first — preserve any sections that are still accurate and update/add sections that have changed or are missing. This allows re-initialization to build on prior work rather than starting from zero.

### Step 1: Discover project structure

Start by understanding the project at a high level:

1. Read the root `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `build.gradle`, `Package.swift`, or equivalent to determine the tech stack
2. Check for monorepo indicators: `lerna.json`, `pnpm-workspace.yaml`, `workspaces` in package.json, `Cargo.toml` workspace members
3. Read any existing `README.md`, `CLAUDE.md`, `ARCHITECTURE.md`, or similar docs
4. If `code-context.md` already exists, read it to understand what was previously documented
5. Map the top-level directory structure

### Step 2: Deep exploration

For each package/module in the project, explore thoroughly using the Agent tool with `subagent_type: "Explore"`. Run multiple agents in parallel for independent packages.

For each package, capture:

**Backend/API packages:**
- All database models/schemas with field names, types, indexes, relationships, and key methods
- All API routes with HTTP method, path, auth requirements, and purpose
- Middleware chain and what each middleware does
- Service layer / business logic modules
- Configuration and environment variables with defaults
- Utility modules and what they export

**Frontend packages:**
- Navigation/routing structure with all routes
- Components with their purpose and key props
- State management (stores/reducers/contexts) with state shape and actions
- Custom hooks and what they return
- Utilities and helpers (especially platform-specific ones)
- Design system tokens (colors, typography, spacing)

**Mobile / Native packages:**
- Architecture pattern (MVVM, MVC, etc.)
- Module structure with responsibilities
- Networking layer and API client design
- Persistence strategy (local storage, database, keychain)
- Authentication flow and state machine

**Infrastructure:**
- CI/CD pipeline (jobs, triggers, branch rules, deploy targets)
- Docker setup
- Environment file hierarchy and key variables
- Build scripts and their purpose
- Auth provider configuration

### Step 3: Generate code-context.md

Get the current git commit hash by running `git rev-parse HEAD`.

Write the file to `code-context.md` in the project root with this structure:

```markdown
# [Project Name] Code Context

> **Purpose**: Persistent codebase understanding for Claude sessions. Eliminates cold-start exploration.
> **Last updated**: [today's date]
> **Update rule**: Update this file when making structural changes (new models, routes, components, stores, modules, config changes, or architectural shifts).

<!-- code-context-marker: [full git commit hash from git rev-parse HEAD] -->

---

## Project Overview
[What the project does, monorepo structure, tech stack summary, top-level directory tree]

## Package: [name] ([path])
[For each package, include ALL of the relevant sections from Step 2]

## Infrastructure
[CI/CD, Docker, env files, scripts, auth config]

## [Domain] Flow (End-to-End)
[Key flows like auth, data pipeline, etc. — how pieces connect across packages]

## Data Flow Summary
[ASCII diagram showing how data moves through the system]

## Key Patterns to Know
[Non-obvious conventions, composite IDs, platform-specific file patterns, transaction patterns, etc.]
```

**The `<!-- code-context-marker: ... -->` HTML comment is critical.** It stores the git commit hash at the time of generation. This marker enables automatic change detection — at the start of future sessions, Claude compares this hash against the current HEAD to determine if the codebase has changed and the context needs updating.

### Step 4: Set up CLAUDE.md for auto-maintenance

After generating `code-context.md`, ensure the project's `CLAUDE.md` has instructions for Claude to read and maintain it. This is what makes the system self-sustaining.

1. Check if `CLAUDE.md` exists in the project root
2. If it exists, read it and check if it already has a "Code Context" section
3. If the section is missing, append it using the Edit tool
4. If `CLAUDE.md` doesn't exist, create it using the Write tool

The section to add (copy exactly):

```markdown
## Code Context

At the start of every session:
1. Read `code-context.md` in the project root if it exists.
2. Check the `<!-- code-context-marker: <hash> -->` comment in the file against the output of `git rev-parse HEAD`.
3. If the hashes differ (codebase has changed since last context update), automatically run the equivalent of `/code-context update` — read recent changes via `git diff <marker-hash>..HEAD --name-only` and `git log <marker-hash>..HEAD --oneline`, then update the affected sections in `code-context.md` and set the marker to the current HEAD hash.

This file contains deep codebase understanding and eliminates exploratory warm-up.

When you make changes that affect anything documented in `code-context.md`, update it before
completing the task and set the marker to the current HEAD hash. This includes structural changes
(new/removed models, routes, components, stores, modules, config) AND behavioral changes
(implementing a deferred feature, changing a status, adding interactions, modifying flows).
If in doubt, scan `code-context.md` for mentions of the area you changed — stale entries
defeat the purpose of the file.
```

**This step is not optional.** Without CLAUDE.md setup, the context file becomes a one-time snapshot that goes stale. The CLAUDE.md instruction is what closes the loop.

### Quality guidelines

- **Be deep**: List individual model fields, route paths, component props — not just file names
- **Be precise**: Include actual values (port numbers, default configs, thresholds) not vague descriptions
- **Be relational**: Show how pieces connect (which store calls which API, which model relates to which)
- **Be practical**: Include the patterns someone needs to know to work in this codebase safely
- **Use tables**: For models, routes, config vars — they're scannable and dense
- **Include types/enums**: State shapes, status enums, error types — these prevent bugs
- **Skip boilerplate**: Don't document obvious framework conventions (React component lifecycle, Express middleware signature)

### After init

Tell the user:
1. `code-context.md` has been generated at the project root
2. `CLAUDE.md` has been set up (or updated) to auto-read and auto-maintain the context file
3. The context file includes a git hash marker for automatic change detection
4. From now on, every Claude session will check for codebase changes and auto-update if needed
5. They can run `/code-context update` manually after large refactors
6. Suggest they review the file and flag any inaccuracies or missing areas

---

## Update Command

Update the existing `code-context.md` file to reflect recent structural changes.

### Step 1: Understand current state

1. Read the existing `code-context.md`
2. If the file doesn't exist, tell the user to run `/code-context init` first and stop
3. Extract the current marker hash from `<!-- code-context-marker: ... -->`
4. If the user described what changed (arguments after "update"), focus on those areas
5. If no description provided, check recent changes:
   - Run `git diff --name-only <marker-hash>..HEAD` to see files changed since last update (fall back to `HEAD~5` if no marker)
   - Run `git log --oneline <marker-hash>..HEAD` to see commits since last update
   - Identify which sections of code-context.md are affected

### Step 2: Verify and update affected sections

For each affected area:

1. Read the current source files to understand what actually exists now
2. Compare with what `code-context.md` currently says
3. Update the relevant sections using the Edit tool

Focus on:
- New models/schemas → add to the models table with full field details
- Removed models → remove from the table
- New routes/endpoints → add to the routes table
- Changed routes → update method, path, auth, or purpose
- New components/stores/hooks → add with purpose and key details
- Removed code → remove from the document
- Config changes → update environment variables, ports, defaults
- New packages/modules → add a new package section
- Architecture changes → update patterns, flows, or overview

### Step 3: Update metadata

1. Update the "Last updated" date in the header to today's date
2. Update the `<!-- code-context-marker: ... -->` to the current `git rev-parse HEAD` hash

### Update guidelines

- **Only change what's affected** — don't rewrite the entire file
- **Match existing style** — use the same table format, heading structure, and level of detail
- **Verify before writing** — read the actual source files, don't guess from commit messages alone
- **Remove stale info** — if something was deleted, remove it from the doc rather than marking it as removed
- **Keep it accurate** — if you're unsure about a change, read the source to confirm

### After update

Tell the user what sections were updated and why. Mention the marker has been updated to the current HEAD.

---

## Uninstall Command

Remove code-context integration from the project while preserving the generated documentation.

### Step 1: Remove CLAUDE.md references

1. Read the project's `CLAUDE.md`
2. If it exists and contains a "Code Context" section, remove the entire section (from `## Code Context` to the next `##` heading or end of file)
3. If removing the section leaves `CLAUDE.md` empty (or with only whitespace), delete the file entirely
4. If `CLAUDE.md` doesn't exist or has no "Code Context" section, note that no changes were needed

### Step 2: Confirm to user

Tell the user:
1. The "Code Context" section has been removed from `CLAUDE.md` — Claude will no longer auto-read or auto-update the context file
2. `code-context.md` has been left in place — they can remove it manually with `rm code-context.md` if they wish
3. If they run `/code-context init` again in the future, it will pick up from the existing `code-context.md` and update it rather than starting from scratch, so keeping it around is safe
4. If they want to fully remove the skill itself, they can run `/code-context global-uninstall`

---

## Global Uninstall Command

Completely remove the code-context skill from the system. This performs the project-level uninstall AND removes the skill files from the global Claude configuration.

### Step 1: Run project-level uninstall

First, perform the same steps as the [Uninstall Command](#uninstall-command) — remove the "Code Context" section from the current project's `CLAUDE.md`.

### Step 2: Remove skill files

1. Check if `~/.claude/skills/code-context/` exists
2. If it exists, remove it: `rm -rf ~/.claude/skills/code-context/`
3. If it doesn't exist, check if the skill is installed per-project at `.claude/skills/code-context/` and remove that instead

### Step 3: Confirm to user

Tell the user:
1. The "Code Context" section has been removed from this project's `CLAUDE.md`
2. The skill files have been removed from `~/.claude/skills/code-context/` (or `.claude/skills/code-context/`)
3. The `/code-context` command will no longer be available in new sessions
4. `code-context.md` has been left in place in the project — they can remove it manually with `rm code-context.md` if they wish
5. To reinstall later, they can follow the install instructions at https://github.com/leverj/claude-code-context