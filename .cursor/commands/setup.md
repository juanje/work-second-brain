Run the Work Agentic Buddy setup.

If a language is specified after the command (e.g., `/setup español`), conduct the entire setup conversation in that language. If no language is specified, detect the user's language from their first response and continue in that language. All **generated content** (files, board items, logs) is always in English regardless of conversation language.

## Pre-flight check

First, determine if this is a fresh setup or a reconfiguration:

1. Read `AGENTS.md`.
2. If it contains `POST-SETUP:` → **Fresh setup.** Proceed with the full setup below.
3. If it does NOT contain that marker → **System already configured.** Switch to reconfiguration mode:
   > The system is already set up. I can help you with:
   > - Update your profile (`agent_brain/identity/USER.md`)
   > - Add or reconfigure tools (`did`, Jira scripts, or new ones)
   > - Seed the board with current sprint/activity data
   > - Add a new project to `agent_brain/projects/`
   >
   > What would you like to update?

   In reconfiguration mode: **never overwrite AGENTS.md**. Only modify the specific files the user asks about. Read `templates/tool-setup.md` if it still exists for tool configuration reference; if it was deleted during initial setup, rely on the existing tool configuration in AGENTS.md as reference.

   After changes, commit: `git add -A && git commit -m "reconfig: <what changed>"`

   **Do not continue with the steps below.** End here for reconfiguration.

---

## Full setup procedure

### Step 1: Introduction

Read `templates/AGENTS.md` to understand the system you're setting up (don't explain the whole system to the user).

Then tell the user (in their language):

> I'll help you set up your Work Agentic Buddy. I'll ask a few questions about your work to personalize the system. It takes about 5 minutes.
>
> After that, the system will be ready to use. You can start brain-dumping tasks, ideas, and notes right away.

### Step 2: Get to know the user

Start with a warm, conversational tone. This is a first meeting — get to know the person before diving into work details. Ask **one group at a time**, wait for answers, and adapt naturally.

**Who are you?** (start here, always)
- What's your name?
- Where do you work?
- What do you do? (open-ended — let them describe it in their own words)
- Anything you'd like to share about yourself? Interests, how you like to work, what matters to you... Whatever feels relevant. (This is optional and open — some people share a lot, others prefer to keep it brief. Both are fine.)

After this first exchange, reflect back briefly what you understood and ask:

> Want to give me more detail about your role, team, tech stack, and tools now? Or would you prefer to skip that and let it fill in naturally as we work together?

If they want to continue, proceed with the groups below. If they prefer to skip, jump to Step 3 with whatever you have.

**Role and team** (optional now):
- What's your role specifically? (e.g., senior engineer, tech lead, product manager)
- What team are you on?
- Who's your manager? (optional, useful for reviews)

**Technical context** (optional now):
- What's your main technical focus? (e.g., CI/CD, backend, infrastructure)
- Key technologies you work with daily?
- What are your main projects or repos? (2-3 is enough to start)

**Tools** (optional now):
- What issue tracker do you use? (Jira, GitLab Issues, GitHub Issues, Bugzilla, etc.)
- What Git platform? (GitLab, GitHub, Bitbucket, etc.)
- Do you already have [`did`](https://github.com/psss/did) installed? (If not, note it for later — it's optional but recommended.)

**Work style** (optional now):
- How would you describe your typical day? (Focused deep work? Lots of context switches? Firefighting?)
- Do you work in sprints? If so, how long?
- Any recurring meetings or rituals? (standups, sprint planning, retros)

**Communication** (optional now):
- What language do you prefer to chat in? (All generated content will be in English regardless.)
- How do you like information presented? (Direct and brief? Detailed? Structured?)

### Step 3: Write agent_brain/identity/USER.md

Using the answers, fill in `agent_brain/identity/USER.md`. Follow the template structure already in the file but replace the placeholder comments with real content. Be concise — this file is reference, not a biography. If the user skipped the detailed questions, fill in what you have and leave the rest as placeholders — it will fill in over time.

### Step 4: Configure tools (if applicable)

Based on the user's answers about their issue tracker and Git platform:

Read `templates/tool-setup.md` for detailed installation and configuration steps for each tool. Follow the instructions there, adapting to what the user actually uses.

The tool-setup guide also covers what to do when the user **doesn't** use Jira or `did` — which skills to simplify and which references to remove from the operational AGENTS.md.

### Step 5: Seed the system with real data (optional but recommended)

This step bootstraps the system with actual work context so the user starts with something useful, not an empty board. Skip entirely if the user prefers to start blank, or skip individual parts if tools aren't configured.

Ask (in their language):
> Want me to pull in your recent work activity and current tasks? This gives us a head start — I can populate the board and log with real data instead of starting empty.

If yes, proceed. If no, skip to Step 6.

**Important: `did` queries can be slow** (30-60 seconds or more) when fetching from multiple remote sources (Jira, GitLab, GitHub). Don't assume it's hanging — wait for it. To speed things up, run source-specific queries in parallel when possible:

```bash
# Run these in parallel rather than one big "did this week":
did today --git &
did today --jira &
did today --gitlab &
wait
```

Then consolidate the outputs.

**5a. Current sprint (if Jira is configured):**

```bash
jira-pending sprint     # current sprint tickets
jira-pending summary    # status overview
```

From the results:
- Add sprint tickets to the board's **Sprint Backlog** section.
- If any are clearly in progress, ask the user which should go in **Doing** or **Next Actions**.
- If any are blocked, add to **Waiting** with the blocker noted.

**5b. Recent activity (if `did` is configured):**

```bash
# Today (fast, local sources)
did today --git

# This week (run per-source in parallel for speed)
did this week --git &
did this week --jira &
did this week --gitlab &
wait

# Last week (same parallel pattern)
did last week --git &
did last week --jira &
did last week --gitlab &
wait
```

From the results:
- Create the first daily log `logs/YYYY-MM-DD.md` with a summary of today's activity.
- Add key recent items (open MRs, ongoing tickets) to the appropriate board sections.
- If there's enough data for a mini weekly summary, mention it to the user as context.

**5c. Manual capture (always offer this):**

Ask:
> Anything else on your plate that these tools didn't pick up? Tasks, ideas, things you're waiting on?

Capture whatever the user mentions into the board (Inbox, or directly to the right section if the user gives enough context).

### Step 6: Adapt skills and tools references

Based on the tools the user has (or doesn't have):

- If they **don't use Jira**: remove references to `jira-pending` and `jira-detail` from `templates/AGENTS.md` in the "Getting work data" section and from any skills that call them (`agent_brain/skills/run-standup.md`, `agent_brain/skills/sync-board.md`, `agent_brain/skills/weekly-review.md`, `agent_brain/skills/next-task.md`). Replace with their actual issue tracker commands if applicable, or simplify to board + logs only.
- If they **don't use `did`**: remove `did` references from the same places. The standup and weekly review will rely on board + logs instead.
- If they use **different tools**: adapt the "Getting work data" section and skill references accordingly.

### Step 7: Set up agent commands (if applicable)

Check what editor/agent the user is running:

- **Cursor**: `.cursor/commands/` is already set up. No action needed.
- **Claude Code**: `CLAUDE.md` symlink and `.claude/commands/` symlinks are pre-created. Verify they resolve correctly (`ls -la CLAUDE.md .claude/commands/`). No copy needed. Additionally, create `.claude/settings.local.json` with permissions for the tools configured in Steps 4-6, so Claude Code doesn't ask for permission on every call:
  ```json
  {
    "permissions": {
      "allow": [
        "Bash(did:*)",
        "Bash(jira-pending:*)",
        "Bash(jira-detail:*)",
        "Bash(git add:*)",
        "Bash(git commit:*)",
        "Bash(ls:*)",
        "Bash(mkdir:*)"
      ]
    }
  }
  ```
  Adjust the list based on which tools the user actually configured. Remove entries for tools not installed.
- **Other agents**: The slash commands won't work, but the skills can be triggered by asking the agent directly (e.g., "run a standup", "do a weekly review"). No changes needed.

### Step 8: Activate the system

Execute these steps in order:

1. Copy the operational AGENTS.md into place:
   ```bash
   cp templates/AGENTS.md ./AGENTS.md
   ```

2. Remove setup-only files:
   ```bash
   rm -rf templates/
   rm -f LICENSE
   ```

3. Remove scripts directory if tools were already linked to `~/.local/bin/`:
   ```bash
   rm -rf scripts/
   ```
   (If the user chose not to set up the scripts, remove them anyway — they're reference files.)

4. Initial commit:
   ```bash
   git add -A && git commit -m "setup: initial configuration"
   ```

5. Tell the user (in their language):

> Your Work Agentic Buddy is ready.
>
> **To activate it now**, run **/refresh** so I reload the new AGENTS.md and start working in operational mode. Alternatively, start a new conversation — I'll pick up the new instructions automatically.
>
> **Quick start:**
> - Brain dump anything: tasks, ideas, decisions, notes. I'll capture and file them.
> - Say **"standup"** to see your priorities and plan the day.
> - Say **"next"** to get the next task with full context.
> - Use **/reflect** to process a conversation into your daily log.
> - Use **/weekly** at the end of the week to review and plan ahead.
>
> The more you use it, the more it knows. Start talking.

## Rules during setup

1. Conduct the conversation in the user's preferred language. If specified with the command, use that. Otherwise, detect from their first message.
2. All **generated file content** (USER.md, board items, logs, brain files) must be in English, regardless of conversation language.
3. Be conversational, not bureaucratic. This should feel like a quick onboarding chat, not a form.
4. Don't overwhelm. Ask one group of questions at a time. Wait for answers.
5. If the user gives short answers, that's fine. The system fills in over time.
6. If the user wants to skip something, skip it. Everything can be added later.
7. Make the tool configuration as hands-on as possible — run the commands, test them, confirm they work.
