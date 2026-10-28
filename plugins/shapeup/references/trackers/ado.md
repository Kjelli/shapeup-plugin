# Tracker: Azure DevOps Boards

Meets [../tracker-contract.md](../tracker-contract.md). Below, `<organization>`, `<project>`, `<areaPath>` and
`<iterationPath>` are the values from `.claude/shapeup.json`.

## Configuration

```json
"tracker": {
  "type": "ado",
  "organization": "contoso",
  "project": "Product",
  "areaPath": "Product\\ShapeUp",
  "iterationPath": "Product\\ShapeUp"
}
```

| Field | Required | Meaning |
|---|---|---|
| `organization` | Yes | The Azure DevOps organization. |
| `project` | Yes | The Azure DevOps project. |
| `areaPath` | Yes | The area path that holds all ShapeUp work items. |
| `iterationPath` | No | The iteration path every work item gets. Defaults to `areaPath`. |

`shapeup:init` can suggest `organization` and `project` from an Azure Repos remote:
`https://dev.azure.com/<organization>/<project>/_git/<repo>` or
`https://<organization>.visualstudio.com/<project>/_git/<repo>`.

## Connection

The Azure DevOps MCP server, named `ado`. The `.mcp.json` entry, with `<organization>` filled in:

```json
{
  "mcpServers": {
    "ado": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@azure-devops/mcp", "<organization>", "-d", "core", "work-items", "repositories"]
    }
  }
}
```

`repositories` is included so the same server also serves Azure Repos as the code host. It needs Node.js 20 or later.
On first use the user approves the project's MCP server and signs in in the browser. The server is connected when the
session has `ado` work item tools (named like `mcp__ado__wit_...`). The agent acts as the signed-in user and can never
do more than they can.

## Scope

Area path `<areaPath>`. The project may also hold production work and other teams' work: a work item is in scope only
if its Area Path is `<areaPath>`. Every work item an agent creates gets Area Path `<areaPath>` and Iteration Path
`<iterationPath>`; never change those fields afterwards.

## Item types

| Concept | Work item type | Parent link |
|---|---|---|
| **Cycle** | Epic in the area path | – |
| **Bet** | Feature in the area path | Parent (Hierarchy-Reverse) is an Epic |
| **Slice** | User Story | Parent is the bet's Feature |
| **Question** | Task | Parent is the slice it is about, or the bet's Feature if it is about the bet as a whole |

- The ID is the work item ID, written `#1958`.
- Web link: `https://dev.azure.com/<organization>/<project>/_workitems/edit/<id>`.
- **Assignee** is the **Assigned To** field; **creator** is **Created By**.

```
Epic             = cycle
└─ Feature       = bet (the pitch is its description)
   ├─ User Story = slice
   │  └─ Task    = question about that slice
   └─ Task       = question about the bet as a whole
```

## States

| Item | Logical state | Azure DevOps state |
|---|---|---|
| Cycle | open | Any state except `Closed` and `Removed` |
| Cycle | closed | `Closed` or `Removed` |
| Bet | `New`, `Active`, `Resolved`, `Closed` | The same names. `Removed` also counts as closed. |
| Slice | `New`, `Active`, `Resolved`, `Closed` | The same names |
| Question | open | `New` |
| Question | `Closed` | `Closed` |

**First-session check:** User Stories must have the states `New`, `Active`, `Resolved` and `Closed`, which depends on
the project's process. Check the work item type's states.

## Tags

The work item **Tags** field, a semicolon-separated list. Keep other tags on the item when adding or removing one.

## Comments

Work item discussion comments. Read them newest first and in full; the first line of a session comment is matched
exactly.

## Descriptions

The **Description** field is HTML. Write Markdown only if the tool takes a Markdown format option. Keep section
headings as headings.

## Queries

Use the `ado` work item tools, with WIQL where a query is needed. Always filter on `[System.AreaPath] = '<areaPath>'`
and `[System.TeamProject] = '<project>'`.

- **Open cycles:** Epics in the area path whose state isn't `Closed` or `Removed`.
- **One bet:** read the work item by ID with its relations; check the type is Feature, the area path matches, and the
  parent is an Epic.
- **Bets by title:** Features in the area path with state `New`, `Active` or `Resolved`, then keep those whose parent is
  an Epic.
- **Descendants of a bet:** a work item links query (tree, Hierarchy-Forward) from the Feature, filtered to the area
  path, with Tags, State and recent comments for each item.

## Changes the skills make

- **Create:** set Work Item Type, Title, Description, Area Path `<areaPath>`, Iteration Path `<iterationPath>`, Tags
  and Assigned To as the skill says, then add the parent link (Hierarchy-Reverse) to the parent's ID.
- **Bet:** a Feature, state `New`, parent the cycle's Epic, Assigned To the shaper.
- **Slice:** a User Story under the Feature.
- **Question:** a Task under the slice or Feature, Assigned To the shaper.
- **State:** update System.State.
- **Pull request link:** add an artifact link of type *Pull Request* from the work item to the pull request (the
  code host adapter says how when the pull request is in Azure Repos).

## One-time setup

None. Tags are created when first used.

## Off-limits

Never create or change sprints, iterations, area paths, teams, queries (shared or personal), dashboards or boards
configuration.

## Commit references

`#<id>` in the commit message, for example `#2001`. Azure Repos links it to the work item.

## Verification

1. The area path `<areaPath>` exists in `<project>` (list the project's area classification nodes).
2. List open cycles. Report how many, by ID and title. None is fine, but say a human must create an Epic in the area
   path before shaping a bet.
3. The User Story states include `New`, `Active`, `Resolved` and `Closed`.
