# claude-code-context

A Claude Code skill that generates and maintains a deep `code-context.md` file for your project. Eliminates cold-start exploration — every Claude session starts with full codebase understanding.

## What it does

- `/code-context` — Generates a comprehensive `code-context.md` from scratch by exploring your entire codebase
- `/update-context` — Updates the existing file after structural changes (new models, routes, components, etc.)

The generated file covers:
- Project structure and architecture
- All models/schemas with fields, indexes, relationships
- API routes/endpoints with methods, auth, and purpose
- Components, stores, hooks, and utilities
- Configuration, environment variables, and infrastructure
- Data flows and key patterns

## Install

### Option A: Global (available in all your projects)

```bash
# Clone the repo
git clone https://github.com/leverj/claude-code-context.git

# Copy skills to your global Claude config
cp -r claude-code-context/skills/* ~/.claude/skills/
```

### Option B: Per-project (committed to your repo)

```bash
# From your project root
mkdir -p .claude/skills
cp -r path/to/claude-code-context/skills/* .claude/skills/
```

### Option C: One-liner

```bash
# Global install
git clone https://github.com/leverj/claude-code-context.git ~/.claude/skills/code-context-tmp && \
  cp -r ~/.claude/skills/code-context-tmp/skills/* ~/.claude/skills/ && \
  rm -rf ~/.claude/skills/code-context-tmp
```

## Permissions and File Changes

### What files will be created or modified

This skill touches exactly **two files** in your project root. Nothing else.

| File | Action | When |
|------|--------|------|
| `code-context.md` | **Created** (or overwritten) | When you run `/code-context` |
| `code-context.md` | **Edited** (targeted sections) | When you run `/update-context`, or when Claude auto-updates after structural changes |
| `CLAUDE.md` | **Created** (if missing) or **appended to** (if exists) | Once, when you run `/code-context` — adds a "Code Context" section with read/update instructions |

**No other files are read-write.** The skill only **reads** your source files to understand the codebase. It never modifies your source code, configs, or any other project files.

### Tool permissions requested

#### `/code-context` (generation)

| Tool | Permission | What it does with it |
|------|-----------|---------------------|
| `Read` | Read any file | Reads source files, configs, READMEs to understand the codebase |
| `Glob` | Search file patterns | Finds files by pattern (e.g., `**/*.ts`, `src/models/*.js`) |
| `Grep` | Search file contents | Searches for keywords, exports, route definitions |
| `Bash(ls *)` | List directories | Maps directory structure |
| `Agent` | Spawn sub-agents | Parallelizes exploration across packages (read-only) |
| `Write` | Write files | Creates `code-context.md` and `CLAUDE.md` (if missing) — **only these two files** |
| `Edit` | Edit files | Appends "Code Context" section to existing `CLAUDE.md` — **only this file** |

#### `/update-context` (incremental update)

| Tool | Permission | What it does with it |
|------|-----------|---------------------|
| `Read` | Read any file | Reads changed source files to verify current state |
| `Glob` | Search file patterns | Finds files related to the change |
| `Grep` | Search file contents | Checks for new/removed exports, routes, models |
| `Bash(ls *)` | List directories | Checks directory structure |
| `Bash(git diff *)` | View git diffs | Identifies what changed recently |
| `Bash(git log *)` | View git history | Understands recent commits |
| `Agent` | Spawn sub-agents | Explores affected areas (read-only) |
| `Edit` | Edit files | Updates sections of `code-context.md` — **only this file** |

### What it does NOT do

- Does not modify any source code files
- Does not install dependencies or run build commands
- Does not make git commits
- Does not push to remote repositories
- Does not access the network or external APIs
- Does not read or write files outside the project root
- Does not delete any files

## Usage

### First time: Generate context

```
/code-context
```

This explores your entire codebase and generates `code-context.md` at the project root. Takes 1-3 minutes depending on project size.

### After structural changes: Update context

```
/update-context
```

Or with a description of what changed:

```
/update-context added new User model and /api/users endpoints
```

### Customization

You can pass arguments to `/code-context` to focus on specific areas:

```
/code-context focus on the API layer and database models
/code-context include deployment and infrastructure details
```

## What gets documented

The skill adapts to your tech stack. It detects and documents:

| Stack | What it captures |
|-------|-----------------|
| **Node.js/Express/Koa** | Routes, middleware, models (Mongoose/Sequelize/Prisma) |
| **React/Next.js** | Components, hooks, stores (Redux/Zustand/Context), pages |
| **React Native/Expo** | Screens, navigation, platform-specific files, native modules |
| **Swift/iOS** | Views, ViewModels, models, networking, persistence |
| **Python/Django/Flask** | Models, views/routes, serializers, management commands |
| **Go** | Handlers, models, middleware, packages |
| **Rust** | Modules, traits, handlers, types |
| **General** | Config files, CI/CD, Docker, environment setup, scripts |

## How it works

1. **Exploration**: Reads package configs, directory structure, and key source files (read-only)
2. **Analysis**: Identifies models, routes, components, stores, and their relationships
3. **Generation**: Writes `code-context.md` to project root and adds a "Code Context" section to `CLAUDE.md`
4. **Maintenance**: The CLAUDE.md instruction ensures Claude reads the context on startup and updates it when making structural changes — no manual upkeep needed

## Philosophy

- **Deep over broad** — Lists individual fields/props, not just file names
- **Relationships matter** — Documents how pieces connect (auth flow, data pipeline, API→store→component)
- **Key patterns** — Captures non-obvious conventions (composite IDs, platform-specific files, transaction patterns)
- **Living document** — Updated as part of the development workflow, not a one-time snapshot

## License

MIT
