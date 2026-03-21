---
last_accessed: YYYY-MM-DD
access_count: 0
created: YYYY-MM-DD
---

# Skill: Process conversation

## When to use

Triggered by the user via the `/reflect` command, or at the end of a work session.

## Procedure

### 1. Review the conversation

Read the current or most recent conversation. Identify:
- Decisions and their reasoning
- Tasks mentioned or captured
- Ideas discussed
- Meeting context or situational notes
- Lessons learned or patterns discovered
- Open threads (discussed but unresolved)

### 2. Update today's log

Open `logs/YYYY-MM-DD.md` (today's date). If it exists, append new content
under the existing sections (avoid duplicating information already logged earlier
in the day). If it doesn't exist, create it with this structure:

```markdown
---
date: YYYY-MM-DD
last_updated: YYYY-MM-DDTHH:MM
---

# Log — YYYY-MM-DD

## Decisions
- [Decision and reasoning, with enough context for future reference]

## Tasks captured
- [What was captured and where it was filed]

## Ideas
- [Ideas discussed, whether filed or not]

## Context
- [Meetings, conversations with colleagues, situational context]

## Lessons
- [Patterns discovered, errors and fixes, things learned]

## Open threads
- [Discussed but unresolved — to pick up later]
```

Always update the `last_updated` timestamp in the frontmatter.

### 3. Verify captures

Check that everything actionable from the conversation has been properly captured:
- Tasks → `work/BOARD.md` (Inbox or appropriate section)
- Concrete improvement ideas / tech debt → `work/BOARD.md` Parked
- Unformed ideas / project concepts / draft proposals → `agent_brain/ideas/`
- Decisions → `agent_brain/projects/` or `agent_brain/concepts/`
- Lessons → `agent_brain/concepts/`

If anything was missed, capture it now.

### 4. Operational introspection

Review the conversation looking for signals that the system itself should evolve.
This is not about *what* was discussed, but about *how* the agent worked and what
could work better.

**Patterns → potential skills:**
- Did the user ask for something that required a multi-step procedure not covered
  by an existing skill? If so, note it in the log under a `## System observations`
  section as a skill candidate.

**Lessons → potential rules:**
- Did the conversation reveal a mistake the agent made, a user correction about
  how to work, or a preference that should apply going forward? If so, consider
  whether it should become a new rule in AGENTS.md or an update to an existing one.
  Note the candidate in the log. Don't modify AGENTS.md directly — present the
  proposed rule to the user for approval.

**Information structure → potential directories:**
- Did the conversation produce information that doesn't fit well in the existing
  `agent_brain/` directories? If a new category is emerging (e.g., recurring
  cross-team requests, meeting notes by team, review archives), note it as a
  candidate for a new directory.

**Staleness signals:**
- Did the agent use information from memory that turned out to be outdated?
  If so, note which file was stale and update it now.

If there are observations, add them to the log:

```markdown
## System observations
- **Skill candidate:** [description of repeated procedure]
- **Rule candidate:** [proposed rule and why]
- **Structure candidate:** [proposed directory and what would go there]
- **Stale data fixed:** [file updated and what changed]
```

If there are no observations, skip this section — don't force it.

### 5. Git commit

```bash
git add -A && git commit -m "reflect: YYYY-MM-DD" 2>/dev/null || true
```

## Quality criteria

- **Be specific.** "Discussed CI/CD" is useless. "Decided to migrate from Jenkins to
  Tekton because of X" is useful.
- **Focus on reasoning.** Decisions and their "why" are the most valuable content.
  Activity tracking is handled by external tools.
- **Don't inflate.** If the conversation was trivial, the log can be minimal.
- **Think of future you.** The log is for someone without today's context who needs
  to understand when and why something was decided.
- **Append, don't overwrite.** If the log already has content from an earlier
  processing pass, add new information without duplicating.
