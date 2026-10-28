# Tracker: GitHub Issues

Meets [../tracker-contract.md](../tracker-contract.md). Below, `<owner>` and `<repo>` are the values from
`.claude/shapeup.json`, and `<o>/<r>` is short for them in API paths.

## Configuration

```json
"tracker": {
  "type": "github",
  "owner": "octocat",
  "repo": "myapp"
}
```

| Field | Required | Meaning |
|---|---|---|
| `owner` | Yes | The user or organization that owns the repository holding the issues. |
| `repo` | Yes | The repository holding the issues. Usually the same repository as the code. |
| `account` | No | The GitHub login of the agent's own account. See "Acting as". |

`shapeup:init` suggests both from a GitHub remote: `https://github.com/<owner>/<repo>.git` or
`git@github.com:<owner>/<repo>.git`.

## Connection

The GitHub CLI, `gh`, run with the Bash tool. It covers everything this adapter needs, including milestones, sub-issues
and repository settings, which the GitHub MCP server's tools don't all reach. No MCP server is needed.

- The user installs `gh` (2.60 or later) and runs `gh auth login` for github.com, then `gh auth setup-git` so
  `git push` works without prompting too.
- The account needs write access to `<owner>/<repo>`.
- The connection works when `gh auth status` succeeds and `gh api repos/<o>/<r> --jq .full_name` prints the
  repository.
- Pass `--repo <owner>/<repo>` to every `gh issue` command, so the command never depends on the current directory.
- Write bodies and comments to a file in a temporary directory and pass it with `--body-file` (or `-F body=@<file>` with
  `gh api`), never inline, so Markdown and quotes survive the shell.

## Acting as

- **`account` set:** the building agent acts as that account and the user's own GitHub setup is never touched. Put the
  plugin's wrapper before every `gh` command: `bash <scripts>/as-github-agent <account> gh ...`, where `<scripts>` is
  the plugin's scripts folder the skill names. The wrapper picks the token gh has stored for `<account>`, whichever
  account is active, for that one command.
- **Shaping** never uses the wrapper: the product owner creates the bet, so it runs plain `gh`, as the active account.
- **`account` not set:** both skills run plain `gh` as the active account.
- The **operator** is always the active account: `gh api user --jq .login` without the wrapper.

## Scope

Issues in `<owner>/<repo>`. A bet is in scope when it has the label `bet` and belongs to a milestone. A slice or
question is in scope when it is a sub-issue, directly or through a slice, of a bet in scope. Items get no extra scope
fields; their place in the tree is what keeps them in scope.

## Item types

| Concept | GitHub | Link to parent or cycle |
|---|---|---|
| **Cycle** | A milestone | – |
| **Bet** | An issue with the label `bet` | Its **milestone** is the cycle |
| **Slice** | An issue with the label `slice` | Sub-issue of the bet |
| **Question** | An issue with the label `question` | Sub-issue of the slice it is about, or of the bet if it is about the bet as a whole |

- The ID is the issue number, written `#12`. Issues and pull requests share one number sequence.
- Web link: `https://github.com/<owner>/<repo>/issues/<number>`.
- **Assignee** is the first of the issue's assignees; **creator** is the issue's author.
- Sub-issue calls take the issue's internal `id`, not its number: `gh api repos/<o>/<r>/issues/<number> --jq .id`.

```
Milestone          = cycle
└─ Issue `bet`     = bet (the pitch is its body)
   ├─ Sub-issue `slice`    = slice
   │  └─ Sub-issue `question` = question about that slice
   └─ Sub-issue `question`    = question about the bet as a whole
```

## States

GitHub issues are only open or closed, so the in-between states are labels. An item has at most one `status:` label.

| Item | Logical state | GitHub |
|---|---|---|
| Cycle | open | Milestone state `open` |
| Cycle | closed | Milestone state `closed` |
| Bet, slice | `New` | Open, no `status:` label |
| Bet, slice | `Active` | Open, label `status:active` |
| Bet, slice | `Resolved` | Open, label `status:resolved` |
| Bet, slice | `Closed` | Closed. Reason *completed* when shipped or merged, *not planned* when cut or dropped |
| Question | open | Open |
| Question | `Closed` | Closed as *completed* |

To change state, add the new `status:` label and remove the old one in the same command:
`gh issue edit <n> --repo <owner>/<repo> --add-label status:active --remove-label status:resolved`. To close:
`gh issue close <n> --repo <owner>/<repo> --reason completed` (or `--reason "not planned"`). To reopen a slice sent back
by review: `gh issue reopen`, then set the label.

**First-session check:** the labels `status:active` and `status:resolved` exist (`gh label list`). If one is missing,
raise a question instead of creating it.

## Tags

Labels. Add and remove with `gh issue edit --add-label` / `--remove-label`, which leaves the item's other labels alone.
Read them from `gh issue view --json labels`. The labels `slice`, `uphill`, `downhill`, `nice-to-have`, `question`,
`bet`, `status:active` and `status:resolved` are created by `shapeup:init`; never create others.

## Comments

Issue comments: `gh issue comment <n> --repo <owner>/<repo> --body-file <file>` to post, and
`gh issue view <n> --repo <owner>/<repo> --json comments` to read them in order with author and time. The first line of
a session comment is matched exactly.

## Descriptions

The issue body, in GitHub Markdown. Keep the pitch's section headings as `##` headings.

## Queries

- **Open cycles:** `gh api "repos/<o>/<r>/milestones?state=open" --jq '.[] | {number, title, due_on}'`.
- **One bet:** `gh issue view <n> --repo <owner>/<repo> --json number,title,body,state,stateReason,labels,assignees,author,milestone,url`.
  Check it is an issue (not a pull request), has the label `bet`, and has a milestone.
- **Bets by title:** `gh issue list --repo <owner>/<repo> --label bet --state open --limit 200 --json number,title,labels,milestone`,
  keeping those with a milestone. Open bets are `New`, `Active` or `Resolved`.
- **Descendants of a bet:** `gh api repos/<o>/<r>/issues/<n>/sub_issues --paginate` lists the direct sub-issues; do the
  same for each slice to find its questions. Then read each item with `gh issue view --json` for labels, state and
  recent comments.

## Changes the skills make

- **Bet:** `gh issue create --repo <owner>/<repo> --title "Bet: <title>" --body-file <file> --label bet --milestone "<milestone title>" --assignee <login>`.
  Run it as the product owner (plain `gh`). Their login is `gh api user --jq .login` unless they name someone else.
- **Slice:** `gh issue create` with `--label slice --label uphill` (or `downhill`), then make it a sub-issue of the bet:
  `gh api -X POST repos/<o>/<r>/issues/<bet>/sub_issues -F sub_issue_id=<slice id>`.
- **Question:** `gh issue create` with `--label question --assignee <shaper>`, then add it as a sub-issue of the slice
  or bet the same way.
- **State:** see "States".
- **Pull request link:** write `Part of #<n>` in the pull request description, which links them both ways in the
  timeline. **Never use closing keywords** (`close`, `closes`, `closed`, `fix`, `fixes`, `fixed`, `resolve`,
  `resolves`, `resolved`, followed by an issue reference) in pull request descriptions, titles or commit messages.
  GitHub would close the issue automatically when the pull request merges, and only the agent closes slices and only
  humans close bets.

## One-time setup

The labels below, created with `gh label create <name> --repo <owner>/<repo> --color <color> --description "<text>"`
when they don't exist:

| Label | Color | Description |
|---|---|---|
| `bet` | `5319e7` | A ShapeUp bet; the body is the pitch |
| `slice` | `1d76db` | A slice of a bet |
| `question` | `d876e3` | A question for the shaper |
| `uphill` | `fbca04` | Still figuring out how |
| `downhill` | `0e8a16` | Only execution left |
| `nice-to-have` | `c5def5` | Cut first when time runs short |
| `status:active` | `0052cc` | Being worked on |
| `status:resolved` | `006b75` | Done, waiting for review or shipping |

## Off-limits

Never create, edit or close milestones, labels (other than the one-time setup), projects, issue types, releases,
repository settings, branch protection or rulesets. Never transfer, lock, pin or delete issues.

## Commit references

`#<n>` in the commit message, for example `Count crossings at one door (#14)`, never with a closing keyword. If the
issues live in a different repository from the code, write `<owner>/<repo>#<n>`.

## Verification

1. `gh auth status` succeeds. Report the active account (`gh api user --jq .login`): that is who shapes. If `account` is
   set, `bash <scripts>/as-github-agent <account> gh api user --jq .login` prints `<account>`, and it differs from the
   active account. If the active account *is* the agent's, the user signed it in last: tell them to run
   `gh auth switch --user <their own login>`.
2. `gh api repos/<o>/<r> --jq '{full_name, has_issues, visibility}'`: the repository is reachable and `has_issues` is
   true.
3. Sub-issues are available: `gh api repos/<o>/<r>/issues/<any issue>/sub_issues` returns a list (an empty one is
   fine). Skip this when the repository has no issues yet.
4. List open cycles. Report how many, by number and title. None is fine, but say a human must create a milestone
   before shaping a bet.
5. List which one-time setup labels are missing.
