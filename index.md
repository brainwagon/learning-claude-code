# Claude Code: Learning Notes

**Note from the human:** These notes are largely an experiment in learning about Claude by asking Claude Code itself 
about how things work, and about how best to use its functionality.   I find that many tutorials and videos are less
than completely helpful in answering questions related to how best to utilize Claude to accomplish tasks in the most
economical way possible.   As time goes on, I may update these notes.

-- 

These are concise, opinionated reference notes on Claude Code features — written by someone actively using the tool and documenting what actually matters. Each page trades breadth for density: no padding, no marketing copy, just the syntax, the mental model, and the gotchas. Use these as a quick-reference when you need to remember how something works, or as a starting point before diving into the official docs.

---

## Topics

- **`tasks-and-subagents`** — [Tasks and Subagents](tasks-and-subagents.md): persistent work tracking with Tasks, parallel isolated Claude instances with Subagents, and how they combine into a managed workflow.
- **`git-worktrees`** — [Git Worktrees](git-worktrees.md): running agents in isolated git worktrees, automatic cleanup behavior, and best practices for sandboxed edits.
- **`billing`** — [Billing and Account Types](billing.md): Claude.ai Pro vs. raw API, which is right for you, how to check your auth status, and how to monitor usage.
- **`skills`** — [Skills](skills.md): named operations that extend Claude Code (update-config, keybindings-help, simplify, loop, schedule, and auto-triggered skills).
- **`hooks`** — coming soon: pre/post-tool hooks, settings.json hook configuration, and setting up automated behaviors.
- **`mcp-servers`** — coming soon: what MCP servers are, configuring them, and the built-in vs. custom server landscape.
- **`memory-system`** — coming soon: the file-based memory system, memory types, and maintaining a personal knowledge base across sessions.
- **`permissions-and-settings`** — coming soon: allow/deny rules, settings.json structure, and per-project vs. global configuration.

---

## Quick Reference

| Topic page | Covers |
|---|---|
| `tasks-and-subagents` | TaskCreate/Update/Get/Stop; subagent types; isolation; background execution |
| `git-worktrees` | Worktree mechanics; isolation parameter; cleanup behavior; shell commands |
| `billing` | Claude.ai Pro vs. API; auth status; usage monitoring; console access |
| `skills` | update-config, keybindings-help, simplify, loop, schedule; auto-triggered skills; invocation |
| `hooks` | Planned — hook events, configuration, automation patterns |
| `mcp-servers` | Planned — server setup, built-in servers, custom server development |
| `memory-system` | Planned — memory types, MEMORY.md index, file structure |
| `permissions-and-settings` | Planned — permission rules, settings.json, project vs. global scope |
