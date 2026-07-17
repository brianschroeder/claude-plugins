# Design: `comment-review` skill

**Date:** 2026-07-17
**Status:** Approved for planning

## Problem

AI-assisted coding tends to over-comment: it narrates the code, restates what
the next line already says, leaves banner dividers and step-by-step commentary,
and dumps changelog-style notes into source. This is noise. Comments should
fight for their lives — a comment earns its place only when it says something
the code cannot. Code and readability are what matter; a comment that restates
code hurts readability rather than helping it.

## Goal

A slash-command-style skill, `/comment-review`, that reviews the comments a
branch *added* relative to `main`, flags the ones that don't earn their place
against an aggressive bar, reports them, and then offers to remove them.

## Non-goals

- Not a subagent and not part of a larger orchestrated review — it is a
  standalone slash command.
- Does not review comment *accuracy* (whether a comment matches the code) —
  that is the existing `pr-review-toolkit:comment-analyzer` agent's job. This
  skill is about comment *excess*.
- Does not fetch or review arbitrary PRs by number in v1. Default and only
  target is the current branch vs. `main`.
- Does not commit or push. Any removal touches the working tree only.

## Scope of review

- **Target:** `git diff main...HEAD` — the changes the current branch
  introduced relative to `main`.
- **Only added comment lines** (diff lines beginning with `+` that are
  comments). Pre-existing comments the branch did not touch are out of scope;
  the point is to catch new noise, not to audit the whole file.

## Taxonomy

### Flag (candidates for removal)

1. **Redundant / restates the code** — e.g. `i++; // increment i`, `//
   constructor` above a constructor.
2. **Section-divider / narration** — e.g. `// Now we loop through the items`,
   `// Step 1:`, banner/box comments.
3. **Commented-out code** — dead code left behind.
4. **Over-documented trivia** — full docstrings on self-explanatory private
   one-liners.
5. **Changelog / process noise** — e.g. `// Added by request`, `// TODO from PR
   feedback`, timestamps, author attributions.

### Preserve (never flag)

- Comments that explain **why**: rationale, non-obvious tradeoffs, workarounds,
  gotchas.
- References to issues, tickets, or external links that give context.
- Public API documentation (docstrings on exported/public surfaces).
- Legal / license / copyright headers.

### Bar

**Aggressive.** The burden of proof is on the comment. When a comment is
genuinely borderline — could be argued either way — flag it. A comment survives
only if it clearly says something the code cannot.

## Behavior

1. Resolve the base branch (`main`) and run `git diff main...HEAD`.
2. Extract added lines (prefix `+`) that are comments. Use the file's language
   to distinguish comments from string literals as best as judgment allows;
   this is a judgment task, not a regex task.
3. Judge each added comment against the taxonomy and the aggressive bar.
4. **Report**, grouped by file. For each finding:
   - `path:line`
   - the comment text
   - category (one of the five)
   - a one-line reason
   - suggested action: **delete** or **tighten**
5. **Offer to apply.** After presenting the report, offer to apply the removals
   to the working tree: delete flagged comments, and rewrite the ones whose
   action is "tighten". Apply only after the user confirms.

## Edge cases

- **No diff vs. `main` / no added comments:** report "no added comments to
  review" and stop.
- **Not a git repository, or no `main` branch:** report the situation clearly
  and stop; do not guess a base.
- **Applying removals:** working tree only. Never stage, commit, or push
  automatically.

## Approach

Pure instruction skill (no helper scripts). The SKILL.md encodes the procedure
above; Claude runs the `git diff`, isolates added comment lines, and applies the
taxonomy. Rationale:

- Matches the repo's existing pattern (`tldr-setup`, `/code-review`): zero
  runtime dependencies, no language toolchain.
- Judging "why vs. noise" and telling a comment from a string literal across
  languages is inherently a judgment task; a regex/AST script buys determinism
  on extraction but not on the judgment, at the cost of a dependency.

## Packaging

- New plugin directory `plugins/comment-review/` mirroring `plugins/tldr/`:
  - `plugins/comment-review/.claude-plugin/plugin.json`
  - `plugins/comment-review/skills/comment-review/SKILL.md`
- Register the plugin in `.claude-plugin/marketplace.json`.
- Add a README entry alongside the existing `tldr` entry.

## Success criteria

- Running `/comment-review` on a branch that added noisy comments produces a
  grouped report that flags the noise and preserves why-comments and public API
  docs.
- After the report, the user is offered removal and, on confirmation, the
  flagged comments are removed from the working tree only.
- On a branch with no added comments, or outside a git repo, the skill reports
  the situation and stops without error.
