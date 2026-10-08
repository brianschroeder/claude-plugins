# legible

Near-STE writing for Claude Code. Chat replies and repo docs follow about 80% of ASD-STE100.

The idea comes from a [post by Andrej Karpathy](https://x.com/karpathy/status/2105819303471976479)
on LLM output readability. Models know the standard well, so you can ask for it by name.

## What ASD-STE100 is

ASD-STE100 (Simplified Technical English) is a writing standard from aerospace maintenance
manuals. Its core rules:

- One fact or one instruction per sentence.
- 20 words or fewer for a step, 25 for a description.
- Active voice.
- One word for one meaning, and common words over complex ones.

The full standard is strict. It has a controlled dictionary that rejects most technical
vocabulary. This plugin applies about 80% of it. Technical terms, product names, commands and
error strings stay exactly as they are, because accuracy matters more than the word list.

## What you get

| Component | Scope | What it does |
|---|---|---|
| `legible:legible` output style | You, in every repo | Writes chat replies in near-STE prose. Escalates to a diagram, then an HTML page, only when prose is the wrong format. |
| `legible-setup` skill | One repo, for everyone | Installs a marker-wrapped block in the repo's `CLAUDE.md` and `AGENTS.md`. Every agent that writes docs there follows the standard. |

The two layers work like Claude Code's native memory and
[simple-agent-memory](https://github.com/brianschroeder/simple-agent-memory). The output style
is personal and lives outside the repo. The block is shared, committed, and reviewed with the
code.

## Use it

1. Install the plugin:

   ```
   /plugin install legible@bschroeder-plugins
   ```

2. Turn on the output style for your own replies. Run `/output-style` and pick
   `legible:legible`.

3. Add the standard to a repo. Open Claude Code in that repo and ask it to set up legible, or
   run the `legible-setup` skill.

4. Commit `CLAUDE.md` and `AGENTS.md`.

## Where the standard applies

The block applies to READMEs, runbooks, setup guides, `docs/`, and other prose a person reads to
do a task. Runbooks and setup guides gain the most, because a reader follows them step by step,
often under pressure.

The block does not apply to code, code comments, commit messages, or text written in a person's
own voice, such as PR descriptions and tickets. Those keep their own conventions.

## Notes

- The block goes into both `CLAUDE.md` and `AGENTS.md` so that tools other than Claude Code see
  it too. If `CLAUDE.md` imports `AGENTS.md`, or one file links to the other, the skill writes
  the block once.
- Running the skill again updates the block in place. It never adds a second copy.
- The block is an instruction, not a lint check. Agents follow it, but nothing blocks a merge.
