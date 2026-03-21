---
last_accessed: YYYY-MM-DD
access_count: 0
created: YYYY-MM-DD
---

# Skill: Sleep maintenance cycle

## When to use

Triggered manually by the user via the `/maintenance` command, or by an
automated scheduler (daily or weekly).

## Phases

Execute in order. Each phase is independent: if one fails, log the error and
continue with the next.

---

### Phase 1: Log compaction

**Goal:** Keep `logs/` manageable while preserving valuable context.

1. List files in `logs/` older than 30 days (exclude maintenance logs).
2. For each old log:
   - Check if its key content (decisions, lessons) has been extracted to `agent_brain/`.
   - If fully extracted: move to `logs/archive/`.
   - If it contains unextracted knowledge: extract it to the appropriate `agent_brain/`
     location first, then archive the log.
3. Never delete logs outright — always archive. Git history provides additional backup.
4. Record what was compacted.

---

### Phase 2: Pruning by disuse

**Goal:** Move stale information out of the active space.

1. Scan all files in `agent_brain/` subdirectories (excluding `identity/`, `skills/`, and `archive/`).
2. Read their metadata (`last_accessed`, `access_count`).
3. If a file hasn't been accessed in >30 days AND has fewer than 5 total accesses:
   - Move it to `agent_brain/archive/`.
   - Record the move.
4. If a file hasn't been accessed in >30 days BUT has >5 accesses:
   - Don't move it. Flag it as a review candidate in the maintenance log.

**Exception:** Never move or prune files in `agent_brain/identity/`, `agent_brain/skills/`, or `work/`.
Those require human decision.

---

### Phase 3: Promotion and degradation (AGENTS.md)

**Goal:** Keep the "Active context" section of AGENTS.md reflecting what's
actually relevant right now.

1. Scan all files in `agent_brain/` subdirectories (excluding `identity/`, `skills/`, and `archive/`).
2. Calculate score: `score = access_count × recency_factor`
   - recency_factor: 1.0 (today), 0.9 (yesterday), 0.7 (this week),
     0.4 (this month), 0.1 (older).
3. Top 5-7 files by score get linked in "Active context" of AGENTS.md.
4. Remove any link whose file is no longer in the top.
5. Link format:
   ```
   - [Brief description](path/to/file.md) — why it's relevant right now
   ```

---

### Phase 4: Board hygiene

**Goal:** Keep the board clean and actionable.

1. Read `work/BOARD.md`.
2. **Doing:** If more than **2** open items, flag WIP exceeded (target 1, max 2 if related; AGENTS.md Rule 9).
3. **Waiting items:** If any have been waiting >7 days, add a note suggesting
   escalation.
4. **Inbox items:** If there are items older than 3 days, flag for triage.
5. **Sprint Backlog:** Flag stale items if sprint scope has shifted.
6. **Next Actions:** Review if items are still relevant. Flag any older than
   14 days without progress. Flag if more than 4 items (WIP limit is 3–4).
7. **Done:** If there are items in Done, remind that weekly review should
   clear them.
8. Record any board changes.

---

### Phase 5: Ideas review

**Goal:** Keep `agent_brain/ideas/` healthy — promote scratchpad items, advance
developing ideas, flag ready ideas for action.

1. Read `agent_brain/ideas/_scratchpad.md`.
   - Any item older than 7 days? Flag it: either promote to its own idea file
     or remove if no longer relevant.
2. Scan all idea files in `agent_brain/ideas/` (excluding `_scratchpad.md`).
3. Read their metadata (`last_accessed`, `status`).
4. For each idea:
   - `seed` not accessed in >14 days → flag as stale. Suggest: develop, archive,
     or move to scratchpad if it was too small for its own file.
   - `developing` not accessed in >21 days → flag for review. Is it stuck?
   - `ready` not accessed in >7 days → flag for action. It's mature enough to
     convert — remind the user.
   - `converted` or `archived` older than 30 days → move to `agent_brain/archive/`.
5. Record findings in the maintenance log.

---

### Phase 6: Skill and rule review

**Goal:** Keep `agent_brain/skills/` up to date, detect opportunities for new
skills, and evolve AGENTS.md rules based on observed patterns.

**6a. Skill review:**

1. Review recent logs from `logs/` (last month), paying attention to
   `## System observations` sections (added by `/reflect`).
2. Look for repeated patterns: procedures done multiple times in similar ways?
   - If found, create a new skill in `agent_brain/skills/` with `verb-object.md` naming.
   - Minimum structure: frontmatter, "When to use" with triggers, "Procedure"
     with numbered steps.
3. Check skill candidates flagged in logs. If a candidate has appeared in 2+ logs,
   it's mature enough — create the skill.
4. Review existing skills: any not referenced in recent logs?
   - Flag as review candidate (may be obsolete or poorly described).
5. If a new interactive skill was created, add it to the Skills section of
   AGENTS.md with a clear trigger description.

**6b. Rule review:**

1. Scan logs for rule candidates (flagged in `## System observations`).
2. Scan logs for user corrections or recurring frustrations with agent behavior.
3. For each candidate:
   - Has the same pattern appeared in 2+ conversations? → Propose as a new rule.
   - Is it a one-off correction? → Leave it as a log entry for now.
4. Present proposed rules to the user for approval. Do not modify AGENTS.md rules
   without explicit confirmation.
5. If approved, add the rule to the Rules section of AGENTS.md.

---

### Phase 7: Contradiction detection

**Goal:** Maintain coherence in the knowledge base.

1. Review recent logs from `logs/` (last month).
2. If information contradicts something in `agent_brain/concepts/` or `agent_brain/projects/`:
   - Clear contradiction + reliable new info → update the old file.
   - Ambiguous → add a note:
     ```
     > ⚠️ CONTRADICTION (YYYY-MM-DD): [description]. Pending human review.
     ```
3. Record contradictions found and how they were resolved.

---

### Phase 8: Structure review

**Goal:** Ensure `agent_brain/` structure matches how the system is actually used.
The directory structure should emerge from use, not from upfront design.

1. Review recent logs for structure candidates (flagged in `## System observations`
   by `/reflect`).
2. Scan files in `agent_brain/` — are there files that don't fit their current
   directory? (e.g., a "concept" that is really a team reference, or a "project"
   file that is actually a cross-team request log.)
3. If a new category of information has accumulated (3+ files of a similar type
   in an ill-fitting directory), propose creating a dedicated directory:
   - Suggest the directory name and what would go there.
   - Present to the user for approval.
   - If approved: create the directory, move the files, and add the new location
     to the "Where to find things" section of AGENTS.md.
4. Check existing directories: any empty or with only 1 file after >30 days?
   - Consider whether the directory is premature. Flag it — the file may belong
     better in a more general directory.
5. Record structural changes and proposals.

---

### Finalize

1. Create `logs/maintenance_YYYY-MM-DD.md`:

```markdown
# Maintenance — YYYY-MM-DD

## Compaction
- Logs archived: [list or "none"]
- Knowledge extracted: [list or "none"]

## Pruning
- Files archived: [list or "none"]
- Review candidates: [list or "none"]

## Promotion
- Promoted to Active context: [list]
- Removed from Active context: [list]

## Board hygiene
- Stale waiting items flagged: [list or "none"]
- Untriaged inbox items flagged: [list or "none"]
- Stale Next Actions flagged: [list or "none"]

## Ideas
- Scratchpad items flagged: [list or "none"]
- Stale seeds flagged: [list or "none"]
- Stuck developing ideas flagged: [list or "none"]
- Ready ideas pending action: [list or "none"]
- Ideas archived: [list or "none"]

## Skills and rules
- New skills created: [list or "none"]
- Skills flagged for review: [list or "none"]
- Rules proposed: [list or "none"]
- Rules added (approved): [list or "none"]

## Structure
- Directories created: [list or "none"]
- Files moved: [list or "none"]
- Structural proposals (pending approval): [list or "none"]

## Contradictions
- Detected: [list or "none"]
- Resolved: [list]
- Pending human review: [list]
```

2. Git commit:

```bash
git add -A && git commit -m "maintenance: YYYY-MM-DD" 2>/dev/null || true
```
