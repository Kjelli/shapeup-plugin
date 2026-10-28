# Tracker contract

The skills speak only in ShapeUp terms: cycle, bet, slice, question, state, tag, comment. A tracker adapter in
[trackers/](trackers/) says exactly what each of these is in one tracker and which tools to use. The skills follow the
adapter literally, so an adapter must be as concrete as instructions to a new colleague: item types, field names,
state names, link types and tool names, never "the equivalent of".

The tracker is the agents' only memory between sessions. A tracker that can't meet a point below can't run ShapeUp
with this plugin, unless the adapter says how it is worked around and what is lost.

## What every tracker adapter defines

Use these headings, in this order.

### Configuration

The `tracker.*` fields in `.claude/shapeup.json`, which are required, their defaults, and how `shapeup:init` can
suggest them (for example from the git remote).

### Connection

The MCP server the adapter uses: an `.mcp.json` entry `shapeup:init` can write, how the user signs in, and how to
tell from the session's tool list that the server is connected.

### Scope

What marks an item as belonging to this ShapeUp setup, as opposed to other work in the same tracker (for example an
area path, a project board, a label), and which fields every created item gets so it stays in scope.

### Item types

| Concept | Required |
|---|---|
| **Cycle** | A container whose children are bets. Created and closed by humans. |
| **Bet** | An item in scope whose parent is a cycle. Its description holds the pitch. Has an assignee (the shaper) and a creator. |
| **Slice** | An item in scope, child of a bet. |
| **Question** | An item in scope, child of a bet or of one of its slices. Can be assigned to the shaper. |

For each: the tracker's item type, how parent and child are linked, how its ID is written, and its web link.

### States

The tracker's state for each logical state the skills use:

| Item | Logical states |
|---|---|
| Cycle | open, closed |
| Bet | `New`, `Active`, `Resolved`, `Closed` (closed also covers removed or dropped) |
| Slice | `New`, `Active`, `Resolved`, `Closed` |
| Question | open, `Closed` |

Plus a **first-session check**: how to confirm that slices can take every slice state, since some trackers let
teams customize them.

### Tags

How to add, remove and read the tags `slice`, `uphill`, `downhill`, `nice-to-have` and `question` (tags, labels or an
equivalent field).

### Comments

How to post and read comments on an item, in order, with author and time. Comments must be plain enough that the
exact first line (`Session 4 of 10 started`, `DECISION (session 3): ...`, `CIRCUIT BREAKER: ...`) survives.

### Descriptions

The format a description takes (Markdown or HTML) and how to write the pitch into a bet's description.

### Queries

How to:

- List open cycles in scope.
- Read one bet by ID, with parent, state, assignee, creator and description.
- Find bets by title: bets in scope whose parent is a cycle and whose state is `New`, `Active` or `Resolved`.
- Read every descendant of a bet in scope, with tags, states and recent comments.

### Changes the skills make

How to create a bet, a slice and a question with every required field, change a state, set the assignee, and link a
pull request to an item.

### Off-limits

Things the tracker has that agents must never create or change (for example sprints, boards, saved queries).

### Commit references

How a commit message references an item so the tracker links them (for example `#2001`).

### Verification

The read-only calls `shapeup:init` makes to prove the configuration works: the scope exists, and open cycles can be
listed.
