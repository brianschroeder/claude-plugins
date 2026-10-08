---
name: legible-setup
description: >-
  Install a documentation standard into a repository: a marker-wrapped block in both
  `CLAUDE.md` and `AGENTS.md` that sets how the repo's docs are written. Sentences follow about
  80% of ASD-STE100 Simplified Technical English, and procedures follow a fixed layout
  (purpose, prerequisites, numbered steps, warnings first, expected results, verify and undo).
  Every agent that writes docs in the repo (Claude Code, Cursor, Codex, Aider and other
  AGENTS.md readers) follows it. Use this whenever the user wants to add, set up, install,
  update, repair or remove the "legible" block, a documentation or docs writing standard, a
  runbook or setup guide style, Simplified Technical English, STE or ASD-STE100 rules, or wants
  a repo's docs to read more clearly and consistently across teams, even if they do not say
  "skill".
---

# Legible setup (repo-level docs standard)

This skill installs a git-tracked documentation standard into a repository, the same way
`simple-agent-memory` installs a learnings log. The standard lives in a marker-wrapped block
in the repo's `CLAUDE.md` and `AGENTS.md`. Both files load at the start of every agent
session, so any agent that writes or edits a doc there follows the standard without being
asked.

The block covers documentation only: READMEs, runbooks, setup guides, how-to guides and
`docs/`. It does not change how agents reply in chat, write code, or word commits and PRs.
The plugin's `legible:legible` output style is a separate, personal setting for chat replies,
and this skill does not depend on it.

The rules sit in the block itself rather than in a separate file. A writing standard only
works if it is in context at the moment of writing, so on-demand reading does not fit here.

## Procedure

Find the repository root with `git rev-parse --show-toplevel`. Run every file operation against
that path, not the current directory. If git is not available, use the project's top directory.

### 1. Read the block

The managed block is the whole of `assets/instructions-block.md` in this skill, from
`<!-- legible:start -->` through `<!-- legible:end -->`. Copy it with a file copy or a shell
command when you write it. Do not retype it.

### 2. Pick the target files

Look at `CLAUDE.md` and `AGENTS.md` at the repo root before writing anything. Check these cases
in order. The first one that matches wins.

1. **One file is a symlink to the other.** Check with `ls -l` or `readlink`. Resolve the link and
   write the block once, to the real file. Report the link as "covered by symlink".
2. **`CLAUDE.md` imports `AGENTS.md`.** Look for a line, outside a code fence, whose first token
   is `@AGENTS.md` or `@./AGENTS.md`. Write the block to `AGENTS.md` only. Claude Code reads it
   through the import, and a second copy would load the rules twice. Report `CLAUDE.md` as
   "covered by import". If `CLAUDE.md` already holds a legible block, tell the user it now
   duplicates the import and offer to remove it. Do not remove it unasked.
3. **Otherwise, write to both files.** The user asked for the standard in both places, so a repo
   with only one of the files gets the other.

Then apply step 3 to each target file.

### 3. Write the block into each target

First count the markers in the file.

- **More than one start marker, more than one end marker, a start marker with no end marker,
  or an end marker before the start marker:** stop. Do not guess where a block begins or ends,
  because a guess can delete the user's own instructions. Show the user the marker lines and
  ask what to do.
- **The file does not exist, is empty, or holds only whitespace:** write the managed block as
  the whole file.
- **One start marker and one end marker, in that order:** take the text from the start marker
  through the end marker, markers included. Compare it with the managed block exactly. Any
  difference, whitespace included, counts. If they are identical, make no change. If they
  differ, replace that span with the managed block and change nothing outside it. Before you
  replace it, note every line in the old span that the managed block does not contain. Step 4
  reports those lines.
- **The file has content but no markers:** trim trailing blank lines from the content, add one
  blank line, then append the managed block.

### 4. Report

For each file, report the path and one outcome:

- **created** (new file)
- **added** (block appended to an existing file)
- **updated** (an older block replaced). List any lines from the old block that the new block
  does not contain, such as a glossary pointer someone added. Say they were removed, and offer
  to put them back inside the markers.
- **already current** (identical block, no change)
- **covered by import** or **covered by symlink** (not written, and why)

Remind the user to commit each file you created or changed. If nothing changed, skip the
reminder. Say that the block is a strong instruction, not a lint check. Agents follow it, but
nothing blocks a merge on it.

## Idempotency

Running this skill again must leave exactly one block in each target file. Never append a
second block. When the markers exist, replace between them.

## Removing it

Delete everything from `<!-- legible:start -->` through `<!-- legible:end -->` in each file.
Delete a file only if the block was its only content and the user agrees.

## Customization notes

- **Monorepos:** ask the user which package. Put the block in that package's `CLAUDE.md` and
  `AGENTS.md` instead of the repo root's. Claude Code loads a subdirectory `CLAUDE.md` when it
  works in that directory.
- **A house glossary:** if the repo keeps a glossary or term list, add one line to the block
  inside the markers that points at it. "One word for one meaning" then has a list to follow.
  A later update replaces the block, and step 4 reports the line so it can go back in.
- **Stricter or looser:** the 80% figure is the point. Full STE uses a controlled dictionary
  and rejects most technical vocabulary, which hurts accuracy in engineering docs. Do not
  tighten the block toward the full standard without the user asking.

## Bundled resources

- `assets/instructions-block.md`: the managed block, copied verbatim into each target.
