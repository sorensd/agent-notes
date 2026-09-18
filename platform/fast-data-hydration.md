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
