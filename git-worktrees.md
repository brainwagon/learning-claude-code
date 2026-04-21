# Claude Code: Git Worktrees

Git worktrees let one repository have multiple working directories checked out simultaneously, each on its own branch. Claude Code exposes worktrees as a first-class isolation primitive through the `isolation="worktree"` parameter on the Agent tool. The payoff: agents doing real file edits run in a sandboxed branch, leaving your working tree untouched until you choose to merge.

---

## What Git Worktrees Are

A worktree is a separate directory linked to the same `.git` object store. Each worktree checks out a different branch; commits in one are immediately visible to all others via the shared object store. Think of it as checking out multiple branches simultaneously — far cheaper than cloning, because you share the object store.

The key distinction from cloning: clones copy the entire object store; worktrees share it. Much faster, no disk duplication. The practical limit: one worktree per branch. You cannot check out the same branch in two worktrees at the same time.

**`git worktree add`**: creates a new worktree. The command checks out an existing branch into a new directory:

```
git worktree add ../myproject-feature feature/auth-refactor
```

This creates a new directory at `../myproject-feature` with the branch `feature/auth-refactor` checked out in it. The `.git` folder remains in the original directory — it is a symbolic reference, not a copy.

**`git worktree list`**: shows all linked worktrees and their current HEAD commits:

```
/home/user/myproject (e4a1f2c) main
/home/user/myproject-feature (7d3b9e2) feature/auth-refactor
```

**`git worktree remove`**: removes a worktree when you are done with it:

```
git worktree remove ../myproject-feature
```

This works only if the worktree has no uncommitted changes. If you want to discard changes, use `git worktree remove --force`.

---

## How Claude Code Uses Worktrees

When you spawn an Agent with `isolation="worktree"`, Claude Code creates a temporary directory, checks out a fresh branch, and runs the entire agent within that worktree. Your main working tree is untouched.

**Mechanics**: the `isolation="worktree"` parameter tells Claude Code to:
1. Create a new git worktree in a temporary location (under `.claude/worktrees/` by default).
2. Check out a new branch from the current HEAD.
3. Run the agent entirely within that directory.
4. After the agent completes, automatically clean up if no file changes were made.

**Auto-cleanup behavior**: if the agent runs but makes no file modifications, the worktree is removed automatically. No garbage accumulation, no manual cleanup needed. This is safe — you can fire many exploratory agents with `isolation="worktree"` defensively; the worktrees vanish if they do not produce work.

**When changes are made**: if the agent does write files, the worktree is left in place. The return value of the `Agent()` call contains the `path` and `branch` name so the parent context can inspect the diff, run tests, and decide whether to merge or discard.

**Code example — spawning a worktree agent**:

```
Agent(
    subagent_type="general-purpose",
    prompt="Refactor the database connection pool in src/db/pool.py to use asyncpg; update all call sites in src/api/",
    isolation="worktree"
)
```

**Code example — result object when changes were made**:

```
# result when the agent made file changes
{
    "path": "/home/user/myproject-worktrees/agent-abc123",
    "branch": "claude/agent-abc123",
    "summary": "Refactored pool.py to asyncpg; updated 7 call sites across src/api/"
}
```

The `path` is the absolute filesystem path to the worktree directory. The `branch` is the branch name Claude Code created for the agent. If the agent made no changes, the worktree is removed and the result contains neither `path` nor `branch` — just the `summary`.

---

## When to Use Worktree Isolation

**Four cases where isolation matters**:

1. **Multiple agents touching overlapping files** — without isolation, two concurrent agents writing to the same file will conflict. Each agent in its own worktree writes to its own branch; merging is explicit and controlled.

2. **Destructive edits** — refactors, renames, large deletions. If the agent takes a wrong turn, you discard the worktree rather than undoing a pile of changes in your working tree.

3. **Clean test baseline** — a worktree branch starts from the current HEAD, so test runs in the agent's context reflect exactly what changed, not accumulated local modifications.

4. **Exploratory spikes** — the agent does the work; you inspect the diff; you merge or delete. No `git stash` archaeology. You maintain full control.

**When not to use it**: read-only agents (`Explore`, `Plan`) make no file changes; isolation adds overhead with no benefit. Similarly, trivial single-file edits where you want the change inline immediately do not warrant a worktree.

---

## Shell Commands Reference

**`git worktree add`**: creates a new worktree by checking out a branch into a new directory. The base form is `git worktree add <path> <branch>`. You can also create a new branch at the same time using the `-b` flag: `git worktree add -b feature/payment-webhooks ../myproject-payments main`. This creates a new worktree at `../myproject-payments` with a new branch `feature/payment-webhooks` based on `main`.

**`git worktree list`**: displays all linked worktrees with their HEAD commits. Claude Code's automatically created worktrees appear here while active. Here is a representative output:

```
/home/user/myproject (e4a1f2c) main
/home/user/myproject-worktrees/agent-abc123 (1b7e3f9) claude/agent-abc123
```

The first column is the path, the second is the commit hash, the third is the branch name.

**`git worktree remove`**: removes a worktree. It only works if the worktree has no uncommitted changes. If the agent left work there, you must either commit or discard first. Use `git worktree remove --force` to discard uncommitted changes and remove the worktree in one command.

**`git worktree prune`**: cleans up stale worktree metadata. If a worktree directory was deleted manually rather than via `git worktree remove`, the metadata remains. `git worktree prune` removes it. Useful after manually discarding Claude Code worktrees.

---

## Quick Reference

| Command / Parameter | What it does |
|---|---|
| `git worktree add <path> <branch>` | Check out an existing branch into a new directory |
| `git worktree add -b <branch> <path> <start>` | Create a new branch and check it out in a new directory |
| `git worktree list` | Show all linked worktrees with their HEAD commits |
| `git worktree remove <path>` | Remove a worktree with no uncommitted changes |
| `git worktree remove --force <path>` | Remove a worktree, discarding any uncommitted changes |
| `git worktree prune` | Remove stale worktree metadata after manual directory deletion |
| `isolation="worktree"` | Run an Agent in its own worktree; result includes path and branch if changes were made |
