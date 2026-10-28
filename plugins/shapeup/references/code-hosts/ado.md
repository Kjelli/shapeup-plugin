# Code host: Azure Repos

Meets [../code-host-contract.md](../code-host-contract.md). The repository is the Azure Repos repository that is this
repo's git remote `origin`.

## Configuration

```json
"codeHost": { "type": "ado" }
```

No other fields. `shapeup:init` recognizes Azure Repos from a remote like
`https://dev.azure.com/<organization>/<project>/_git/<repo>` or
`https://<organization>.visualstudio.com/<project>/_git/<repo>`.

## Connection

The same `ado` MCP server as the Azure DevOps tracker adapter, with the `repositories` domain enabled (see
[../trackers/ado.md](../trackers/ado.md)). If the tracker isn't Azure DevOps, add that `.mcp.json` entry anyway, with
`-d core repositories`.

`git push` to `origin` must work without prompting, for example with Git Credential Manager.

## Pull requests

Use the `ado` repository tools (named like `mcp__ado__repo_...`).

- **Open:** source and target branch, title, description, and `isDraft` true for a draft.
- **Update / publish:** update the pull request; publishing a draft sets `isDraft` to false.
- **Link to a work item:** link the work item when creating or updating the pull request, so it shows under the pull
  request's work items and as a *Pull Request* link on the work item.
- **Read:** reviewers with their votes, threads with their status, and merge status (conflicts).
- **Reply:** add a comment to the existing thread; don't open a new thread for a reply.

**Fallback:** if the tools can't create or update draft pull requests, open a normal one titled `WIP: ...`.

## Approval

Each reviewer has a vote: `10` *Approved*, `5` *Approved with suggestions*, `0` no vote, `-5` *Waiting for author*,
`-10` *Rejected*.

- Approved when at least one reviewer votes `10` or `5` and no reviewer votes `-5` or `-10`. Any teammate's vote
  counts; the shaper usually reviews. The agent never votes, so every vote is a person's, even one cast from the
  account the agent works under.
- With `5`, the suggestions are in that reviewer's threads: apply or answer them first.
- **Merged by a person:** the pull request's status is `completed` and you didn't complete it. That counts as approval.
- The repository's branch policies may reset votes when new commits are pushed. Read the vote again after pushing.

## Merging

Complete the pull request with these completion options:

| Pull request | Merge strategy | Delete source branch |
|---|---|---|
| Slice | `noFastForward` (merge commit) | Yes |
| Bet | `squash` | No |

If the tools can't set the merge strategy, don't complete it with another one.

## Definition of done

Suggest only when the repository has a pipeline (`azure-pipelines.yml`, or a build validation policy on the target
branch):

- **The pull request's checks pass.** Build validation and every other branch policy on the pull request succeed
  before it is published or merged. A failing build is fixed, never queued again until it passes by luck.

## Verification

1. The `ado` repository tools can read the repository behind `origin`.
2. `git ls-remote origin` works without prompting.
