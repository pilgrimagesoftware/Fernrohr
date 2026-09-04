---
name: "ADR"
description: Create a new Architecture Decision Record from the template
allowed-tools: Bash(ls:*), Bash(date:*), Read, Write, Edit
category: Workflow
tags: [adr, architecture, decision]
---

Create a new Architecture Decision Record.

**Input**: a short title after `/adr` (e.g. `/adr use nats as the message bus`). If
omitted, infer it from the current conversation; if that is ambiguous, ask.

**Steps**

1. **Get the next number.** `ls` the `docs/adr/` directory. Take the highest `NNNN` in a
   `*-*.md` filename (ignoring `0000-template.md`), add 1, zero-pad to 4 digits.

2. **Build the filename.** `<NNNN>-<title-kebab-cased>.md`. Lowercase, hyphens for
   spaces, strip punctuation.

3. **Write the file** from the template (`docs/adr/0000-template.md`). If it does not exist
   yet, create `docs/adr/` with a `0000-template.md` (frontmatter: `id`, `title`, `status`,
   `date`, `scope`, `tags`, `affected_repos`) and a minimal `README.md` index. Fill frontmatter:
   - `id`: `<NNNN>`
   - `title`: the title as given, sentence case
   - `status`: `proposed`
   - `date`: today (`date +%Y-%m-%d`)
   - `scope`: this repository
   - `tags`, `affected_repos`: best guess from context, leave `[]` if unclear
   Leave the body sections as template prompts for the author to fill, but pre-fill Context from
   the conversation if there is enough signal.

4. **Add the index row.** Append a row to the `docs/adr/README.md` index table:
   `| [NNNN](<filename>) | <title> | proposed | <one-line decision or —> | <date> |`
   Keep rows ordered by number.

5. **Report** the path and remind the author: fill the Options section including rejected
   options, and flip status to `accepted` once the decision survives contact with the code.

**Guardrails**
- Never reuse or guess a number without listing the directory first.
- Never write `status: accepted` on creation — new ADRs start `proposed`.
- Do not edit an existing `accepted` or `superseded` ADR. A changed decision is a new ADR that
  supersedes the old one.
