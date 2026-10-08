<!-- legible:start -->
## Writing standard: near-STE

This repo writes its docs in about 80% of ASD-STE100 (Simplified Technical English), the
writing standard from aerospace maintenance manuals. You know the standard. This block sets
where to apply it and how strictly.

**Applies to:** docs written or edited here (READMEs, runbooks, setup guides, `docs/`, and any
other prose a person reads to do a task), and your chat replies to people working in this repo.

**Does not apply to:** code, code comments, commit messages, generated files, or text written
in a person's own voice (PR descriptions, tickets, emails, chat posts). Those follow their own
conventions.

**Rules:**

- One fact or one instruction per sentence. Steps: 20 words or fewer. Descriptions: 25 or fewer.
- Use the active voice. Name who or what does the action. Write instructions as commands.
- Use one word for one meaning. Once a thing has a name, use that name every time. Prefer the
  name the code uses.
- Prefer common words: "use", not "utilize"; "start", not "initiate"; "because", not "due to
  the fact that".
- Keep the articles ("the", "a"). Do not write in telegraph style.
- Write procedures as numbered lists: one action per step, in the order the reader does them.
  Put a warning before the step it applies to.
- Put the outcome or answer first. Put the reason after it.
- Keep one topic per paragraph.

**The other 20%:** keep technical terms, product names, commands, error strings and identifiers
exactly as they are, even where the STE dictionary would reject them. Never swap a precise term
for a vague approved word. When a rule would make a sentence wrong, accuracy wins.

**Editing existing docs:** apply the standard to the sections you change. Do not rewrite a whole
file unless someone asks.
<!-- legible:end -->
