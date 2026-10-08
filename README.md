# claude-plugins

My personal marketplace of [Claude Code](https://code.claude.com) plugins.

A Claude Code *marketplace* is a git repo with a `.claude-plugin/marketplace.json` file that
indexes one or more plugins. Anyone with Claude Code can add this marketplace and install
plugins from it. No website or hosting is involved.

## Add this marketplace

```
/plugin marketplace add brianschroeder/claude-plugins
```

> Note: the *repo* is `claude-plugins`, but the *marketplace name* (used after the `@` when
> installing) is `bschroeder-plugins`. Claude Code reserves `claude-*` marketplace names for
> official use.

## Install a plugin

```
/plugin install <plugin-name>@bschroeder-plugins
```

## Available plugins

- **legible**: A documentation standard based on about 80% of ASD-STE100 Simplified
  Technical English, after a
  [post by Andrej Karpathy](https://x.com/karpathy/status/2105819303471976479). Run its
  `legible-setup` skill in a repo to add the standard to `CLAUDE.md` and `AGENTS.md`. Every
  agent that writes runbooks, setup guides and READMEs there then follows it. An optional
  `legible:legible` output style applies the same sentence rules to your chat replies. See
  [plugins/legible](./plugins/legible/README.md).

### The repo-context family

Three companion patterns. Each one installs a git-tracked file at a repo's root plus a matching
block in `CLAUDE.md`/`AGENTS.md`, so the context survives fresh agent sessions. Each answers a
different question, and they compose:

| Plugin | Installs | Answers |
|---|---|---|
| [**simple-agent-memory**](./plugins/simple-agent-memory/) | `agent-memory.md` | *What is true here?* Root causes, dead ends, conventions, quirks. |
| [**agent-references**](./plugins/agent-references/) | `references.md` | *Where does the context live?* A map of the surrounding material. |
| [**agent-methods**](./plugins/agent-methods/) | `methods.md` | *How do I find out, right now?* Safe, repeatable procedures for fetching real-time answers. |

`agent-methods` is the direct counterpart to `simple-agent-memory`. Memory stores a durable
conclusion someone already worked out. Methods store the procedure that gets *today's* answer.
The split matters because a fact about a live system expires, and a stale entry still gets read
and trusted. So where the answer will not keep, the derivation is what gets recorded, behind an
admission gate that keeps unsafe procedures from being written down and replayed unattended.

All three used to live in their own repos. They are now bundled here, and this repo is their
only home.

See [CONTRIBUTING.md](./CONTRIBUTING.md) to add another plugin.

## Managing the marketplace

```
/plugin marketplace list                       # see added marketplaces
/plugin marketplace update bschroeder-plugins  # pull the latest plugin list
/plugin marketplace remove bschroeder-plugins  # remove it
```

## Repository layout

```
claude-plugins/
├─ .claude-plugin/
│  └─ marketplace.json     # the plugin index
└─ plugins/                # bundled plugins, one directory per plugin
```

Each plugin is a directory under `plugins/`, named in `marketplace.json` by a bare `source` that
resolves under `metadata.pluginRoot`. `pluginRoot` applies only to bare names. It is ignored for
a `source` that already starts with `./`.

## Validate

Before committing changes, validate the marketplace and its plugins:

```
claude plugin validate .
```
