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

## Adding your own plugin

See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Validate

Before committing changes, validate the marketplace and its plugins:

```
claude plugin validate .
```
