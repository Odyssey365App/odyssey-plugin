---
name: board-kanban
description: Work an Odyssey project board like a kanban — move tasks between lists (for example To do → Doing → Done), mark tasks complete, add new tasks to the right list, and take items off a board. Use when the user says "move X to done", "I finished Y", "start working on Z", "add this to the backlog list", "clean up the Done column", or reports progress on items that live on a board.
---

# Move work across a board

On an Odyssey board a **list** is an `itemTile` component whose `containItems` references tasks and
notes. Moving an item between lists is a change to two components; completing a task is a change to
the task itself. Keep both in step.

## Steps

1. **Load the current board.** `get_project` with `includeComponents: true`. Map each list's `text`
   (its title) to its component id, and each item to the list that holds it. Resolve the items the
   user named by title; if a title matches more than one item, ask.
2. **Confirm the plan in one line** when more than three items change, e.g.
   "Move 4 tasks from Doing to Done and mark them complete?" Single, clearly named moves need no
   confirmation.
3. **Apply each change.**

   | User intent | Calls |
   |---|---|
   | Move between lists | `update_project_component` on the source list with `mode: "remove"`, then on the target list with `mode: "append"`, both with `containItems: [{uid, type}]` |
   | Mark done | `complete_todo` (it records the completion on the timeline). If the board has a Done list, also move the task there |
   | Start working | `update_todo` with `status: "inProgress"`; move it to the Doing list if the board has one |
   | Add a new task to a list | `create_todo` with `projectUid` and `attachToComponentId` = that list's id |
   | Take off the board | `update_project_component` with `mode: "remove"` — the task itself still exists |
   | Delete for good | Ask first. Then `delete_todo` / `delete_memo` in addition to removing it from the list |

   Always use `mode: "append"` or `"remove"` for lists. `replace` rewrites the whole list and silently
   drops anything you did not include; on shared boards it is refused.
4. **Verify.** Re-read with `get_project` when you changed more than a few items, and fix any list that
   does not match what you reported.
5. **Report** in the user's language: what moved where, what was completed, and anything you could not
   do (for example another member's item on a shared board that the user cannot edit).

## Guardrails

- "Remove" is ambiguous. Ask whether the user means off this board or deleted everywhere.
- On a shared board, edit only the user's own items unless the response shows `canEditItems: true`.
- An item can be on only one shared board at a time. If a call is refused for that reason, tell the
  user and ask before retrying with `moveFromOtherSharedBoard: true`.
- Do not call `tidy_board` after moving items; lists keep their place on the canvas.
