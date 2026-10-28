# Project configuration

Each repository that runs ShapeUp with this plugin has a `.claude/shapeup.json` at its root, checked into git. The
`shapeup:init` skill writes it. It holds only *where* things live, never bet numbers, names or the state of a cycle:
those are always read fresh from the tracker.

```json
{
  "tracker": {
    "type": "ado",
    "organization": "contoso",
    "project": "Product",
    "areaPath": "Product\\ShapeUp",
    "iterationPath": "Product\\ShapeUp"
  },
  "codeHost": {
    "type": "ado"
  },
  "mainBranch": "main",
  "worktree": {
    "symlink": ["models"],
    "neverLink": ["data"]
  }
}
```

| Key | Required | Meaning |
|---|---|---|
| `tracker.type` | Yes | Which tracker adapter to use: a file name in [trackers/](trackers/) without `.md`. |
| `tracker.*` | Per adapter | The fields that adapter's "Configuration" section asks for. |
| `codeHost.type` | Yes | Which code host adapter to use: a file name in [code-hosts/](code-hosts/) without `.md`. The repository is always this repo's git remote `origin`. |
| `codeHost.*` | Per adapter | The fields that adapter's "Configuration" section asks for. |
| `mainBranch` | No | The branch bets ship to. Defaults to `main`. |
| `worktree.symlink` | No | Paths, relative to the repository root, to symlink from the operator's checkout into a session's worktree because they are needed to run the app but aren't in git. Defaults to none. |
| `worktree.neverLink` | No | Paths never to link or copy into a worktree, for example local data the app would read and overwrite. Defaults to none. |

## Supported adapters

| Kind | Types |
|---|---|
| Tracker | `ado` (Azure DevOps Boards), `github` (GitHub Issues) |
| Code host | `ado` (Azure Repos), `github` (GitHub) |

Tracker and code host are chosen separately, for example Azure DevOps Boards with code on GitHub.

Adding a backend means writing one adapter file that meets [tracker-contract.md](tracker-contract.md) or
[code-host-contract.md](code-host-contract.md), and adding it to this table. The skills don't change.

## Reading the configuration

Every skill starts by reading `.claude/shapeup.json` from the repository root.

- **Missing:** tell the operator to run `/shapeup:init` and stop.
- **Unknown `tracker.type` or `codeHost.type`:** say which types are supported (table above) and stop.
- **A field the adapter requires is missing:** name it, tell the operator to run `/shapeup:init`, and stop.

Never guess a missing value and never write the file yourself outside `shapeup:init`.
