# References

Where the context for **this repository** lives. Read this before going looking for something
outside the repo; add to it whenever you come to rely on something that isn't here yet.

This file is committed on purpose: it is shared across every teammate, every machine, and every
agent session, and it is reviewed like any other change. It is an index, not a bookmark dump —
volume is what kills it.

---

## How to use this file

**Read it when:**
- You need something that isn't in this repo and don't already know where it is.
- You are about to assume a fact about a system this repo depends on. The answer to "which file
  actually defines this?" may already be written down.

**Add to it when:**
- You came to rely on something not listed here. Add it *while you have the context* — the
  moment you found it is the moment you know what question it answers.

**Fix it when:**
- An entry points somewhere that has moved, been renamed, or no longer exists. Correct or delete
  it in passing rather than working around it. The failure mode of this file is not absence, it
  is drift: an index that is trusted and wrong.

---

## Entry style

Every entry says **what you would go there to find out.** A path with no annotation leaves the
reader to open it and discover that for themselves, which is the work this file exists to save.

Not this:

```markdown
- `../platform-infra` — the infrastructure repo.
```

This:

```markdown
- **`platform-infra` → `environments/prod/iam.tf`** — the pipeline role and its state prefix.
  Read the prefix here before setting a backend key in this repo: the role's S3 grant covers that
  prefix and nothing else, and a mismatch only fails on the first apply.
```

Point at **files**, not just repos, when you know which file matters. Group entries under
headings that match **the question being asked** ("Where the schema is defined") rather than the
kind of thing being listed ("Repositories") — that is how a reader actually arrives here. And say
whether something is authoritative ("copy this") or illustrative ("older shape, don't copy it");
that distinction is invisible from the path.

---

## Only reference things that persist

This file is committed, so it is read by people who don't have your machine and by you in six
months. An entry that no longer resolves is worse than a missing one: it is a promise the file
can't keep, and the reader spends their time finding that out.

- **Name repos canonically** — repo name plus repo-relative path, or a URL. A path like
  `../other-repo/...` only works for readers who lay their checkouts out the way you do; offer it
  as a convenience if you like, but not as the identity of the thing.
- **Never index scratch.** Gitignored directories and agent or tool working dirs (`.superpowers/`,
  `docs/superpowers/`, `.remember/`, build output) exist precisely because they are disposable.
  Check before adding an in-repo path: `git check-ignore -v <path>`.
- **Don't index documents with an expiry date.** Change proposals, migration plans, and cutover
  runbooks describe the past as soon as the change lands. Index the repo and file the change
  *acts on*, and put the "why it was broken" in the learnings log — both halves outlive the
  document.
- If something genuinely lives only on one machine and has no versioned home, either leave it out
  or put it under a heading that says so, so nobody mistakes it for a link they can follow.

**Never record secrets** — name where a credential lives, never its value. **Never add a guessed
URL** — an admitted gap is better than a link that gets trusted once and wastes an afternoon.

---

<!--
  Replace the headings below with ones matching the questions this repo actually raises.
  These are prompts, not a required structure — delete the ones that don't apply.
-->

## Where <the thing this repo depends on> is defined

<!-- Sibling repos, upstream modules, the specific files that matter. -->

## Where the permissions / credentials are configured

<!-- Name the file. Say where secrets live, never what they are. -->

## Design docs and decisions

<!-- Specs, ADRs, change proposals — especially ones explaining WHY something is as it is. -->

## Reference implementations

<!-- The ones to copy, and the ones deliberately not to. -->

## Operations

<!-- Dashboards, runbooks, alert definitions, status pages, on-call rotation. -->

## Tickets and tracking

<!-- The queue, epic, or board this repo's work hangs off. -->

## External and vendor

<!-- Docs for the services this repo integrates with. Leave empty rather than guessing URLs. -->
