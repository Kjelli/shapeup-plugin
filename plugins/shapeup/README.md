# ShapeUp

Run [Shape Up](https://basecamp.com/shapeup) with Claude agents. Product owners shape the pitches and decide what
ships. Agents help them shape, and build the bets autonomously, one session at a time.

| Skill | Run it with | Does |
|---|---|---|
| `shapeup:init` | `/shapeup:init` | Sets up a repository: tracker, code host, their connection, Definition of done, `.claude/shapeup.json`. |
| `shapeup:shape` | `Help me shape a pitch`, or `/shapeup:shape <title>` | Helps a product owner shape a pitch and posts it as a bet. |
| `shapeup:build` | `Continue with bet #<id>`, `Continue with <bet title>`, or `/shapeup:build <id or name>` | Builds a bet, one session at a time. |

Installation, setup, the proposed flow and troubleshooting are in the [repository README](../../README.md).

## Inside the plugin

| Path | What it holds |
|---|---|
| `skills/` | The three skills, written only in Shape Up terms. |
| `references/config.md` | The `.claude/shapeup.json` format and the supported adapters. |
| `references/tracker-contract.md`, `references/code-host-contract.md` | What an adapter for a tracker or code host must define. |
| `references/trackers/`, `references/code-hosts/` | The adapters: `ado` (Azure DevOps) and `github`. |
| `scripts/as-github-agent` | Runs one command as the agent's own GitHub account, without touching the user's gh or git setup. |
