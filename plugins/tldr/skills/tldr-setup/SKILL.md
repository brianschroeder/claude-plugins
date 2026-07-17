---
name: tldr-setup
description: Use when the user wants Claude to append a TLDR summary to the end of every response — installs a durable TLDR instruction into the global ~/.claude/CLAUDE.md. Triggers on requests like "add a tldr to every output", "summarize each response", "set up response summaries".
---

# TLDR Setup

Install a durable instruction into the user's global memory so that every
substantive Claude response ends with a standardized TLDR block.

## What this does

Writes a marker-delimited managed block into `~/.claude/CLAUDE.md`. Once
present, that instruction is loaded every session and nudges Claude to append
the TLDR block to each response. This is a strong nudge, not a hard-enforced
hook — tell the user that when you confirm.

## Procedure

1. **Resolve the target path.** Expand `~` to the user's home directory. The
   target is `<home>/.claude/CLAUDE.md`. Do not target any project file.

2. **Read the file if it exists.** Use the Read tool. If the file does not
   exist, treat its content as empty (you will create it in step 4).

3. **Check for an existing managed block.** Search the content for the marker
   `<!-- TLDR:START -->`.
   - If found: replace everything from `<!-- TLDR:START -->` through
     `<!-- TLDR:END -->` (inclusive) with the current managed block below.
     Use the Edit tool with the old block as `old_string`.
   - If not found and the file has other content: append a blank line followed
     by the managed block to the end of the file.
   - If the file does not exist or is empty: create it with the managed block
     as its entire content (use the Write tool).

4. **The managed block to install** (verbatim):

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

5. **Confirm to the user.** Report the exact path written and whether the block
   was **created** (new file), **updated** (existing block replaced), or
   **added** (appended to an existing file). Remind them it is a strong nudge,
   not a hard guarantee, and that removing the block between the markers
   disables it.

## Idempotency

Running this skill repeatedly must converge to exactly one managed block. Never
duplicate the block — if the markers already exist, replace between them.
