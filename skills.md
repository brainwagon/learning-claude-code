# Claude Code: Skills

Skills are specialized tools that extend Claude Code's capabilities beyond basic code editing and bash execution. They provide domain-specific automation, configuration helpers, and integrations. Each skill is a reusable, invoke-by-name operation that can be triggered from within a Claude Code session.

---

## What Skills Are

A skill is a named operation you invoke with `/skill-name` or via the Skill tool. It's distinct from:

- **Built-in commands** (like `/help`, `/clear`) — these are hardcoded features of Claude Code
- **Tools** (like Bash, Read, Edit) — these are low-level operations Claude can call directly
- **Agents** (subagents) — these are isolated Claude instances that run in parallel; skills are synchronous

Skills are often **hooks into your environment** — they read settings, modify configuration, run local commands, and feed results back into your current session.

---

## Available Skills

### update-config

Configure Claude Code settings via `settings.json`. Used for:
- Permissions: allow/deny rules, move permissions between scopes
- Environment variables: `set DEBUG=true`
- Hook configuration: set up pre/post-tool automation
- Settings troubleshooting

**Invoke:** `/update-config` or use the `update-config` skill in tooling.

**Example use cases:**
- "allow npm commands"
- "move permission to user settings"
- "add bq permission to global settings"
- "set DEBUG=true"

---

### keybindings-help

Customize keyboard shortcuts and rebind keys. Modifies `~/.claude/keybindings.json`.

Used for:
- Adding custom key bindings (e.g., `Ctrl+K Ctrl+F` for format)
- Rebinding existing keys
- Chord bindings (multi-key sequences)
- Viewing current bindings

**Invoke:** `/keybindings-help` or use the `keybindings-help` skill.

**Example use cases:**
- "rebind ctrl+s to submit"
- "add a chord shortcut for formatting"
- "change the submit key"
- "customize keybindings"

---

### simplify

Review changed code for quality, reuse, and efficiency. Scans your changes and suggests improvements.

Used for:
- Code review of your edits
- Finding duplicate logic
- Identifying inefficiencies
- Quality and style checks

**Invoke:** `/simplify`

**Note:** This skill audits the current session's changes; it does not modify code directly.

---

### fewer-permission-prompts

Reduce permission prompts by scanning your transcript for common read-only operations and adding an allowlist to `settings.json`.

Used for:
- Auditing repeated permission requests
- Creating smart permission presets
- Reducing friction in development flow

**Invoke:** `/fewer-permission-prompts`

**Behavior:** Scans your command history, identifies safe operations (like `Read`, `Grep`, `Bash` with certain patterns), and suggests a prioritized allowlist to add to project settings. This lets you approve common operations once and avoid re-prompting.

---

### loop

Run a prompt or slash command on a recurring interval. Self-paces when no interval is specified.

Used for:
- Polling for status: "check the deploy every 5 minutes"
- Recurring tasks: "run the tests every 30 seconds"
- Long-running monitors: "keep babysitting the PRs"

**Invoke:** `/loop [interval] <prompt>` or `/loop <prompt>` (self-paced)

**Interval format:** `5s`, `1m`, `30m`, etc. Omit for dynamic pacing.

**Example:**
```
/loop 5m check git status
/loop 30s run the tests
/loop every 10 minutes verify the deploy
```

---

### schedule

Create, update, list, or run scheduled remote agents (triggers) that execute on a cron schedule.

Used for:
- Persistent automated tasks: "run this job every Tuesday at 9am"
- Recurring agents: "check PRs every hour"
- Cron job management: create, list, update, delete schedules

**Invoke:** `/schedule` or use the `schedule` skill.

**Example use cases:**
- "schedule a reminder at 2pm"
- "create a daily deploy check at 9am"
- "list my scheduled tasks"
- "delete the 3pm job"

---

### claude-api

Build, debug, and optimize Claude API / Anthropic SDK apps. Handles:
- Building new API integrations
- Debugging existing API code
- Optimizing with prompt caching
- Model migrations (upgrading between Claude versions)

**Trigger conditions:**
- You're writing or modifying code that imports `anthropic` or `@anthropic-ai/sdk`
- You ask about Claude API, Anthropic SDK, or Managed Agents
- You're adding/tuning Claude features (caching, thinking, tool use, batch, files)

**Note:** This skill auto-triggers when appropriate; you can also invoke it explicitly.

---

### frontend-design

Create distinctive, production-grade frontend interfaces. Generates creative, polished web UI without generic AI aesthetics.

**Trigger conditions:**
- You ask to build web components, pages, dashboards, or applications
- You request styling/beautification of web UI
- Examples: websites, landing pages, React components, HTML/CSS layouts

**Note:** This skill auto-triggers when appropriate; can also invoke explicitly.

---

### project-wiki

Build and maintain a persistent auto-updating knowledge base for your codebase (Karpathy-style wiki).

**Trigger conditions:**
- You modify project files (code, configs, docs)
- You add new modules or refactor architecture
- You ask about project architecture ("how does auth work", "where is X implemented")
- You request architecture/documentation health checks

**Note:** This skill auto-triggers when appropriate; useful for large projects with evolving architecture.

---

### wiki

Build and maintain a persistent Karpathy Wiki — a knowledge base for external sources and research.

**Trigger conditions:**
- You add sources to a `raw/` directory (articles, papers, notes)
- You ask synthesis questions ("what do we know about X", "compare A and B")
- You ask to add content ("ingest this", "add to wiki")

**Note:** Distinct from `project-wiki` — this is for external knowledge, not project code.

---

### init

Initialize a new `CLAUDE.md` file with codebase documentation.

Used for:
- Starting a new project's conventions file
- Setting up content guidelines and rules
- Establishing per-project Claude instructions

**Invoke:** `/init`

---

### review

Review a pull request.

**Invoke:** `/review`

Used for:
- Formal PR review of the pending changes
- Code quality and testing verification

---

### security-review

Complete a security review of pending changes on the current branch.

**Invoke:** `/security-review`

Used for:
- Vulnerability scanning of code changes
- OWASP top-10 checks
- Security anti-patterns

---

## How to Invoke Skills

Skills are invoked in two ways:

### Direct slash command (most common)

```
/skill-name
```

Example: `/update-config` or `/simplify`

### Via the Skill tool (programmatic)

If you're using Claude Code in a script or custom agent:

```
Skill("skill-name", args="optional arguments")
```

---

## Skill Availability

Not all skills are always available. Skills can be:

- **Always available** (core skills): `update-config`, `keybindings-help`, `simplify`, `loop`, `init`, `review`
- **Conditional** (auto-trigger when context matches): `claude-api`, `frontend-design`, `project-wiki`, `wiki`
- **Situational** (specific to your configuration): custom skills defined in `.claude/settings.json`

To see available skills in your session:

```
/help
```

This shows active skills and built-in commands. If a skill is missing, it may be disabled in settings or not applicable to the current context.

---

## Tips

**Skill vs. Tool:** A skill is high-level and often interactive (asks questions, gives you options). A tool is low-level and deterministic. Use skills when you want Claude to reason about what to do; use tools for direct, programmatic operations.

**Skill vs. Agent:** Skills are synchronous and run in the current session. Agents are parallel, isolated Claude instances. Use skills for interactive configuration or review; use agents for long-running, parallel work.

**Permission prompts:** Skills that modify your system (like `update-config`) may trigger permission prompts. These are expected and safe — they're built-in safeguards. Use `/fewer-permission-prompts` to reduce friction if needed.

**Skill discovery:** Check `/help` regularly as new skills are added to Claude Code. Your installed version may have skills not documented here yet.

