# Contributing

This is a notes repository, not a library. The bar for adding something is not "is it
true?" — it is **"did this cost real time, and was the cause somewhere you did not first
look?"**

---

## When to add a note

Add an entry when all of these hold:

- It cost more than an hour.
- The cause was not where you first looked.
- It will recur — on the next project, or for the next person on this one.
- It is not already recorded by the code, the git history, or the framework's own docs.

Do **not** add:

- General advice available in any tutorial.
- Anything that only makes sense inside one codebase.
- Restatements of official documentation.
- Predictions. Write it down after it happened, not before.

## How to write it

**Symptom first.** That is how anyone will search for it. Someone hitting this problem knows
what they saw, not what caused it.

The shape that works:

```markdown
### The thing that appears to be wrong

**Symptom.** What you actually observed — the error text, the wrong behaviour, the thing
that silently did nothing.

**Cause.** What was really happening.

**Fix.** What to do, concretely enough to act on.

**Why it was hard to find.** The most valuable line. What sent you the wrong way — the
misleading error, the test that passed anyway, the layer you assumed was fine.
```

That last section is the one people skip and the one that saves the afternoon.

## Style

- **Plain Markdown.** No tooling, no build step, no front-matter, no shortcodes.
- **Wrap prose at about 95 characters.** Diffs stay readable.
- **Tables for anything with more than two dimensions.** Prose for reasoning.
- **Be specific and be willing to be wrong.** "Use Radix" is useless; "Radix if you want the
  largest ecosystem, Base UI if you are starting fresh" is a note. State a recommendation,
  then say what would change it.
- **Mark provenance when it is not a scar.** Most of these notes come from doing the thing.
  If an entry is a survey, a snapshot of a fast-moving ecosystem, or someone else's claim,
  say so at the top so a reader knows to re-verify before acting on it.
- **No emoji. No decorative headings. No marketing voice.**

## Cross-linking

Notes are more useful joined up than alone. When you add one:

- Link it from the relevant section of `README.md`.
- Link it from any existing note whose rule it extends, and link back.
- Prefer extending an existing note to creating a near-duplicate. Two notes on the same
  subject means neither gets read.

## What must never go in

- Credentials, tokens, keys, or connection strings — including expired ones.
- Real hostnames. Use `example.com`.
- Client names, customer names, or anything identifying a private individual.
- Internal URLs, ticket links, or dashboards that need a login.
- Proprietary code belonging to a client.

The repository is public. Anonymise the worked example; the technical content never depends
on who the client was.

## Commits

- Present tense, scoped by area: `design: ...`, `platform: ...`, `process: ...`.
- Say what changed and why it is worth recording, not just which file moved.
- Commit as yourself, with your own name and email.

## Pull requests

`main` is protected: no force pushes, no deletions, admin bypass is off, and every change
comes through a pull request. Open one against `main` and describe what the note teaches, not
just that it exists. The PR template asks for provenance; fill it in.

## When a coding agent adds a note

Agents contribute here often, sometimes several at once from different projects. These rules
keep their changes from colliding or degrading the notes:

- **Branch, never push to `main`.** `notes/<topic>` from the current `origin/main`. Open a PR;
  the human merges. If a push to `main` is refused, that is the rule working — do not look
  for a way around it.
- **One subject per PR.** A note plus its README rows. Two agents each adding one note merge
  cleanly; one agent touching five notes conflicts with everyone.
- **Append README rows at the end of their table.** Insertions in the middle are the one
  conflict that recurs. The human reorders later if it matters.
- **Extend before creating.** Grep the repo for the symptom first. A second note on the same
  subject means neither is read; add a section to the existing one and link back.
- **State only what you verified.** A vendor price, a licence, a default, a limit: fetched
  from the vendor's own page or repository during the task, with the date, or written as
  "to verify". Recall is not a source.
- **Anonymise as you write, not after.** No project names, hostnames, ticket links, client
  names. The technical content never depends on who the client was.
- **Do not edit the always-load notes casually.** `coding-agent-instructions.md` and
  `delivery-rules.md` are loaded into every session; a change there costs every future task
  tokens. Prefer a new routed note and a one-line pointer.
- **Commit as the human** who is running you, with their name and email, no co-author
  trailers, in the repo's `area: ...` message style.
