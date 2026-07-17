# comment-review Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `comment-review` plugin whose `/comment-review` skill reviews the comments a branch added vs. `main`, flags ones that don't earn their place against an aggressive bar, reports them, and offers to remove them.

**Architecture:** A new bundled plugin under `plugins/comment-review/`, mirroring the existing `plugins/tldr/` layout: a `plugin.json` manifest plus a single instruction-only skill at `skills/comment-review/SKILL.md`. No scripts, no runtime dependencies — the skill encodes a procedure Claude follows (run `git diff main...HEAD`, isolate added comment lines, judge them, report, then offer removal). The plugin is registered in the marketplace index and README.

**Tech Stack:** Markdown (SKILL.md, README), JSON (plugin manifest, marketplace index), `git` (diff target), `claude plugin validate` (validation).

## Global Constraints

- Plugin and skill `name` must be **kebab-case**: `comment-review` for both.
- Bundled-plugin layout: only `plugin.json` lives in `.claude-plugin/`; component dirs (`skills/`) sit at the plugin root.
- Marketplace `metadata.pluginRoot` is `./plugins`, so `source` is just `"./comment-review"`.
- Author is `{ "name": "Brian Schroeder" }`; license `MIT`; starting `version` `0.1.0` — matching the `tldr` plugin verbatim.
- `claude plugin validate .` must pass after every task that touches plugin/manifest/marketplace files.
- The skill never commits, stages, or pushes; comment removal touches the working tree only.
- Default and only review target in v1: `git diff main...HEAD`.

---

### Task 1: Scaffold the plugin manifest

Create the plugin directory and its manifest so the marketplace can resolve a real plugin. This task's deliverable is a valid `plugin.json`; the skill file comes in Task 2.

**Files:**
- Create: `plugins/comment-review/.claude-plugin/plugin.json`

**Interfaces:**
- Consumes: nothing (first task).
- Produces: a plugin named `comment-review` at path `plugins/comment-review/`, referenced by Task 3's marketplace entry (`source: "./comment-review"`).

- [ ] **Step 1: Write the manifest**

Create `plugins/comment-review/.claude-plugin/plugin.json`:

```json
{
  "$schema": "https://json.schemastore.org/claude-code-plugin-manifest.json",
  "name": "comment-review",
  "description": "Reviews comments a branch added vs. main and flags ones that don't earn their place — redundant, narration, commented-out code, over-documented trivia, changelog noise — then offers to remove them.",
  "version": "0.1.0",
  "author": {
    "name": "Brian Schroeder"
  },
  "license": "MIT",
  "keywords": ["comments", "code-review", "cleanup", "readability"],
  "category": "code-review"
}
```

- [ ] **Step 2: Validate the manifest**

Run: `claude plugin validate .`
Expected: PASS. Note: because the marketplace does not yet list `comment-review` (added in Task 3), validation confirms the manifest JSON/frontmatter is well-formed; the plugin is not yet indexed. If validation reports the plugin is present-but-unlisted as an error rather than ignoring it, proceed — Task 3 resolves the listing. Do not "fix" by inventing extra manifest fields.

- [ ] **Step 3: Commit**

```bash
git add plugins/comment-review/.claude-plugin/plugin.json
git commit -m "feat: scaffold comment-review plugin manifest"
```

---

### Task 2: Write the `comment-review` skill

Write the instruction-only SKILL.md that encodes the full review procedure. This is the heart of the plugin.

**Files:**
- Create: `plugins/comment-review/skills/comment-review/SKILL.md`

**Interfaces:**
- Consumes: the plugin dir from Task 1.
- Produces: a skill named `comment-review` (invoked as `/comment-review`), listed in Task 3's marketplace entry and Task 4's README entry.

- [ ] **Step 1: Write the skill file**

Create `plugins/comment-review/skills/comment-review/SKILL.md` with exactly this content:

````markdown
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
````

- [ ] **Step 2: Validate the plugin**

Run: `claude plugin validate .`
Expected: PASS — the skill frontmatter (`name`, `description`) parses and the plugin is well-formed.

- [ ] **Step 3: Behavioral dry-run against a fixture**

Create a throwaway fixture branch with one noisy comment and one why-comment, then confirm the diff surface the skill relies on works as written. This verifies the procedure's data source, not Claude's judgment.

Run:

```bash
git stash -u 2>/dev/null; \
BASE=$(git rev-parse --abbrev-ref HEAD); \
git checkout -b comment-review-fixture main && \
printf 'def f(x):\n    x += 1  # increment x\n    # Retry once: the upstream API 500s on cold start (see ISSUE-42)\n    return x\n' > /tmp/fixture.py && \
git add /tmp/fixture.py 2>/dev/null; \
cp /tmp/fixture.py fixture_sample.py && git add fixture_sample.py && \
git commit -q -m "fixture: noisy + why comment" && \
echo "=== git diff main...HEAD (added comment lines) ===" && \
git diff main...HEAD -- fixture_sample.py | grep -E '^\+.*#'
```

Expected output includes both added comment lines:
```
+    x += 1  # increment x
+    # Retry once: the upstream API 500s on cold start (see ISSUE-42)
```

Confirm by inspection that the skill's taxonomy would **flag** `# increment x` (redundant) and **preserve** the retry rationale (why-comment). This is a manual read, not an assertion.

- [ ] **Step 4: Tear down the fixture**

Run:

```bash
git checkout "$BASE" && \
git branch -D comment-review-fixture && \
rm -f fixture_sample.py /tmp/fixture.py && \
git stash pop 2>/dev/null; \
git status --short
```

Expected: back on the original branch, no `fixture_sample.py`, clean status (aside from the plan/skill files you are working on). The fixture branch and file must NOT be committed to the feature branch.

- [ ] **Step 5: Commit**

```bash
git add plugins/comment-review/skills/comment-review/SKILL.md
git commit -m "feat: add comment-review skill"
```

---

### Task 3: Register the plugin in the marketplace

Add the plugin to the marketplace index so it is installable.

**Files:**
- Modify: `.claude-plugin/marketplace.json`

**Interfaces:**
- Consumes: the plugin at `plugins/comment-review/` (Task 1) with skill `comment-review` (Task 2).
- Produces: a listed plugin `comment-review@bschroeder-plugins`.

- [ ] **Step 1: Add the marketplace entry**

In `.claude-plugin/marketplace.json`, add this object to the `plugins` array, after the existing `tldr` entry (keep valid JSON — add a comma after the `tldr` object's closing brace):

```json
    {
      "name": "comment-review",
      "source": "./comment-review",
      "description": "Reviews comments a branch added vs. main and flags ones that don't earn their place — redundant, narration, commented-out code, over-documented trivia, changelog noise — then offers to remove them.",
      "version": "0.1.0",
      "author": { "name": "Brian Schroeder" },
      "license": "MIT",
      "keywords": ["comments", "code-review", "cleanup", "readability"],
      "category": "code-review"
    }
```

- [ ] **Step 2: Validate the full marketplace**

Run: `claude plugin validate .`
Expected: PASS — both `tldr` and `comment-review` resolve, manifests and skill frontmatter are valid.

- [ ] **Step 3: Verify JSON is well-formed**

Run: `python3 -c "import json;d=json.load(open('.claude-plugin/marketplace.json'));print([p['name'] for p in d['plugins']])"`
Expected: `['tldr', 'comment-review']`

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: register comment-review plugin in marketplace"
```

---

### Task 4: Document the plugin in the README

Add a README entry so the plugin is discoverable, matching the existing `tldr` bullet's style.

**Files:**
- Modify: `README.md` (the `## Available plugins` list, currently around lines 30-37)

**Interfaces:**
- Consumes: the registered plugin from Task 3.
- Produces: user-facing documentation. Terminal deliverable — no later task depends on this.

- [ ] **Step 1: Add the README bullet**

In `README.md`, under `## Available plugins`, add a second bullet after the `tldr` bullet (before the `See CONTRIBUTING.md` line):

```markdown
- **comment-review** — Reviews the comments your current branch added vs.
  `main` and flags ones that don't earn their place (redundant, narration,
  commented-out code, over-documented trivia, changelog noise), then offers to
  remove them. Run its `comment-review` skill (`/comment-review`) on a branch
  before opening a PR.
```

- [ ] **Step 2: Validate nothing broke**

Run: `claude plugin validate .`
Expected: PASS.

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "docs: document comment-review plugin in README"
```

---

## Self-Review

**1. Spec coverage:**
- Slash-command skill, branch diff vs. main → Task 2, Steps 1 (procedure §1–2).
- Only added comment lines → Task 2, procedure §2.
- Five flag categories + preserve list + aggressive bar → Task 2, procedure §3.
- Report grouped by file with path:line/text/category/reason/action → Task 2, procedure §4.
- Offer to apply removal, working tree only, never commit → Task 2, procedure §5 + Global Constraints.
- Edge cases (no diff, not a repo, no main, HEAD==main, trailing comments) → Task 2, procedure §1, §4, and Notes.
- Packaging: plugin dir + manifest → Task 1; marketplace registration → Task 3; README → Task 4.
- Validation with `claude plugin validate .` → every task.
All spec sections map to a task. No gaps.

**2. Placeholder scan:** No TBD/TODO/"handle edge cases" placeholders. Every code and content step shows the exact content or command. Clear.

**3. Type consistency:** Names are consistent throughout — plugin `comment-review`, skill `comment-review`, source `./comment-review`, marketplace name `bschroeder-plugins`, invocation `/comment-review`. Manifest `description` matches between Task 1 (plugin.json) and Task 3 (marketplace entry). Keywords/category (`code-review`) consistent across manifest and marketplace entry.
