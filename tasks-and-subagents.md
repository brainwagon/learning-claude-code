# Claude Code: Tasks and Subagents

Claude Code can handle complex, multi-file work without losing track of where it is. Tasks give Claude a persistent work ledger visible across tool calls; Subagents let you spawn isolated Claude instances for parallelism, clean context, or specialized roles. Together they transform a single-shot prompt into something closer to a managed development workflow.

---

## Tasks

A Task is a tracked work unit — think of it as a sticky note Claude can read back at any point during a long session. Unlike a mental note buried in the conversation, a Task persists, has a status, and can carry structured output.

**When to create one**: any time work has more than two discrete steps, might span multiple tool calls, or needs a clear definition of done. Refactoring a module, implementing a feature end-to-end, writing a test suite — these warrant tasks. Answering a quick question does not.

**Creating and updating**:

```
TaskCreate(
  title="Add JWT refresh token support",
  description="Implement refresh token issuance in auth/token.py, add rotation logic, update tests in tests/test_auth.py",
  acceptance_criteria="All existing auth tests pass; new refresh token tests cover issuance, rotation, and expiry"
)
```

During work, call `TaskUpdate` to record progress and keep the status accurate:

```
TaskUpdate(task_id="abc123", status="in_progress", notes="Issued token logic done; starting rotation")
TaskUpdate(task_id="abc123", status="completed")
```

Use `TaskGet` to re-read a task's current state mid-session, and `TaskList` to see everything in flight. If a task gets pre-empted, `TaskStop` marks it cleanly rather than leaving it dangling.

**Why this matters**: without tasks, Claude's working state lives only in the conversation context. As that context grows, earlier intent gets diluted. A task externalizes the goal — Claude can check `TaskGet` at any point and know exactly what "done" looks like without re-reading the entire thread. For you, it also creates a clear audit trail of what was attempted and why.

---

## Subagents

Subagents are independent Claude instances. They start with no knowledge of the parent conversation, which is both a constraint and a feature.

**Three reasons to reach for them**:

1. **Parallelism** — run multiple investigations simultaneously instead of sequentially. Exploring how three different subsystems handle error propagation is three separate reads; there is no reason to do them one at a time.

2. **Context protection** — a subagent that goes deep into a rabbit hole (reading 40 files, fetching docs) does not pollute the parent context. The parent gets a summary; the noise stays isolated.

3. **Specialization** — some agent types have restricted tool sets that prevent accidents. A `Plan` agent cannot write files, so you get a design doc without risk of premature edits.

**Agent types and when to use each**:

```
# Fast codebase search — no write access, very cheap
Agent(subagent_type="Explore", prompt="Find all uses of the legacy Config struct and map which files will need changes")

# Full-capability research or multi-step task
Agent(subagent_type="general-purpose", prompt="Research the best approach for connection pooling in asyncpg, then draft an implementation plan")

# Design without risk of modifying files
Agent(subagent_type="Plan", prompt="Design the migration path from callbacks to async/await in src/networking/, including file-by-file sequencing")
```

**Foreground vs. background**: foreground agents block and return results inline. Background agents (`run_in_background=True`) fire and continue — use them when you want work happening in parallel while you proceed with something else.

```
Agent(subagent_type="Explore", prompt="Audit test coverage gaps in src/parser/", run_in_background=True)
```

**Worktree isolation**: when a subagent will make actual file changes, `isolation="worktree"` gives it a separate git worktree. Changes are sandboxed until you decide to merge. Essential when running multiple agents that touch overlapping parts of the codebase.

```
Agent(subagent_type="general-purpose", prompt="Implement the new caching layer per the design doc", isolation="worktree")
```

---

## Combining Them: A Real Scenario

You need to add WebSocket support to an existing HTTP service. Here is how tasks and subagents work together:

```
TaskCreate(title="Add WebSocket support", description="...", acceptance_criteria="Integration tests pass for WS upgrade path")
```

Then fire parallel exploration agents while the task tracks the overall goal:

```
Agent(subagent_type="Explore", prompt="Map current HTTP handler structure in src/server/", run_in_background=True)
Agent(subagent_type="Explore", prompt="Find all integration test fixtures relevant to connection handling", run_in_background=True)
```

Once you have the map, a `Plan` agent designs the approach without touching files. You review, then a `general-purpose` agent with `isolation="worktree"` does the implementation. When it's done, you merge the worktree and call `TaskUpdate(status="completed")`.

The task kept the goal crisp across all those context switches. The subagents kept the parent context clean and the exploration parallel.

---

## Quick Reference

| Tool / Parameter | Use it when |
|---|---|
| `TaskCreate` | Work has >2 steps or needs explicit acceptance criteria |
| `TaskUpdate` | Status changes, or you want to record progress notes |
| `TaskGet` / `TaskList` | Re-orienting mid-session or reviewing all open work |
| `TaskStop` | Pre-empting a task cleanly before it completes |
| `subagent_type="Explore"` | Read-only codebase mapping; fastest, cheapest |
| `subagent_type="Plan"` | Design work that must not write files |
| `subagent_type="general-purpose"` | Multi-step tasks that need full tool access |
| `run_in_background=True` | Parallel work you do not need to block on |
| `isolation="worktree"` | Any subagent that will write files |
| `model="opus"` | Tasks requiring stronger reasoning; slower and more expensive |
| `model="haiku"` | Fast, cheap subtasks where output quality is less critical |
