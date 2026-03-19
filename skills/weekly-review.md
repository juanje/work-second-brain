---
last_accessed: YYYY-MM-DD
access_count: 0
created: YYYY-MM-DD
---

# Skill: Weekly review

## When to use

Use when the user says "weekly review", "what did I do this week", "end of week",
or for broader reviews ("what have I done this quarter", "review for my manager").

## Procedure

### 1. Gather data

- Run `did this week` to get objective activity data.
- Run `jira-pending summary` to get current ticket status overview.
- Run `jira-pending sprint` to see sprint ticket states.
- Read logs from `memory/logs/` for the current week.
- Read `board/BOARD.md`.

### 2. Present the weekly summary

**Activity summary:**
- Key work items, grouped by project or theme.

**Completed this week:**
- Items from "Done" on the board.
- Cross-reference with logs for things done but not on the board.

**Still in progress:**
- Items in "Doing". On track or at risk?

**Waiting/Blocked:**
- Current blockers and their age.
- Any resolved this week?

**Decisions and context** (from logs):
- Key decisions made and their reasoning.
- Lessons learned.

**Unplanned work:**
- Things that came up during the week that weren't planned.

### 3. Clean the board

- Move "Done" items to `brain/reviews/YYYY-WNN.md` (ISO week number).
- Clear the "Done" section on the board.
- Review "Inbox" — triage anything remaining.
- Review "Next Actions" — still relevant? Anything to add?
- Review "Waiting" — any stale items to escalate?

### 3b. Review ideas

- Read `brain/ideas/_scratchpad.md` — any items worth promoting to their own file?
- Scan idea files in `brain/ideas/`:
  - `seed` ideas: still interesting? Promote to `developing` or archive.
  - `developing` ideas: any progress this week? Anything to add from the week's context?
  - `ready` ideas: should any be converted this week? (create ticket, start project, etc.)
- Present a brief ideas summary to the user alongside the board review.

### 4. Prepare next week

- Ask: "What should be your top priorities next week?"
- Suggest items from the board based on age, context, and patterns.
- Move agreed priorities to "Next Actions".

### 5. Broader review mode

If the user asks for a monthly or quarterly review:

- Run `did` with the appropriate range (`did last quarter`, `did this month`).
- Scan review files in `brain/reviews/` for the period.
- Scan logs in `memory/logs/` for the period.
- Group by project or theme.
- Highlight: major deliverables, cross-team contributions, problems solved, skills developed.
- Present in a format suitable for sharing with a manager.
