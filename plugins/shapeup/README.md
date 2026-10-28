# ShapeUp

Run ShapeUp with Claude agents. Product owners shape the pitches, decide what ships and set the rhythm. Agents help
them shape, and build the bets autonomously, one session at a time.

| Skill | Run it with | Does |
|---|---|---|
| `shapeup:init` | `/shapeup:init` | Sets up a repository: tracker, code host, their connection, `.claude/shapeup.json`. |
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
- **Humans approve, agents merge.** The agent never approves its own work and never merges without a teammate's
  approval, usually the shaper's. A teammate can also merge a pull request themselves.
- `Session N of M started` / `ended` comments on the bet act as a lock. When the appetite is spent before the bet pull
  request is open, the agent posts a `CIRCUIT BREAKER` comment instead.

## Trackers and code hosts

The skills speak only in ShapeUp terms. What a cycle, bet, slice or pull request is in a particular tool is defined
by an adapter:

| Kind | Supported | Contract |
|---|---|---|
| Tracker | Azure DevOps Boards (`ado`), GitHub Issues (`github`) | [references/tracker-contract.md](references/tracker-contract.md) |
| Code host | Azure Repos (`ado`), GitHub (`github`) | [references/code-host-contract.md](references/code-host-contract.md) |

The tracker and code host are chosen separately, so for example Azure DevOps Boards with code on GitHub works, and
a Jira tracker would too once its adapter exists. To add one, write an adapter file that meets the contract and add it to
[references/config.md](references/config.md).

## Set up a repository

1. Install the plugin (see the marketplace README).
2. In the repository, run `/shapeup:init`. It asks for the tracker and code host, sets up the connection, verifies
   access and writes `.claude/shapeup.json`.
3. Agree on a Definition of done. `/shapeup:init` offers to shape one with you from the repository's own build and
   test commands, and writes it to the root `CLAUDE.md`. The building agent holds every slice to it.
4. Commit `.claude/shapeup.json`, and `.mcp.json` if it wrote one.

The agent acts as the signed-in user and can never do more than they can.

### On GitHub

GitHub doesn't let anyone approve their own pull request. The building agent can act as its own GitHub account or as
the operator's:

- **The operator's account:** teammates approve its pull requests with a normal review, or the operator merges them
  themselves. Rulesets that allow only merge commits into `bet/**` and only squash into the main branch keep a manual
  merge from picking the wrong type.
- **Its own account:** a machine account with write access to the repository. Anyone, the operator included, can
  approve its pull requests with a normal review.

To give the agent its own account:

1. Install the GitHub CLI and sign in with **your own** account: `gh auth login`.
2. Sign in the agent's account too, then switch back to yours:

   ```bash
   gh auth login                       # log in as the agent's account
   gh auth switch --user <your login>
   ```

   gh keeps both tokens. Your account stays the active one for everything you do; the building agent picks the agent's
   token for its own commands only, through the plugin's `as-github-agent` wrapper. Its commits are authored by the
   agent's account and pushed with its token. Your git identity and credentials aren't touched.
3. Use an HTTPS remote (or an HTTPS push URL), so pushes can use the agent's token.

Either way, the repository needs issues turned on, both merge commits and squash merges allowed, *Automatically delete
head branches* turned off, and at least one open milestone as the first cycle. `/shapeup:init` asks which account the
agent acts as, checks the rest, and creates the labels the skills use. Shaping always runs as the operator, so bets are
created by the product owner.
