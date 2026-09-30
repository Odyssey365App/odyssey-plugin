---
name: board-mindmap
description: Brainstorm on an Odyssey project board as a mindmap — put a central topic in the middle, branch ideas out in several directions, and turn chosen branches into tasks. Use when the user wants to "map out", "brainstorm on my board", "break this idea down", "make a mindmap of", or explore options visually before planning.
---

# Brainstorm as a mindmap

A mindmap on an Odyssey board is a **relationship**, not a special card: one `mindmap` component is
the center, and any component becomes a branch by naming its parent with `mindmapParentId`. The
board's layout engine spreads the branches, so you never compute coordinates.

## Steps

1. **Agree on the center.** Restate the topic in a few words. Pick the board (existing one via
   `list_projects`, or `create_project` after the user agrees to a name).
2. **Sketch the tree in chat first**: 3–6 first-level branches, 2–5 ideas under each, two levels
   deep at most. Ask whether to adjust before writing anything to the board.
3. **Create the center.** `add_project_component` with `type: "mindmap"` and `text` = the topic.
4. **Create the branches.** For each first-level branch, `add_project_component` with `type: "text"`
   (or `roundRect` for emphasis), `text` = the branch label, `mindmapParentId` = the center's id, and
   `mindmapDirection` spread across `right`, `left`, `down`, `up` so the map stays balanced. Add the
   second-level ideas the same way with the branch as parent and the same direction as their branch.
5. **Lay it out.** Call `tidy_board` once when the map is complete. Skip this if the board already
   had components the user arranged by hand — tell the user they can use the app's auto-arrange
   instead.
6. **Turn ideas into action (optional).** When the user picks branches to act on, make an `itemTile`
   list for them with `mindmapParentId` = that branch, then add tasks with `create_todo`
   (`projectUid` + `attachToComponentId` = the list's id). Tasks stay linked to the branch they came
   from.
7. **Report back**: the center, the branches created, and any tasks made from them.

## Guardrails

- Keep labels short (under about six words); long text belongs in a note, not a branch.
- Reuse an existing mindmap if `get_project` shows one on the same topic under `structure.mindmaps`.
- Removing branches uses `delete_project_component` and needs an explicit yes from the user.
