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

The MCP server the adapter uses, if it isn't the tracker's: an `.mcp.json` entry `shapeup:init` can write and how the
user signs in. Also what `git push` to `origin` needs to work without prompting.

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
to recognize suggestions that come with an approval.

### Merging

How to complete a pull request with:

- a **merge commit**, deleting the source branch (slices), and
- a **squash merge**, keeping the source branch (the bet).

And what to do if the tools can't do one of these.
