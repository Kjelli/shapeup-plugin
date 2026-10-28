---
name: init
description: Set up or check a repository for ShapeUp with this plugin - choose the tracker and code host, connect their MCP server, verify access and write .claude/shapeup.json. Use when someone asks to set up, initialize, configure or check ShapeUp in a repository, or when another ShapeUp skill says the configuration or connection is missing.
---

# Setting up ShapeUp in a repository

You set up one repository so the `shapeup:shape` and `shapeup:build` skills can run in it. You write configuration
only: `.claude/shapeup.json`, the repository's `.mcp.json` and, if asked, a section in `CLAUDE.md`. You never create,
change or comment on items in the tracker, and never create branches or pull requests.

The configuration format and the supported adapters are in
[config.md](${CLAUDE_PLUGIN_ROOT}/references/config.md). Each adapter is in
`${CLAUDE_PLUGIN_ROOT}/references/trackers/` or `${CLAUDE_PLUGIN_ROOT}/references/code-hosts/`.

## Steps

Show what you found and ask before every file you write. Go one step at a time.

1. **Check what is there.** Read `.claude/shapeup.json`, `.mcp.json` and the git remote `origin`. If a configuration
   exists, show it and ask whether to check it (go to step 5) or change it.
2. **Tracker.** Ask which tracker holds the cycles and bets, listing the supported types from config.md. If the answer
   isn't supported yet, say so, name the contract an adapter would have to meet
   ([tracker-contract.md](${CLAUDE_PLUGIN_ROOT}/references/tracker-contract.md)), and stop.
3. **Code host.** Recognize it from `origin` using each code host adapter's "Configuration" section, and ask the
   operator to confirm. If it isn't supported yet, say so as in step 2 and stop.
4. **Fields.** Read both adapters' "Configuration" sections and ask for each field, suggesting values from `origin`
   where the adapter says how. Then ask about `mainBranch` (suggest the remote's default branch) and about
   `worktree.symlink` and `worktree.neverLink`: files or folders the app needs to run that aren't in git, and local
   state a worktree must never touch. Look at `.gitignore` to make suggestions, but let the operator decide.
5. **Connection.** Check whether the tools each adapter's "Connection" section names are available in this session.
   - **Available:** go on.
   - **Not available:** show the `.mcp.json` entries the adapters give, with the fields filled in, and offer to add
     them to the repository's `.mcp.json`. Merge them in: keep every other server, and never replace an existing
     entry with the same name without asking. Then tell the operator to restart the session, approve the server and
     sign in, and to run `/shapeup:init` again to finish. Stop here. If the configuration file isn't written yet, write
     it first (step 7) so the next run only has to verify.
6. **Verify.** Run each adapter's "Verification" calls, read-only. Report what you found: the scope, the open cycles
   by ID and title, and any check that failed with what the operator can do about it. A failed check isn't fixed by
   changing the tracker: that is for humans.
7. **Write the configuration.** Show the full `.claude/shapeup.json` and write it after an explicit yes.
8. **Definition of done.** `shapeup:build` expects the Definition of done in the repository's root `CLAUDE.md`. If
   there isn't one, say so and offer to add a short ShapeUp section that names the skills and leaves a Definition of
   done for the team to fill in. Never write one on the team's behalf.
9. **Wrap up.** List the files you wrote, suggest committing them so everyone who clones the repository gets the same
   setup, and say how to start: `Help me shape a pitch`, or `Continue with bet #<id>`.

Never put bet numbers, names or the state of a cycle in any file.
