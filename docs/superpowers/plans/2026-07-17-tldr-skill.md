# TLDR Output Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `tldr` plugin to this marketplace whose `tldr-setup` skill installs an idempotent TLDR instruction into the user's global `~/.claude/CLAUDE.md`, making every substantive Claude response end with a summary / key points / next steps / outstanding block.

**Architecture:** A single bundled plugin under `plugins/tldr/` with one skill (`skills/tldr-setup/SKILL.md`). The skill is pure instructions: it tells Claude how to edit `~/.claude/CLAUDE.md` idempotently using `<!-- TLDR:START -->` / `<!-- TLDR:END -->` markers. The marketplace index and README are updated to list the plugin.

**Tech Stack:** Claude Code plugin format (JSON manifest + markdown skill with YAML frontmatter). Verification via `claude plugin validate .`.

## Global Constraints

- Plugin and marketplace `name` fields must be kebab-case (lowercase letters, digits, hyphens only).
- Plugin names must be unique within the marketplace.
- Only `plugin.json` goes inside `.claude-plugin/`; component dirs (`skills/`) sit at the plugin root.
- The marketplace's `metadata.pluginRoot` is `./plugins`, so a bundled plugin's `source` is just its directory name (e.g. `./tldr`).
- Author name: `Brian Schroeder`. License: `MIT`.
- `claude plugin validate .` must pass with no errors before completion.
- The skill targets `~/.claude/CLAUDE.md` (global), not any project file.

---

### Task 1: Scaffold the `tldr` plugin and its setup skill

**Files:**
- Create: `plugins/tldr/.claude-plugin/plugin.json`
- Create: `plugins/tldr/skills/tldr-setup/SKILL.md`

**Interfaces:**
- Consumes: nothing (first task).
- Produces: a plugin directory `plugins/tldr/` containing a valid `plugin.json` (name `tldr`) and a skill named `tldr-setup`. Task 2 references the plugin by directory name `./tldr` and by plugin name `tldr` in the marketplace index and README.

- [ ] **Step 1: Write the plugin manifest**

Create `plugins/tldr/.claude-plugin/plugin.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-plugin-manifest.json",
  "name": "tldr",
  "description": "Appends a standardized TLDR block (summary, key points, next steps, outstanding) to the end of every Claude response, via a one-time setup skill.",
  "version": "0.1.0",
  "author": {
    "name": "Brian Schroeder"
  },
  "license": "MIT",
  "keywords": ["tldr", "summary", "output", "productivity"],
  "category": "productivity"
}
```

- [ ] **Step 2: Write the setup skill**

Create `plugins/tldr/skills/tldr-setup/SKILL.md`. The frontmatter `description` must state both what it does and when to use it (this drives skill triggering). The body is a precise, ordered procedure.

````markdown
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
````

- [ ] **Step 3: Verify the files exist and are well-formed**

Run: `cat plugins/tldr/.claude-plugin/plugin.json | python3 -m json.tool >/dev/null && echo "plugin.json OK"`
Expected: `plugin.json OK`

Run: `head -4 plugins/tldr/skills/tldr-setup/SKILL.md`
Expected: the YAML frontmatter opening `---`, a `name: tldr-setup` line, and the `description:` line.

- [ ] **Step 4: Commit**

```bash
git add plugins/tldr/.claude-plugin/plugin.json plugins/tldr/skills/tldr-setup/SKILL.md
git commit -m "feat: add tldr plugin with tldr-setup skill"
```

---

### Task 2: Register the plugin in the marketplace and README

**Files:**
- Modify: `.claude-plugin/marketplace.json` (the `plugins` array, currently `[]`)
- Modify: `README.md` (the "Available plugins" section, currently "_None yet._")

**Interfaces:**
- Consumes: the `tldr` plugin directory from Task 1 (referenced as source `./tldr`, plugin name `tldr`).
- Produces: a marketplace that lists and validates the `tldr` plugin.

- [ ] **Step 1: Add the plugin entry to the marketplace index**

In `.claude-plugin/marketplace.json`, replace the empty `plugins` array:

```json
  "plugins": []
```

with:

```json
  "plugins": [
    {
      "name": "tldr",
      "source": "./tldr",
      "description": "Appends a standardized TLDR block (summary, key points, next steps, outstanding) to the end of every Claude response, via a one-time setup skill.",
      "version": "0.1.0",
      "author": { "name": "Brian Schroeder" },
      "license": "MIT",
      "keywords": ["tldr", "summary", "output", "productivity"],
      "category": "productivity"
    }
  ]
```

- [ ] **Step 2: Update the README "Available plugins" section**

In `README.md`, replace:

```markdown
## Available plugins

_None yet._ See [CONTRIBUTING.md](./CONTRIBUTING.md) to add one.
```

with:

```markdown
## Available plugins

- **tldr** — Appends a standardized TLDR block (summary, key points, next
  steps, outstanding) to the end of every Claude response. Run its
  `tldr-setup` skill once to install the instruction into your global
  `~/.claude/CLAUDE.md`.

See [CONTRIBUTING.md](./CONTRIBUTING.md) to add another.
```

- [ ] **Step 3: Validate the whole marketplace**

Run: `claude plugin validate .`
Expected: success with no errors (validates `marketplace.json`, the `tldr` plugin manifest, and the `tldr-setup` skill frontmatter).

- [ ] **Step 4: Confirm the marketplace JSON is still valid JSON**

Run: `python3 -m json.tool .claude-plugin/marketplace.json >/dev/null && echo "marketplace.json OK"`
Expected: `marketplace.json OK`

- [ ] **Step 5: Commit**

```bash
git add .claude-plugin/marketplace.json README.md
git commit -m "feat: register tldr plugin in marketplace and README"
```

---

## Self-Review

**1. Spec coverage:**
- New `tldr` plugin → Task 1. ✓
- `tldr-setup` skill that edits `~/.claude/CLAUDE.md` → Task 1, Step 2. ✓
- Idempotent marker-delimited block with the four confirmed sections → Task 1, Step 2 (block + idempotency note). ✓
- Create / update / append file-handling paths → Task 1, Step 2, procedure steps 2–4. ✓
- marketplace.json entry → Task 2, Step 1. ✓
- README "None yet" → listed → Task 2, Step 2. ✓
- `claude plugin validate .` passes → Task 2, Step 3. ✓
- Out-of-scope items (no hook, no per-project targeting, no uninstall skill) correctly omitted. ✓

**2. Placeholder scan:** The `<...>` tokens inside the managed block are the literal template Claude fills at response time — they are intended content, not plan placeholders. No "TBD"/"TODO"/"implement later" in the plan itself. ✓

**3. Type consistency:** Plugin name `tldr` and skill name `tldr-setup` are used identically across both tasks; source `./tldr` matches the directory created in Task 1; markers `<!-- TLDR:START -->` / `<!-- TLDR:END -->` are identical in the skill body and referenced consistently. ✓
