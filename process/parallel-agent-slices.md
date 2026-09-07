# Running several agent slices in parallel (worktrees)

One day, six merged slices, four of them built simultaneously by agents in isolated git
worktrees. It worked — and every friction point below cost real time. Read this before
fanning out.

## Before launching
- **Reserve migration numbers up front.** Two agents both picked `0024`. Hand each slice its
  number in the brief (roles 0024, finance 0025, capabilities 0026, csv 0027).
- **Name the files that always conflict** and tell each agent to keep its edit there
  minimal and self-contained: the route index (`routes/v1/index.ts`), the sidebar/nav,
  `api.ts`, the app router, `schema.ts`. Conflicts there were all "both sides added a
  line" — trivially resolved by keeping both, if the edits are small.
- **Give each agent a strict "do not" list**: no commits/push/deploy, no edits to the build
  stamp / changelog / issues log (the human finalises), no touching other worktrees.
- **Exclude worktrees from the test glob** (`vitest` `exclude: ['**/.claude/**']`) and
  gitignore the worktree directory *before* the first run. Otherwise the main checkout's
  test run collects every worktree's suites and runs them against the wrong worker (62
  files, 168 "failures", all noise), and a snapshot commit indexes the worktrees as
  embedded repos.

## While they run
- **A coordinator can stall silently.** One agent that had spawned three sub-agents never
  resumed after they finished — no error, no notification, 40 minutes idle. Watch the
  tree's most-recent-edit time; if it is far older than the last sub-report, take over:
  gate the tree yourself and ship, or launch a fresh agent with the same brief.
- **Rebase small onto big.** Land the slice with the largest, most invasive diff first
  (the restyle), then rebase the small ones onto it — never the reverse. Resolve by
  keeping both intents: the restyle's markup *and* main's functional change.
- **A moved folder is a semantic conflict git won't flag.** After `dashboard/ui/*` moved to
  `components/ui/*`, every branch cut earlier compiled against a path that no longer
  existed (19 `TS2307`). Typecheck after every rebase.

## Gating and shipping
- **Build before test when tests read `dist/`** (per-host SPA document tests). In a fresh
  worktree, `test` before `build` fails with empty documents.
- **Merge by PR number, not branch**, immediately after creating the PR — the branch lookup
  lags. Don't pass `--delete-branch` from a checkout where `main` lives in another
  worktree: it tries to check out `main` locally and reports a failure after a successful
  merge.
- **Keep one deploy worktree parked on `main`** (with `node_modules`). Never switch the main
  checkout while an agent is editing it; pull, build and deploy from the parked worktree.
- **Apply the remote migration before the deploy**, then query the remote DB for the thing
  the migration promised (the roles left, the columns added, the permissions seeded).
- **Verify the live version stamp** after every deploy. It is the only check that catches a
  chain that quietly redeployed the previous build.
- Shell gotchas that bit: `grep -c` exits 1 on zero matches (under `pipefail` that kills a
  chain — use `! grep -q`); piping a gating command into `tail` masks its exit code.
