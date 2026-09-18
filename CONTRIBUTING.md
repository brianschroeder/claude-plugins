# Adding a plugin

A plugin can either be **bundled in this repo** (under `plugins/`) or **hosted
in another repo** and merely referenced. Both work.

## Option A — bundle the plugin here

1. Create the plugin directory under `plugins/`, following this layout:

   ```
   plugins/my-plugin/
   ├─ .claude-plugin/
   │  └─ plugin.json        # name, description, version, author, license
   ├─ commands/             # slash commands (*.md), optional
   ├─ agents/               # subagents (*.md), optional
   ├─ skills/               # skills (<name>/SKILL.md), optional
   ├─ hooks/hooks.json      # hooks, optional
   └─ .mcp.json             # MCP servers, optional
   ```

   Only `plugin.json` goes inside `.claude-plugin/`; every component directory
   sits at the plugin root. Component directories are auto-discovered, so you
   generally don't need to list them in `plugin.json`.

2. Add an entry to `.claude-plugin/marketplace.json` under `plugins`. Because
   `metadata.pluginRoot` is `./plugins`, the `source` is just the directory
   name:

   ```json
   {
     "name": "my-plugin",
     "source": "./my-plugin",
     "description": "What it does.",
     "version": "0.1.0",
     "author": { "name": "Brian Schroeder" },
     "license": "MIT",
     "keywords": ["..."],
     "category": "..."
   }
   ```

## Option B — reference a plugin in another repo

Add an entry whose `source` points at the external repo instead of a local
path:

```json
{
  "name": "my-plugin",
  "source": { "source": "github", "repo": "bschroeder/my-plugin-repo" },
  "description": "What it does."
}
```

Other supported `source` forms: a raw git `url`, `git-subdir` (a subdirectory
of a monorepo), and `npm`. See the
[plugin marketplace docs](https://code.claude.com/docs/en/plugin-marketplaces).

## Naming rules

- `name` (marketplace and plugin) must be **kebab-case** — lowercase letters,
  digits, and hyphens only, no spaces.
- Plugin names must be unique within this marketplace.

## Before you commit

```
claude plugin validate .
```

This checks `marketplace.json` and each plugin's manifest, frontmatter, and
hook syntax for errors.
