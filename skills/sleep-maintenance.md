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

**Goal:** Keep `memory/logs/` manageable while preserving valuable context.

1. List files in `memory/logs/` older than 30 days (exclude maintenance logs).
2. For each old log:
   - Check if its key content (decisions, lessons) has been extracted to `brain/`.
   - If fully extracted: move to `memory/archive/`.
   - If it contains unextracted knowledge: extract it to the appropriate `brain/`
     location first, then archive the log.
3. Never delete logs outright — always archive. Git history provides additional backup.
4. Record what was compacted.

---

### Phase 2: Pruning by disuse

**Goal:** Move stale information out of the active space.

1. Scan all files in `brain/projects/`, `brain/concepts/`, and `brain/requests/`.
2. Read their metadata (`last_accessed`, `access_count`).
3. If a file hasn't been accessed in >30 days AND has fewer than 5 total accesses:
   - Move it to `brain/archive/`.
   - Record the move.
4. If a file hasn't been accessed in >30 days BUT has >5 accesses:
   - Don't move it. Flag it as a review candidate in the maintenance log.

**Exception:** Never move or prune files in `identity/`, `skills/`, or `board/`.
Those require human decision.

---

### Phase 3: Promotion and degradation (AGENTS.md)

**Goal:** Keep the "Active context" section of AGENTS.md reflecting what's
actually relevant right now.

1. Scan all files in `brain/projects/`, `brain/concepts/`, `brain/requests/`,
   and `brain/reviews/`.
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

1. Read `board/BOARD.md`.
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

**Goal:** Keep `brain/ideas/` healthy — promote scratchpad items, advance
developing ideas, flag ready ideas for action.

1. Read `brain/ideas/_scratchpad.md`.
   - Any item older than 7 days? Flag it: either promote to its own idea file
     or remove if no longer relevant.
2. Scan all idea files in `brain/ideas/` (excluding `_scratchpad.md`).
3. Read their metadata (`last_accessed`, `status`).
4. For each idea:
   - `seed` not accessed in >14 days → flag as stale. Suggest: develop, archive,
     or move to scratchpad if it was too small for its own file.
   - `developing` not accessed in >21 days → flag for review. Is it stuck?
   - `ready` not accessed in >7 days → flag for action. It's mature enough to
     convert — remind the user.
   - `converted` or `archived` older than 30 days → move to `brain/archive/`.
5. Record findings in the maintenance log.

---

### Phase 6: Skill review

**Goal:** Keep `skills/` up to date and detect opportunities for new skills.

1. Review recent logs from `memory/logs/` (last month).
2. Look for repeated patterns: procedures done multiple times in similar ways?
   - If found, create a new skill in `skills/` with `verb-object.md` naming.
   - Minimum structure: frontmatter, "When to use" with triggers, "Procedure"
     with numbered steps.
3. Review existing skills: any not referenced in recent logs?
   - Flag as review candidate (may be obsolete or poorly described).
4. If a new interactive skill was created, add it to the Skills section of
   AGENTS.md with a clear trigger description.

---

### Phase 7: Contradiction detection

**Goal:** Maintain coherence in the knowledge base.

1. Review recent logs from `memory/logs/` (last month).
2. If information contradicts something in `brain/concepts/` or `brain/projects/`:
   - Clear contradiction + reliable new info → update the old file.
   - Ambiguous → add a note:
     ```
     > ⚠️ CONTRADICTION (YYYY-MM-DD): [description]. Pending human review.
     ```
3. Record contradictions found and how they were resolved.

---

### Finalize

1. Create `memory/logs/maintenance_YYYY-MM-DD.md`:

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

## Skills
- New skills created: [list or "none"]
- Skills flagged for review: [list or "none"]

## Contradictions
- Detected: [list or "none"]
- Resolved: [list]
- Pending human review: [list]
```

2. Git commit:

```bash
git add -A && git commit -m "maintenance: YYYY-MM-DD" 2>/dev/null || true
```
