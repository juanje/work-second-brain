---
last_accessed: YYYY-MM-DD
access_count: 1
created: YYYY-MM-DD
---

# Memory-first lookup

## Pattern

Before querying external tools (`did`, `jira-pending`, `jira-detail`), check internal memory first. The board, daily logs, and brain files often already contain the answer.

## Why

External calls are slower and consume tokens. The board, logs, and brain files frequently have the exact tickets, MRs, and context needed — especially for recent or already-captured work.

## Where to look

1. **Board (`board/BOARD.md`)** — Sections are ordered actionable-first: Doing, Next Actions, Waiting, Sprint Backlog, Inbox, Parked, Done.
2. **`memory/logs/`** — Curated decisions and context from recent days.
3. **`brain/projects/`**, **`brain/concepts/`** — Durable context for recurring topics.

## Rule

If the answer is already in memory, use it. Only call external tools when the data is missing or must be fresh.
