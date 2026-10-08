<!-- methods:start -->
## Methods

This repo keeps a git-tracked log of safe, repeatable procedures at `./methods.md` — how
to fetch a real-time answer or get something done here. It is the **counterpart to the
durable learnings log**: memory stores a conclusion someone already worked out, methods
store the procedure that gets *today's* answer.

- **Check `./methods.md`** before investigating something from scratch, and before
  answering a question about current state. The procedure may already be written down.
- **Append to `./methods.md`** when you work out a procedure that took real effort and
  that someone will need again. Use the entry format at the top of that file, write it
  while it's fresh, and scan for an existing entry on the same question first — revise or
  mark `SUPERSEDED:` rather than stacking a near-duplicate.
- **Route it correctly** — the split is *conclusion vs. procedure*, not durable vs.
  volatile. Ask: *if I wrote the answer down, would it be wrong in a month?* If **yes**,
  it belongs in `./methods.md`, and storing the answer anywhere is harmful — a later
  session reads the stale fact and trusts it. If **no**, it's a durable learning and the
  learnings log is usually right, *unless* the procedure itself is the valuable part and
  working it out took real effort — a reusable way to fetch or do something earns a place
  here even when its output is stable. When both files could hold it, **split it**:
  conclusion in the learnings log, procedure here, cross-linked. Never copy the conclusion
  into `./methods.md` — that is the staleness the file exists to prevent.

**Only record a method that is safe to replay unattended.** A recorded method is not a
command judged once in context — a future agent will run it months from now without
re-deciding whether it's a good idea. So:

- **Read-only unless it genuinely cannot be.** If a read-only path exists, that path *is*
  the method. A mutating method must state its blast radius and how to undo it.
- **Least privilege** — name the narrowest role or scope that works, never the broadest.
- **No secret values.** Say where a credential lives, never what it is, and never record a
  step that prints one to a terminal, a log, or CI output.
- **Use the paved path.** If there's a pipeline, PR flow, or approval gate for this, the
  method points at it rather than around it.
- **Never record** a procedure that weakens a control to get an answer (disabling cert
  verification, widening a policy, suspending a guardrail, `chmod 777`), bypasses review or
  audit (force-push to a protected branch, `--no-verify`, disabling logging), runs unpinned
  remote code (`curl … | sh`), exfiltrates data, or works around a missing permission
  instead of fixing it where it's owned.

This gate is about *how* you get an answer, not *whether* you look: reading live state —
querying a cluster, describing a resource, listing what's deployed — is the point of the
file. **If the only procedure you found is inadmissible, still record the question** with
`Method: none safe yet`, what you tried, and what would make a safe one possible — so the
next agent doesn't reach for the path you rejected.

**Using a tool that follows the `AGENTS.md` convention instead of `CLAUDE.md`** (e.g.
Cursor, Aider)? Add this block to `AGENTS.md` instead of, or in addition to, `CLAUDE.md` —
the rules are identical either way.

> Read the file on demand (as above) rather than importing it with `@methods.md`: imports
> load in full at every session start and grow your baseline context as the log grows. The
> gate above is the only part that needs to be always loaded, and it already is.
<!-- methods:end -->
