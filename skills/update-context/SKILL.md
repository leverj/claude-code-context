---
name: update-context
description: Update the existing code-context.md file after structural changes (new/removed models, routes, components, stores, modules, config). Use after making changes that affect the codebase architecture.
allowed-tools: Agent Bash(ls *) Bash(git diff *) Bash(git log *) Glob Grep Read Edit
argument-hint: "[description of what changed]"
effort: medium
---

# Update Code Context

You are updating the existing `code-context.md` file to reflect recent structural changes.

## What changed

$ARGUMENTS

## Process

### Step 1: Understand current state

1. Read the existing `code-context.md`
2. If the user described what changed, focus on those areas
3. If no description provided, check recent changes:
   - Run `git diff --name-only HEAD~5` to see recently changed files
   - Run `git log --oneline -10` to see recent commits
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

Update the "Last updated" date in the header to today's date.

## Guidelines

- **Only change what's affected** — don't rewrite the entire file
- **Match existing style** — use the same table format, heading structure, and level of detail
- **Verify before writing** — read the actual source files, don't guess from commit messages alone
- **Remove stale info** — if something was deleted, remove it from the doc rather than marking it as removed
- **Keep it accurate** — if you're unsure about a change, read the source to confirm

## After updating

Tell the user what sections were updated and why.
