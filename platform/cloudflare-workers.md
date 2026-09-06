# Cloudflare Workers — traps, and what they actually were

A portable list of problems that cost real time on Cloudflare Workers projects, written so
it can be dropped into any repo. Each entry is **symptom → cause → fix**, because the
symptom is usually misleading and the cause is usually somewhere you were not looking.

Sources: two production systems — an operations platform (Workers + D1 + R2 + Turnstile +
Better Auth, serving two hostnames from one Worker) and an outreach engine (Workers + D1,
high-frequency dashboard polling).

If you are an agent reading this: **most of these are invisible to `curl` and to unit
tests.** Several passed every server-side probe while production was broken. When a
measurement disagrees with the person reporting the bug, the measurement is usually
answering a different question.

---

## 1. Turnstile pre-clearance silently couples every subdomain

**Symptom.** Two apps on different subdomains of one zone. Open both login pages at once
and one starts returning **403 on everything**, symmetrically and persistently. An
unrelated third app on the same zone is affected too. `curl` shows everything healthy.

**Cause.** The Turnstile widget had *"Skip future security rule challenges for verified
visitors"* (**pre-clearance**) enabled. That makes Turnstile issue a **`cf_clearance`
cookie**, and `cf_clearance` is scoped to the **zone** (`.example.com`), not the hostname.
Solving a challenge on one host writes a new clearance and invalidates the previous one, so
the other tab's next request presents a stale clearance and **Cloudflare's edge returns 403
before the Worker runs**.

**Fix.** Turn pre-clearance **off** unless you specifically want Turnstile to satisfy your
WAF rules. If you verify tokens yourself via `siteverify`, you gain nothing from it.

**Why it was hard.** `curl` carries no `cf_clearance` and never triggers a managed
challenge, so every server-side probe returned 200. The evidence that settled it was the
**browser's request headers**, not the response body.

> **Rule of thumb:** a 403 that `curl` cannot reproduce is an edge decision, not your code.
> Ask for request headers and look for `cf_clearance`. A Cloudflare block is HTML with a
> Ray ID and a `cf-mitigated` header; your app's 403 is your own JSON.

## 2. Turnstile tokens are single-use and short-lived

**Symptom.** Sign-in works sometimes. Leave the page open a few minutes, or press resend,
and it 403s.

**Cause.** Tokens are redeemable once and expire in roughly five minutes. Fetching one on
mount and reusing it fails on the second attempt; a page left open sends an expired one.

**Fix.** Treat a token as spent after every send, whether it succeeded or not, and reset the
widget. Set `refresh-expired: 'auto'`, and on `expired-callback` call `turnstile.reset()` —
clearing the token alone can leave the widget never issuing another. Add a timer that
replaces the token well inside its lifetime, and re-check on `visibilitychange`, because
backgrounded tabs do not reliably run timers.

## 3. One Turnstile widget per property, and check the hostname back

Site keys are domain-scoped, and `siteverify` returns the `hostname` the challenge was
solved on. **Check it.** Two apps sharing one widget can otherwise accept each other's
tokens. One widget per hostname, with only that hostname in its allow-list.

## 4. Serving two SPAs from one Worker

**Symptom.** Two hostnames, two different single-page apps, one Worker. The root path `/`
serves the wrong app on one host — while deep links serve the right one.

**Cause.** Two layers, and the first one is invisible from inside the Worker:

- `not_found_handling: "single-page-application"` is **host-blind**. It cannot pick a
  different document per hostname.
- Even with it off, **the asset layer maps `/` to `/index.html` on its own, before the
  Worker is invoked.** So any `index.html` in the output is served to every host.

**Fix.** Set `not_found_handling: "none"`, name the documents something other than
`index.html` (`app.html`, `vendor.html`), and let the Worker choose by hostname in its
not-found handler. **Assert in a test that no `index.html` exists in the build output** —
that is the load-bearing invariant, and it is the one the request-level tests cannot see.

**Cost check:** this does *not* put the database back on the page-load path. Real asset
files are still served without invoking the Worker; the fallback is one `ASSETS.fetch`.

## 5. Module-level state is shared by every host in an isolate

**Symptom.** One hostname occasionally serves data computed for another.

**Cause.** A module-level `Map` used as an in-memory cache, keyed on `"overview:30"` with no
host in the key. One Worker serves several hostnames, and module state is per-isolate, not
per-host. (The `caches.default` tier was fine because its key is a URL, which includes the
origin — so the bug hid in the faster tier.)

**Fix.** Namespace every in-memory key by origin, including prefix invalidation. More
generally: **any module-level state in a multi-hostname Worker needs the host in its key.**

## 6. `c.notFound()` inside a not-found handler recurses forever

Hono's `app.notFound()` handler calling `c.notFound()` re-enters itself and blows the stack
on every unmatched path. Return a response directly (`c.env.ASSETS.fetch(...)`,
`c.text(...)`). Worth a regression test — the failure is a 500 with no useful message.

## 7. D1 error details hide on a nested `cause`

**Symptom.** A unique-constraint violation surfaces as a generic 500 instead of a 409.

**Cause.** D1 wraps driver errors. The `UNIQUE constraint failed: …` text is not on
`error.message`; it is further down the `cause` chain.

**Fix.** Walk the chain when classifying:

```ts
function describeError(err: unknown): string {
  const parts: string[] = [];
  let current: unknown = err;
  while (current instanceof Error) {
    parts.push(current.message);
    current = (current as { cause?: unknown }).cause;
  }
  return parts.join(' | ');
}
```

## 8. `datetime('now')` has one-second granularity

**Symptom.** "Has anything changed since?" logic misses changes made in the same second —
which is exactly when it matters, because rapid successive edits are what make people want
an undo.

**Fix.** Order and compare by `rowid` (insertion order), not by a text timestamp. Keep the
timestamp for display.

## 9. Drizzle: `sql\`id in ${array}\`` does not expand

It does not become an `IN (?, ?, ?)` list. Use `inArray(table.column, values)`. Silent wrong
results, not an error.

## 10. Aggregate in one query, not one query per number

**Symptom (outreach engine).** A dashboard polling a dozen separate `COUNT(*)` queries put
D1 into overload once a few tabs were open.

**Fix.** Collapse them into a single scan:

```sql
SELECT COUNT(*)                                              AS total,
       SUM(CASE WHEN status = 'active' THEN 1 ELSE 0 END)    AS active,
       SUM(CASE WHEN created_at >= datetime('now','-7 days')
                THEN 1 ELSE 0 END)                           AS last_7d
  FROM things
```

Same for per-row counts on a list: one grouped query over the page's ids, never one query
per row.

## 11. Two-tier caching, and why both tiers are needed

From the outreach engine, and the thing that actually stopped the overload:

1. **In-memory map, per isolate.** Free, but every isolate recomputes.
2. **`caches.default`, shared across the colo.** This is the tier that collapses a burst of
   polls from several tabs into one computation per TTL.

Write with `waitUntil` so the response is never delayed by storing it. Invalidate on writes
rather than relying on TTL alone — otherwise a create leaves the dashboard showing the old
count, which reads as a failed save. If keys are parameterised (`overview:7`, `overview:30`),
invalidate the whole prefix, not just the bare name.

## 12. Take the database off the page-load path

**Symptom (outreach engine).** The whole dashboard appeared to go down when D1 merely
slowed.

**Cause.** Serving the shell ran several database lookups — settings, schema self-heal —
before returning any HTML.

**Fix.** Serve the shell straight from static assets. Move self-heal off the critical path
with `ctx.waitUntil`. Let the data panels surface their own errors.

```js
if (method === 'GET' && isAppPath) {
  ctx.waitUntil((async () => { /* self-heal, off the critical path */ })());
  return env.ASSETS.fetch(new Request(url.origin + '/app.html'));
}
```

The client half matters too: a page that replaces its entire body with a spinner or an error
makes a slow query look like an outage. Render the chrome unconditionally; degrade the
panels.

## 13. Give schema self-heal a cooldown

A struggling D1 hammered with the same failing DDL on every request turns *slow* into
*stalled*. Put a cooldown (60s) on the attempt and let a failure fall through to the handler,
which returns a clean error.

## 14. `tsc --noEmit` on a solution file checks nothing

**Symptom.** Typecheck passes; the build fails.

**Cause.** With project references, `tsc --noEmit` silently does no work.

**Fix.** Use `tsc -b --force`. Verify your typecheck actually fails by injecting a
deliberate type error once — a green check that cannot go red is worse than none.

## 15. The test pool is not the edge

`@cloudflare/vitest-pool-workers` runs real `workerd` and real D1, which is excellent — but
it does **not** reproduce the asset layer's `/` → `/index.html` mapping, Cloudflare's cache,
managed challenges, or WAF rules.

A per-host document test passed in the pool while production served the wrong document.
Where behaviour depends on a layer the pool does not model, **pin the invariant instead of
the behaviour** (assert the file does not exist), and verify the real thing with a request
against the deployed URL.

Also: pin `compatibility_date` to one the pool's `workerd` supports — the pool often ships an
older binary than the Vite plugin. And `defineWorkersConfig` was removed in v0.22; use the
`cloudflareTest` plugin with `readD1Migrations` / `applyD1Migrations`.

## 16. Zod defaults only exist after `parse`

**Symptom.** A provider received `{}` and sent mail with `from: undefined`.

**Cause.** A code path handed the *raw* stored config to the consumer instead of the parsed
result. `.default()` materialises on parse, not on the schema.

**Fix.** Always pass `schema.parse(config)` downstream, including on fallback paths.

## 17. Better Auth specifics worth knowing

- **OTP send returns 200 regardless** of whether the mail actually went, by design
  (anti-enumeration). Record your own send result or a broken mailer looks healthy.
- **Cookies are host-only by default** (no `Domain` attribute), so subdomains genuinely do
  not share sessions. Good. Do not enable `crossSubDomainCookies` unless you want that.
- **The admin plugin's own gate is separate from your RBAC.** It reads `user.role` on the
  Better Auth user record. A user who is a super admin in your permission tables still has
  `role: 'user'` there, so `impersonateUser` refuses. Bridge it deliberately, from one
  source of truth, rather than maintaining two.
- Generate the auth schema with the CLI and share the plugin list between the runtime config
  and the generator config, so generated tables cannot drift from the running app.
- Removing a login method: **delete the plugin, don't hide the UI.** Hidden endpoints still
  answer, still cost money, and are a second way in on a channel nobody watches.

## 18. Uploads stream through the Worker

Every byte of an upload passes through the Worker on its way to R2, so an oversized file
fails slowly and expensively. Cap by type — a generous limit for images and documents, a
much lower one for video — and validate the type server-side against an allow-list. Keep
separate allow-lists for separate purposes so widening one does not widen the other.

Generate the object key server-side and never take it from the filename: that keeps unicode
and traversal characters out of storage entirely. Store the original name in the database.

## 19. `wrangler` rewrites your config file

`wrangler r2 bucket create` (and friends) can rewrite `wrangler.jsonc` — renaming bindings
and **stripping every comment**. Commit before running resource-creating commands, and diff
afterwards.

## 20. Public config vs secrets

Site keys, environment names and public identifiers belong in `wrangler.jsonc` under `vars`.
Anything that authenticates goes in `wrangler secret put` and is referenced by name only.
Providers should **declare** the secrets they need and receive the values at call time —
never read credentials from the database.

## 21. Custom domains are provisioned by deploy

Adding `{ "pattern": "app.example.com", "custom_domain": true }` to `routes` creates the DNS
record and certificate on the next `wrangler deploy`. Expect a minute or two before the
hostname answers; a connection failure immediately after deploying is usually provisioning,
not breakage.

---

## A checklist for multi-hostname Workers

One Worker serving several hostnames is where most of the above bit hardest:

- [ ] Every module-level cache key includes the origin
- [ ] Each hostname has its own Turnstile widget, with pre-clearance off
- [ ] `siteverify`'s returned `hostname` is checked against the request host
- [ ] `not_found_handling` is `"none"` and no `index.html` exists in the build output
- [ ] The Worker picks its SPA document by hostname
- [ ] A test asserts the no-`index.html` invariant
- [ ] Session cookies are host-only (no `Domain` attribute)
- [ ] Cross-host authorisation is enforced server-side on every route, not in the UI
- [ ] Client bundles are split per app if one audience should not receive the other's code


## Better Auth's origin check is gated on the cookie, so a cookieless test lies

**Symptom.** A cross-origin `POST /api/auth/*` (e.g. an SSO/impersonation handoff from
`app.example.com` to `portal.example.com`) returns **403** — but only for users who are
**already logged in** to the target host. Fresh browsers, and your integration tests, pass.

**Cause.** Better Auth's `validateOrigin` checks the `Origin` against `trustedOrigins`
(default: the request's own host) **only when the request carries a Cookie**:
`if (!(forceValidate || useCookies)) return;` with `useCookies = headers.has("cookie")`. No
cookie → no check → it passes. So the failure appears intermittent (depends on an existing
session cookie) and a cookieless test is a false green.

**Fix.** Set `trustedOrigins` to include the sibling first-party host(s) — a function
`(request) => string[]` is evaluated per request, so you can trust the exact sibling
(`app.` ↔ `portal.`) without a wildcard onto every subdomain. Keep the real authority (a
single-use token) separate from CSRF.

**Rule of thumb.** When reproducing an auth 403, send a **cookie** in the repro — an auth
library's CSRF/origin guard is often cookie-gated, and the logged-in path is the one that
breaks. Distinguish it from an *edge* 403 (Cloudflare block: HTML + Ray ID + `cf-mitigated`)
vs your *app* 403 (your own JSON, reaches the Worker and `wrangler tail`).

## `wrangler deploy` uses the vite-plugin's generated config, not your `wrangler.jsonc`

**Symptom.** You add a route / custom domain / binding to `wrangler.jsonc`, run `wrangler
deploy`, and the change does not take — no error, the deploy just uses the old config. A new
custom domain never gets provisioned and the host keeps not resolving.

**Cause.** With `@cloudflare/vite-plugin`, `vite build` emits a *redirected* deploy config at
`dist/<name>/wrangler.json`, and `wrangler deploy` uses **that**, not your source
`wrangler.jsonc` (it even prints "Using redirected Wrangler configuration"). Edit the source,
skip the rebuild, and you deploy the stale generated copy.

**Fix.** Always `vite build` **after** any `wrangler.jsonc` change and **before** `wrangler
deploy`. The project's `deploy` script chains them (`vite build && wrangler deploy`) for
exactly this reason — a bare `wrangler deploy` is the trap.

**Bonus.** Custom domains *can* be provisioned from config: add
`{ "pattern": "x.example.com", "custom_domain": true }` to `routes`, rebuild, deploy, and
wrangler creates the DNS record + route. No dashboard step needed (the zone must be in the
same account). Adding the hostname to a Turnstile widget does **not** create DNS — that is a
separate thing people conflate.

## A brand-new custom domain has an edge warm-up window — don't blame the app

**Symptom.** You add a new custom domain to a Worker, it resolves and serves, but one flow —
often a **cross-site POST navigation** to the new host — returns 403 in every browser for a
while, then **starts working on its own** with no change.

**Cause.** A freshly-provisioned Cloudflare custom domain takes minutes to fully settle: SSL
issuance, routing, and security/config propagation across the edge. In that window the new
host can get inconsistent edge decisions (managed challenge / bot rules / stale `cf_clearance`
from pre-clearance) that an *established* sibling host does not. It clears once propagation
completes.

**How to know it's this, not your code.** `curl` (even mimicking the browser's exact headers)
returns the normal 2xx/3xx and reaches the Worker — only real browsers 403, only on the new
host. That is the tell: a 403 curl can't reproduce is an edge decision. Prove the app with a
direct probe (`--resolve newhost:443:<zone-ip>`); if the Worker's own handler runs (its own
redirect/JSON, no `cf-mitigated` header), the app is fine — wait, then retry.

**So:** after adding a custom domain, give it a few minutes before debugging a flaky
cross-host/edge 403 on it. And keep Turnstile **pre-clearance off** so its zone-scoped
`cf_clearance` never adds to the noise.
