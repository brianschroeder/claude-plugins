---
name: methods
description: >-
  Set up repo-level methods, the counterpart to agent memory: create a `methods.md` log at
  the repository root and wire up `CLAUDE.md` so that where memory stores durable
  learnings, agents record and reuse safe, repeatable procedures for fetching real-time
  answers and other useful information. Use this whenever the user wants to add, set up,
  bootstrap, install, enable, or repair "methods," a "how-to log," a procedures file, a
  recipes or playbook file, or wants agents to stop re-deriving the same investigation
  from scratch, or to stop trusting a cached answer about live state that has since gone
  stale — even if they don't say the word "skill." Also use it to re-apply the pattern to
  a new repo, and to add, revise, or retire a single method in a repo that already has
  `methods.md`.
---

# Methods (repo-level)

This skill installs a git-tracked pattern into a repository so that the *way* you answer a
recurring question survives across fresh agent sessions — even when the answer itself does
not. It sets up two things:

1. **`methods.md`** at the repo root — a curated log of safe, repeatable procedures for
   finding something out or getting something done in this repo.
2. **A block in `CLAUDE.md`** instructing agents to *check* `methods.md` before
   investigating something from scratch, to *append* a method when they work one out, and
   to *refuse to record* one that only works by doing something unsafe.

As with the sibling patterns, `CLAUDE.md` holds the always-loaded behavior and `methods.md`
holds the accumulated content, read on demand.

## The counterpart to agent memory

These two are a pair, and the pairing is the point:

| | `agent-memory.md` | `methods.md` |
|---|---|---|
| Stores | **durable learnings** — a conclusion you read and act on | **procedures** — something you run to get a current answer |
| Holds | root causes, dead ends, conventions, quirks | real-time lookups, live state, reusable how-tos |
| Fails by | going stale and being believed anyway | never — a procedure outlives the answer it returns |

The split is **conclusion vs. procedure**, not durable vs. volatile. Memory hands you an
answer someone already worked out. Methods hand you the way to get *today's* answer.

Two questions route an entry. The first is the sharp one:

> **If I wrote the answer down, would it be wrong in a month?**

If **yes**, it belongs here, and storing the answer anywhere is actively harmful — a later
session reads the stale fact and trusts it. Which version is running, who currently holds
an approval, what the live quota is, what exists in the account right now: written down as
facts these become confident wrong answers, and written down as methods they stay correct.

If **no**, it is a durable learning and `agent-memory.md` is usually the right home — but
not always, which is what the second question is for:

> **Is the procedure itself the valuable part, and did working it out take real effort?**

A reusable way to fetch something or get something done earns a place here even when what
it returns is fairly stable. The value is the procedure, not the output. But one obvious
command is not a method: record the ones where the obvious approach fails, where the right
invocation is non-obvious, where the answer needs cross-referencing several places, or
where getting it wrong is expensive.

**When both files could hold it, split it rather than choosing.** Put the durable
conclusion in `agent-memory.md`, the procedure here, and cross-link the two. Do not copy
the conclusion into this file — that is precisely the staleness this file exists to
prevent.

**Methods are not runbooks.** A runbook is written for a human under pressure during an
incident. A method is written for an agent establishing current state, on an ordinary day,
with no pager and no one watching. If a repo has both, link them rather than merging them.

## Safe by construction

This is the part of the skill that is load-bearing, and it is why the rules also appear in
the always-loaded `CLAUDE.md` block rather than only in `methods.md`: an agent deciding
whether to *write* a method may never have read the file it is writing into.

A recorded method is not a command you run once and judge in context. It is a procedure
that a future agent will run **unattended, months from now, without re-deciding whether it
is a good idea.** A shortcut written down here becomes standard practice by default. So the
bar for admission is higher than the bar for doing something once, by hand, while watching.

**A method may only be recorded if all of these hold:**

- **Read-only unless it genuinely cannot be.** If a read-only path to the answer exists,
  that path *is* the method. A method that changes state must say so in its `Mutates:`
  field, state its blast radius, and say how to undo it.
- **Least privilege.** Name the narrowest role, profile, or scope that actually works.
  Never "use the admin profile, it's easier."
- **No secret values.** Say *where* a credential lives ("the `app-config` Secret", "the
  read-only SSO profile"), never what it is. No step may print a credential, token, or
  key to a terminal, a log, or CI output.
- **It goes through the paved path.** If the repo has a pipeline, a PR flow, or an approval
  gate for this, the method points at it. A local or console procedure that sidesteps a
  pipeline is not a method, it is an incident waiting to be cited as precedent.
- **It is verifiable.** The method says how you know the answer is right. A procedure that
  silently returns a plausible wrong answer is worse than none.

**Never record a method that:**

- **Weakens a control to get an answer** — disabling TLS or certificate verification,
  widening a security group or bucket policy, attaching a broad managed policy, suspending
  a guardrail, SCP, or admission policy, `chmod 777`, or turning off a required check.
- **Bypasses review or audit** — force-pushing a protected branch, `--no-verify`, merging
  past CODEOWNERS, editing state directly to dodge a plan, or disabling logging.
- **Executes unpinned remote code** — `curl … | sh` against an unpinned source. Pin to a
  tag or digest and verify it.
- **Exfiltrates data** to somewhere it does not belong, including pasting live data into an
  external service to "just check something."
- **Works around a missing permission or a broken upstream.** The workaround is the wrong
  artifact. Record the method for *confirming the gap*, and note that the fix is a change
  in the repo that owns it.

**What the gate is not.** It does not prohibit touching production, reading live
infrastructure, or using real credentials through the normal path. Reading current state —
querying a cluster, describing a resource, listing what is deployed — is the entire point
of the pattern. The gate is about *how* you get the read, not *whether* you read.

**When the only procedure you found is inadmissible**, still record the question. Write the
entry with **`Method: none safe yet`**, say what you tried, say what would make a safe one
possible, and stop. A negative entry is worth writing: it preserves the finding and it
stops the next agent reaching for the unsafe path you already rejected.

Because `methods.md` is committed, this gate is also a review checklist. A method arriving
in a pull request gets read by a human before it becomes something an agent will replay.

## Setting it up

Do this directly with your own file tools — read the bundled templates below, then create
or edit the files in the target repo. No script is needed. Work at the **repository root**
(confirm with `git rev-parse --show-toplevel` if git is available, otherwise use the
project's top directory).

Both steps are deliberately **non-destructive — check before you write:**

1. **Create `methods.md` at the repo root — only if it's missing.**
   - If `./methods.md` already exists, leave it untouched; it holds accumulated methods.
   - If it does not exist, create it with the exact contents of
     `assets/methods.template.md`.

2. **Add the instruction block to `CLAUDE.md` — exactly once.**
   - Read `./CLAUDE.md` (create it if it doesn't exist).
   - If it already contains the marker `<!-- methods:start -->`, leave it as-is.
   - Otherwise, append the full contents of `assets/claude-md-block.md`, preceded by a
     blank line if the file already has content.

Then report what changed and remind the user to commit both files.

## Adding a method to a repo that already has one

This is the common case after setup, and it is worth doing deliberately:

1. **Check for an existing entry on the same question** before writing a new one. If one
   exists and is still right, leave it. If it has drifted, revise it in place. If it is
   now wrong, mark it `SUPERSEDED:` and put the corrected method above it.
2. **Apply the admission gate above.** If the procedure fails it, record the question with
   `Method: none safe yet` instead of recording the procedure.
3. **Write it while it is fresh**, in the entry format at the top of `methods.template.md`,
   newest at the top.
4. **Confirm the volatility test still applies.** If the answer turns out to be durable
   after all, it belongs in `agent-memory.md`, not here.

## Customization notes

- **Different filename:** if the user prefers `PROCEDURES.md` or `RECIPES.md`, rename the
  file and update the paths inside the `CLAUDE.md` block to match. Keep the `methods:start`
  / `methods:end` markers as-is so the block stays idempotent.
- **`AGENTS.md` instead of `CLAUDE.md`:** the block works unchanged in either; add it to
  whichever file the user's tooling reads, or both.
- **Monorepos:** apply per package. A subdirectory `CLAUDE.md` loads on demand when an
  agent works there, so a package-local `methods.md` plus a package-local block scopes
  methods to that package.
- **On-demand vs. always-loaded:** do not switch to an `@methods.md` import. Imports load
  the whole file at every session start; the gate in the `CLAUDE.md` block is the only part
  that needs to be always loaded, and it already is.
- **Alongside the sibling patterns:** this composes with
  [`simple-agent-memory`](https://github.com/brianschroeder/claude-plugins/tree/main/plugins/simple-agent-memory) (durable
  facts) and [`agent-references`](https://github.com/brianschroeder/claude-plugins/tree/main/plugins/agent-references)
  (where context lives). Three files, three questions: *what is true*, *where is it*, *how
  do I find out, right now*. If a repo has more than one, cross-link them so a reader who lands in the
  wrong one is redirected rather than stuck.

## Bundled resources

- `assets/methods.template.md` — the starter log, including the volatility test, the
  admission gate, and the entry format. Copy it verbatim to create the file.
- `assets/claude-md-block.md` — the marker-wrapped block to append to `CLAUDE.md`.
