# Shaping a Definition of done

`shapeup:build` holds every slice to the Definition of done in the repository's root `CLAUDE.md`. It is the team's
standard, so the operator decides every line: you suggest, they accept, change or drop. Walk through it the way
`shapeup:shape` walks through a pitch: one part at a time, a short draft for each, and on to the next only when they are
happy.

## Gather first

Before suggesting anything, read what the repository already says about how it is built and checked:

- **Build and test commands:** `package.json` scripts, `*.sln` / `*.csproj`, `pyproject.toml` / `setup.cfg`,
  `go.mod`, `Cargo.toml`, `pom.xml` / `build.gradle`, `Makefile`, `justfile`, `Taskfile.yml`.
- **Formatting and linting:** `.editorconfig`, `.prettierrc*`, `eslint.config.*` / `.eslintrc*`, `ruff.toml`, `.golangci.yml`,
  `Directory.Build.props` (analyzers, `TreatWarningsAsErrors`), pre-commit hooks.
- **Continuous integration:** `.github/workflows/`, `azure-pipelines.yml`, `.gitlab-ci.yml`.
- **Conventions:** existing `CLAUDE.md` files, `CONTRIBUTING.md`, `README.md`.

Use only commands you found. Never invent one: if there is no linter, say so and let the operator decide whether that
belongs in the Definition of done.

## The parts

1. **Conventions.** Where the code's conventions live (for example folder-level `CLAUDE.md` files, `CONTRIBUTING.md`)
   and that a change follows them. If nothing is written down, suggest "match the style of the code around you".
2. **The checks pass.** The exact commands for build, formatting, linting and tests that you found, run locally, with
   no new warnings. Suggest the rule that a check is never disabled, a warning never suppressed and a test never
   skipped to make it pass; if a check seems wrong, the agent says so instead.
3. **The pull request's checks pass.** Take this from the code host adapter's "Definition of done" section, and only
   when the repository has the CI it depends on.
4. **Logic is tested.** Anything with rules or edge cases has automated tests, including the edge cases the agent
   decided on. Ask whether there are kinds of tests the team expects (end-to-end, integration against a real
   database, UI tests) and with which commands.
5. **It is easy to change.** Logic separated from I/O and UI, names from the problem's domain, one job per function
   and class, nothing duplicated that should be shared.
6. **It is clean.** No dead code, commented-out code, debug output or leftover experiments. No `TODO` without an item
   reference, written the way the tracker adapter's "Commit references" section says (for example `TODO(#12)`).
7. **Reviewed first.** The agent reads its own diff as a reviewer would, against this list, and fixes what it finds.
8. **Anything else.** Ask what the team cares about that isn't covered yet: documentation, accessibility,
   performance budgets, security checks, changelog entries. Suggest nothing here the repository doesn't already
   point to.

Close with the sentence that the Definition of done is the minimum standard, not gold-plating: when time gets tight,
cut scope, never the standard.

## Writing it

Show the whole section, then add it to the root `CLAUDE.md` after an explicit yes, under the heading
`## Definition of done`, opening with "Code you hand over will be maintained by others. Cutting corners is not the same
as cutting scope. Every change meets this standard before it is offered for review:". Keep the rest of `CLAUDE.md`
as it is. Keep each item to one or two lines: the agent reads it in every session.

## Example

```markdown
## Definition of done

Code you hand over will be maintained by others. Cutting corners is not the same as cutting scope. Every change
meets this standard before it is offered for review:

- **It follows the conventions for its area.** Each project folder under `src/` has a `CLAUDE.md`; read it first.
- **The checks pass.** `dotnet build`, `dotnet format --verify-no-changes`, `npm run lint` and `dotnet test` all pass
  locally, with no new warnings. Never disable a check, suppress a warning or skip a test to make it pass. If a check
  seems wrong, say so instead.
- **The pull request's checks pass.**
- **Logic is tested.** Anything with rules or edge cases has automated tests, including the edge cases you decided on.
- **It is easy to change.** Logic is separated from I/O and UI, names come from the problem's domain, functions and
  classes do one thing, and nothing is duplicated that should be shared.
- **It is clean.** No dead code, commented-out code, debug output or leftover experiments. No `TODO` without an issue
  reference next to it.
- **You reviewed it first.** Read your own diff as a reviewer would, against this list, and fix what you find.

This is the minimum standard, not gold-plating. When time gets tight, cut scope, never this standard.
```
