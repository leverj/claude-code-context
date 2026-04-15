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

## Setup

After installing the skills, add these lines to your project's `CLAUDE.md` (create one if it doesn't exist):

```markdown
## Code Context

At the start of every session, read `code-context.md` in the project root if it exists.
This file contains deep codebase understanding and eliminates exploratory warm-up.

When you make structural changes (new/removed models, routes, components, stores, modules,
config changes, or architectural shifts), update `code-context.md` before completing the task.
```

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

1. **Exploration**: Reads package configs, directory structure, and key source files
2. **Analysis**: Identifies models, routes, components, stores, and their relationships
3. **Generation**: Produces a structured markdown document optimized for Claude's context window
4. **Maintenance**: The CLAUDE.md instruction ensures Claude updates the file when making structural changes

## Philosophy

- **Deep over broad** — Lists individual fields/props, not just file names
- **Relationships matter** — Documents how pieces connect (auth flow, data pipeline, API→store→component)
- **Key patterns** — Captures non-obvious conventions (composite IDs, platform-specific files, transaction patterns)
- **Living document** — Updated as part of the development workflow, not a one-time snapshot

## License

MIT
