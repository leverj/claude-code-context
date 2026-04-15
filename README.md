# claude-code-context

A Claude Code skill that generates and maintains a deep `code-context.md` file for your project. Eliminates cold-start exploration — every Claude session starts with full codebase understanding.

## What it does

`/code-context` is a single command with three subcommands:

- `/code-context init` — Generates a comprehensive `code-context.md` from scratch (or rebuilds from an existing one)
- `/code-context update` — Updates the existing file after structural changes
- `/code-context uninstall` — Removes CLAUDE.md integration while preserving `code-context.md`
- `/code-context global-uninstall` — Full removal: project uninstall + deletes skill files from `~/.claude/skills/`

The generated file covers:
- Project structure and architecture
- All models/schemas with fields, indexes, relationships
- API routes/endpoints with methods, auth, and purpose
- Components, stores, hooks, and utilities
- Configuration, environment variables, and infrastructure
- Data flows and key patterns

### Automatic change detection

The generated `code-context.md` includes a git commit hash marker (`<!-- code-context-marker: <hash> -->`). At the start of every new Claude session, Claude checks if the codebase has changed since the last update. If it has, it automatically updates the affected sections — no manual intervention needed.

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
| `code-context.md` | **Created** (or overwritten) | When you run `/code-context init` |
| `code-context.md` | **Edited** (targeted sections) | When you run `/code-context update`, or when Claude auto-updates after detecting changes |
| `CLAUDE.md` | **Created** (if missing) or **appended to** (if exists) | Once, during `/code-context init` — adds a "Code Context" section |
| `CLAUDE.md` | **Section removed** | When you run `/code-context uninstall` |

**No other files are read-write.** The skill only **reads** your source files to understand the codebase. It never modifies your source code, configs, or any other project files.

### Tool permissions requested

| Tool | Permission | What it does with it |
|------|-----------|---------------------|
| `Read` | Read any file | Reads source files, configs, READMEs to understand the codebase |
| `Glob` | Search file patterns | Finds files by pattern (e.g., `**/*.ts`, `src/models/*.js`) |
| `Grep` | Search file contents | Searches for keywords, exports, route definitions |
| `Bash(ls *)` | List directories | Maps directory structure |
| `Bash(git diff *)` | View git diffs | Identifies what changed recently (used by `update`) |
| `Bash(git log *)` | View git history | Understands recent commits (used by `update`) |
| `Bash(git rev-parse *)` | Get git commit hash | Reads current HEAD hash for the change detection marker |
| `Bash(rm -rf *)` | Remove directories | Removes skill files during `global-uninstall` only |
| `Agent` | Spawn sub-agents | Parallelizes exploration across packages (read-only) |
| `Write` | Write files | Creates `code-context.md` and `CLAUDE.md` (if missing) — **only these two files** |
| `Edit` | Edit files | Updates sections of `code-context.md` and `CLAUDE.md` — **only these files** |

### What it does NOT do

- Does not modify any source code files
- Does not install dependencies or run build commands
- Does not make git commits
- Does not push to remote repositories
- Does not access the network or external APIs
- Does not read or write files outside the project root
- Does not delete any files (uninstall leaves `code-context.md` in place)

## Usage

### Initialize: Generate context

```
/code-context init
```

This explores your entire codebase and generates `code-context.md` at the project root. Also sets up `CLAUDE.md` for automatic change detection and maintenance.

If a `code-context.md` already exists (e.g., from a previous init or after an uninstall), it picks up from that file — preserving what's still accurate and updating the rest.

### Update: Refresh after changes

```
/code-context update
```

Or with a description of what changed:

```
/code-context update added new User model and /api/users endpoints
```

This is also triggered automatically at the start of each Claude session if the git HEAD has changed since the last update.

### Uninstall: Remove integration

```
/code-context uninstall
```

This removes the "Code Context" section from `CLAUDE.md`, which stops Claude from auto-reading and auto-updating the context file. The `code-context.md` file itself is **not deleted** — you can remove it manually if you wish, or leave it for a future re-init.

### Global uninstall: Full removal

```
/code-context global-uninstall
```

This does everything `uninstall` does, plus removes the skill files from `~/.claude/skills/code-context/` (or `.claude/skills/code-context/` for per-project installs). The `/code-context` command will no longer be available after this. `code-context.md` is still preserved.

### Customization

You can pass arguments to `init` to focus on specific areas:

```
/code-context init focus on the API layer and database models
/code-context init include deployment and infrastructure details
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
3. **Generation**: Writes `code-context.md` (with git hash marker) to project root and adds a "Code Context" section to `CLAUDE.md`
4. **Auto-maintenance**: At each session start, Claude compares the marker hash against current HEAD — if the codebase changed, it auto-updates the affected sections
5. **Uninstall**: Cleanly removes CLAUDE.md integration while preserving the generated documentation
6. **Global uninstall**: Full removal of skill files from the system

## Philosophy

- **Deep over broad** — Lists individual fields/props, not just file names
- **Relationships matter** — Documents how pieces connect (auth flow, data pipeline, API->store->component)
- **Key patterns** — Captures non-obvious conventions (composite IDs, platform-specific files, transaction patterns)
- **Living document** — Auto-updated via git hash change detection, not a one-time snapshot
- **Reversible** — Uninstall disables without destroying; re-init picks up where you left off

## License

MIT