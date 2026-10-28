# Branches, pull requests and merging

How each pull request step is done on this host (tools, draft flags, votes, merge options) is in the code host
adapter. The rules below are the same on every host.

## Branches

- All of the bet's work comes together in the bet branch, `bet/<bet-id>`.
- Each slice has its own branch, `slice/<slice-id>-<short-name>`, branched from the latest bet branch (or from
  another slice's branch, see "Dependencies").
- Slice pull requests target the bet branch. The main branch only receives the finished bet, through the bet pull
  request.

## Worktree

Each session works in one git worktree for the bet, never in the operator's checkout. The worktree lives next to the
operator's checkout and is named after it: `../<checkout-folder>-bet-<bet-id>`, for example
`../myapp-bet-2004`. Switch between the bet's branches inside it.

**At the start of a session**, before touching any branch, run `git worktree prune` and then `git worktree list`. Any
worktree on `bet/<bet-id>` or on a branch of one of the bet's slices belongs to the bet, wherever it is and whatever it
is called. It is left over from an earlier session that didn't run its end steps. For each one:

1. Check it for work that isn't on `origin`: uncommitted changes, untracked files that aren't symlinks, commits not
   pushed and stashes made in it.
2. **Nothing found:** remove it with `git worktree remove --force <path>`. `--force` only drops symlinks and files git
   ignores; check step 1 first.
3. **Work found:** don't remove it and don't discard anything. Tell the operator what is there and ask whether to keep
   it (commit and push it to its branch, then treat it as unreviewed work in this session) or discard it.

Then create the worktree at the path above from the latest `origin`.

**To run the app** in the worktree, symlink each path in the configuration's `worktree.symlink` from the operator's
checkout, because those aren't in git. Never link or copy a path in `worktree.neverLink`: the app would then read and
overwrite the operator's own local state.

**At the end of a session**, after every branch is pushed, remove the worktree with `git worktree remove --force
<path>`, after the same check as at the start. If there is still work in it that isn't on `origin`, don't remove it:
say so in the session summary.

## One pull request per slice

Each slice gets exactly one pull request, linked to its item. Later sessions push more commits to the same branch and
update the same pull request; never open a second one for the same slice.

Every session that works on a slice ends with its pull request existing and up to date:

- **Work remains:** the pull request is a **draft** and the slice stays `Active`. The description lists what is done
  and what remains. A draft is never `Resolved`.
- **Complete:** working, integrated, meeting the Definition of done, with nothing left you intend to do. Bring the
  branch up to date, make sure it merges cleanly, publish the pull request (or mark the draft ready for review) and
  set the slice to `Resolved`. The description says what it does, how to try it, and lists the decision comments it
  relies on.

If review comments send a slice back, move it from `Resolved` to `Active` while you fix them, then back to `Resolved`.

**Fallback:** if the tools can't create or update draft pull requests, use the code host adapter's fallback, keep the
slice `Active`, and post a decision comment noting the fallback.

## Dependencies

If a slice relies on another slice whose pull request isn't approved yet, don't wait. Branch the dependent slice from
the other slice's branch, say so in both descriptions ("Depends on #2001" / "Blocks #2003"), and add a one-line comment
on the dependent slice. Never merge a dependent pull request before the one it depends on. Once that one is merged,
bring the bet branch into the dependent branch.

## Work in progress

At most two slices may have unmerged pull requests at a time, drafts included. At the limit, finish or update those
instead of starting a new slice. If both are waiting for review with nothing to do on them, end the session.

## Keeping branches current

Other bets are merged into the main branch while you work, so drift happens at two levels.

- At the start of every session, merge the latest main branch into the bet branch through a pull request if it has
  fallen behind, or say in the session summary why you didn't.
- At the start of every session, and again right before publishing a pull request, merge the latest bet branch into
  each `Active` slice's branch. Resolve conflicts, run the checks, and push.
- Merge, never rebase, and never force-push.
- If a conflict is about behavior rather than text, don't pick silently: resolve it and post a decision comment
  explaining the choice. If it conflicts with another bet's work, raise a question.

## Merging

You complete pull requests, but only approved ones.

- **Approved** means approved by the shaper, exactly as the code host adapter's "Approval" section defines it. Any
  other verdict, no verdict, or a verdict by anyone else is not approval. If the approval comes with suggestions,
  apply or answer them first.
- If pushing new commits resets the approval, wait for the shaper to approve again.
- Before completing, make sure the branch is up to date and the pull request has no conflicts and no unresolved
  threads from the shaper.
- **Slice pull requests:** complete with a regular **merge commit** and delete the source branch. Then set the slice to
  `Closed` with a one-line comment.
- **The bet pull request:** complete with a **squash merge** and keep the bet branch; it preserves the bet's history.
- If the tools can't complete a pull request with the required merge type, don't use another type: raise a question
  and leave the pull request open.
