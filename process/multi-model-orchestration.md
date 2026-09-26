# Multi-model orchestration: one senior model, many worktrees, the repo as the bus

How a large build was split across a strongest-tier orchestrator and several cheaper
worktree agents in one night, what each model class is for, and the frictions that cost
time. Companion to [parallel-agent-slices.md](parallel-agent-slices.md), which covers the
git mechanics; this note covers *who does what* and *how they talk*.

---

## The shape

```
Orchestrator (strongest model, one session)
  ├─ reads spec + notes, verifies repo reality, writes the plan
  ├─ lands the FOUNDATION commit: every shared file, every contract
  ├─ spawns worktree agents, each with a brief, a branch, a model
  ├─ integrates green branches, runs the suite, deploys, verifies live
  └─ reads handoffs and diff stats — never whole diffs
Worktree agents (one per closed vertical slice)
  ├─ own disjoint paths, build against fixed names
  ├─ commit to their branch, push for CI, write a handoff file
  └─ report back; ask the orchestrator, not the human, for contract answers
```

The orchestrator is the only session that knows the whole picture. It should spend its
tokens on contract-setting, integration and judgement, and delegate every long or
mechanical stretch. If it is reading a 700-line diff, something is misrouted.

## Model routing by task type

Use capability classes, not version names — they change.

| Route to the strongest tier | Route to the mid tier | Route to the fast tier |
|---|---|---|
| auth and session architecture | CRUD API + UI once contracts are fixed | docs, runbooks, registries |
| authorisation and tenant isolation | screen loaders, tables, forms | licence/notice inventories |
| anything cryptographic or key-handling | test harness setup | handoff and slice-note templates |
| OS-level services, installers, uninstallers | integration agents for merges | fixture generation |
| network policy / isolation | e2e smoke scripts | type/lint cleanup after an interface is fixed |
| adversarial security review (independent of the author) | | |

Rules that held:

- **The reviewer is never the implementer.** Security review is a separate strongest-tier
  agent reading the integrated branch. It grades the work; it does not write it.
- **Foundation on the strongest tier, or done by the orchestrator itself.** The commit
  that fixes every shared file (schema, migration, route index, nav, dependencies, lockfile,
  workflow bindings) sets every contract the fan-out builds against. A handoff round trip
  there costs more than it saves.
- **Mid-tier agents get the most work.** Once names are fixed, CRUD, screens, and tables are
  well-specified and broad; that is the mid tier's job. Do not spend the strongest model on
  a data table.
- **The fast tier gets volume, not judgement.** It must be told which facts it may state and
  to write "to verify" for anything else — it will otherwise fill a notices file with
  plausible licences.

## The brief is the interface

Each agent gets one brief file committed in the repo, plus a shared rules block. A brief
that worked had, in this order: model, branch and base, reserved migration number, the notes
to load (by routing table, not "read everything"), owned paths, the fixed contract names
it builds against, deliverables as a numbered closed loop (data → API → authz → UI → audit →
tests → demo path), and the handoff it must commit before stopping.

Two things to put in every brief because they were forgotten first time:

- **Environment facts.** "This machine has no dotnet; push a draft PR so CI on the Windows
  runner builds it, iterate on `gh run view --log-failed`." An agent that cannot build
  locally and is not told how to get a build will either stall or claim success.
- **Who answers questions.** "Contract questions go to the orchestrator via your handoff
  or a message, not to the human." Otherwise every agent interrupts the person.

## Communication: the repository, not the chat

Chats do not share memory. What travels between agents is:

1. the foundation contract file (fixed names, routes, table columns, error shape),
2. each agent's handoff file, committed as its last act,
3. the branch itself, with CI results on a draft PR,
4. a worktree registry the orchestrator keeps current.

The orchestrator pulls the branch, reads the handoff and `git diff --stat`, runs the
suite, and only opens files where the suite or the handoff points. Direct messages between
sessions exist and are useful for a one-line contract answer; they are not where state
lives.

## Two ways to run it

- **Spawned from the orchestrator** (background subagents with a model override, one per
  worktree): the person opens nothing, questions come back to the orchestrator, results
  arrive as notifications. Best for a deadline night.
- **Separate chats per worktree**, model chosen per chat, brief pasted as the first message:
  more human attention per question, but each agent has a full-size context and a person
  watching it. Best for the long, sensitive slices.

Mixing is fine: spawn the mechanical ones, hand-run the sensitive ones.

## Scars from the first night

- **Worktree base ref.** The harness's "new worktree" defaults to branching from
  `origin/<default-branch>`, which was the empty initial commit — not the phase branch with
  the scaffold. Create worktrees yourself: `git worktree add .claude/worktrees/<name> -b
  <branch> <phase-branch>` and hand the agent the absolute path.
- **`pnpm ci` is a built-in.** A `"ci"` script in `package.json` is shadowed by pnpm's own
  clean-install command; it reinstalled and never ran the checks, and a deploy workflow that
  relied on it would have shipped without a build. Name the script `verify`.
- **Merges can be gated.** An auto-approval mode refused `gh pr merge` as "merge without
  review" even with green CI. Plan for the human to click merge, or to say so explicitly;
  do not have the orchestrator's timeline depend on a merge it may not be allowed to do.
- **Verify vendor claims before routing work at them.** Three network-mesh vendors were
  evaluated in one hour by fetching their own pricing and licence pages and repository
  LICENSE files. The first two recommendations were wrong on a detail that mattered to the
  owner (a hosted control plane, a per-device ceiling). Fetch the page; do not recall it.
- **The plan is a repo artefact.** Plans and briefs written into a scratch folder were
  invisible to the worktree agents until copied into `docs/plans/` and committed. Put them
  in the repo before spawning.
- **Never `git stash`** with shared worktrees — see the parallel-agent-slices note. Put the
  ban in every brief.

## What to hand a fresh orchestrator

The spec, the notes routing table, the repo, and these three sentences: verify repo reality
before planning; land shared files before fan-out; read handoffs, not diffs.
