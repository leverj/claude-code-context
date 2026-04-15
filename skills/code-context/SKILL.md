---
name: code-context
description: Generate a comprehensive code-context.md file that captures deep codebase understanding — models, routes, components, stores, architecture, data flows, and key patterns. Use when starting a new project or when code-context.md is missing/outdated.
allowed-tools: Agent Bash(ls *) Glob Grep Read Write Edit
argument-hint: "[optional focus area]"
effort: high
---

# Generate Code Context

You are generating a `code-context.md` file for this project. This file serves as a persistent codebase understanding cache — it eliminates the need for every Claude session to re-explore the codebase from scratch.

## User instructions

$ARGUMENTS

## Process

### Step 1: Discover project structure

Start by understanding the project at a high level:

1. Read the root `package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, `build.gradle`, `Package.swift`, or equivalent to determine the tech stack
2. Check for monorepo indicators: `lerna.json`, `pnpm-workspace.yaml`, `workspaces` in package.json, `Cargo.toml` workspace members
3. Read any existing `README.md`, `CLAUDE.md`, `ARCHITECTURE.md`, or similar docs
4. Map the top-level directory structure

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

Write the file to `code-context.md` in the project root with this structure:

```markdown
# [Project Name] Code Context

> **Purpose**: Persistent codebase understanding for Claude sessions. Eliminates cold-start exploration.
> **Last updated**: [today's date]
> **Update rule**: Update this file when making structural changes (new models, routes, components, stores, modules, config changes, or architectural shifts).

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

## Quality guidelines

- **Be deep**: List individual model fields, route paths, component props — not just file names
- **Be precise**: Include actual values (port numbers, default configs, thresholds) not vague descriptions
- **Be relational**: Show how pieces connect (which store calls which API, which model relates to which)
- **Be practical**: Include the patterns someone needs to know to work in this codebase safely
- **Use tables**: For models, routes, config vars — they're scannable and dense
- **Include types/enums**: State shapes, status enums, error types — these prevent bugs
- **Skip boilerplate**: Don't document obvious framework conventions (React component lifecycle, Express middleware signature)

### Step 4: Set up CLAUDE.md for auto-maintenance

After generating `code-context.md`, ensure the project's `CLAUDE.md` has instructions for Claude to read and maintain it. This is what makes the system self-sustaining — Claude will automatically update `code-context.md` when making structural changes in future sessions.

1. Check if `CLAUDE.md` exists in the project root
2. If it exists, read it and check if it already has a "Code Context" section
3. If the section is missing, append it using the Edit tool
4. If `CLAUDE.md` doesn't exist, create it using the Write tool

The section to add:

```markdown
## Code Context

At the start of every session, read `code-context.md` in the project root if it exists.
This file contains deep codebase understanding and eliminates exploratory warm-up.

When you make structural changes (new/removed models, routes, components, stores, modules,
config changes, or architectural shifts), update `code-context.md` before completing the task.
```

**This step is not optional.** Without CLAUDE.md setup, the context file becomes a one-time snapshot that goes stale. The CLAUDE.md instruction is what closes the loop — making every future session both a consumer and maintainer of the context.

## After generation

Tell the user:
1. `code-context.md` has been generated at the project root
2. `CLAUDE.md` has been set up (or updated) to auto-read and auto-maintain the context file
3. From now on, every Claude session will start with full codebase understanding and update the file when making structural changes
4. They can run `/update-context` manually after large refactors or to describe what changed
5. Suggest they review the file and flag any inaccuracies or missing areas
