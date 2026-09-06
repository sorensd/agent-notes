# agent-notes

Reusable engineering notes for working with coding agents — rules, platform traps and
design guidance, kept out of any one project so they can be dropped into the next.

Everything here was written from real work, not from general advice. The platform notes in
particular are a list of scars.

---

## What is here

| File | What it covers |
|---|---|
| [engineering/coding-agent-instructions.md](engineering/coding-agent-instructions.md) | Universal rules for a coding agent. Integrate before authoring; when to use a library and when not to. |
| [engineering/api-versioning.md](engineering/api-versioning.md) | Why `/api/v1` belongs in the path from the first endpoint — path vs header, what stays unversioned, and how retirement is announced. |
| [engineering/cross-host-impersonation.md](engineering/cross-host-impersonation.md) | "Log in as" across two hosts that share no cookie: a hash-only single-use grant, a postMessage handoff (no token in any URL), and the auth library's own session primitives. |
| [process/delivery-rules.md](process/delivery-rules.md) | The closed-loop rule, deploying small, authorisation, auditing, deletes and undo, and how to debug when your measurements disagree with the person reporting the bug. |
| [process/skills-and-tooling.md](process/skills-and-tooling.md) | Choosing a technology means finding its skills too; and never install a skill unsighted — scan every one with NVIDIA SkillSpector first, how to read its noise, and whether a big harness is worth it. |
| [platform/cloudflare-workers.md](platform/cloudflare-workers.md) | 21 traps on Workers, D1, R2, Turnstile and Better Auth. Symptom → cause → fix. Most are invisible to `curl` and to unit tests. |
| [design/design-system.md](design/design-system.md) | Design-system rules — tokens, density, dark mode, what the system should not look like — with one project's tokens as a worked example. |
| [design/ux-patterns.md](design/ux-patterns.md) | Interaction patterns that earned their place: one-time codes, nothing-written-until-save, unsaved-changes tracking that survives autofill, stacking contexts, operational tables, honest empty states. |
| [ai/data-intelligence.md](ai/data-intelligence.md) | Building analytics and suggestions into a product without producing something confident and wrong. |
| [ai/agentic-operations.md](ai/agentic-operations.md) | The business case: how instrumentation → data intelligence → automation compound into 10X, the agency spectrum (observe→recommend→assist→automate), and R.O.B./Rael Sports as the worked example. |

## Using it with an agent

Reference the relevant files from the project's own agent instructions, so they are read
before work starts rather than after something breaks:

```markdown
Read before touching anything:
- ../agent-notes/engineering/coding-agent-instructions.md
- ../agent-notes/process/delivery-rules.md
- ../agent-notes/platform/cloudflare-workers.md   # if on Cloudflare
```

Or clone it next to the project and point at it directly. It is deliberately plain
markdown with no tooling.

## The three that have earned their keep

**The closed-loop rule.** Every increment is database → API → UI → tests, working end to
end. An agent will otherwise produce a beautiful API with no screen, and it looks like
progress in a diff.

**Evidence over theory.** When a measurement disagrees with the person reporting the bug,
the measurement is usually answering a different question. Ask for the actual failing
request — headers included — early.

**Write down what cost you time.** Symptom, cause, fix, and *why it was hard to find*. That
last part is what saves the next person an afternoon.

**Commit as the human.** Ask for the author name and email before the first commit, match
what the repo already uses, and verify the attribution on the host afterwards — GitHub
matches on the email, and the wrong one credits a stranger.

## Contributing to it

Add an entry when something costs more than an hour and the cause was not where you first
looked. Keep the symptom first — that is how anyone will search for it.

Nothing project-specific, nothing confidential, no credentials.
