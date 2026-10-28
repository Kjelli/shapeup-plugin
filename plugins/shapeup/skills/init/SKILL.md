---
name: init
description: Set up or check a repository for ShapeUp with this plugin - choose the tracker and code host, connect them, verify access and write .claude/shapeup.json. Use when someone asks to set up, initialize, configure or check ShapeUp in a repository, or when another ShapeUp skill says the configuration or connection is missing.
---

# Setting up ShapeUp in a repository

You set up one repository so the `shapeup:shape` and `shapeup:build` skills can run in it. You write configuration
only: `.claude/shapeup.json`, the repository's `.mcp.json` when an adapter uses an MCP server, what the tracker
adapter's "One-time setup" section lists, and, if asked, a section in `CLAUDE.md`. You never create, change or
comment on items in the tracker, never create cycles, and never create branches or pull requests.

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
   where the adapter says how. If an adapter offers modes (for example how the shaper approves pull requests), explain
   them and ask which one. If an adapter supports an agent account ("Acting as"), ask whether the agent has its own
   account and which login; it is the same for both adapters when they use the same service. The plugin's scripts
   folder is `${CLAUDE_PLUGIN_ROOT}/scripts`. Then ask about `mainBranch` (suggest the remote's default branch) and
   about
   `worktree.symlink` and `worktree.neverLink`: files or folders the app needs to run that aren't in git, and local
   state a worktree must never touch. Look at `.gitignore` to make suggestions, but let the operator decide.
5. **Connection.** Check the connection each adapter's "Connection" section describes.
   - **Works:** go on.
   - **An MCP server is missing:** show the `.mcp.json` entries the adapters give, with the fields filled in, and offer
     to add them to the repository's `.mcp.json`. Merge them in: keep every other server, and never replace an
     existing entry with the same name without asking. Then tell the operator to restart the session, approve the
     server and sign in, and to run `/shapeup:init` again to finish. Stop here. If the configuration file isn't
     written yet, write it first (step 8) so the next run only has to verify.
   - **A command-line tool is missing or not signed in:** say exactly what to install or run, as the adapter says, and
     to run `/shapeup:init` again afterwards. Never sign in on the operator's behalf. Stop here, writing the
     configuration first as above.
6. **Verify.** Run each adapter's "Verification" checks, read-only. Report what you found: the scope, the open cycles
   by ID and title, any warnings the adapters raise, and any check that failed with what the operator can do about it.
   Settings and cycles are for humans to change, not you.
7. **One-time setup.** If the tracker adapter's "One-time setup" lists anything that is missing, show exactly what
   you would create and create it after an explicit yes.
8. **Write the configuration.** Show the full `.claude/shapeup.json` and write it after an explicit yes.
9. **Definition of done.** `shapeup:build` holds every slice to the Definition of done in the repository's root
   `CLAUDE.md`.
   - **None yet:** offer to shape one together now, following
     [references/definition-of-done.md](references/definition-of-done.md). It draws on the repository's own build and
     test commands and on the code host adapter. If the operator declines, say the building skill can't run well
     without one, and move on.
   - **One exists:** show it and ask whether to review it the same way. Change only what the operator agrees to.
   Never write a Definition of done the operator hasn't read and confirmed.
10. **Wrap up.** List the files you wrote, suggest committing them so everyone who clones the repository gets the
    same setup, and say how to start: `Help me shape a pitch`, or `Continue with bet #<id>`.

Never put bet numbers, names or the state of a cycle in any file.
