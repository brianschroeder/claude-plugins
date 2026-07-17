# Design: `claude-plugins` — native Claude Code plugin marketplace

**Date:** 2026-07-17
**Owner:** Brian Schroeder
**Status:** Approved

## Goal

A public git repo on Brian's personal GitHub account that functions as a
native Claude Code plugin marketplace. Others add it with
`/plugin marketplace add <account>/claude-plugins` and install plugins with
`/plugin install <name>@claude-plugins`. No web app, no hosting — the repo
*is* the marketplace.

## Non-goals (YAGNI)

- No web UI / backend / database (that is a separate, much larger project).
- No CI/GitHub Actions.
- No real plugins yet beyond one working example. This is an empty scaffold
  Brian fills in later.

## Repository layout

```
claude-plugins/
├─ .claude-plugin/
│  └─ marketplace.json        # the plugin index
├─ plugins/
│  └─ hello-world/            # one working example / template plugin
│     ├─ .claude-plugin/
│     │  └─ plugin.json
│     └─ commands/
│        └─ hello.md          # a /hello slash command
├─ docs/superpowers/specs/    # this design doc
├─ README.md                  # what it is + add/install/validate instructions
├─ CONTRIBUTING.md            # how to add a new plugin (bundled or referenced)
├─ LICENSE                    # MIT
└─ .gitignore
```

## Schema decisions (verified against Claude Code v2.1.196+ docs)

`marketplace.json` required fields: `name` (kebab-case), `owner.name`,
`plugins[]`. Note: the marketplace `name` is `bschroeder-plugins`, **not**
`claude-plugins` — the validator reserves `claude-*` names as official
Anthropic marketplaces. The GitHub *repo* is still named `claude-plugins`; only
the internal marketplace identifier (what follows `@` in `/plugin install`)
differs. We also set `$schema`, `description`, `version`, and
`metadata.pluginRoot: "./plugins"` so plugin `source` values can be bare
directory names. Each plugin entry uses `name` + `source` (required) plus
`description`, `version`, `author`, `license`, `keywords`, `category`.

`plugin.json` required field: `name`. The example also sets `displayName`,
`description`, `version`, `author`, `license`, `keywords`. Component
directories (`commands/`, `agents/`, `skills/`, `hooks/`, `.mcp.json`) live at
the plugin root and are auto-discovered.

Sources: docs.claude.com plugin-marketplaces, plugins-reference, plugins,
discover-plugins.

## The example plugin

`hello-world` is a genuine, installable plugin — a single `/hello` slash
command — so the marketplace is non-empty and passes `claude plugin validate`.
It doubles as the template referenced by CONTRIBUTING.md.

## Delivery plan

1. Scaffold files locally in `/workspace/code/claude-plugins`. ✔
2. `git init`, commit.
3. Install the `gh` CLI in this environment.
4. Brian runs `! gh auth login` interactively (this selects the personal
   account — deliberately **not** the `NBA` org that `vpm` uses).
5. `gh repo create claude-plugins --public --source=. --remote=origin --push`.
6. Verify the repo exists and the push succeeded.

## Verification

- Structural JSON validity of both manifests (`python -m json.tool` / parser).
- `claude plugin validate .` if the `claude` CLI is available in this
  environment; otherwise Brian validates from his own Claude Code after push.
