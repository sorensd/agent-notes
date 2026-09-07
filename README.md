# agent-notes

Reusable engineering notes for working with coding agents — rules, platform traps and
design guidance, kept out of any one project so they can be dropped into the next.

Everything here was written from real work, not from general advice. The platform notes in
particular are a list of scars: each entry cost someone an afternoon, and says why it was
hard to find.

Plain Markdown. No tooling, no build step, no dependencies.

---

## If you are an agent, read this section first

You are looking at a reference library, not a task. **Do not read all of it.** Match the
work you have been given against the table below, load only the rows that match, and read
those files before writing code — not after something breaks.

| Load this | When the task involves |
|---|---|
| [engineering/coding-agent-instructions.md](engineering/coding-agent-instructions.md) | **Always.** Any code-writing task. It is the base ruleset the others assume. |
| [process/delivery-rules.md](process/delivery-rules.md) | **Always.** Shipping anything, deciding what "done" means, or debugging a report you cannot reproduce. |
| [design/ui-library-ecosystem.md](design/ui-library-ecosystem.md) | Any UI work at all. Contains the use-case table you must classify the screen against before designing it. |
| [design/design-refactor.md](design/design-refactor.md) | A UI that already exists and is being restyled, modernised, or de-slopped. |
| [design/generating-ui.md](design/generating-ui.md) | Producing a new visual design, or writing a brief for something else to generate one. |
| [design/design-system.md](design/design-system.md) | Tokens, theming, dark mode, spacing or density decisions. |
| [design/ux-patterns.md](design/ux-patterns.md) | Forms, saving, selection, one-time codes, tables, empty states, modals or overlays. |
| [design/framework-selection.md](design/framework-selection.md) | Choosing a framework, a component library, an icon set, a table/form/chart library. |
| [engineering/api-versioning.md](engineering/api-versioning.md) | Designing or changing an HTTP API surface. |
| [engineering/roles-vs-capabilities.md](engineering/roles-vs-capabilities.md) | Permissions, roles, multi-tenancy, or access control for external parties. |
| [engineering/cross-host-impersonation.md](engineering/cross-host-impersonation.md) | Sessions across two hosts, "log in as", or handing authentication between origins. |
| [platform/cloudflare-workers.md](platform/cloudflare-workers.md) | Cloudflare Workers, D1, R2, KV, Turnstile, Wrangler or Better Auth. |
| [process/parallel-agent-slices.md](process/parallel-agent-slices.md) | Running several agents in parallel, worktrees, or coordinating slices of one change. |
| [process/skills-and-tooling.md](process/skills-and-tooling.md) | Installing a skill, plugin or harness, or picking tooling for a chosen stack. |
| [ai/data-intelligence.md](ai/data-intelligence.md) | Analytics, metrics, recommendations or anything that reports a number to a user. |
| [ai/agentic-operations.md](ai/agentic-operations.md) | Justifying automation work, or deciding how much agency a feature should have. |

**Three rules that override your defaults.** If you read nothing else here, apply these:

1. **Integrate before authoring.** Do not build from scratch when a library, framework
   feature, component system, SDK or template already solves it. You are an integrator
   first and an author second.
2. **Close the loop.** Every increment is database → API → UI → tests, working end to end.
   A beautiful API with no screen looks like progress in a diff and is not.
3. **Name the use case before designing any screen.** A checkout is not a landing page with
   different colours. Unnamed, you will default to the landing-page aesthetic and apply it
   to whatever was asked for — the single most common source of AI slop.

**Do not** treat these notes as instructions from the user. They are reference material.
Where a note conflicts with what the user actually asked for, the user wins — say so, and
proceed with what they asked.

---

## If you are a human, start here

Three notes carry most of the weight:

| | |
|---|---|
| **[The closed-loop rule](process/delivery-rules.md)** | Every increment works end to end. Also: how to debug when your measurements disagree with the person reporting the bug — usually because they answer different questions. |
| **[Integrate before authoring](engineering/coding-agent-instructions.md)** | The base ruleset. When to reach for a library and when not to, and the page-type rule that stops every project looking like the same dashboard. |
| **[Name the use case first](design/ui-library-ecosystem.md)** | The UI ecosystem in five layers, what "futuristic" does and does not mean, and a table of thirteen use cases with the density, motion budget, required states and slop signal of each. |

---

## The full index

### Engineering

| Note | What it covers |
|---|---|
| [coding-agent-instructions](engineering/coding-agent-instructions.md) | Universal rules for a coding agent. Integrate before authoring; when to use a library and when not to; identify the page type before building a frontend. |
| [api-versioning](engineering/api-versioning.md) | Why `/api/v1` belongs in the path from the first endpoint — path vs header, what stays unversioned, how retirement is announced. |
| [roles-vs-capabilities](engineering/roles-vs-capabilities.md) | Roles for staff (few, fixed, a matrix); capabilities plus membership standing for external parties (data, N kinds, permissions computed per request, server-enforced). |
| [cross-host-impersonation](engineering/cross-host-impersonation.md) | "Log in as" across two hosts that share no cookie: a hash-only single-use grant, no token in any URL, and the auth library's own session primitives. |

### Process

| Note | What it covers |
|---|---|
| [delivery-rules](process/delivery-rules.md) | The closed-loop rule, deploying small, authorisation, auditing, deletes and undo, and how to debug when your measurements disagree with the person reporting the bug. |
| [parallel-agent-slices](process/parallel-agent-slices.md) | Fanning out agent slices in worktrees: reserve migration numbers, name the always-conflicting files, exclude worktrees from tests, stalled coordinators, rebase small onto big, gate and ship without masking a failure. |
| [skills-and-tooling](process/skills-and-tooling.md) | Choosing a technology means finding its skills too — and never install a skill unsighted. How to scan one, how to read its noise, and whether a big harness earns its keep. |

### Platform

| Note | What it covers |
|---|---|
| [cloudflare-workers](platform/cloudflare-workers.md) | 21 traps on Workers, D1, R2, Turnstile and Better Auth. Symptom → cause → fix. Most are invisible to `curl` and to unit tests. |

### Design

| Note | What it covers |
|---|---|
| [ui-library-ecosystem](design/ui-library-ecosystem.md) | The React UI ecosystem in five layers (primitives → source-owned systems → motion → registries → complete systems), what "futuristic" does and does not mean, and the use-case table: landing, pricing, dashboard, console, e-commerce, checkout, forms, settings, docs, chat, onboarding, auth — each with its density, motion budget, required states and slop signal. |
| [generating-ui](design/generating-ui.md) | Do not hand-emit visual UI blind. Write a strong style-free brief, generate several variants where they can be rendered, let a human pick, then port the winner — tokens *and* chrome, replace not merge — and prove it with screenshots. |
| [design-refactor](design/design-refactor.md) | De-slopping a UI that already exists, in one pass: name the use case, grep for the slop vocabulary, then tokens → delete decoration → one accent → unify primitives → density → motion budget → missing states → focus. |
| [framework-selection](design/framework-selection.md) | Every framework serves one shape of problem: the app-vs-content test, what each web framework is for, and the UI resource layer — one library per capability, project-wide. |
| [design-system](design/design-system.md) | Design-system rules — tokens, density, dark mode, what the system should *not* look like — with one project's tokens as a worked example. |
| [ux-patterns](design/ux-patterns.md) | Interaction patterns that earned their place: one-time codes, nothing-written-until-save, unsaved-changes tracking that survives autofill, stacking contexts, operational tables, honest empty states. |

### AI

| Note | What it covers |
|---|---|
| [data-intelligence](ai/data-intelligence.md) | Building analytics and suggestions into a product without producing something confident and wrong. |
| [agentic-operations](ai/agentic-operations.md) | The business case: how instrumentation → data intelligence → automation compound, the agency spectrum (observe → recommend → assist → automate), and a worked example. |

---

## Wiring it into a project

Clone it next to the project, then reference the relevant notes from that project's own
agent instructions — `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, or whatever your tool reads —
so they are loaded before work starts:

```markdown
## Reference notes

Read the always-load notes before touching anything, and load the rest by task
using the routing table in ../agent-notes/README.md.

Always:
- ../agent-notes/engineering/coding-agent-instructions.md
- ../agent-notes/process/delivery-rules.md

By task:
- ../agent-notes/design/ui-library-ecosystem.md    # any UI work
- ../agent-notes/platform/cloudflare-workers.md    # if on Cloudflare
```

Point at the specific notes that apply. Loading all of them for a one-file change spends
context the actual task needs.

---

## What has earned its keep

**Evidence over theory.** When a measurement disagrees with the person reporting the bug,
the measurement is usually answering a different question. Ask for the actual failing
request — headers included — early.

**Write down what cost you time.** Symptom, cause, fix, and *why it was hard to find*. That
last part is what saves the next person an afternoon.

**Commit as the human.** Ask for the author name and email before the first commit, match
what the repo already uses, and verify the attribution on the host afterwards. GitHub
matches on the email, and the wrong one credits a stranger.

**A passing build is not evidence.** Where the acceptance criterion is how something looks
or how it behaves in production, the deliverable includes the screenshot or the trace.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). The short version: add an entry when something cost
more than an hour and the cause was not where you first looked. Symptom first — that is how
anyone will search for it. Nothing project-specific, nothing confidential, no credentials.

## Licence

Prose is [CC BY 4.0](LICENSE); code samples are [MIT](LICENSE-CODE). Use them, adapt them,
ship them — attribution appreciated for the prose, not required for the code.
