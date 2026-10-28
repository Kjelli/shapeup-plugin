# ShapeUp

Run ShapeUp with Claude agents. Product owners shape the pitches, decide what ships and set the rhythm. Agents help
them shape, and build the bets autonomously, one session at a time.

| Skill | Run it with | Does |
|---|---|---|
| `shapeup:init` | `/shapeup:init` | Sets up a repository: tracker, code host, MCP server, `.claude/shapeup.json`. |
| `shapeup:shape` | `Help me shape a pitch`, or `/shapeup:shape <title>` | Helps a product owner shape a pitch and posts it as a bet. |
| `shapeup:build` | `Continue with bet #<id>`, `Continue with <bet title>`, or `/shapeup:build <id or name>` | Builds a bet, one session at a time. |

Shaping and building are kept separate on purpose: run them in separate conversations.

## How it works

Each **cycle** is a container in the tracker. The **bets** of the cycle are its children, each with its pitch in the
description. A product owner shapes a pitch together with an agent, which posts it as a bet once they confirm it. Then
an agent is pointed at one bet by ID or by name, reads everything it needs from the tracker, works for one focused
session, records what it did there, and stops. Nothing about a cycle or a bet is stored in the repository, so any
number of people and agents can work at the same time, as long as each bet has only one session running at a time.

- The agent discovers **slices** (children of the bet) and **questions** for the shaper, tags each slice `uphill` or
  `downhill`, and records every judgment call as a `DECISION` comment.
- Each slice has a branch and one pull request into the bet branch, `bet/<id>`. When every slice is merged or cut, the
  agent opens one bet pull request to the main branch.
- **Humans approve, agents merge.** The agent never approves its own work and never merges without the shaper's
  approval.
- `Session N of M started` / `ended` comments on the bet act as a lock. When the appetite is spent before the bet pull
  request is open, the agent posts a `CIRCUIT BREAKER` comment instead.

## Trackers and code hosts

The skills speak only in ShapeUp terms. What a cycle, bet, slice or pull request is in a particular tool is defined
by an adapter:

| Kind | Supported | Contract |
|---|---|---|
| Tracker | Azure DevOps Boards (`ado`) | [references/tracker-contract.md](references/tracker-contract.md) |
| Code host | Azure Repos (`ado`) | [references/code-host-contract.md](references/code-host-contract.md) |

The tracker and code host are chosen separately, so for example a Jira tracker with GitHub as the code host is a
valid combination once both adapters exist. To add one, write an adapter file that meets the contract and add it to
[references/config.md](references/config.md).

## Set up a repository

1. Install the plugin (see the marketplace README).
2. In the repository, run `/shapeup:init`. It asks for the tracker and code host, connects the MCP server, verifies
   access and writes `.claude/shapeup.json`.
3. Put a Definition of done in the repository's root `CLAUDE.md`. The building agent holds every slice to it.
4. Commit `.claude/shapeup.json` and `.mcp.json`.

The agent acts as the signed-in user and can never do more than they can.
