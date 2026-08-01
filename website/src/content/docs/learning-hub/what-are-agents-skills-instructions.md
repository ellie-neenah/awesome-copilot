---
title: 'What are Agents, Skills, and Instructions'
description: 'Understand the primary customization primitives that extend GitHub Copilot for specific workflows.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-01
estimatedReadingTime: '9 minutes'
prev: false
---

Building great experiences with GitHub Copilot starts with understanding the core primitives that shape how Copilot behaves in different contexts. This article clarifies what each artifact does, how it is packaged inside this repository, and when to use it.

## Agents

Agents are configuration files (`*.agent.md`) that describe:

- The tasks they specialize in (for example, "Terraform Expert" or "LaunchDarkly Flag Manager").
- Which tools or MCP servers they can invoke.
- Optional instructions that guide the conversation style or guardrails.

When you assign an issue to Copilot or open the **Agents** panel in VS Code, these configurations let you swap in a specialized assistant. Each agent in this repo lives under `agents/` and includes metadata about the tools it depends on.

In products that support delegation, a primary agent can also launch temporary subagents for focused work such as planning, research, or review. See [Agents and Subagents](../agents-and-subagents/) for the coordination model.

### When to reach for an agent

- You have a recurring workflow that benefits from deep tooling integrations.
- You want Copilot to proactively execute commands or fetch context via MCP.
- You need persona-level guardrails that persist throughout a coding session.
- You want a coordinator that can delegate narrower work to subagents.

## Skills

Skills are self-contained folders that package reusable capabilities for GitHub Copilot. Each skill lives in its own directory and contains a `SKILL.md` file along with optional bundled assets such as reference documents, templates, and scripts.

A `SKILL.md` defines:

- A **name** (used as a `/command` in VS Code Chat and for agent discovery).
- A **description** that tells agents and users when the skill is relevant.
- Detailed instructions for how the skill should be executed.
- References to any bundled assets the skill needs.

Skills follow the open [Agent Skills specification](https://agentskills.io/home), making them portable across coding agent systems beyond GitHub Copilot.

### Why skills over prompts

Skills replace the earlier prompt file (`*.prompt.md`) pattern and offer several advantages:

- **Agent discovery**: Skills include extended frontmatter that lets agents find and invoke them automatically—prompts could only be triggered manually via a slash command.
- **Richer context**: Skills can bundle reference files, scripts, templates, and other assets alongside their instructions, giving the AI much more to work with.
- **Cross-platform portability**: The Agent Skills specification is supported across multiple coding agent systems, so your investment travels with you.
- **Slash command support**: Like prompts, skills can still be invoked via `/command` in VS Code Chat.

### When to reach for a skill

- You want to standardize how Copilot responds to a recurring task.
- You need bundled resources (templates, schemas, scripts) to complete the task.
- You want agents to discover and invoke the capability automatically.
- You prefer to drive the conversation, but with guardrails and rich context.

## Instructions

Instructions (`*.instructions.md`) provide background context that Copilot reads whenever it works on matching files. They often contain:

- Coding standards or style guides (naming conventions, testing strategy).
- Framework-specific hints (Angular best practices, .NET analyzers to suppress).
- Repository-specific rules ("never commit secrets", "feature flags must live in `flags/`").

Instructions sit under `instructions/` and can be scoped globally, per language, or per directory using glob patterns. They help Copilot align with your engineering playbook automatically.

### When to reach for instructions

- You need persistent guidance that applies across many sessions.
- You are codifying architecture decisions or compliance requirements.
- You want Copilot to understand patterns without manually pasting context.

## Hooks

Hooks are shell commands or scripts that run automatically at key lifecycle events during a Copilot agent session — when a session starts or ends, when the agent uses a tool, or when a prompt is submitted. Unlike instructions and skills (which guide the AI), hooks are deterministic: they run outside the model and always execute reliably.

Hooks are stored as JSON files in `.github/hooks/` and support events like `preToolUse`, `postToolUse`, `sessionStart`, `sessionEnd`, `agentStop`, and more.

```json
{
  "version": 1,
  "hooks": {
    "postToolUse": [
      {
        "type": "command",
        "bash": "npx prettier --write .",
        "cwd": ".",
        "timeoutSec": 30
      }
    ]
  }
}
```

### When to reach for hooks

- You want formatting, linting, or validation to happen reliably after every edit.
- You need to block dangerous commands or enforce security policies.
- You want to inject dynamic context (git state, environment info) into sessions automatically.
- You need audit logs, notifications, or compliance checks for autonomous agent work.

## Agentic Workflows

Agentic Workflows are AI-powered automations that run coding agents inside GitHub Actions. Written as a single Markdown file with natural language instructions and YAML frontmatter (triggers, permissions, safe outputs), they handle recurring tasks like issue triage, daily reports, and compliance checks — triggered by schedules, events, or slash commands.

```markdown
---
name: "Daily Issues Report"
description: "Generates a daily summary of open issues"
on:
  schedule: daily on weekdays
permissions:
  contents: read
  issues: read
safe-outputs:
  create-issue:
    title-prefix: "[daily-report] "
    labels: [report]
---

Summarize the open issues that need attention today...
```

Agentic workflows are compiled from `.md` sources to `.lock.yml` files using the `gh aw` CLI, then run by GitHub Actions.

### When to reach for agentic workflows

- You need scheduled or event-driven automation that goes beyond static GitHub Actions.
- You want a task that requires reasoning, summarization, or context-aware decisions.
- You want to automate repository maintenance without writing YAML actions syntax.

## Plugins

Plugins are installable packages that bundle agents, skills, hooks, and MCP server configurations into a single installable unit. Instead of manually copying files across projects, you can install a curated set of capabilities with one command:

```bash
copilot plugin install database-data-management@awesome-copilot
```

Plugins are distributed via marketplaces (the `awesome-copilot` and `copilot-plugins` marketplaces are registered by default), making it easy for teams to standardize tooling across all their projects.

### When to reach for plugins

- You want to share a curated set of agents, skills, and hooks across multiple repositories or team members.
- You want one-command installation rather than copying individual files into each project.
- You're building internal tooling that your organization should be able to install consistently.

## How the artifacts work together

Think of these artifacts as complementary layers:

1. **Instructions** lay the groundwork with long-lived guardrails — always active, no invocation needed.
2. **Skills** let you trigger rich, reusable workflows on demand, and let agents discover those workflows automatically.
3. **Agents** bring the most opinionated behavior, bundling tools and instructions into a single persona.
4. **Hooks** add deterministic automation — reliable formatting, linting, and governance that doesn't depend on the AI remembering to do it.
5. **Agentic Workflows** extend the above into scheduled and event-driven GitHub Actions automations.
6. **Plugins** package all of the above into installable, shareable units for teams and organizations.

By combining these primitives, teams can achieve:

- Consistent onboarding for new developers.
- Repeatable operations tasks with reduced context switching.
- Tailored experiences for specialized domains (security, infrastructure, data science, etc.).
- Automated repository maintenance that runs on schedule or in response to events.

## Next steps

- Explore the **Fundamentals** track for deeper dives: [Agents and Subagents](../agents-and-subagents/), [Automating with Hooks](../automating-with-hooks/), [Agentic Workflows](../agentic-workflows/).
- Browse the [Agents](../../agents/), [Skills](../../skills/), [Instructions](../../instructions/), [Hooks](../../hooks/), and [Plugins](../../plugins/) directories for ready-to-use resources.
- Try generating your own artifacts, then add them to the repo to keep the Learning Hub evolving.

---
