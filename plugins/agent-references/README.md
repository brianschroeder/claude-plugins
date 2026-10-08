# agent-references

A repo-level references index for AI coding agents. Stop re-deriving *where things live*
every session.

Companion to [simple-agent-memory](../simple-agent-memory/):
that one remembers what you learned, this one remembers where to look.

## The problem

Most of what you need to work in a repo isn't in the repo. The VPC is defined in a sibling
clone. The permissions are in a third one. The reason the schema is shaped that way is in a
design doc nobody links to. Every fresh session — human or agent — goes looking for the same
things, in the same order, and finds them in the same twenty minutes.

Meanwhile the README lists the repos by name, which tells you nothing about which file in
them actually matters.

## The pattern

Two files, working together:

1. **`references.md`** at the repo root — a curated, git-tracked map of where the surrounding
   context lives: sibling repos *and the specific files in them*, upstream modules, design
   docs, dashboards, runbooks, tickets, vendor documentation.
2. **A block in `CLAUDE.md`** (or `AGENTS.md`) telling agents to read it before going looking
   for something, and to fix or extend it as they go.

`CLAUDE.md`/`AGENTS.md` holds the always-loaded behavior (when to read, when to update);
`references.md` holds the content, read on demand — so the index can grow without growing
every session's baseline context cost.

## The one rule that makes it work

**Every entry says what you would go there to find out.**

A link with no annotation leaves the reader to open it and discover that for themselves,
which is exactly the work the file exists to save. So not this:

```markdown
- `../platform-infra` — the infrastructure repo.
```

but this:

```markdown
- **`platform-infra` → `environments/prod/iam.tf`** — the pipeline role and its state prefix.
  Read the prefix here before setting a backend key in this repo; the role's S3 grant covers
  that prefix and nothing else, and a mismatch only fails on the first apply.
```

The second one saves a session. The first one costs one.

Two corollaries: point at **files** rather than repos when you know which file matters, and
group entries under headings that match **the question being asked** ("Where the permissions
live") rather than the kind of thing listed ("Repositories") — that's how a reader actually
arrives at the file.

## Only reference things that persist

The index is committed, so it gets read by people who don't have your machine, and by you in
six months. An entry that no longer resolves is worse than a missing one — it's a promise the
file can't keep, and the reader burns their time finding that out. Three ways it happens:

**Scratch.** Agent and tool working directories — `.superpowers/`, `docs/superpowers/`,
`.remember/`, build output — are gitignored *because* they're disposable, so they won't exist
in a fresh clone. `git check-ignore -v <path>` is the one-command test before you write a path
down.

**Machine-local paths.** `../other-repo/some/file` only resolves for readers who lay their
checkouts out the way you do, and a file in a parent folder that isn't itself a repo is
versioned nowhere at all. Name repos canonically — repo plus repo-relative path, or a URL — and
offer the local path as a convenience, not as the identity of the thing.

**Documents with an expiry date.** Change proposals, migration plans, and cutover runbooks
describe the past the moment the change lands. Split them along the same seam as location and
lessons: index the repo and file the change *acts on*, and put the "why it was broken" in the
learnings log. Both halves outlive the document.

## Install (Claude Code plugin)

```
/plugin marketplace add brianschroeder/claude-plugins
/plugin install agent-references@bschroeder-plugins
```

Then, in any repo, ask Claude Code to set up references (e.g. "set up a references file for
this repo") and the skill will:
- create `references.md` at the repo root, if it doesn't already exist, and seed it with real
  entries from what the repo actually depends on
- append the instruction block to `CLAUDE.md` (or `AGENTS.md`), if it isn't already there

Both steps are non-destructive — existing content is never overwritten.

## Manual install (no Claude Code plugins)

Copy the two files by hand:

1. Copy [`skills/references/assets/references.template.md`](skills/references/assets/references.template.md)
   to `references.md` at your repo root, then replace the placeholder headings with ones
   matching the questions your repo actually raises.
2. Copy the block from
   [`skills/references/assets/claude-md-block.md`](skills/references/assets/claude-md-block.md)
   into your `CLAUDE.md` or `AGENTS.md`.

Works with any agent that reads `CLAUDE.md`/`AGENTS.md` at session start — not Claude
Code-specific.

## Location vs. lessons

If you run this alongside a learnings log, keep the boundary clean — a file that tries to be
both becomes neither:

| | Answers | Shape |
|---|---|---|
| `references.md` | *Where is it?* | Stable, mostly additive, grouped by question |
| `agent-memory.md` | *What did we learn the hard way?* | Dated, newest-first, grouped by incident |

A path belongs in references. A root cause belongs in memory. When something is both, put the
pointer in references with a one-line warning and the full story in memory.

## Drift is the failure mode

Absence isn't the risk — a stale index that people trust is. So the maintenance rules are part
of the pattern, not an afterthought: confirm a reference still exists before recommending it,
fix drift in passing rather than working around it, delete dead entries outright, and when
something has moved, keep both locations for as long as the old one still does anything. A
half-migrated dependency is exactly what this file should be warning you about.

Two things never go in: secrets (name where a credential lives, never its value) and guessed
URLs (an admitted gap beats a link that gets trusted once and wastes an afternoon).

## Why on-demand, not `@import`

Don't switch to importing `references.md` (e.g. `@references.md`) into `CLAUDE.md` unless the
index is small and always relevant. Imports load the whole file at every session start, so the
baseline cost grows as the index grows. Reading it on demand keeps sessions lean and only pays
the cost when it's actually needed.

## License

MIT. See [LICENSE](../../LICENSE).
