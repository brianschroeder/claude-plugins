# legible

A documentation standard for repos, based on about 80% of ASD-STE100. Agents that write or
edit a repo's docs follow it automatically.

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

## Why docs

Runbooks and setup guides gain the most. A reader follows them step by step, often under
pressure, and a long or vague step is where they go wrong. A shared standard also makes docs
from different teams read the same way.

## What the standard covers

The `legible-setup` skill installs a block in the repo's `CLAUDE.md` and `AGENTS.md`. The block
sets two things.

**Sentences.** One fact per sentence, active voice, instructions as commands, one name per
thing, common words, and no telegraph style.

**Structure.** Each doc opens with its purpose and audience. Prerequisites come before the first
step. Procedures are numbered, one action per step. Warnings come before the step they apply
to. Steps that change something say what the reader should see. Runbooks end with how to
verify and how to undo.

It covers READMEs, runbooks, setup guides, how-to guides and `docs/`. Code, comments, commits
and PR text keep their own conventions.

### Before and after

Before:

> The deployment can be verified by checking the pods, and if any are in a CrashLoopBackOff
> state it's probably due to the secret not having been created yet, so make sure that's done
> first (see the secrets section).

After:

> **Prerequisite:** the `app-config` secret exists. To create it, see "Create the secret".
>
> 3. Check the pods:
>
>    ```
>    kubectl get pods -n app
>    ```
>
>    All pods show `Running`.
>
> If a pod shows `CrashLoopBackOff`, the secret is probably missing. Create it, then run step 3
> again.

## Use it

1. Install the plugin:

   ```
   /plugin install legible@bschroeder-plugins
   ```

2. Open Claude Code in the repo. Ask it to set up legible, or run the `legible-setup` skill.

3. Commit `CLAUDE.md` and `AGENTS.md`.

## Notes

- The block goes into both `CLAUDE.md` and `AGENTS.md` so that tools other than Claude Code see
  it too. If `CLAUDE.md` imports `AGENTS.md`, or one file links to the other, the skill writes
  the block once.
- Running the skill again updates the block in place. It never adds a second copy.
- The block is an instruction, not a lint check. Agents follow it, but nothing blocks a merge.
- The plugin also ships an optional output style, `legible:legible`, that applies the same
  sentence rules to your own chat replies. Turn it on with `/output-style`. The docs standard
  does not need it.
