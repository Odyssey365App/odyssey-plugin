---
name: board-kanban
description: Run an Odyssey project as a kanban — set up columns (for example To do → Doing → Done), move tasks between columns, add new or existing tasks to a column, mark tasks complete, and take items off the kanban. Use when the user says "make a kanban for this project", "move X to done", "I finished Y", "start working on Z", "add this to the backlog", "clean up the Done column", or reports progress on items in a project.
---

# Move work across a project kanban

An Odyssey project has two independent views of its items:

- **Kanban** — ordered **columns** (stages or statuses), each holding items in order. Columns have no
  coordinates; you work with a column id and a position (`index`, 0 = top / leftmost).
- **Board canvas** — cards and **lists** (`itemTile` components) placed freely on a canvas.

They do not mirror each other. Putting a task in a kanban column does not put it on a canvas list, and
the reverse is also true; the same task can be in both. Use the kanban tools for kanban requests, and
never fake a kanban with canvas lists, sections or coordinates.

## Steps

1. **Read the kanban.** `get_kanban(projectId)` returns the columns in order, each with its items
   (`uid`, `type`, `title`, `status`). Resolve the items the user named by title; if a title matches
   more than one item, ask.
   - No columns yet and the user wants a kanban: create the columns first (see the table). The common
     set is *To do · Doing · Done*; use the user's own stage names when they give them. Do not create
     a column whose name already exists.
   - No columns, but the user is talking about lists that already sit on the canvas (for example a
     board built with *To do / Doing / Done* lists): follow **Canvas lists** below instead.
2. **Confirm the plan in one line** when more than three items change or when deleting a column, e.g.
   "Move 4 tasks from Doing to Done and mark them complete?" Single, clearly named moves need no
   confirmation.
3. **Apply each change.**

   | User intent | Calls |
   |---|---|
   | Set up columns | `add_kanban_column(title)` once per column, left to right (`index` inserts at a position) |
   | Rename or recolor a column | `update_kanban_column(columnId, title / color)` |
   | Reorder columns | `reorder_kanban_columns(columnIds)` with the full left-to-right order |
   | Move between columns, or within one | `move_kanban_item(itemUid, toColumnId, index?)`. If the item is in more than one column, pass `fromColumnId` |
   | Add a new task to a column | `create_todo` with `projectUid` and `attachToComponentId` = the column id (same for `create_memo`) |
   | Add existing items | `add_to_kanban(columnId, items: [{uid, type}], index?)` |
   | Mark done | `complete_todo` (it records the completion on the timeline), then move it to the Done column if there is one. Moving to Done alone does **not** complete the task |
   | Start working | `update_todo` with `status: "inProgress"`, and move it to the Doing column if there is one |
   | Take off the kanban | `remove_from_kanban(columnId, items)` — the task itself still exists |
   | Delete a column | Ask first. `delete_kanban_column(columnId)` — its items stay in the project, only the column goes |
   | Delete an item for good | Ask first. Then `delete_todo` / `delete_memo` |

   A board **table** can sit in a column as a tile: `add_to_kanban` with `{uid: <table componentId>,
   type: "projectComponent"}`. The tile shows the same table — edit its cells with
   `update_project_component(table)`, not through the kanban.
4. **Verify.** Re-read with `get_kanban` when you changed more than a few items, and fix any column
   that does not match what you reported.
5. **Report** in the user's language: what moved where, what was completed, and anything you could not
   do (for example another member's item on a shared board).

## Canvas lists

Some boards track progress with lists on the canvas instead of a kanban. Read them with
`get_project(projectId, includeComponents: true)`; each list is an `itemTile` whose `text` is its title.

- Move between lists: `update_project_component` on the source list with `mode: "remove"`, then on the
  target list with `mode: "append"`, both with `containItems: [{uid, type}]`.
- Add a new task to a list: `create_todo` with `projectUid` and `attachToComponentId` = the list id.
- Always use `mode: "append"` or `"remove"`. `replace` rewrites the whole list and silently drops
  anything you did not include; on shared boards it is refused.
- If the user would rather work in a kanban, offer to set one up — do not move their canvas lists.

## Guardrails

- Kanban column ids work only with the kanban tools. `update_project_component`,
  `delete_project_component` and `tidy_board` refuse them; `get_project` does not list them as
  components.
- "Remove" is ambiguous. Ask whether the user means off the kanban, off the board, or deleted everywhere.
- On a shared board you can add and take off only your own items; deleting a column needs the board
  owner.
- An item can be shared on only one shared board at a time. If a call is refused for that reason, tell
  the user and ask before retrying with `moveFromOtherSharedBoard: true`.
- Do not call `tidy_board` for kanban work; columns have no place on the canvas.
