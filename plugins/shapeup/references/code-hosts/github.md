# Code host: GitHub

Meets [../code-host-contract.md](../code-host-contract.md). The repository is the GitHub repository that is this repo's
git remote `origin`; below, `<o>/<r>` is its owner and name.

## Configuration

```json
"codeHost": { "type": "github", "approval": "review" }
```

| Field | Required | Meaning |
|---|---|---|
| `approval` | Yes | How the shaper approves a pull request: `review` or `comment`. See "Approval". |

`shapeup:init` recognizes GitHub from a remote like `https://github.com/<owner>/<repo>.git` or
`git@github.com:<owner>/<repo>.git`.

**Choosing the approval mode.** The agent opens pull requests as the account `gh` is signed in as, and GitHub never
lets the author of a pull request approve it.

- **`review`** (recommended): the agent works under its own GitHub account (a machine account with write access to the
  repository), and the shaper approves with a normal review. Choose this whenever the agent's account isn't a shaper's.
- **`comment`**: for when the agent has to use a shaper's own account, for example a solo project. The shaper approves
  by commenting `/approve`. This relies on the agent never posting that comment itself, which the building skill
  forbids, but GitHub can't enforce it.

## Connection

The GitHub CLI, `gh`, as in the GitHub tracker adapter ([../trackers/github.md](../trackers/github.md)): installed,
signed in with `gh auth login`, and `gh auth setup-git` run so `git push` works without prompting. Pass
`--repo <o>/<r>` to every `gh pr` command, and pass bodies with `--body-file`.

## Pull requests

- **Open:** `gh pr create --repo <o>/<r> --base <target> --head <branch> --title "<title>" --body-file <file>`, with
  `--draft` for a draft.
- **Update:** `gh pr edit <n> --repo <o>/<r> --title ... --body-file <file>`.
- **Publish a draft:** `gh pr ready <n> --repo <o>/<r>`.
- **Link to an issue:** `Part of #<issue>` in the description, never a closing keyword (see the tracker adapter).
- **Read:** `gh pr view <n> --repo <o>/<r> --json number,state,isDraft,mergeable,headRefName,headRefOid,reviews,comments,commits`.
  Review threads and whether they are resolved need GraphQL:

  ```
  gh api graphql -F owner=<o> -F name=<r> -F number=<n> -f query='
    query($owner:String!,$name:String!,$number:Int!){repository(owner:$owner,name:$name){pullRequest(number:$number){
      reviewThreads(first:100){nodes{isResolved comments(first:50){nodes{databaseId author{login} body}}}}}}}'
  ```

- **Reply in a review thread:** `gh api -X POST repos/<o>/<r>/pulls/<n>/comments/<comment databaseId>/replies -F body=@<file>`,
  replying to the thread's first comment. Reply to general pull request comments with `gh pr comment`.

**Fallback:** drafts need a public repository or a paid plan. If `gh pr create --draft` is refused, open a normal one
titled `WIP: ...`.

## Approval

Read the reviews and comments with `gh pr view --json reviews,comments,commits`. Only the shaper counts, as defined by
the tracker; anything by another account, including the agent's own, is ignored.

**`approval: review`**

- Approved when the shaper's most recent review with state `APPROVED` or `CHANGES_REQUESTED` is `APPROVED`. Reviews with
  state `COMMENTED` or `DISMISSED` don't count either way.
- An `APPROVED` review with a body or line comments is approval with suggestions: apply or answer them first.
- If the repository's branch protection dismisses stale approvals, a push turns an approval into `DISMISSED`: wait for a
  new one.

**`approval: comment`**

- Approved when the shaper has a pull request comment whose first line is exactly `/approve`, posted after the newest
  commit on the pull request, and has posted no review with state `CHANGES_REQUESTED` and no unresolved review thread
  since. A push after the comment cancels it.
- Any other text after `/approve` on the same comment is a suggestion: apply or answer it first.
- Never post `/approve`, or anything starting with it, yourself.

## Merging

Merge through the API, not `gh pr merge`, which changes branches in the local checkout:

| Pull request | Command | Then |
|---|---|---|
| Slice | `gh api -X PUT repos/<o>/<r>/pulls/<n>/merge -f merge_method=merge` | `git push origin --delete <slice branch>` |
| Bet | `gh api -X PUT repos/<o>/<r>/pulls/<n>/merge -f merge_method=squash` | Keep the bet branch |

Pass `-f sha=<headRefOid>` from the last read so the merge fails if the branch moved since. If the repository doesn't
allow a merge method, the call is refused: don't use another method.

## Verification

1. `gh api repos/<o>/<r> --jq '{visibility, allow_merge_commit, allow_squash_merge, delete_branch_on_merge}'`:
   - `allow_merge_commit` and `allow_squash_merge` must both be true. If not, a human must allow them in the
     repository settings under *Pull Requests*.
   - `delete_branch_on_merge` must be false, or GitHub deletes the bet branch when the bet ships. A human turns off
     *Automatically delete head branches*.
   - If `visibility` is `private`, warn that draft pull requests need a paid plan, and that the `WIP:` fallback is used
     otherwise.
2. With `approval: review`: compare `gh api user --jq .login` with the people likely to shape bets (ask the operator).
   If the agent's account is one of them, warn that they can't approve the agent's pull requests and suggest
   `comment` or a separate account.
3. `git ls-remote origin` works without prompting.
