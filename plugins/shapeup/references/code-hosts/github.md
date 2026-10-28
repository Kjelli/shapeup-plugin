# Code host: GitHub

Meets [../code-host-contract.md](../code-host-contract.md). The repository is the GitHub repository that is this repo's
git remote `origin`; below, `<o>/<r>` is its owner and name.

## Configuration

```json
"codeHost": { "type": "github", "account": "myapp-bot" }
```

| Field | Required | Meaning |
|---|---|---|
| `account` | No | The GitHub login the building agent acts as, when it has its own account. See "Acting as". When the tracker is also GitHub, it is the same as `tracker.account`. Without it, the agent acts as the operator's active gh account. |

`shapeup:init` recognizes GitHub from a remote like `https://github.com/<owner>/<repo>.git` or
`git@github.com:<owner>/<repo>.git`.

**Asking who the agent acts as.** `gh auth status --hostname github.com` lists every account gh has a token for and
marks the active one, which is the operator's. Ask "Which GitHub account should the building agent act as?",
offering:

- **Each other stored account** by login, described as: "A separate account for the agent. Anyone on the team,
  including you, can approve its pull requests with a normal review." Sets `account`.
- **The operator's own account** by login, described as: "The agent works as you. GitHub won't let you review pull
  requests opened under your own account, so a teammate approves them, or you merge them yourself." No `account`.
- **"An agent account that isn't signed in yet"**: tell the operator to run `gh auth login` as that account, then
  `gh auth switch --user <their own login>`, and to run `/shapeup:init` again. Stop there.

## Connection

The GitHub CLI, `gh`, as in the GitHub tracker adapter ([../trackers/github.md](../trackers/github.md)): installed,
signed in with `gh auth login`, and `gh auth setup-git` run so `git push` works without prompting. Pass
`--repo <o>/<r>` to every `gh pr` command, and pass bodies with `--body-file`.

## Acting as

With `account` set, the building agent puts the plugin's wrapper before every `gh` command **and every `git commit`,
`git push`, `git fetch` and `git ls-remote`**: `bash <scripts>/as-github-agent <account> git push ...`. For that one
command the wrapper makes gh use the agent's token, makes git authenticate to `https://github.com` with it (other hosts
keep the user's own credentials), and makes the agent the commit's author and committer, with its GitHub noreply
address. Nothing is changed in the user's gh or git configuration.

The wrapper only works with an HTTPS remote. With an SSH remote (`git@github.com:...`), `git push` would use the user's
own SSH key.

Without `account`, everything runs plainly as the active gh account and the user's git identity.

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

Read the reviews with `gh pr view <n> --repo <o>/<r> --json state,reviews,mergedAt,mergedBy,commits`. Ignore reviews by
the account the agent acts as: the agent never reviews, and GitHub doesn't let that account approve anyway.

- **Approved** when, taking each reviewer's most recent review with state `APPROVED` or `CHANGES_REQUESTED`, at least
  one is `APPROVED` and none is `CHANGES_REQUESTED`. Reviews with state `COMMENTED` or `DISMISSED` don't count either
  way. Any teammate's review counts; the shaper usually reviews.
- An `APPROVED` review with a body or line comments is approval with suggestions: apply or answer them first.
- If the repository's branch protection dismisses stale approvals, a push turns an approval into `DISMISSED`: wait for a
  new one.
- **Merged by a person:** `state` is `MERGED` and you didn't merge it. That counts as approval; handle it as the
  building skill describes.
- **The agent works under the operator's account** (no `account`): the operator can't review these pull requests.
  Teammates review them as usual, or the operator merges them. When you open or publish a pull request in this setup,
  end its description with: "Opened under your own account, so GitHub won't let you review it: ask a teammate to
  approve it, or merge it yourself (merge commit for a slice, squash for the bet)."

## Merging

Merge through the API, not `gh pr merge`, which changes branches in the local checkout:

| Pull request | Command | Then |
|---|---|---|
| Slice | `gh api -X PUT repos/<o>/<r>/pulls/<n>/merge -f merge_method=merge` | `git push origin --delete <slice branch>` |
| Bet | `gh api -X PUT repos/<o>/<r>/pulls/<n>/merge -f merge_method=squash` | Keep the bet branch |

Pass `-f sha=<headRefOid>` from the last read so the merge fails if the branch moved since. If the repository doesn't
allow a merge method, the call is refused: don't use another method.

## Definition of done

Suggest only when the repository has workflows in `.github/workflows/` that run on pull requests:

- **The pull request's checks pass.** Every GitHub Actions check on the pull request is green before it is published
  or merged. Read them with `gh pr checks <n> --repo <o>/<r>`. A failing check is fixed, never re-run until it passes
  by luck.

## One-time setup

None.

## Verification

1. `gh api repos/<o>/<r> --jq '{visibility, allow_merge_commit, allow_squash_merge, delete_branch_on_merge}'`:
   - `allow_merge_commit` and `allow_squash_merge` must both be true. If not, a human must allow them in the
     repository settings under *Pull Requests*.
   - `delete_branch_on_merge` must be false, or GitHub deletes the bet branch when the bet ships. A human turns off
     *Automatically delete head branches*.
   - If `visibility` is `private`, warn that draft pull requests need a paid plan, and that the `WIP:` fallback is used
     otherwise.
2. Recommend, without requiring, rulesets that offer only the right merge type, so a person merging by hand can't
   pick the wrong one: `bet/**` branches allow only merge commits, and the main branch allows only squash. Check with
   `gh api repos/<o>/<r>/rulesets` and each ruleset's `pull_request` rule (`allowed_merge_methods`). If they are
   missing, say how to add them under *Settings > Rules > Rulesets*. Changing settings is for the operator.
3. With `account` set: `origin` is an HTTPS URL; the wrapper's `gh api repos/<o>/<r> --jq .permissions.push` is
   `true`; and `bash <scripts>/as-github-agent <account> git ls-remote origin` works without prompting. If `origin` is
   SSH, suggest switching it to HTTPS, or keeping SSH for fetching and setting an HTTPS push URL:
   `git remote set-url --push origin https://github.com/<o>/<r>.git`. The operator decides.
4. Without `account`: `git ls-remote origin` works without prompting.
