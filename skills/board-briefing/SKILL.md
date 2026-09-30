---
name: board-briefing
description: Read an Odyssey project board and brief the user on where it stands — progress per list and section, what is done, in progress, stuck, or unassigned to any list, and what to do next. Use when the user asks "where are we on this project", "summarize my board", "what's left on X", "catch me up on this project", or wants a status update to share.
---

# Brief the user on a board

This is a **read-only** workflow. Nothing is changed unless the user asks at the end.

## Steps

1. **Find the board.** Use `list_projects` or `search` with the name the user gave. If several match,
   list them briefly and ask which one.
2. **Read it.** Call `get_project` (no components needed). Use:
   - `items[]` — each item carries `title`, and tasks also `status` (`notStarted` / `inProgress` /
     `done`) and `dueAt` (deadline end). `listTitle` and `sectionTitle` say where it sits, and
     `componentId` groups items that share a list even when the list has no title. `missing: true`
     means the card points at an item that no longer exists.
   - `structure.sections` / `structure.mindmaps` — the user's own grouping. Follow it in the report.
   - `autoFilterLists[]` — lists filled by a saved filter; their tasks are summaries only.
   - `shared` — if present, the board belongs to someone else; mention whose items are whose only
     when it matters.
3. **Open details only when needed.** Titles and statuses are already in `items[]`. Call `get_todo` /
   `get_memo` only for items whose title is not enough to judge (for example a checklist's progress
   or a note that holds decisions). Do not open every item.
4. **Write the briefing** in the user's language, in this order:
   1. One-line status: e.g. "12 of 20 tasks done; the Launch section is blocked on copy."
   2. Per section or list: done / in progress / not started counts and the notable items.
   3. **Needs attention**: tasks in progress for a long time, overdue deadlines, lists that are empty,
      notes that contain open questions.
   4. **Suggested next steps**: at most three, each tied to a specific item.
   Keep it scannable — short bullets, item titles quoted exactly, no raw ids.
5. **Offer, don't act.** At the end you may offer to (a) move or complete items (see the
   `board-kanban` skill) or (b) post the briefing to the board's comment thread.

## Posting the briefing as a board comment

`comment_on_project` notifies the board owner and every member and cannot be recalled, so call it
only after the user explicitly asks. Attach the items you mention with `containItems` instead of
copying their titles into the text, post once per briefing, and keep `content` to plain text.
