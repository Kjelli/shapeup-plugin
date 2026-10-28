# ShapeUp plugin for Claude Code

A plugin marketplace for Claude Code. It holds one plugin so far:

| Plugin | What it does |
|---|---|
| [shapeup](plugins/shapeup/) | Run ShapeUp with agents: shape pitches with a product owner and build bets autonomously, one session at a time. |

## Install

```bash
claude plugin marketplace add kjelli/shapeup-plugin
claude plugin install shapeup@shapeup-plugin
```

Or, inside a session, `/plugin marketplace add kjelli/shapeup-plugin` and `/plugin install shapeup@shapeup-plugin`.

To have everyone who clones a project offered the plugin, commit this to the project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "shapeup-plugin": {
      "source": { "source": "github", "repo": "kjelli/shapeup-plugin" }
    }
  },
  "enabledPlugins": {
    "shapeup@shapeup-plugin": true
  }
}
```

## Develop

Validate after every change, and bump `version` in the plugin's `plugin.json` when you release, or installed copies
won't update:

```bash
claude plugin validate .
claude plugin validate ./plugins/shapeup
```

To try local changes in a project without publishing, add this folder as a marketplace
(`claude plugin marketplace add ../shapeup-plugin`); edits then load at the next session start or with
`/reload-plugins`.

## License

[MIT](LICENSE)
