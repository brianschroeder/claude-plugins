# agent-methods

A repo-level *methods* pattern for AI coding agents, and the **counterpart to
agent memory**: where memory stores durable learnings, methods store safe,
repeatable procedures for fetching real-time answers and other useful
information.

## The problem

Agent memory solves half of the cold-start problem. It stores **what is
true** — root causes, dead ends, conventions — and that works well right up
until the fact expires.

Plenty of what you need to know about a live system is volatile: which
versions are running, what the current quota is, who holds an approval, what
exists in the account right now. Writing those down as facts is worse than
not writing them down at all, because the next session reads a stale fact and
trusts it.

But the *investigation* that produced the answer is stable, and it is usually
the expensive part — the obvious command that doesn't work, the right
invocation nobody could guess, the three places you have to cross-reference.
That gets thrown away every session and re-derived from scratch.

## The pattern

Two files, working together:

1. **`methods.md`** at the repo root — a curated log of safe, repeatable
   procedures: how to find something out or get something done here.
2. **A block in `CLAUDE.md`** (or `AGENTS.md`) instructing agents to *check*
   `methods.md` before investigating from scratch, to *append* a method when
   they work one out, and to *refuse to record* one that only works by doing
   something unsafe.

`CLAUDE.md`/`AGENTS.md` holds the always-loaded behavior; `methods.md` holds
the accumulated content, read on demand.

## Conclusion vs. procedure

The two files are a pair, and the split between them is **conclusion vs.
procedure** — not durable vs. volatile:

| | [`agent-memory.md`](../simple-agent-memory/) | `methods.md` |
|---|---|---|
| Stores | **durable learnings** — a conclusion you read and act on | **procedures** — something you run to get a current answer |
| Holds | root causes, dead ends, conventions, quirks | real-time lookups, live state, reusable how-tos |
| Fails by | going stale and being believed anyway | never — a procedure outlives the answer it returns |

Memory hands you an answer someone already worked out. Methods hand you the
way to get *today's* answer.

Two questions route an entry. The sharp one:

> **If I wrote the answer down, would it be wrong in a month?**

**Yes** → it belongs here, and storing the answer anywhere is harmful.
**No** → it's a durable learning, and the learnings log is usually right.
Usually, because of the second question:

> **Is the procedure itself the valuable part, and did working it out take
> real effort?**

A reusable way to fetch something or get something done earns a place here
even when what it returns is fairly stable — the value is the procedure, not
the output. One obvious command is still not a method.

When both files could hold it, **split it**: conclusion in the learnings log,
procedure here, cross-linked. Never copy the conclusion into `methods.md` —
that's the staleness the file exists to prevent.

## Safety is part of the pattern, not a disclaimer

A recorded method is not a command you judge once in context. It is a
procedure a future agent will run **unattended, months from now, without
re-deciding whether it's a good idea.** Anything written down becomes
standard practice by default.

So the skill ships an admission gate, and the gate lives in the
always-loaded `CLAUDE.md` block — not only in `methods.md` — because an agent
deciding whether to *write* a method may never have read the file it's
writing into. In short: read-only unless it genuinely can't be; least
privilege; no secret values and no step that prints one; go through the paved
path; and never record a procedure that weakens a control, bypasses review or
audit, runs unpinned remote code, exfiltrates data, or works around a missing
permission instead of fixing it where it's owned.

The gate is about *how* you get an answer, not *whether* you look — reading
live state is the entire point of the file.

When the only procedure you found is inadmissible, the method is recorded as
**`Method: none safe yet`**, with what was tried and what would make a safe
one possible. A negative entry stops the next agent reaching for the path you
already rejected.

Because `methods.md` is committed, the gate doubles as a review checklist: a
method arriving in a pull request gets read by a human before it becomes
something an agent replays.

## Install (Claude Code plugin)

```
/plugin marketplace add brianschroeder/claude-plugins
/plugin install agent-methods@bschroeder-plugins
```

Then, in any repo, ask Claude Code to set up methods (e.g. "set up methods
for this repo") and the skill will:
- create `methods.md` at the repo root, if it doesn't already exist
- append the instruction block to `CLAUDE.md` (or `AGENTS.md`), if it isn't
  already there

Both steps are non-destructive — existing content is never overwritten.

After setup, the skill is also what you invoke to add, revise, or retire a
single method in a repo that already has the file.

## Manual install (no Claude Code plugins)

1. Copy [`skills/methods/assets/methods.template.md`](skills/methods/assets/methods.template.md)
   to `methods.md` at your repo root.
2. Copy the block from
   [`skills/methods/assets/claude-md-block.md`](skills/methods/assets/claude-md-block.md)
   into your `CLAUDE.md` or `AGENTS.md`.

Works with any agent that reads `CLAUDE.md`/`AGENTS.md` at session start —
not Claude Code-specific.

## The family

Three files, three questions:

| Pattern | File | Answers |
|---|---|---|
| [simple-agent-memory](../simple-agent-memory/) | `agent-memory.md` | *What is true here?* |
| [agent-references](../agent-references/) | `references.md` | *Where does the context live?* |
| **agent-methods** (this plugin) | `methods.md` | *How do I find out, right now?* |

They compose. If a repo has more than one, cross-link them so a reader who
lands in the wrong file gets redirected rather than stuck.

## Why on-demand, not `@import`

Don't import `methods.md` (e.g. `@methods.md`) into `CLAUDE.md`. Imports load
the whole file at every session start, so the baseline cost grows as the log
grows. The gate in the `CLAUDE.md` block is the only part that needs to be
always loaded, and it already is.

## License

MIT. See [LICENSE](../../LICENSE).
