# Closed sign-in with one-time codes: no passwords, no self-created accounts

The sign-in model that two production systems settled on after trying the open version
first. It is short because the rules are few; the value is in the traps underneath them.

---

## The rules

1. **One-time codes for every human sign-in.** Console, customer portal, desktop client: the
   same six-box code screen, delivered by email (or another out-of-band channel the identity
   owns). No password field exists anywhere, no reset flow, no credential table for humans.
   Step-up for a sensitive action is a fresh code, not a password prompt.
2. **Sign-in never creates an account.** A row for a person exists only because an
   administrator created it: a staff-role grant, a membership add, or the bootstrap admin
   seeded from configuration. An email that has no row gets the *same* response and the
   same screen as one that does, and nothing is stored, generated or sent for it.
3. **Membership is granted, not invited.** An authorised admin adds a person by email; the
   row and the active membership exist immediately and the person can sign in. No invite
   token, no pending state, no acceptance step, no expiry job.
4. **Machine credentials are not human passwords.** Random per-device account passwords a
   service rotates, bearer tokens for agents, enrollment codes: these stay. The rule is that
   no human is ever asked to type or remember a secret.

## Staff and customers are two identity systems, not two roles

The single most expensive design mistake in this area: one auth library instance, one
`user` table, one session cookie, one `/api/auth/*` prefix, with staff and customers told
apart by a role column and route allowlists. It is what a library's quick start produces and
it is wrong for any product where staff operate on customers.

**Rule:** staff identity and customer identity share nothing. Separate tables (`staff_*`
and `customer_*`, never a bare `user`), separate sessions with different cookie names and
different secrets, separate endpoint prefixes (`/api/ops/auth/*` vs `/api/auth/*`),
separate sign-in pages, and disjoint plugin lists (the staff instance has no code-based
sign-in; the customer instance has no password plugin). Two instances of the same library
with different configuration is the cheap way to get this.

Why it matters:

- **Isolation by construction, not by allowlist.** A customer session cannot satisfy a
  staff route because the staff middleware only reads the staff table. No allowlist can
  drift, no role column can be flipped, no shared endpoint can be reached by the wrong
  audience.
- **Different threat models, different controls.** Staff get password + authenticator,
  tight lockouts, an undisclosed base path, and a WAF rule scoped to it. Customers get
  one-time codes, anti-enumeration, and a bot challenge under abuse. Sharing one instance
  forces one set of compromises onto both.
- **Different lifecycles.** Staff are a handful of rows created by administrators; customers
  are many rows created by tenant admins. Separate tables keep every customer query away
  from staff data and make audits and exports trivial.
- **A discreet staff surface.** The staff console lives under a base path that customer
  pages never link to, with `X-Robots-Tag: noindex`, unknown paths under it indistinguishable
  from any other 404, and the path configurable so it can be a long random slug.

Retrofitting this after the fact costs a migration rewrite and a fixture rewrite across every
test. Decide it before the first table is created.

## Why closed beats open

The open version ("any email that verifies a code gets a row") is the default an auth
library nudges you toward, and it has a plausible justification: identical responses for
known and unknown emails. It costs you:

- **Rows that grow for no reason.** Every typo, every probe, every bot attempt is a user row
  you keep forever, in a table your permission checks join against on every request.
- **Mail you send for strangers.** An unauthenticated caller makes your sender emit codes to
  arbitrary addresses, on your domain reputation.
- **A second audience in your UI.** Signed-in people with no role need a "you have no
  access" surface that would not otherwise exist.

The closed version keeps the indistinguishability guarantee at the response layer instead of
the storage layer. Same status, same body, same screen, roughly the same time; nothing
written.

## What it takes to do it right

- **Mask the library, do not trust it.** A "no sign-up" option in the auth library usually
  returns an error for unknown emails. That error is the enumeration leak. Wrap the send
  endpoint: look up the email yourself, and for an unknown one return the success shape
  without calling the library. Do the same on verify: a code submitted for an unknown email
  fails exactly like a wrong code.
- **Bootstrap must pre-create.** If sign-in cannot create rows, the first administrator's row
  has to exist before their first sign-in. Seed it idempotently from configuration on the
  first request (one conditional insert, audited with a system actor), not on first
  successful verify, or nobody can ever get in.
- **Allowlist the auth library's routes.** Mounting the library's handler on `/api/auth/*`
  exposes every endpoint it ships: password reset, credential update, session revocation,
  code checking outside your audited hook. Enumerate the three or four paths your UI
  actually calls and return 404 for the rest. This is also how "no passwords" is enforced
  rather than merely intended.
- **Key the send cap on the requester, not only the target.** A per-email cap of "N codes
  per window" lets any anonymous caller lock any account, including the only administrator,
  whose address is usually public. Key the tight cap on (email, client address), and while an
  unexpired code exists do not send another and do not return 429; return the same 200.
- **Bound anonymous writes.** Recording "unknown email attempted" in an audit log is useful
  until an attacker writes a million rows into a table you can never prune. Rate-limit the
  record per client, or fold it into the per-client limiter.
- **Audit the grants, not the sign-ins.** Role grant, membership add, bootstrap seed: full
  before/after rows. Successful sign-ins are events, not state changes.

## The code screen (unchanged from the UX note)

Six separate boxes, submit on the sixth digit, paste distributes, `autocomplete="one-time-code"`,
a visible 30-second resend timer, an off-screen honeypot enforced server-side. See
[design/ux-patterns.md](../design/ux-patterns.md) "One-time codes".

## Where it sits with roles

Rows only from admin actions pairs naturally with [roles-vs-capabilities.md](roles-vs-capabilities.md):
the same action that creates the row assigns the standing, so there is never a person
without a place, and permission checks never meet an account nobody granted anything to.
