---
title: 'Managing Multiple Copilot Sessions'
description: 'Learn how to run multiple GitHub Copilot CLI sessions in parallel, switch between them, and use worktrees for isolated concurrent work.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-09
estimatedReadingTime: '6 minutes'
tags:
  - sessions
  - worktrees
  - parallel-work
  - copilot-cli
relatedArticles:
  - ./using-copilot-coding-agent.md
  - ./github-copilot-app.md
  - ./automating-with-hooks.md
prerequisites:
  - GitHub Copilot CLI installed (v1.0.79+)
  - Basic familiarity with Copilot CLI sessions
---

GitHub Copilot CLI supports running multiple sessions simultaneously, so you can work on several tasks at once without losing context between them. A **Sessions sidebar** (or dedicated Sessions tab) lets you see all active sessions at a glance, spawn new ones, and switch between them — all from a single CLI window.

## The Sessions Sidebar

*(v1.0.79+)* The Sessions sidebar shows all your active sessions as a vertical list alongside your current conversation. From the sidebar you can:

- **Switch sessions** with arrow keys or a click
- **Spawn a new session** by pressing `n`
- **Close a session** by pressing `x` twice
- See the **status** of each session (idle, working, waiting for input)

### Enabling the Sessions Sidebar

The sidebar is on by default in v1.0.79+. If you don't see it, make sure you're on an up-to-date release:

```bash
copilot --version
```

You can also control the sidebar behavior from `/settings`:

```
/settings sidebar.hoverFocus on      # focus a session on mouse hover (off by default)
/settings sidebar.accentActiveSession off  # disable accent on the active session card
```

To restore sessions from a previous CLI run, the sidebar persists sessions across restarts automatically.

## Starting a Session in a New Worktree

Each session runs in its own directory. For parallel feature work, the best approach is to put each session in its own **git worktree** — an isolated checkout of your repository on a separate branch.

### Using `/worktree`

From any session, create a new worktree and switch into it:

```
/worktree fix-auth-bug
```

This creates a new branch `fix-auth-bug`, checks it out in a new worktree directory, and switches the current session to work there. Your uncommitted changes stay behind in the original worktree.

### Using `/new-worktree` (Experimental)

*(v1.0.78+)* The `/new-worktree` command creates a new worktree **and** starts a brand-new conversation in it, without affecting your current session:

```
/new-worktree add-dark-mode
```

This is the fastest way to kick off parallel work: your existing session keeps running, while a new session starts in the new `add-dark-mode` worktree. Both appear in the Sessions sidebar.

### `worktreeBaseRef` Setting

*(v1.0.79+)* By default, `/worktree`, `/worktree new`, and `--worktree` all start from `HEAD`. To change the base branch for new worktrees, set `worktreeBaseRef`:

```json
// settings.json
{
  "worktreeBaseRef": "origin/main"
}
```

Setting this to `origin/main` means new worktrees always branch from the remote default branch, which is useful in trunk-based workflows where you want to start from the latest main rather than your current working branch.

## Practical Multi-Session Workflows

### Pattern 1: Work + Review in Parallel

1. Start a session to implement a feature
2. Open the Sessions sidebar (`n` to spawn a new session)
3. In the new session, review a separate PR or triage issues
4. Switch between sessions as needed without losing context in either

### Pattern 2: Two Parallel Features

1. In session A: `/new-worktree feature-search` — start building the search feature
2. In session B (automatically opened): work on the search feature
3. Switch back to your original session: continue unrelated work
4. Monitor both from the Sessions sidebar

### Pattern 3: Long-Running Agent + Interactive Work

1. Set session A to autopilot on a well-defined task:
   ```
   /mode autopilot
   ```
2. Open a new session with `n`
3. In session B: do interactive work while the agent in session A runs autonomously
4. Check back on session A from the sidebar when the task completes

## Switching Sessions

Within the Sessions sidebar:

- **Arrow keys** navigate the session list
- **Enter** or a **click** switches to the selected session
- `n` spawns a new session
- `x` twice closes the focused session

MCP servers and hook state are preserved when switching sessions — switching never restarts your MCP connections or rebuilds hook state.

## Common Questions

**Q: Do sessions share MCP servers?**

A: Yes. MCP servers are shared across sessions within a single CLI process. Switching between sessions doesn't disconnect or restart them.

**Q: What happens to a running agent when I switch sessions?**

A: The agent keeps running in the background. You can switch back to monitor its progress at any time. You'll also see status indicators in the sidebar when an agent is working.

**Q: How many sessions can I run at once?**

A: There's no hard limit. Performance depends on your machine and the number of active agents. Each idle session uses minimal resources.

**Q: Can I resume sessions after restarting the CLI?**

A: Yes. The Sessions sidebar persists sessions across restarts. Use `copilot --resume` to pick a previous session from the session picker.

## Further Reading

- [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — Autonomous agent sessions and remote control
- [Getting Started with the GitHub Copilot app](../github-copilot-app/) — Parallel sessions in the desktop app with worktrees
- [Automating with Hooks](../automating-with-hooks/) — Add guardrails that run across all your sessions
- [GitHub Copilot CLI changelog](https://github.com/github/copilot-cli/blob/main/changelog.md) — Full release notes for session management improvements

---
