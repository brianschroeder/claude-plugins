<!-- references:start -->
## References

This repo keeps a git-tracked index of **where the surrounding context lives** at
`./references.md` — sibling repos and the specific files in them, upstream modules, design
docs, dashboards, runbooks, tickets, and vendor documentation.

- **Read `./references.md`** before going looking for something outside this repo, and before
  assuming a fact about a system this repo depends on. It is faster than searching, and it is
  where the "which file actually defines this?" answers are written down.
- **Add to `./references.md`** whenever you come to rely on something not listed — while you
  still have the context, because the moment you found it is the moment you know what question
  it answers. Record the path or URL, what is in it, and the one thing a future reader would
  go there for. A bare link with no annotation does not earn its place.
- **Only reference things that persist.** This file is committed and gets read by people who
  don't have your machine, so an entry has to still resolve for them in six months. That rules
  out gitignored scratch and agent working directories (`.superpowers/`, `docs/superpowers/`,
  `.remember/`, build output — check with `git check-ignore -v <path>`), paths that only work in
  your local checkout layout, and documents with a built-in expiry date like change proposals and
  migration plans. For those last ones, index the repo and file the change *lands in*, and put
  the "why it was broken" in the learnings log instead.
- **Fix drift in passing.** If an entry points somewhere that has moved, been renamed, or no
  longer exists, correct or delete it rather than working around it. An index that is trusted
  and stale is worse than no index.
- **Never record secrets** — name *where* a credential lives, never its value. And never add a
  guessed URL: an admitted gap is better than a link that will be trusted once and waste
  someone's afternoon.

> Keep location and lessons separate. `references.md` answers *where is it?*; a learnings log
> like `agent-memory.md` answers *what did we learn the hard way?* When something is both, put
> the pointer here with a one-line warning and the full story there.

> Read the file on demand (as above) rather than importing it with `@references.md`: imported
> files load in full at every session start and grow your baseline context as the index grows,
> whereas on-demand reads keep sessions lean and only pay the cost when it's actually needed.
<!-- references:end -->
