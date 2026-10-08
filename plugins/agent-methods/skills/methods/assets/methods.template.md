# Methods

Safe, repeatable procedures for **this repository** — how to find something out or get
something done here, especially when the answer itself goes stale.

Check this file before investigating something from scratch. Add to it when you work out
a procedure that took real effort and that someone will need again.

This file is committed on purpose: it is shared across every teammate, every machine, and
every agent session, and it is reviewed like any other change. That review is part of the
safety model — see "The admission gate" below.

---

## What belongs here, and what belongs in agent memory

This file is the **counterpart to `agent-memory.md`**, and the pairing is the point:

| | `agent-memory.md` | `methods.md` (this file) |
|---|---|---|
| Stores | **durable learnings** — a conclusion you read and act on | **procedures** — something you run to get a current answer |
| Holds | root causes, dead ends, conventions, quirks | real-time lookups, live state, reusable how-tos |
| Fails by | going stale and being believed anyway | never — a procedure outlives the answer it returns |

The split is **conclusion vs. procedure**, not durable vs. volatile. Memory hands you an
answer someone already worked out. Methods hand you the way to get *today's* answer.

Two questions route an entry. The first is the sharp one:

> **If I wrote the answer down, would it be wrong in a month?**

If **yes**, it belongs here, and storing the answer anywhere is actively harmful — a later
session reads the stale fact and trusts it. Which versions are running, what the live
quota is, who currently holds an approval, what exists right now: written down as facts
these become confident wrong answers, and written down as methods they stay correct.

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

**Methods are not runbooks.** A runbook is for a human under pressure during an incident.
A method is for establishing current state on an ordinary day. If this repo has both,
link them rather than merging them.

---

## The admission gate

A method here is not a command judged once in context. It is a procedure a future agent
will run **unattended, months from now, without re-deciding whether it is a good idea.**
Anything written down becomes standard practice by default, so the bar is higher than for
doing something once by hand while watching.

**Only record a method if all of these hold:**

- **Read-only unless it genuinely cannot be.** If a read-only path to the answer exists,
  that path *is* the method. A mutating method must set `Mutates:`, state its blast
  radius, and say how to undo it.
- **Least privilege.** Name the narrowest role, profile, or scope that actually works.
  Never "use the admin profile, it's easier."
- **No secret values.** Say *where* a credential lives, never what it is. No step may
  print a credential, token, or key to a terminal, a log, or CI output.
- **It goes through the paved path.** If there is a pipeline, a PR flow, or an approval
  gate for this, the method points at it. A procedure that sidesteps one is not a method.
- **It is verifiable.** The method says how you know the answer is right. A procedure that
  silently returns a plausible wrong answer is worse than none.

**Never record a method that:**

- **Weakens a control to get an answer** — disabling TLS or certificate verification,
  widening a security group or bucket policy, attaching a broad managed policy, suspending
  a guardrail, SCP, or admission policy, `chmod 777`, or turning off a required check.
- **Bypasses review or audit** — force-pushing a protected branch, `--no-verify`, merging
  past CODEOWNERS, editing state directly to dodge a plan, or disabling logging.
- **Executes unpinned remote code** — `curl … | sh` against an unpinned source.
- **Exfiltrates data**, including pasting live data into an external service to check it.
- **Works around a missing permission or a broken upstream.** Record the method for
  *confirming the gap*, and note that the fix belongs in the repo that owns it.

**What the gate is not.** It does not prohibit touching production, reading live
infrastructure, or using real credentials through the normal path. Reading current state
is the entire point of this file. The gate is about *how* you get the read, not *whether*.

**If the only procedure you found is inadmissible, still record the question** — with
`Method: none safe yet`, what you tried, and what would make a safe one possible. A
negative entry stops the next person reaching for the path you already rejected.

---

## Entry format

Newest at the top of the Log. Lead with the question the way someone would actually ask it.

```
### <the question, phrased the way someone would actually ask it>
- **Use when:** the trigger — what makes you need this answer
- **Access:** the narrowest role / profile / scope that works
- **Mutates:** no — or: yes, <blast radius>; undo with <…>
- **Method:**
  1. <step>
  2. <step>
- **Verify:** how you know the answer is right
- **Volatile because:** why this isn't just a fact in `agent-memory.md`
- _Added YYYY-MM-DD._
```

**Keep it curated.** Before adding an entry, scan for one on the same question. If it is
still right, leave it. If it has drifted, revise it in place. If it is now wrong, mark it
`SUPERSEDED:` and put the corrected method above it, so the reasoning isn't lost. Merge
near-duplicates rather than stacking them.

**Retire methods that stop being needed.** A method for a system that no longer exists is
worse than nothing — delete it, or mark it `RETIRED:` with a one-line reason.

---

## Log

<!-- Newest entries at the top. Delete the examples below once you have real entries. -->

### (example) Which add-on versions are actually running, vs. what Terraform declares?
- **Use when:** a plan shows an add-on change you didn't expect, or you need to know
  whether the cluster has drifted from the declared versions.
- **Access:** the read-only profile; cluster read via the normal auth path.
- **Mutates:** no.
- **Method:**
  1. Read the declared versions out of the Terraform source (don't trust a past summary).
  2. List what the cluster actually reports for the same add-ons.
  3. Diff the two lists field by field; a version that differs is drift, a version that is
     absent is a resource that was never created.
- **Verify:** re-read one add-on directly by name; if it disagrees with your list, the
  list was scoped to the wrong cluster or region.
- **Volatile because:** add-on versions change on every apply and on some AWS-side
  updates, so a version written down here would be wrong within weeks.
- _Added 2026-01-01. (Placeholder — delete once real entries exist.)_

### (example) How do I confirm a Secret is populated correctly without printing its value?
- **Use when:** a workload is failing auth and you need to rule out an empty or truncated
  secret.
- **Access:** namespace read.
- **Mutates:** no.
- **Method:**
  1. Check that the expected **keys** exist on the object — keys, not values.
  2. Compare the **byte length** of each value against what the provider issues.
  3. If both look right, stop here and move to the consumer: check the pod actually
     mounted the current version rather than a cached one.
- **Verify:** a key present with a plausible length rules the secret out as the cause; it
  does not confirm the value is correct. Confirm correctness at the issuing system, which
  has an audit trail.
- **Volatile because:** secrets rotate; any value or hash recorded here would be stale and
  recording it would leak it besides.
- _Added 2026-01-01. (Placeholder — delete once real entries exist.)_

### (example) How do I check a production Terraform change will apply cleanly before merging?
- **Use when:** you want confidence in a risky change ahead of the production gate.
- **Access:** n/a.
- **Mutates:** n/a.
- **Method:** **none safe yet.** The only local procedure needs credentials the pipeline
  holds, and running it locally would either fail on auth or touch real state — so it
  fails the paved-path rule. Do not do it.
  - **Tried:** running the plan locally against the real backend; borrowing the apply
    role. Both rejected.
  - **What would make a safe one:** a plan-only role assumable from a pull request, so the
    speculative plan runs in CI with no write path. That is a change in the repo that owns
    the pipeline IAM, not a workaround here.
- **Verify:** n/a.
- **Volatile because:** n/a — recorded so the unsafe path stays rejected.
- _Added 2026-01-01. (Placeholder — delete once real entries exist.)_
