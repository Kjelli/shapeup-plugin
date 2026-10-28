# Code host contract

The building skill talks about branches, pull requests, approval and merging in general terms. A code host adapter in
[code-hosts/](code-hosts/) says exactly how each is done on one host and with which tools. Git itself (branches,
worktrees, commits, merges, pushes to `origin`) is the same everywhere and is described in the building skill.

## What every code host adapter defines

Use these headings, in this order.

### Configuration

The `codeHost.*` fields in `.claude/shapeup.json`, if any, and how `shapeup:init` recognizes the host from the git
remote `origin`.

### Connection

What the agent uses to reach the host, if it isn't the same as the tracker's: an MCP server (with the `.mcp.json`
entry `shapeup:init` can write) or a command-line tool, and how the user signs in. Also what `git push` to `origin`
needs to work without prompting.

### Pull requests

How to:

- Open a pull request from a branch to a target branch, as a **draft** or ready for review.
- Update its title and description, and mark a draft ready for review.
- Link it to a tracker item, in both directions where the host and tracker allow.
- Read its reviewers' verdicts, review threads and whether it has conflicts.
- Reply in a review thread.

And the **fallback** when drafts aren't supported.

### Approval

Exactly which reviewer verdicts count as the shaper's approval, which don't, whether new commits reset it, and how
to recognize suggestions that come with an approval. If the host offers more than one way to approve (for example
because a host forbids approving your own pull request), the adapter lists them as modes and the configuration picks
one.

### Merging

How to complete a pull request with:

- a **merge commit**, deleting the source branch (slices), and
- a **squash merge**, keeping the source branch (the bet).

And what to do if the tools can't do one of these.

### Verification

The read-only checks `shapeup:init` runs: the repository is reachable, both merge types are allowed, and anything
else the rules above depend on, such as draft support or settings that would delete the bet branch on merge.
