---
name: board-builder
description: Turn a conversation, plan, meeting notes, or brainstorm into a laid-out Odyssey project board — task lists for things to do, note cards for things to read, grouped into sections. Use when the user says "put this on a board", "make a project for this", "organize this into Odyssey", "save these ideas to my board", or asks to add a batch of tasks and notes to an existing board.
---

# Build an Odyssey board

A board is a canvas of components. Items (tasks, notes) live in their own documents and a board only
**references** them from `itemTile` components. Creating an item does not put it on a board, and
removing it from a board does not delete it.

## Layout rule: the container follows the item

| Item | How to place it | Why |
|---|---|---|
| **Task** | Many tasks inside **one** `itemTile` list | Tasks are scanned and moved through states |
| **Note** | **One** note per `itemTile` card, laid out as a grid | Notes are opened and read one at a time |

A list of twenty notes shows only titles; a grid of twenty task cards is noise. Keep to the table above
unless the user asks for something else.

## Steps

1. **Pick the board.** If the user named one, find it with `list_projects` (or `search`). Otherwise ask
   whether to create a new board; create it with `create_project` only after they agree to a name.
2. **Read before writing.** For an existing board call `get_project` with `includeComponents: true`.
   Reuse its sections and lists (see `structure.sections` and each component's `text`) instead of
   creating duplicates. Never move or restyle components the user arranged by hand.
3. **Draft the outline in chat first** when you will create more than about five items: sections,
   lists, and which items go where. Wait for a go-ahead, then build.
4. **Create containers.**
   - Section (optional grouping frame): `add_project_component` with `type: "section"` and `text`.
   - Task list: `add_project_component` with `type: "itemTile"`, `text` = list title, and `sectionId`
     when it belongs to a section. The section positions it; do not compute coordinates.
5. **Create items straight into their container** — one call each, no separate attach step:
   - Task: `create_todo` with `projectUid` = board id and `attachToComponentId` = the list's id.
   - Note: first create an empty `itemTile` card for it at its grid position (step 6), then
     `create_memo` with `text` (the body), `projectUid`, and `attachToComponentId` = that card's id.
   Existing items are added with `update_project_component` using `mode: "append"` and `containItems`.
6. **Place note cards as a grid** by giving each card `size: {width: 400, height: 220}` and an explicit
   `offset`: 5 columns, gap 40, starting below the lists (for example `dy` 560). Card `i` goes to
   `dx = (i % 5) * 440`, `dy = 560 + floor(i / 5) * 260`. Cards cannot be narrower than 400.
7. **Tidy only boards you just made.** If the board was new and has no note grid, call `tidy_board`
   once at the end. Do not call it after placing a grid (it re-flows the grid) or on a board the user
   arranged by hand.
8. **Report back** in chat: the board name, each section/list with item counts, and anything you
   skipped. Do not paste raw tool output.

## Guardrails

- A call that names a missing component or item fails and creates nothing; fix the id and retry
  rather than re-creating items (an `attachError` in a success response means the item exists — do
  not create it again).
- On a board shared with the user (`shared: true`), you can add only the user's own items, and a list
  can be edited only with `mode: "append"` or `"remove"`.
- Deleting components (`delete_project_component`) or items needs an explicit yes from the user.
- Write item titles in the user's language, short and actionable ("Draft landing copy", not
  "Landing copy things").
