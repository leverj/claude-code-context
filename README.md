# claude-code-context

A Claude Code skill that generates a persistent `code-context.md` for your project — models, routes, components, stores, architecture, data flows, and key patterns — so every Claude session starts with full codebase understanding instead of re-exploring from scratch.

**Why use it:**
- Eliminates cold-start exploration — Claude knows your codebase immediately
- Auto-updates when your code changes — tracks a git hash marker, detects drift, and updates itself
- Team-friendly — one developer runs `init` and commits `code-context.md`, everyone else gets instant context and only incremental updates from there

## Install

```bash
git clone https://github.com/leverj/claude-code-context.git /tmp/cc && cp -r /tmp/cc/skills/* ~/.claude/skills/ && rm -rf /tmp/cc
```

## Usage

```
/code-context init                  # Generate context for your project
/code-context update                # Update after structural changes
/code-context uninstall             # Remove from project (keeps code-context.md)
/code-context global-uninstall      # Full removal (deletes skill files too)
```

You can also pass hints to `init`:

```
/code-context init focus on the API layer and database models
```

After init, every new Claude session automatically checks if the codebase has changed and updates the context file — no manual upkeep needed.

---

## How it works

1. **Exploration** — Reads package configs, directory structure, and key source files (read-only)
2. **Generation** — Writes `code-context.md` (with a git hash marker) and adds a "Code Context" section to `CLAUDE.md`
3. **Auto-maintenance** — Each session compares the marker hash against current HEAD; if the codebase changed, it auto-updates the affected sections
4. **Uninstall** — Cleanly removes `CLAUDE.md` integration while preserving the generated documentation

## What gets documented

The skill adapts to your tech stack:

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

## Permissions and File Changes

This skill touches exactly **two files** in your project root. Nothing else.

| File | Action | When |
|------|--------|------|
| `code-context.md` | Created / edited | `init`, `update`, or auto-update on session start |
| `CLAUDE.md` | Created / appended / section removed | `init` adds a section, `uninstall` removes it |

It only **reads** your source files — never modifies source code, configs, or any other project files.

### What it does NOT do

- Does not modify any source code files
- Does not install dependencies or run build commands
- Does not make git commits or push to remote
- Does not access the network or external APIs
- Does not delete any files (uninstall leaves `code-context.md` in place)

## Philosophy

- **Deep over broad** — Lists individual fields/props, not just file names
- **Relationships matter** — Documents how pieces connect across packages
- **Living document** — Auto-updated via git hash change detection, not a one-time snapshot
- **Reversible** — Uninstall disables without destroying; re-init picks up where you left off

## License

MIT