# Work Second Brain

A persistent, file-based memory system for AI coding assistants. The user does brain dumps throughout the day — tasks, decisions, ideas, context, random thoughts — and the AI agent captures, organizes, and maintains everything in Markdown files.

> **Note:** After setup, delete this README, the LICENSE file, and the `scripts/` directory (once the scripts are copied to their final location, e.g. `~/.local/bin/`). These are reference files for the human, not for the agent. Keeping them adds unnecessary tokens to the agent's context and can distract from operational files.

## Design principles

1. **Files first.** Markdown is the single source of truth. Human-readable, Git-versionable, portable across agents. No databases, no vendor lock-in.
2. **Minimum viable, then grow.** The system starts nearly empty. Structure emerges from use, not from upfront design.
3. **Hebbian plasticity.** What gets used gets promoted (to active context). What doesn't gets archived. The agent's attention mirrors actual relevance.
4. **Progressive disclosure.** The agent only loads what it needs. AGENTS.md is a lightweight index; skills and knowledge load on demand.

## Quick start

1. Clone or copy this skeleton into a new directory.
2. Open it as a workspace in your AI-enabled editor (Cursor, VS Code + Copilot, etc.).
3. Edit `identity/USER.md` with your profile — role, team, tools, work style.
4. Set up the CLI tools (see [Tool setup](#tool-setup) below).
5. Start talking to the agent. Brain dump away.
6. **Delete this README** once you're familiar with the system.

The system populates itself through use. The more you talk to it, the more it knows.

> **Note:** Remember, this is an initial and generic setup. Personalize it to your needs, tools, and workflows.

## Structure

```
work_brain/
├── AGENTS.md                    → Agent working memory. Loaded automatically.
├── README.md                    → This file. Delete after setup.
├── scripts/
│   ├── jira-pending.sh          → Query pending Jira tickets.
│   └── jira-detail.sh           → Get full ticket details.
├── board/
│   └── BOARD.md                 → Kanban board (Inbox, Next Actions, Doing, Waiting, Done, Parked).
├── identity/
│   ├── USER.md                  → Your work profile and preferences.
│   └── SOUL.md                  → Agent identity, values, and limits.
├── skills/
│   ├── run-standup.md           → Interactive: standup summary.
│   ├── capture-item.md          → Interactive: complex item classification.
│   ├── weekly-review.md         → Interactive: weekly/quarterly review.
│   ├── next-task.md             → Interactive: show next task with context.
│   ├── process-conversation.md  → Triggered: brain dump → structured log.
│   └── sleep-maintenance.md     → Triggered: deep maintenance cycle (7 phases).
├── brain/
│   ├── projects/                → Active project context and decisions.
│   ├── concepts/                → Generalized knowledge, lessons, patterns.
│   ├── teams/                   → Team structure, people, who works on what.
│   ├── requests/                → Cross-team requests.
│   ├── reviews/                 → Weekly/sprint review summaries.
│   ├── ideas/                   → Unformed thoughts with lifecycle (seed → ready).
│   │   └── _scratchpad.md       → Quick idea capture for one-liners.
│   └── archive/                 → Files degraded by disuse.
├── memory/
│   ├── logs/                    → Daily conversation logs (curated summaries).
│   └── archive/                 → Old logs (compacted by maintenance).
└── .cursor/commands/            → Slash commands for Cursor.
    ├── reflect.md               → /reflect — process conversation into log.
    ├── standup.md               → /standup — standup summary.
    ├── weekly.md                → /weekly — weekly review.
    ├── maintenance.md           → /maintenance — deep maintenance cycle.
    └── next.md                  → /next — show next task.
```

## Tool setup

The system includes three CLI tools that provide objective work data to the agent. All are optional but recommended for a fully functional system.

### `did` — activity aggregator

[`did`](https://github.com/psss/did) is a CLI tool that gathers status reports from development tools (Git, GitLab, GitHub, Jira, Bugzilla, etc.).

**Install:**

```bash
pip install did
```

**Configure** (`~/.did/config`):

```ini
[general]
email = your.email@example.com

[git]
type = git
apps = /path/to/your/repos/*

[github]
type = github
url = https://api.github.com/
token = <your-github-token>
login = <your-github-username>

[jira]
type = jira
url = https://your-instance.atlassian.net
token = <your-jira-api-token>
user = your.email@example.com
project = YOUR_PROJECT
```

See the [`did` documentation](https://github.com/psss/did) for all available plugins (GitLab, Bugzilla, Pagure, Gerrit, etc.) and configuration options.

**Test:**

```bash
did yesterday
did this week
did yesterday --jira    # filter by source for faster queries
```

### `jira-pending` — current Jira state

Queries your pending Jira tickets. Included in `scripts/jira-pending.sh`.

**Setup:**

1. Edit `scripts/jira-pending.sh` and set your Jira URL, email, and project:
   ```bash
   JIRA_URL="https://your-instance.atlassian.net"
   JIRA_USER="your.email@example.com"
   PROJECT="YOUR_PROJECT"
   ```
2. Create a Jira API token at https://id.atlassian.com/manage-profile/security/api-tokens
3. Save the token:
   ```bash
   mkdir -p ~/.did
   echo "your-api-token" > ~/.did/jira.token
   chmod 600 ~/.did/jira.token
   ```
4. Make executable and link:
   ```bash
   chmod +x scripts/jira-pending.sh
   ln -s "$(pwd)/scripts/jira-pending.sh" ~/.local/bin/jira-pending
   ```

**Usage:**

```bash
jira-pending assigned   # all open tickets assigned to me
jira-pending sprint     # my tickets in the current sprint
jira-pending summary    # count by status
```

### `jira-detail` — ticket details

Shows full ticket detail (description, comments, links). Included in `scripts/jira-detail.sh`. Uses the same credentials as `jira-pending`.

**Setup:**

1. Edit `scripts/jira-detail.sh` and set the same Jira URL and email.
2. Make executable and link:
   ```bash
   chmod +x scripts/jira-detail.sh
   ln -s "$(pwd)/scripts/jira-detail.sh" ~/.local/bin/jira-detail
   ```

**Usage:**

```bash
jira-detail PROJ-1234                # full ticket detail
jira-detail PROJ-1234 --comments-only  # just comments
jira-detail PROJ-1234 --last 3        # last 3 comments
```

## How it works

### During the day

Talk to the agent naturally. It will:
- Capture tasks → Board Inbox
- Capture decisions → `brain/projects/` or `brain/concepts/`
- Capture ideas → `brain/ideas/`
- Capture lessons → `brain/concepts/`
- Confirm what it captured

### Commands

| Command | What it does |
|---------|-------------|
| `/standup` | Shows what was done, current priorities, blockers, inbox |
| `/next` | Shows the next task with full context |
| `/reflect` | Processes the conversation into a structured daily log |
| `/weekly` | Compiles the week's work, cleans the board, plans next week |
| `/maintenance` | Runs deep maintenance: compaction, pruning, promotion, hygiene |

### Maintenance cycle

Run `/maintenance` periodically (daily or weekly). It handles:
1. **Log compaction** — archives old logs after extracting knowledge
2. **Pruning** — moves unused brain files to archive
3. **Promotion** — updates Active Context in AGENTS.md based on usage
4. **Board hygiene** — flags stale items
5. **Ideas review** — flags stale ideas, reminds about ready ones
6. **Skill review** — detects repeated patterns, suggests new skills
7. **Contradiction detection** — maintains knowledge base coherence

## Customization

### Adding new skills

Create a new file in `skills/` with this structure:

```markdown
---
last_accessed: YYYY-MM-DD
access_count: 0
created: YYYY-MM-DD
---

# Skill: Verb object

## When to use
<Clear trigger description>

## Procedure
<Numbered steps>
```

Then add it to the Skills section in `AGENTS.md` with a descriptive trigger.

### Creating new brain directories

If you need a new category of knowledge, create a directory under `brain/` and add it to the "Where to find things" section in `AGENTS.md` with a clear trigger description.

## Compatibility

This system works with any AI editor that reads `AGENTS.md` from the workspace root:

- **Cursor** — full support (AGENTS.md + .cursor/commands/)
- **Claude Code** — full support (AGENTS.md + .claude/commands/)
- **GitHub Copilot** — reads AGENTS.md
- **Windsurf, Zed, RooCode** — reads AGENTS.md

For Claude Code, copy `.cursor/commands/` to `.claude/commands/`.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

## 🤖 AI Tools Disclaimer

This project was developed with the assistance of artificial intelligence tools:

**Tools used:**
- **Cursor**: Code editor with AI capabilities
- **Claude-4.6-Opus**: Anthropic's language model

**Division of responsibilities:**

**AI (Cursor + Claude-4.6-Opus)**:
- 🔧 Initial code prototyping
- 📝 Generation of examples and test cases
- 🐛 Assistance in debugging and error resolution
- 📚 Documentation and comments writing
- 💡 Technical implementation suggestions

**Human (Juanje Ojeda)**:
- 🎯 Specification of objectives and requirements
- 🔍 Critical review of code and documentation
- 💬 Iterative feedback and solution refinement
- 📋 Definition of project's educational structure
- ✅ Final validation of concepts and approaches

**Collaboration philosophy**: AI tools served as a highly capable technical assistant, while all design decisions, educational objectives, and project directions were defined and validated by the human.
