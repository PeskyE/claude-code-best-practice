# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a best practices repository for Claude Code configuration, demonstrating patterns for skills, subagents, hooks, and commands. It serves as a reference implementation rather than an application codebase.

## Key Components

### Weather System (Example Workflow)
A demonstration of two distinct skill patterns via the **Command → Agent → Skill** architecture:
- `/weather-orchestrator` command (`.claude/commands/weather-orchestrator.md`): Entry point — asks user for C/F, invokes agent, then invokes SVG skill
- `weather-agent` agent (`.claude/agents/weather-agent.md`): Fetches temperature using its preloaded `weather-fetcher` skill (agent skill pattern)
- `weather-fetcher` skill (`.claude/skills/weather-fetcher/SKILL.md`): Preloaded into agent — instructions for fetching temperature from Open-Meteo
- `weather-svg-creator` skill (`.claude/skills/weather-svg-creator/SKILL.md`): Skill — creates SVG weather card, writes `orchestration-workflow/weather.svg` and `orchestration-workflow/output.md`

Two skill patterns: agent skills (preloaded via `skills:` field) vs skills (invoked via `Skill` tool). See `orchestration-workflow/orchestration-workflow.md` for the complete flow diagram.

### Skill Definition Structure
Skills in `.claude/skills/<name>/SKILL.md` use YAML frontmatter:
- `name`: Display name and `/slash-command` (defaults to directory name)
- `description`: When to invoke (recommended for auto-discovery)
- `argument-hint`: Autocomplete hint (e.g., `[issue-number]`)
- `disable-model-invocation`: Set `true` to prevent automatic invocation
- `user-invocable`: Set `false` to hide from `/` menu (background knowledge only)
- `allowed-tools`: Tools allowed without permission prompts when skill is active
- `model`: Model to use when skill is active
- `context`: Set to `fork` to run in isolated subagent context
- `agent`: Subagent type for `context: fork` (default: `general-purpose`)
- `hooks`: Lifecycle hooks scoped to this skill

### Presentation System
See `.claude/rules/presentation.md` — presentation work is delegated per-presentation to `presentation-vibe-coding` (for `presentation/vibe-coding-to-agentic-engineering/`) or `presentation-claude-gemini` (for `presentation/2026-04-25-gdg-kolachi-cli-claude-code-gemini/`).

### Hooks System
Cross-platform sound notification system in `.claude/hooks/`:
- `scripts/hooks.py`: Main handler for Claude Code hook events
- `config/hooks-config.json`: Shared team configuration
- `config/hooks-config.local.json`: Personal overrides (git-ignored)
- `sounds/`: Audio files organized by hook event (generated via ElevenLabs TTS)

Hook events configured in `.claude/settings.json`: PreToolUse, PostToolUse, UserPromptSubmit, Notification, Stop, SubagentStart, SubagentStop, PreCompact, SessionStart, SessionEnd, Setup, PermissionRequest, TeammateIdle, TaskCompleted, ConfigChange.

Special handling: git commits trigger `pretooluse-git-committing` sound.

## Critical Patterns

### Subagent Orchestration
Subagents **cannot** invoke other subagents via bash commands. Use the Agent tool (renamed from Task in v2.1.63; `Task(...)` still works as an alias):
```
Agent(subagent_type="agent-name", description="...", prompt="...", model="haiku")
```

Be explicit about tool usage in subagent definitions. Avoid vague terms like "launch" that could be misinterpreted as bash commands.

### Subagent Definition Structure
Subagents in `.claude/agents/*.md` use YAML frontmatter:
- `name`: Subagent identifier
- `description`: When to invoke (use "PROACTIVELY" for auto-invocation)
- `tools`: Comma-separated allowlist of tools (inherits all if omitted). Supports `Agent(agent_type)` syntax
- `disallowedTools`: Tools to deny, removed from inherited or specified list
- `model`: Model alias: `haiku`, `sonnet`, `opus`, or `inherit` (default: `inherit`)
- `permissionMode`: Permission mode (e.g., `"acceptEdits"`, `"plan"`, `"bypassPermissions"`)
- `maxTurns`: Maximum agentic turns before the subagent stops
- `skills`: List of skill names to preload into agent context
- `mcpServers`: MCP servers for this subagent (server names or inline configs)
- `hooks`: Lifecycle hooks scoped to this subagent (all hook events are supported; `PreToolUse`, `PostToolUse`, and `Stop` are the most common)
- `memory`: Persistent memory scope — `user`, `project`, or `local` (see `reports/claude-agent-memory.md`)
- `background`: Set to `true` to always run as a background task
- `effort`: Effort level override: `low`, `medium`, `high`, `max` (default: inherits from session)
- `isolation`: Set to `"worktree"` to run in a temporary git worktree
- `color`: CLI output color for visual distinction

### Configuration Hierarchy
1. **Managed** (`managed-settings.json` / MDM plist / Registry): Organization-enforced, cannot be overridden
2. Command line arguments: Single-session overrides
3. `.claude/settings.local.json`: Personal project settings (git-ignored)
4. `.claude/settings.json`: Team-shared settings
5. `~/.claude/settings.json`: Global personal defaults
6. `hooks-config.local.json` overrides `hooks-config.json`

### Disable Hooks
Set `"disableAllHooks": true` in `.claude/settings.local.json`, or disable individual hooks in `hooks-config.json`.

## Answering Best Practice Questions

When the user asks a Claude Code best practice question, **always search this repo first** (`best-practice/`, `reports/`, `tips/`, `implementation/`, and `README.md`) before relying on training knowledge or external sources. This repo is the authoritative source — only fall back to external docs or web search if the answer is not found here.

## Workflow Best Practices

From experience with this repository:

- Keep CLAUDE.md under 200 lines per file for reliable adherence
- `.claude/rules/*.md` with `paths:` YAML frontmatter are lazy-loaded only when Claude touches matching files; without frontmatter they load into every session like CLAUDE.md
- Use commands for workflows instead of standalone agents
- Create feature-specific subagents with skills (progressive disclosure) rather than general-purpose agents
- Perform manual `/compact` at ~50% context usage
- Start with plan mode for complex tasks
- Use human-gated task list workflow for multi-step tasks
- Break subtasks small enough to complete in under 50% context

### Debugging Tips

- Use `/doctor` for diagnostics
- Run long-running terminal commands as background tasks for better log visibility
- Use browser automation MCPs (Claude in Chrome, Playwright, Chrome DevTools) for Claude to inspect console logs
- Provide screenshots when reporting visual issues

## Git Commit Rules

When committing changes, **create separate commits per file**. Do NOT bundle multiple file changes into a single commit. Each file gets its own commit with a descriptive message specific to that file's changes.

For example, if `README.md`, `best-practice/claude-subagents.md`, and a skill file all changed:
- Commit 1: `git add README.md` → commit with README-specific message
- Commit 2: `git add best-practice/claude-subagents.md` → commit with subagents-doc-specific message
- Commit 3: `git add .claude/skills/weather-fetcher/SKILL.md` → commit with skill-specific message

This makes the git history cleaner and easier to review, revert, or cherry-pick individual changes.

## Documentation

See `.claude/rules/markdown-docs.md` for documentation standards. Key docs:
- `best-practice/claude-subagents.md`: Subagent frontmatter, hooks, and repository agents
- `best-practice/claude-commands.md`: Slash command patterns and built-in command reference
- `orchestration-workflow/orchestration-workflow.md`: Weather system flow diagram


---

## REUSE-FIRST DISCOVERY GATE

Before writing any new code, workflow, automation, agent, integration, dashboard, scraper, generator, prompt system, SaaS component, or repo update, you MUST complete a reuse-first discovery pass and present the result to the user. **Do not start building until the gate has been cleared.**

### When this rule fires

Applies to anything that could plausibly already exist as a maintained open-source or commercial component:
- Custom features, tools, scripts, CLIs
- Workflow / automation designs (n8n, GitHub Actions, cron, Zapier)
- AI agent designs (sub-agents, skills, orchestrators, eval harnesses)
- SaaS / API integration code
- Dashboards, scrapers, generators, parsers, RAG pipelines
- Prompt systems, prompt libraries, prompt-eval scaffolding
- Repo scaffolding, starter templates, dev tooling

Does NOT apply to: trivial one-line edits, bug fixes in existing code, config tweaks, or work explicitly scoped by the user as "from scratch / no dependencies."

### Required search scope

Before writing code, search at minimum (skip a source only if you state out loud why it's not relevant):

- **GitHub** — repo search, topic search, awesome-lists, gists
- **Hugging Face** — models, datasets, spaces (for any ML/AI work)
- **npm** (Node) / **PyPI** (Python) / **crates.io** (Rust) — packages
- **Docker Hub** — pre-built images
- **n8n template library** — for workflow automation
- **MCP server registries** — modelcontextprotocol.io, awesome-mcp lists, Claude Code marketplace
- **Claude / ClawHub skill or agent resources** — if present in this repo or its referenced docs
- **Official vendor docs** — when integrating a specific API/service
- **Awesome-\* lists and starter kits** for the relevant domain

If a search dimension is impossible in the current environment (no web access, sandboxed, MCP server down), say so explicitly. Do not silently skip.

### Required output before any code is written

Produce these five fields, in this order, then STOP. Do not write code until the user has reviewed and approved them.

1. **Interpreted goal** — one sentence: what the user actually wants accomplished. Restate so they can correct any misreads.
2. **Contained build boundary** — what is IN scope and what is explicitly OUT of scope for this task. Prevents scope creep.
3. **Best existing options** — top 2–4 maintained candidates discovered, each with: name, link, last-updated / maintenance signal, license, what it gives you, what it does not.
4. **Build-vs-borrow recommendation** — pick exactly one: (a) use option X as-is, (b) fork / wrap option X, (c) compose multiple options, or (d) build from scratch — and **why**. Building from scratch requires explicit justification (no maintained option, licensing blocker, performance ceiling, etc.).
5. **One concrete next step** — the single smallest action that moves the chosen path forward. Not a plan, not a phased roadmap. One step.

### Trigger

The user can invoke this gate at any time by typing any of: `/reuse-first`, `/repo-scan`, `/dont-reinvent`. The agent should also self-trigger the gate whenever it detects it is about to begin a buildable task that fits the scope above.

### Failure mode to avoid

Do not produce vague reuse summaries ("there are probably libraries for this"). Either name specific maintained projects with links, or explicitly say no maintained option was found and why a build is warranted. The whole point of the gate is to stop reinventing — vague is the same as skipping.
