# Fast data hydration on Workers + D1 — rules that keep screens instant

**Symptom.** Dashboards and lists on two production systems (an operations platform and an
outreach engine, both Workers + D1 + a React SPA) hydrated slowly enough that the owner called
it "snail" speed, on infrastructure that should answer in tens of milliseconds. Each screen
looked fine in isolation; the product felt slow.

**Cause.** Not one bug but a pattern, and every part of it is invisible in a diff:

- A screen opened **six to twelve API calls**, one per panel, each doing its own D1 query.
  On Workers every D1 query is a round trip to the database's region, so a screen paid for a
  dozen round trips in series before it could paint.
- Aggregates were **one `COUNT(*)` per number** instead of one grouped scan
  (cloudflare-workers.md #10).
- The shell did **database work on the page-load path**: settings, schema self-heal, session
  lookups, before any HTML was served (#12, #13).
- Lists ran **a query per row** for counts and related names (N+1), and read every column.
- Panels **polled** on a timer; several open tabs multiplied the load until D1 overloaded (#11).
- No shared cache tier, so every isolate recomputed the same hot configuration.

Individually each was a small choice. Together they are why hydration felt like reading the
whole database on every click.

**Fix.** Treat performance as a requirement with tests, not a tuning pass at the end. These
rules are portable to any Workers + D1 project.

## Data placement

- Create the D1 primary with a `--location` hint in the region where users are (`apac`, `weur`,
  …) and **enable read replication** (dashboard → D1 → Settings; no wrangler command). Check
  with `wrangler d1 info`: `running_in_region` and `read_replication.mode`.
- Use the **D1 Sessions API** on every request (`env.DB.withSession(bookmark ?? "first-unconstrained")`),
  pass the bookmark back to the client in a header, so reads hit the nearest replica and a
  client never sees its own write disappear.
- Hot configuration (settings, pricing, feature flags) comes from KV or an in-memory
  per-isolate cache keyed by tenant **and origin** (#5), never from D1 on the request path.

## Queries

- **Ceiling: three D1 round trips per request**, and those go through `db.batch()`. A handler
  that needs more is redesigned. Make the ceiling a test, not a code comment (below).
- **Keyset pagination with an explicit column list** on every list. No `SELECT *`, no unbounded
  reads, no `OFFSET` past the first page.
- **One grouped scan per aggregate set**: `SUM(CASE WHEN …)` over the rows the screen already
  needs. Per-row counts come from one grouped query over the page's ids.
- **No N+1**: explicit joins; do not use an ORM's lazy relational loader inside a handler.
- **Every query has an index**, and every migration ships a test that runs
  `EXPLAIN QUERY PLAN` for the queries it introduces and fails on `SCAN TABLE` against any
  large or tenant-scoped table.
- State that needs serialization (a booking slot, a wallet balance, a conversation) lives in a
  Durable Object and is written to D1 once; it is not read-modify-written through D1.

## API shape

- **One request per screen.** Each screen has a single loader endpoint
  (`GET /api/v1/screens/<name>`) that returns everything the first paint needs, assembled
  server-side with the batched queries above. Panels never fan out into their own calls on
  load; details load on demand from a separate endpoint.
- Loader responses carry an **ETag**; unchanged data returns 304. The payload contains only the
  fields the screen renders.
- **Push, not polling.** Live panels (inbox, calendar, balances) subscribe over WebSocket to a
  Durable Object that broadcasts on change. If a poll must exist, it is one call every 30 s or
  more against a cached response, never one call per panel.

## Frontend

- The shell is a static asset with **no data on the load path** (#12). Route loaders fire the
  one screen request; links prefetch on hover and intent.
- A client cache with **stale-while-revalidate** (TanStack Query or equivalent) so returning to
  a screen paints from cache instantly and refreshes in the background.
- **Optimistic updates** for the frequent writes; skeletons that match the real structure.
- Code split per route; set a first-load JavaScript budget (≈150 KB gzipped for a console)
  and fail the build when it is exceeded.

## Client-side readiness: do not start the journey on the click

**Owner requirement (November 2026):** architect every customer, partner and staff surface
for immediate interaction, small downloads and ready-to-use critical paths. This is a design
constraint from the beginning, not a promise that a third-party network call is instant.

**Observed failure.** A checkout waited for its payment-start API and only then downloaded
its payment SDK. A fast API could not make that serial waterfall feel fast. Rebuilding the
same server checkout repeatedly also hid behind one apparently simple browser request.

- **Load essential interaction code before the action.** Frequent controls, form validation
  and the next required checkout step must not start a cold chunk/SDK download only when
  clicked. Warm the selected checkout SDK while details are being entered, without creating
  an intent, buying anything or capturing money. Observe vendor/privacy/consent constraints.
- **Do not eagerly download the whole application.** Keep route splitting; prefetch the
  likely next route/dialog on hover, keyboard focus, touch intent or bounded idle time.
  Prioritize visible and probable work; deduplicate in-flight downloads, limit concurrency,
  respect data-saving connections and discard obsolete speculative work. Rare heavy tools
  stay isolated, with explicit progress if their first use really must wait.
- **Parallelize independent work.** Fetch route code and authorized first-paint data together;
  do not chain component import → effect → configuration → SDK → data when dependencies do
  not require it. Keep one owner for each shared request/provider initialization.
- **Keep the main thread free.** Avoid giant synchronous parsing, repeated serialization,
  sorting on every render, full-table DOM and unbounded layout work. Use pagination or
  virtualization where measured; batch DOM reads/writes. Move substantial CPU work to a
  worker only when transfer/coordination costs earn it. IndexedDB is asynchronous storage,
  not a CPU worker: do not put cache hydration on the critical render path.
- **Cache only safe, scoped data.** Reuse the existing query cache with explicit freshness,
  identity/tenant keys and invalidation; clear private state on sign-out/account changes.
  Never persist payment secrets or use stale client prices, permissions, stock or balances
  as authority. Warm public SDK code, not transactions or privileged operations.
- **Respond immediately without pretending success.** Preserve entered details; show the
  selected/loading state within a frame where possible. Financial/security actions remain
  pending until the server/provider confirms. No fake progress or optimistic paid state.
- **Measure action readiness as well as page load.** Record cold/warm route bytes, request
  waterfalls, click-to-feedback, click-to-usable and long tasks on a representative phone.
  Use explicit project budgets and test regressions. Useful starting targets: visible
  feedback within 100 ms, p75 INP at most 200 ms and p75 LCP at most 2.5 s; provider latency
  gets its own measured budget, never an invented guarantee. Keep browser and server timing
  evidence separate; an offline delay simulation is not a production latency measurement.
- **Acceptance includes failure.** Test slow/blocked SDKs, retry without unhandled promises,
  double clicks, stale responses, keyboard/touch intent and the no-prefetch fallback.
  A skeleton or spinner is recovery UI, not evidence the underlying wait was optimized.

**Retrofit order:** capture the actual browser waterfall → remove click-triggered critical
downloads → remove serial/redundant requests → reduce main-thread work → add byte, query and
interaction budgets. Preserve security and correctness throughout. See
[delivery rules](../process/delivery-rules.md#evidence-over-theory-when-debugging).

## A fast shell can still hide a five-second checkout

**Observed in a production retrofit, November 2026.** A warm document answered in about
143 ms, but the same fresh checkout read took 5,447 / 5,519 / 5,480 ms. The endpoint returned
only about 9 KB. Bundle size and a skeleton alone could not explain or remove this wait.

**Cause.** Independent enrichments ran after one another, totals and lines were rebuilt
inside the same request, and configuration resolved one credential per database call. The
client waited for route translations before starting the authorized checkout read. The
Worker also ran far from its non-replicated database.

**Fix.** Remove the serial work without caching financial authority:

- Run independent authorized enrichments alongside pricing. Preserve each branch's existing
  failure behavior; optional artwork failing must not silently authorize a payment.
- Reuse fresh totals and lines only within the same request and exact order. Never replace
  the later payment/quote writer's fresh checks with the boot response.
- Join product names into the ordered line query, batch guide metadata/rows, and resolve
  credential names with a bounded indexed query before decrypting. Preserve store precedence,
  environment fallback and rotation behavior; do not introduce a credential cache.
- Start a one-shot checkout transport beside route/locale loading, then hand its exact
  response to the ordinary query parser. Keep normal error parsing, cancellation and abandoned
  request cleanup. Exclude payment-return paths and do not reuse it for later reads.
- Warm the existing deduplicated public configuration and challenge script in parallel.
  Creating/validating a challenge and expiring its token still happen normally.

After the integrated release, equivalent reads measured 507 / 476 / 655 ms, with handler
samples of 98 / 102 / 200 ms. The sample median fell from 5,480 to 507 ms, about 91% less
wait. These are repeated endpoint/network observations from one machine, not cold-device
measurements, browser LCP/INP, global percentiles or a guarantee for other projects.

**Placement has a cost.** An explicit compute hint toward the database helped this measured
path, but warm Worker-served HTML on the distant client route became roughly 200 ms slower.
Static assets remained edge-served. Verify database location and the actual document/API path;
do not assume every response becomes faster, enable replicas without reviewing consistency,
or move a database as an incidental performance fix. Follow the current vendor placement
contract and use the project's approved deployment path.

**Why it was hard to find.** The document was already fast and the JSON was small. The delay
was the dependency graph and remote round trips, not the visible loading component.

## Startup optimizations must preserve session and route boundaries

**Symptom.** A console paid for two session lookups, sidebar badges loaded full operational
lists, and ordinary auth reads initialized unrelated provider/PDF modules.

**Fix verified in the same retrofit:**

- Let entry and shell share the existing authoritative current-user request when it already
  performs session and access validation. Forward renewal cookies; a protected read must not
  consume renewal while dropping the cookie. A connection failure offers Retry, not a false
  sign-out. Revalidate broadcasts rather than trusting another tab as authority.
- Read only permission-filtered scalar badge counts, with a grouped/indexed query. An
  unauthorized count is absent, not a fabricated zero. Writes still invalidate badges.
- Lazily select route groups while keeping global host, CSRF, API-version and error boundaries
  outside them. Preserve handler/middleware order and construct the full routing inventory
  for parity checks. Cache route definitions, never requests, sessions or database results.
- Keep essential interactions eager and optional tabs/dialogs intent- or proximity-loaded.
  Serialize bounded speculative imports so background warming cannot swamp the critical
  path. A token-based startup skeleton improves feedback but is not a latency measurement.

**Why it was hard to find.** Every individual read was legitimate. The expensive duplication
and eager module initialization lived across entry, shell, middleware and route boundaries.

## Several artwork files in one revision must not share one thumbnail key

**Symptom.** Different products displayed the same image even though their approved revision
contained separate artwork files. A revision-level cache key collapsed distinct images.

**Fix.** Key the private derivative by immutable file identity, source checksum and transform
version. Key the client in-flight request by authorized checkout scope and immutable file
identity. Rows using the same file share one transfer; different files in the same revision
remain separate. If legacy metadata has no file ID, use a conservative line/revision fallback.

Keep thumbnail bounds explicit and reject unsupported, corrupt or oversized sources before
expensive decoding. A failed derivative must not silently download the original. Derivative
cache hits do not grant access: resolve current order/grant/approval/assignment authority on
every request and recheck it before serving bytes. Keep private blobs out of durable client
storage and revoke object URLs when unused.

This records the shipped thumbnail/identity behavior. It does not claim that fullscreen
previews were already converted to bounded optimized images; that is a separate delivery.

**Why it was hard to find.** Revision IDs looked stable and deduplicating them looked like a
performance win. The revision was a container, not the identity of the image bytes.

## A second deployment can silently undo a verified speed fix

**Symptom.** A checkout measured near 0.6 seconds, then returned to about 6.5 seconds after an
unrelated feature release. The performance commit still existed on main.

**Cause.** Another agent deployed a stale branch's complete runtime tree. A green focused
feature check did not prove that its tree contained the current production fixes.

**Fix.** Name one release owner, integrate both branches, verify the exact deployed build,
and require the deployment command to fetch current main and prove it is an ancestor of
HEAD. Refuse unverifiable/stale bases; test the guard with temporary current, stale and
integrated repositories. An intentional rollback is a reviewed revert on current main.
A tooling-only guard change need not cause another runtime deployment.

**Why it was hard to find.** Git history correctly showed both fixes, but the deployed artifact
was built from only one branch. Preserve the last verified tree, not merely individual commits.
See [parallel-agent slices](../process/parallel-agent-slices.md).

## Budgets, measured

- Log **query count and D1 duration per request** in structured logs, and surface them per
  endpoint on an internal reporting screen. Regressions must be visible to the team, not only
  to someone with a profiler.
- Targets that held: p95 under 100 ms server time for a screen loader on a warm replica, under
  50 ms for a detail endpoint.

## The two tests that make it stick

```ts
// 1. Query-plan test per migration: no table scans on tenant-scoped tables.
for (const q of queriesIntroducedByThisMigration) {
  const plan = await db.prepare(`EXPLAIN QUERY PLAN ${q.sql}`).bind(...q.params).all();
  const scans = plan.results.filter((r) => String(r.detail).startsWith('SCAN ') && !r.detail.includes('USING INDEX'));
  expect(scans, q.name).toHaveLength(0);
}

// 2. Query-count ceiling per screen loader: wrap the D1 binding and count calls.
const counted = countingD1(env.DB);              // proxies prepare/batch and increments a counter
const res = await app.request('/api/v1/screens/today', { headers }, { ...env, DB: counted });
expect(res.status).toBe(200);
expect(counted.roundTrips).toBeLessThanOrEqual(3);  // batch() counts as one
```

**Why it was hard to find.** Every individual query was fast and every endpoint returned 200
in a few milliseconds when probed alone. The slowness lived in the *number* of calls a screen
made and in *where* it made them (before paint, on a timer, per row). Nothing in a unit test
or a `curl` shows that; the browser's network waterfall does. Look at the waterfall for every
new screen before calling it done.

**Retrofitting an existing system.** In this order: (1) serve the shell static and move
self-heal off the request path; (2) collapse each screen's panel calls into one loader with
batched queries; (3) replace polling with push, or at least with one cached call; (4) add the
two tests so it cannot regress. Steps 1 and 2 alone removed most of the perceived slowness.
