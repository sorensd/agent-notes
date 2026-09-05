# API versioning — put `/v1` in the path from the first endpoint

Version the API from day one, in the URL path (`/api/v1/…`), even when there is only one
consumer and no plan for a v2. The cost is one path segment now; the cost of adding it later
is a migration across every client you do not control.

## Why path versioning, and why now

An API gets consumers you do not deploy — a separate SPA, a partner integration, a second
app, a webhook receiver. The moment one exists, you cannot change the wire shape on your own
schedule without breaking it. Versioning is what buys back that freedom: v2 can change shape
while v1 keeps serving the old contract, and each consumer moves when it is ready.

Doing it from the first endpoint matters because the alternative is what unversioned APIs
actually produce — a second file called `apiv2.php`, or a pile of `?legacy=true` flags, or a
breaking change shipped on a Friday. Retrofitting a version prefix means touching every
caller at once, which is exactly the coordinated break the prefix exists to avoid.

**Path over header.** `Accept: application/vnd.app.v2+json` is more "correct" and worse in
practice: harder to read in a log, harder to cache (varying on a header is a footgun),
harder to curl, and harder for a consumer to pin. `/api/v1/thing` is legible everywhere and
routes with a single mount.

## The rules that make it hold

- **`/api/v1/*` is the contract.** Version the wire shape, not the internals. Keep the
  request/response schemas in one place (`v1/schemas/`), so a future v2 changes shape while
  the service and data layers stay shared and untouched.
- **A few endpoints stay unversioned on purpose:** `/api/health`, `/api/version`. They carry
  no data and clients need them before they know anything else.
- **Standards dictate their own paths.** OIDC/OAuth (`/.well-known/openid-configuration`),
  ACME, etc. live where the spec says, not under your version prefix.
- **Every response carries the version** (`X-API-Version: v1`), so a confused client is one
  header away from the answer.
- **Retirement is announced, not sprung.** A superseded version returns `Deprecation` and
  `Sunset` headers (RFC 8594) for a real window before it stops answering.
- **Your own frontend pins to v1 like any other client.** No private back-channel. That is
  what keeps "API-first" honest instead of aspirational.

## Free documentation falls out of it

If the schemas are already written in a validation library (Zod, Pydantic, etc.), an OpenAPI
generator turns them into a spec and reference docs at `/api/v1/docs` with no separate docs
effort. Integrators get a typed client; you get the "integrate, don't author" win.

## The trap

The version is not a place to dump breaking changes whenever you feel like it. Bumping to v2
is a commitment to run two surfaces in parallel until every consumer has moved. Keep v1
stable, add to it additively (new optional fields, new endpoints), and only mint v2 when the
shape genuinely cannot evolve in place. A version you break casually is not a version.
