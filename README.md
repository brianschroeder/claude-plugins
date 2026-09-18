# claude-plugins

My personal marketplace of [Claude Code](https://code.claude.com) plugins.

A Claude Code *marketplace* is just a git repo with a
`.claude-plugin/marketplace.json` file that indexes one or more plugins. Anyone
with Claude Code can add this marketplace and install plugins from it — no
website or hosting involved.

## Add this marketplace

```
/plugin marketplace add bschroeder/claude-plugins
```

(Replace `bschroeder` with the account this repo lives under if different.)

> Note: the *repo* is `claude-plugins`, but the *marketplace name* (used after
> the `@` when installing) is `bschroeder-plugins` — Claude Code reserves
> `claude-*` marketplace names for official use.

## Install a plugin

Once this marketplace lists plugins, install one with:

```
/plugin install <plugin-name>@bschroeder-plugins
```

## Available plugins

- **tldr** — Appends a standardized TLDR block (summary, key points, next
  steps, outstanding) to the end of every Claude response. Run its
  `tldr-setup` skill once to install the instruction into your global
  `~/.claude/CLAUDE.md`.

- **comment-review** — Reviews the comments your current branch added vs.
  `main` and flags ones that don't earn their place (redundant, narration,
  commented-out code, over-documented trivia, changelog noise), then offers to
  remove them. Run its `comment-review` skill (`/comment-review`) on a branch
  before opening a PR.

### The repo-context family

Three companion patterns, each maintained in its own repo and indexed here.
They install a git-tracked file at a repo's root plus a matching block in
`CLAUDE.md`/`AGENTS.md`, so the context survives fresh agent sessions. Each
answers a different question, and they compose:

| Plugin | Installs | Answers |
|---|---|---|
| [**simple-agent-memory**](https://github.com/brianschroeder/simple-agent-memory) | `agent-memory.md` | *What is true here?* — root causes, dead ends, conventions, quirks |
| [**agent-references**](https://github.com/brianschroeder/agent-references) | `references.md` | *Where does the context live?* — a map of the surrounding material |
| [**agent-methods**](https://github.com/brianschroeder/agent-methods) | `methods.md` | *How do I find out, right now?* — safe, repeatable procedures for fetching real-time answers |

`agent-methods` is the newest of the three, and the direct counterpart to
`simple-agent-memory`: memory stores a durable conclusion someone already
worked out, methods store the procedure that gets *today's* answer. The split
is conclusion vs. procedure. It matters because a fact about a live system
expires, and a stale entry still gets read and trusted — so where the answer
won't keep, the derivation is what gets recorded, behind an admission gate
that keeps unsafe procedures from being written down and replayed unattended.

See [CONTRIBUTING.md](./CONTRIBUTING.md) to add another.

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
└─ plugins/                # bundled plugins live here (one dir per plugin)
```

Plugins listed here are either **bundled** (a directory under `plugins/`, named
by a bare `source` that resolves under `metadata.pluginRoot`) or **external** (a
`source` object pointing at another GitHub repo). `pluginRoot` applies only to
bare names — it is ignored for a `source` that already starts with `./`, and it
does not apply to external sources at all.

## Adding your own plugin

See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Validate

Before committing changes, validate the marketplace and its plugins:

```
claude plugin validate .
```
