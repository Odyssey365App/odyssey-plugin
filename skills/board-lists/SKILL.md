---
name: board-lists
description: Run an Odyssey project in its Lists view — set up lists (for example To do → Doing → Done), move tasks between lists, add new or existing tasks to a list, mark tasks complete, and take items off. Use when the user says "set up To do / Doing / Done for this project", "move X to done", "I finished Y", "start working on Z", "add this to the backlog", "clean up the Done list", or reports progress on items in a project. Older Odyssey apps call the Lists view "kanban"; treat that word as the same thing.
---

# Move work across a project's Lists view

An Odyssey project shows the same components through two views (on each sheet):

- **Lists view** — ordered **lists** (stages or statuses), each holding items in order. Lists have no
  coordinates; you work with a list id (`columnId`) and a position (`index`, 0 = top / leftmost).
- **Board** — cards and lists placed freely on a canvas.

A list is one component that can show in either view or both. When a list also shows on the board,
adding an item in the Lists view puts it on that board list too — it is the same data. Tables, single
item cards, links, text cards and calendars can sit in a list as tiles. Use the list tools for List
view requests, and never fake one with sections or coordinates.

## Steps

1. **Read the Lists view.** `get_lists(projectId)` returns the lists in order — whether each also
   shows on the board (`onBoard`), its `status` if it has one, and its items (`uid`, `type`, `title`,
   `status`). Resolve the items the user named by title; if a title matches more than one item, ask.
   - No lists yet: if the board already has a matching list (for example *To do / Doing / Done* lists
     on the canvas), show that list in the Lists view with
     `set_component_visibility(componentId, lists: true)` instead of creating a new one. Otherwise
     create the lists (see the table). The common set is *To do · Doing · Done*; use the user's own
     stage names when they give them. Do not create a list whose name already exists.
2. **Confirm the plan in one line** when more than three items change or when deleting a list, e.g.
   "Move 4 tasks from Doing to Done and mark them complete?" Single, clearly named moves need no
   confirmation.
3. **Apply each change.**

   | User intent | Calls |
   |---|---|
   | Set up lists | `save_list(projectId, title, status?)` once per list, left to right — give *To do · Doing · Done* the status `notStarted` · `inProgress` · `done`; stage names get none |
   | Rename, recolor or set a status | `save_list(projectId, columnId, title / color / status)` (`status: "none"` clears it) |
   | Reorder lists | `reorder_lists(columnIds)` with the full left-to-right order |
   | Move between lists, or within one | `move_list_item(itemUid, toColumnId, index?)` |
   | Add a new task to a list | `create_todo` with `projectUid` and `attachToComponentId` = the list id (same for `create_memo`) |
   | Add existing items | `add_to_list(columnId, items: [{uid, type}], index?)` |
   | Mark done | Move it to the list whose status is `done` — that completes the task (the response's `statusChanged` says so). With no such list, `complete_todo` |
   | Start working | Move it to the `inProgress` list; with no such list, `update_todo` with `status: "inProgress"` |
   | Take off a list | `remove_from_list(columnId, items)` — the task itself still exists |
   | Hide a list from the Lists view | `set_component_visibility(componentId, lists: false)` — it stays on the board if it was there |
   | Delete a list | Ask first. `delete_list(columnId)` — it goes from both views; its items stay in the project |
   | Delete an item for good | Ask first. Then `delete_item` with `type` (`todo` or `memo`) and the item's `id` |

   A list without a status leaves task status alone — change the task with `update_todo` too when
   the user wants both. If a response has `statusFailed`, the move happened but the status did not;
   fix it with `update_todo` / `complete_todo`.

   A board **table** can sit in a list as a tile: `add_to_list` with `{uid: <table componentId>,
   type: "projectComponent"}`. The tile shows the same table — edit its cells with
   `update_project_component(table)`.
4. **Verify.** Re-read with `get_lists` when you changed more than a few items, and fix any list that
   does not match what you reported.
5. **Report** in the user's language: what moved where, what was completed, and anything you could not
   do (for example another member's item on a shared board).

## Board-only lists

Some boards track progress with lists that show only on the canvas. Read them with
`get_project(projectId, includeComponents: true)`; each list is an `itemTile` whose `text` is its title.

- Move between lists: `update_project_component` on the source list with `mode: "remove"`, then on the
  target list with `mode: "append"`, both with `containItems: [{uid, type}]`.
- Add a new task to a list: `create_todo` with `projectUid` and `attachToComponentId` = the list id.
- Always use `mode: "append"` or `"remove"`. `replace` rewrites the whole list and silently drops
  anything you did not include; on shared boards it is refused.
- If the user would rather work in the Lists view, offer to show those lists there with
  `set_component_visibility(lists: true)` — do not rebuild them.

## Guardrails

- Lists that show only in the Lists view have no place on the board. Do not give them board positions
  or sizes (refused until `board: true`), and do not call `tidy_board` for Lists view work.
- The archive is not a list. Do not add to it or move into it; it only reports a count.
- "Remove" is ambiguous. Ask whether the user means off the list, off the board, or deleted everywhere.
- On a shared board you can add and take off only your own items; deleting a list needs the board
  owner.
- An item can be shared on only one shared board at a time. If a call is refused for that reason, tell
  the user and ask before retrying with `moveFromOtherSharedBoard: true`.
- Each sheet has its own Lists view. Pass `sheetId` to `get_lists` / `save_list` for a sheet other than
  the default one.
