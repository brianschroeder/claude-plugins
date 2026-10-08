---
name: references
description: Set up or maintain a repo-level references index — a `references.md` at the repository root recording where the important context actually lives (sibling repos, upstream modules, design docs, dashboards, runbooks, tickets, vendor docs), with `CLAUDE.md`/`AGENTS.md` wired to read it. Use this whenever someone wants to add, set up, bootstrap, install, or repair a "references file", a "links file", a "pointers doc", an index of important links, or wants agents and teammates to stop re-deriving where things live — even if they never say the word "skill". Also use it when adding an entry to an existing `references.md`, when auditing one for stale entries, or when re-applying the pattern to another repo.
---

# References (repo-level)

Installs a lightweight, git-tracked index of **where the context lives** into a repository, so
that a fresh session — human or agent — can find the surrounding material without re-deriving
it. It sets up two things:

1. **`references.md`** at the repo root — a curated map of the things outside this repo that
   you need in order to work inside it: sibling repos and the specific files in them, upstream
   modules, design docs, dashboards, runbooks, tickets, vendor documentation.
2. **A block in `CLAUDE.md`** (or `AGENTS.md`) telling agents to read it before going looking
   for something, and to fix or extend it as they go.

The division of labor is the same one that makes this cheap: `CLAUDE.md` holds the
*always-loaded behavior* (when to read, when to update), and `references.md` holds the
*content*, read on demand.

## References vs. agent memory

These are companion patterns and it is worth keeping the boundary clean, because a file that
tries to be both becomes neither:

- **`references.md` is about location.** *Where* is the thing? Stable, mostly additive, and
  answers "where do I look?"
- **`agent-memory.md` is about lessons.** *What did we learn the hard way?* Dated, newest-first,
  and answers "why did this bite me?"

A path belongs in references. A root cause belongs in memory. When an entry wants to be both —
"the pipeline IAM lives here, **and** the state prefix it sets will silently break your backend
key" — put the location in references with a one-line warning, and the full story in memory.

## Setting it up

Do this directly with your own file tools — read the bundled assets below, then create or edit
the files in the target repo. No script is needed. Work at the **repository root** (confirm with
`git rev-parse --show-toplevel` if git is available).

Both steps are deliberately **non-destructive — check before you write:**

1. **Create `references.md` at the repo root — only if it's missing.**
   - If `./references.md` already exists, leave it in place and add to it instead (see "Adding
     an entry"). Overwriting discards curation someone already did.
   - If it does not exist, create it from `assets/references.template.md`, then **immediately
     fill in the first few real entries.** An empty template is the one outcome that guarantees
     nobody reads the file again. Look at what the repo actually depends on — sibling clones,
     module sources in the Terraform/package manifests, links already sitting in the README or
     `CLAUDE.md` — and seed from that.
   - **Check each candidate persists before you write it down.** Read `.gitignore` and run
     `git check-ignore -v <path>` on in-repo paths; for anything outside the repo, confirm it has
     a versioned home rather than only existing on this machine. Scratch that you indexed today
     is a dead link tomorrow — see "Durability" below.

2. **Add the instruction block to `CLAUDE.md` — exactly once.**
   - Read `./CLAUDE.md` (create it if absent; use `AGENTS.md` instead if that is what the repo
     and its tooling use).
   - If it already contains the marker `<!-- references:start -->`, leave it alone so the block
     isn't duplicated.
   - Otherwise append the contents of `assets/claude-md-block.md`, preceded by a blank line if
     the file already has content.

Then report what changed — created the index? seeded which entries? added the block? already
present? — and remind the user to commit both files.

## What makes the file worth reading

This is the part that decides whether the pattern pays off, so spend your effort here rather
than on formatting.

**Every entry must say what you would go there to find out.** A bare list of repo names and
URLs is close to useless: the reader still has to open each one to discover whether it holds
what they need. The value is in the annotation, not the link.

Compare:

```markdown
- `../platform-infra` — the infrastructure repo.
```

against:

```markdown
- **`platform-infra` → `environments/prod/iam.tf`** — the pipeline role and its state prefix.
  Read the prefix here before setting a backend key anywhere in this repo; the role's S3 grant
  covers that prefix and nothing else, and a mismatch only fails on the first apply.
```

The second one saves a session. The first one costs one.

**Point at files, not just repos,** when you know which file matters. "The config lives in
`config/` somewhere" is a search; `config/limits.yaml` is an answer.

**Group by the question being asked,** not by repo or by media type. Headings like "Where the
VPCs live", "Where the permissions live", "Where the schema is defined" match how someone
arrives at the file — they have a question, not a repo name. Headings like "Repositories" and
"Links" make the reader do the mapping themselves.

**Say when a reference is authoritative and when it is only illustrative.** "Copy this" and
"this is the older shape, don't copy it" are both valuable, and the distinction is invisible
from the path.

**Reference the most durable form of a thing.** See below — this one is easy to get wrong and
expensive when you do.

## Durability: reference what outlives the task

`references.md` is committed, so it outlives the session that wrote it and gets read by people
who do not have your machine. That makes an entry pointing at something temporary worse than no
entry: it is a promise the file cannot keep, and the reader spends their time discovering that
rather than getting an answer.

Three failure modes, each with a fix:

**Scratch that is not in version control.** Agent and tool working directories are the common
case — `.superpowers/`, `docs/superpowers/`, `.remember/`, `.plan/`, scratch and tmp dirs,
`node_modules/`, build output. These are gitignored precisely because they are disposable, so
they will not exist in a fresh clone or after a cleanup. Never index them. The mechanical check
before adding any in-repo path:

```bash
git check-ignore -v <path> && echo "ignored — do not reference"
```

**Paths that only exist on one machine.** A sibling-clone path like `../platform-infra/...`
depends on the reader having laid their checkouts out the way you did, and files in a parent
folder that is not itself a repo are versioned nowhere at all. Give the **canonical** location —
repo name plus repo-relative path, or a URL — and offer the local path as a convenience, clearly
marked:

```markdown
- **`platform-infra` → `environments/prod/iam.tf`** ([repo](https://github.com/acme/platform-infra))
  — the pipeline role and its state prefix. Locally: `../platform-infra/environments/prod/iam.tf`
  if you keep your clones side by side.
```

If something genuinely lives only on one machine and has no versioned home, either leave it out
or put it under a heading that says so plainly, so nobody mistakes it for a link they can follow.

**Documents that are consumed and then obsolete.** Change proposals, migration plans, cutover
runbooks, and "here is what we're about to do" specs have an expiry date built in: once the
change lands, the document describes the past. Pointing at one bakes in a reference that quietly
goes stale.

Split it instead, along the same seam as the references/memory boundary:

- The **location** that stays true is the thing the document acts *on* — the repo and file where
  the change lands. Index that.
- The **insight** worth keeping — why it was blocked, what the fix was, what surprised you — is a
  lesson. Put it in the learnings log, which is built to be dated and to accumulate.

So rather than "see `docs/plans/fix-the-dns-share.md`", index the repo and file that defines the
DNS share, and record the "and here is why it was broken" in memory. Both halves survive the
document.

## What belongs, and what doesn't

**Belongs:**
- Repos this one depends on, named canonically, and the specific files in them that matter.
- Upstream module or package sources, when you routinely need to read them to know what a flag
  does.
- Design docs and ADRs **that are committed somewhere** — especially ones explaining why
  something is the way it is.
- Dashboards, runbooks, alert definitions, status pages.
- Ticket queues or epics tracking the work this repo serves.
- Vendor and API documentation for the services this repo integrates with.
- Reference implementations, marked as such, including the ones to *avoid* copying.

**Doesn't belong:**
- Secrets, tokens, or credentials. Say *where* a secret lives, never what it is.
- **Anything not in version control** — gitignored scratch, agent and tool working directories,
  build output, files sitting in a folder that is not itself a repo. See "Durability" above; this
  is the most common way a fresh index is dead on arrival.
- **Documents with an expiry date** — change proposals, migration plans, cutover runbooks. Index
  what they act on and record what they taught you; don't index the document.
- Anything discoverable in seconds from within the repo — a path to a file in this repo, unless
  it is genuinely hard to find.
- Speculative or unverified links. If you have not opened it, either check it or mark it
  explicitly as unverified.
- Guessed URLs. An invented link is worse than an admitted gap, because it will be trusted once
  and then quietly wastes someone's time. Leave the section marked empty instead.
- A dump of every link anyone ever mentioned. This is a curated index; volume is what kills it.

## Adding an entry

When you come to rely on something not listed, add it while you have the context — the moment
you found it is the moment you know what it answers. Put it under the heading matching the
question it answers, creating a heading if none fits. Include the path or URL, what is in it,
and the one thing a future reader would go there for.

Then ask one question before you commit it: **will this still be here in six months, for someone
who is not me?** If the answer is no, you are indexing scratch or a soon-obsolete document —
reach for the durable form instead.

## Keeping it true

The failure mode of this pattern is not absence, it is **drift**: an index that is trusted and
wrong. So the maintenance rule matters as much as the setup.

- Before recommending something out of `references.md`, confirm it still exists — a repo moved
  forge, a file was renamed, a dashboard was retired.
- When you find an entry that has drifted, fix it in passing rather than working around it.
- Delete dead references outright. Unlike a learnings log, there is nothing to preserve in a
  pointer to something that no longer exists.
- If a repo has moved (new forge, new name), record **both** the old and new locations for as
  long as the old one still exists and still does something — a half-migrated dependency is
  exactly the situation this file should warn about.

## Customization notes

- **Different filename:** if the user prefers `LINKS.md` or `CONTEXT.md`, rename the created
  file and update the path references inside the `CLAUDE.md` block to match.
- **`AGENTS.md` instead of `CLAUDE.md`:** the block is tool-agnostic. Put it wherever the repo's
  agents already read from; the read/update rules are identical.
- **Monorepos:** the pattern applies per package. A subdirectory `CLAUDE.md` loads on demand when
  an agent works in that directory, so a package-local `references.md` plus a package-local block
  scopes the index to that package.
- **On-demand, not imported:** do not switch the `CLAUDE.md` block to an `@references.md` import
  unless the index is small and always relevant. Imports load in full at every session start and
  grow the baseline cost as the file grows; on-demand reads only pay when the index is needed.

## Bundled resources

- `assets/references.template.md` — the starter index, including the embedded rules for what
  belongs and the entry style. Copy it to create the file, then seed real entries.
- `assets/claude-md-block.md` — the marker-wrapped block to append to `CLAUDE.md`/`AGENTS.md`.
