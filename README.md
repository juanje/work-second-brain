# Work Agentic Buddy

A persistent, file-based memory system for AI coding assistants. You do brain dumps throughout the day — tasks, decisions, ideas, context, random thoughts — and the AI agent captures, organizes, and maintains everything in Markdown files.

## Table of contents

- [Getting started](#getting-started)
- [What it does](#what-it-does)
- [Structured workflows](#structured-workflows)
- [The board](#the-board)
- [Maintenance cycle](#maintenance-cycle)
- [Structure](#structure)
- [External tools (optional)](#external-tools-optional)
- [Compatibility](#compatibility)
- [Customization](#customization)
- [Design principles: why this works](#design-principles-why-this-works)
- [License](#license)

## Getting started

1. Clone or copy this repository into a new directory.
2. Open it as a workspace in your AI-powered editor (Cursor, VS Code + Copilot, Claude Code, etc.).
3. Run `/setup` to start the guided configuration, or `/setup <language>` to run it in your preferred language (e.g., `/setup español`).
4. The agent will walk you through: your profile, tool configuration, and optionally seeding the board with real data from your issue tracker and Git activity.
5. After setup, the system is ready. Start brain-dumping.

## What it does

Talk to the agent naturally. It will:

- **Capture tasks** → Board Inbox (triage into Sprint Backlog or Next Actions when ready)
- **Capture decisions** → Project files or concept files, with reasoning
- **Capture ideas** → Ideas directory, with a lifecycle (seed → developing → ready → converted)
- **Capture lessons** → Concept files for future reference
- **Confirm what it captured** — always

You don't need to think about where things go. The agent classifies and files them based on what you say. Over time, your workspace becomes a searchable, structured knowledge base of everything you've worked on, decided, and learned.

## Structured workflows

| Command | What it does |
|---------|-------------|
| `/standup` | Shows what was done, current priorities, blockers, inbox to triage |
| `/next` | Shows the next task with full context from the issue tracker and agent brain |
| `/reflect` | Processes the conversation into a structured daily log |
| `/weekly` | Compiles the week's work, cleans the board, plans next week |
| `/sync` | Quick board sync: cross-references board with the issue tracker |
| `/maintenance` | Deep maintenance: compaction, pruning, promotion, board hygiene |
| `/refresh` | Re-reads AGENTS.md — useful when the agent loses context in long conversations |

These commands are available as slash commands in Cursor and Claude Code. For other agents, trigger them by asking directly (e.g., "run a standup", "do a weekly review").

## The board

A Kanban-style task board ordered by actionability:

**Doing** (WIP target: 1, max 2 if related) → **Next Actions** (max 3-4) → **Waiting** → **Sprint Backlog** → **Inbox** → **Parked** → **Done**

Items flow from capture to completion. The agent enforces WIP limits — if you have too much in Doing, it will tell you. During weekly reviews, Done items are archived into review files for future reference (quarterly reviews, manager conversations).

## Maintenance cycle

A periodic deep maintenance that keeps the system healthy. Run it with `/maintenance` at the end of the day or weekly.

1. **Log compaction** — archives old logs after extracting knowledge
2. **Pruning** — moves unused brain files to archive (Hebbian degradation)
3. **Promotion** — updates the agent's active context based on usage patterns (Hebbian promotion)
4. **Board hygiene** — flags stale blockers, untriaged inbox, WIP violations
5. **Ideas review** — flags stale ideas, reminds about mature ones ready for action
6. **Skill review** — detects repeated patterns in logs and suggests new skills
7. **Contradiction detection** — finds and flags inconsistencies in the knowledge base

## Structure

```
├── AGENTS.md                    → Agent working memory. Loaded automatically.
├── work/
│   └── BOARD.md                 → Kanban board.
├── logs/                        → Daily conversation logs.
│   └── archive/                 → Compacted old logs.
└── agent_brain/
    ├── identity/
    │   ├── USER.md              → Your work profile and preferences.
    │   └── SOUL.md              → Agent identity, values, and limits.
    ├── skills/                  → Reusable procedures, loaded on demand.
    ├── projects/                → Active project context and decisions.
    ├── concepts/                → Lessons learned, patterns, knowledge.
    ├── ideas/                   → Ideas with lifecycle tracking.
    └── archive/                 → Files degraded by disuse.
```

The system starts nearly empty. Directories populate through use. The agent creates files and new directories inside `agent_brain/` as needed — you don't have to set up anything manually beyond the initial configuration. For example, if you start tracking cross-team requests, a `requests/` directory will emerge. If you need team structure notes, a `teams/` directory will be created. The structure grows organically to match how you actually work.

## External tools (optional)

The system can integrate with CLI tools that provide objective work data. These are optional — the system works with board + logs alone, but external data makes standups, syncs, and reviews richer.

- **[`did`](https://github.com/psss/did)** — Aggregates activity from Git, GitLab, GitHub, Jira, Bugzilla, and other development tools into status reports. Used by the standup and weekly review skills.
- **Jira scripts** — Two included scripts (`jira-pending`, `jira-detail`) that query your Jira instance for current ticket state and full ticket details. Used by the standup, sync, next-task, and weekly review skills.

These tools are configured during the interactive setup. If you don't use Jira or prefer a different issue tracker, the setup agent will adapt the system accordingly.

## Compatibility

The system works with any AI agent that reads `AGENTS.md` from the workspace root:

- **Cursor** — full support (AGENTS.md + slash commands)
- **Claude Code** — full support (AGENTS.md + commands)
- **GitHub Copilot** — reads AGENTS.md
- **Windsurf, Zed, Gemini CLI, RooCode** — reads AGENTS.md

Slash commands are provided for Cursor (`.cursor/commands/`). For Claude Code, copy them to `.claude/commands/`. For other agents, trigger workflows by asking directly.

## Customization

### Adding skills

Skills are reusable procedures in `agent_brain/skills/`. Create a new `.md` file with a "When to use" trigger and a numbered "Procedure", then add it to the Skills section in `AGENTS.md`. The agent will pick it up on the next conversation.

### Adding brain directories

The agent creates new directories inside `agent_brain/` as needed based on use. You can also create them manually — just add the new directory to the "Where to find things" section in `AGENTS.md` with a description of when the agent should look there.

### Adapting to different tools

The "Getting work data" section in `AGENTS.md` defines which CLI tools the agent can call. Replace or extend these with whatever tools your workflow uses — the agent just needs to know the command, what it returns, and when to use it.

## Design principles: why this works

Most AI coding assistants are stateless. Each conversation starts from zero. You repeat your context, re-explain your priorities, and lose the thread of what you were doing yesterday. This system gives the agent a persistent memory that grows with use.

But it's not just a note-taking system with an AI front-end. The design is grounded in principles from neuroscience, complex systems theory, and practical experience with AI agent limitations.

### Files as memory substrate

The brain doesn't store memories in a single location — it distributes them across networks that strengthen or weaken based on use. This system uses plain Markdown files as its memory substrate: distributed, human-readable, Git-versionable, and portable across any AI agent that can read files.

There are no databases, no embeddings, no vendor-specific formats. If your agent breaks, switches, or disappears, your knowledge is still there in files you can read, search, and edit yourself.

### Hebbian plasticity: use it or lose it

In neuroscience, Hebb's principle states that neurons that fire together wire together — connections strengthen with use and weaken without it. This system applies the same idea to information management.

Every file tracks when it was last accessed and how often. A periodic maintenance cycle uses these metrics to **promote** frequently-used files to the agent's active context (making them immediately visible) and **archive** files that haven't been touched in weeks. The agent's attention automatically mirrors what's actually relevant to your work right now, without manual curation.

### Progressive disclosure: load only what's needed

The agent doesn't read everything at startup. It reads a lightweight index file (`AGENTS.md`, ~150 lines) that contains just enough to know where things are and when to look deeper. Skills, project context, and knowledge files are loaded on demand — only when a task requires them.

This mirrors how human expertise works: you don't recall everything you know before starting a task. You activate relevant knowledge as the context demands it. For AI agents, this has a practical benefit too — it keeps the context window clean, which directly improves response quality.

### Emergence from simple rules

Complex systems theory shows that sophisticated behavior can emerge from simple rules applied consistently. This system doesn't try to be a complete project management tool. Instead, it gives the agent a small set of clear behaviors:

- **Capture everything the user mentions.** Tasks, ideas, decisions, notes — file them in the right place.
- **Confirm what was captured.** Brief acknowledgment, no ceremony.
- **Don't reorganize proactively.** Structure emerges from use, not from upfront design.
- **When in doubt, capture.** A rough note is better than a lost thought.

Over time, these simple rules produce a knowledge base that reflects how you actually work — not how you planned to work.

## License

MIT License. See [LICENSE](LICENSE) for details.
