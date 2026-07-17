# Design: `tldr` plugin — TLDR output skill

Date: 2026-07-17

## Summary

A new plugin, `tldr`, added to this marketplace. It contains a single skill,
`tldr-setup`. Running the skill installs a durable instruction into the user's
global `~/.claude/CLAUDE.md` so that every subsequent Claude response ends with
a standardized TLDR block (summary, key points, next steps, outstanding).

## Motivation

The user wants a consistent, skimmable footer on Claude's responses that
distills each output down to: a plain-language TLDR, the most important
extracted points, recommended next steps, and what remains open. Rather than a
per-response manual action, the skill performs a one-time setup that makes the
behavior automatic across all projects.

## Approach

A **setup skill** (not a hook, not an output style). When invoked, the skill
edits the global memory file. Thereafter the loaded memory instruction nudges
the model to append the TLDR block on every response.

Tradeoff (accepted): a `CLAUDE.md` instruction is a strong, near-always-followed
nudge, but it is **not enforced** the way a Stop hook would be. The user
explicitly chose the skill approach for its simplicity and portability.

## Target file

`~/.claude/CLAUDE.md` (user-global memory). Effect is global across every
project. The skill creates the file if it does not exist.

## The managed block

The skill writes a clearly delimited, idempotent block bounded by HTML comment
markers so it can be detected, updated, or removed cleanly:

```
<!-- TLDR:START -->
## Response TLDR

At the end of every substantive response, append the following block:

---
**TLDR:** <1–3 sentence plain-language summary of the response>

**Key points:**
- <the most important extracted takeaways>

**Next steps:**
- <recommended actions to take next>

**Outstanding:**
- <open questions, blockers, or decisions needed — or "Nothing outstanding">

Rules:
- Keep every section concise.
- Omit a section (except Outstanding) only if it would be empty.
- Outstanding always appears; write "Nothing outstanding" when nothing is open.
- Skip the entire block for trivial one-line replies or pure acknowledgements.
<!-- TLDR:END -->
```

## Skill behavior (when invoked)

1. Resolve the target path `~/.claude/CLAUDE.md`.
2. If the file does not exist, create it (with the managed block as its content).
3. If it exists, check for an existing `<!-- TLDR:START -->` marker:
   - Present → replace the block between markers (idempotent update).
   - Absent → append the block, separated by a blank line.
4. Report to the user: the exact path written, and whether it was created,
   updated, or already current.

Idempotency: running the skill repeatedly converges to a single managed block;
it never duplicates.

## TLDR block contents (confirmed with user)

- **TLDR summary** — 1–3 sentence plain-language summary.
- **Key points** — most important extracted takeaways, as bullets.
- **Next steps** — recommended actions.
- **Outstanding** — what is open/blocked/needs a decision.

## Files

```
plugins/tldr/
├─ .claude-plugin/plugin.json      # name, description, version, author, license
└─ skills/tldr-setup/SKILL.md      # the setup skill
```

Plus:
- New entry in `.claude-plugin/marketplace.json` under `plugins`.
- README "Available plugins" updated from "None yet" to list `tldr`.

## Validation

`claude plugin validate .` must pass (marketplace + plugin manifest +
skill frontmatter).

## Out of scope (YAGNI)

- No Stop hook / enforced injection.
- No per-project targeting or runtime prompts (global is fixed).
- No uninstall skill (the delimited markers make manual removal trivial; an
  uninstall command can be added later if wanted).
