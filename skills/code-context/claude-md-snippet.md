## CLAUDE.md Snippet

Add the following to your project's `CLAUDE.md` file:

---

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

---