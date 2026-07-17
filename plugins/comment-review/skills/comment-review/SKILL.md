---
name: comment-review
description: Use when the user wants to review comments a branch added for excess or verbosity — flags redundant, narration, commented-out, over-documented, and changelog-noise comments on the current branch's diff vs. main, then offers to remove them. Triggers on requests like "review my comments", "check for excessive comments", "did I over-comment this branch".
---

# Comment Review

Review the comments this branch *added* relative to `main` and flag the ones
that do not earn their place. Comments fight for their lives: code and
readability are what matter, and a comment survives only when it says something
the code cannot. The bar is aggressive — the burden of proof is on the comment.

This reviews comment *excess*, not comment *accuracy* (whether a comment matches
the code). If the user wants accuracy checking, that is a different job.

## Procedure

1. **Establish the target.** Confirm this is a git repository and that a `main`
   branch exists. If not, report the situation plainly and stop — do not guess a
   base branch.

2. **Get the added lines.** Run `git diff main...HEAD`. Consider only *added*
   lines (diff lines beginning with `+`, excluding the `+++` file header) that
   are **comments**. Ignore pre-existing comments the branch did not touch — the
   goal is to catch new noise, not audit the whole file. Use each file's
   language to tell a comment apart from a string literal; this is a judgment
   call, not a regex match.

3. **Judge each added comment** against the taxonomy below with an aggressive
   bar. When a comment is genuinely borderline — arguable either way — flag it.

   **Flag (candidates for removal):**
   - **Redundant / restates the code** — e.g. `i++; // increment i`, or
     `// constructor` above a constructor.
   - **Section-divider / narration** — e.g. `// Now we loop through the items`,
     `// Step 1:`, banner or box comments.
   - **Commented-out code** — dead code left behind.
   - **Over-documented trivia** — full docstrings on self-explanatory private
     one-liners.
   - **Changelog / process noise** — e.g. `// Added by request`,
     `// TODO from PR feedback`, timestamps, author attributions.

   **Preserve (never flag):**
   - Comments that explain **why**: rationale, non-obvious tradeoffs,
     workarounds, gotchas.
   - References to issues, tickets, or external links that give context.
   - Public API documentation (docstrings on exported/public surfaces).
   - Legal / license / copyright headers.

4. **Report the findings**, grouped by file. For each flagged comment give:
   - `path:line`
   - the comment text
   - the category (one of the five above)
   - a one-line reason it does not earn its place
   - the suggested action: **delete** or **tighten**

   If there are no added comments, or none are flagged, say so plainly and stop.

5. **Offer to apply.** After presenting the report, offer to apply the changes
   to the working tree: delete the flagged comments, and rewrite the ones whose
   action is **tighten**. Apply only after the user confirms. Touch the working
   tree only — never stage, commit, or push.

## Notes

- If the branch is `main` itself, or `HEAD` equals `main`, there is nothing to
  review; report that and stop.
- Removing a comment must never change code behavior. When a comment shares a
  line with code (a trailing comment), delete only the comment portion and leave
  the code intact.
