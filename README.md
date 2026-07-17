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

```
/plugin install hello-world@bschroeder-plugins
```

Then try it:

```
/hello
```

## Available plugins

| Plugin | Description |
| ------ | ----------- |
| `hello-world` | Minimal example plugin that adds a `/hello` command. Use it as a template. |

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
└─ plugins/
   └─ hello-world/         # an example bundled plugin
      ├─ .claude-plugin/
      │  └─ plugin.json
      └─ commands/
         └─ hello.md
```

## Adding your own plugin

See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Validate

Before committing changes, validate the marketplace and its plugins:

```
claude plugin validate .
```
