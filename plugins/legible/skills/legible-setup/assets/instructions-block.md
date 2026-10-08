<!-- legible:start -->
## Documentation standard: near-STE

Write the docs in this repo in about 80% of ASD-STE100 (Simplified Technical English), the
writing standard from aerospace maintenance manuals. You know the standard. This block sets how
strictly to apply it and how to lay out a doc.

**Covers:** READMEs, runbooks, setup guides, how-to guides, `docs/`, and any other page a person
reads to understand or operate this system. Code, code comments, commit messages and PR text
keep their own conventions.

### Sentences

- One fact or one instruction per sentence. Steps: 20 words or fewer. Descriptions: 25 or fewer.
- Use the active voice. Name who or what does the action.
- Write instructions as commands: "Run the script", not "The script should be run".
- Use one word for one meaning. Once a thing has a name, use that name everywhere. Prefer the
  name that the code, console or CLI uses.
- Prefer common words: "use", not "utilize"; "start", not "initiate"; "because", not "due to
  the fact that".
- Keep the articles ("the", "a"). Do not write in telegraph style.
- Keep one topic per paragraph.

### Structure

- Open each doc with what it is for and who it is for, in one or two sentences.
- List the prerequisites before the first step: access, tools and versions, and anything that
  must already exist.
- Write each procedure as a numbered list. Use one action per step, in the order the reader
  does them.
- Put a warning before the step it applies to, never after. Say what goes wrong and how to
  avoid it.
- After a step that changes something, say what the reader should see.
- Put every command, path and value in code formatting, exactly as the reader types it. Mark
  placeholders clearly, for example `<account-id>`.
- Use a table for reference data the reader looks up. Do not use a table for steps.
- End a runbook or setup guide with how to confirm it worked and how to undo it.
- Put a known failure and its fix next to the step that causes it, or in a troubleshooting
  section at the end.
- Link to the source of a value that changes, such as a config file or a console page. Do not
  copy the value into the doc.

### The other 20%

Keep technical terms, product names, commands, error strings and identifiers exactly as they
are, even where the STE dictionary would reject them. Never swap a precise term for a vague
approved word. When a rule would make a sentence wrong, accuracy wins.

### Editing existing docs

Apply the standard to the sections you change. Do not rewrite a whole file unless someone asks.
<!-- legible:end -->
