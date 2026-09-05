# Cross-host impersonation without a shared cookie

The problem: a staff console (`oms.example.com`) and a customer/vendor portal
(`vendor.example.com`) are deliberately on **separate hosts** so they share no cookies, no
localStorage, and no XSS blast radius. Now an admin wants to "log in as" a portal user and
see the portal exactly as they do. There is no shared session to hand over, and you must not
weaken the isolation to get one.

## What not to do

- **Don't enable cross-subdomain cookies** just for this. It permanently couples the two
  hosts you separated on purpose.
- **Don't put the session (or a session-minting token) in a URL.** A token in a URL is a
  copyable bearer credential: paste `vendor.example.com/impersonate#token=…` into any
  browser, even incognito, and you are signed in as that user. Fragments keep it off the
  server and out of Referer, but not out of history, the clipboard, or a shoulder.
- **Don't reuse the auth library's built-in "impersonate" endpoint if it demands a global
  admin role.** Granting that role to your super-admins usually exposes the library's entire
  admin API (ban, set-password, set-role) *outside* your own permission system and audit
  log. Keep the authority in your RBAC.

## The pattern that works

A one-time **grant** minted on the console, handed to the portal tab in memory, redeemed on
the portal host with the auth library's own session primitives.

1. **Mint (console host).** Admin clicks "Login as vendor". The server checks the admin's
   live session and a dedicated permission (`users.impersonate`), then creates a grant:
   random token, store **only its SHA-256 hash**, bind it to the target user + issuer, TTL
   ~2 minutes, single-use. Return the raw token and the portal origin to the console tab.
   No token is ever minted without an active, authorised session.

2. **Hand off in memory (postMessage).** The console opens `portal/impersonate` in a new tab
   **with no token in the URL**. The portal page posts `ready` to `window.opener` at the
   exact console origin; the console replies with the token, targeted to the exact portal
   origin. Both sides check `event.origin` (and `event.source`). The token lives only in
   memory — never in a URL, history, or Referer. A copied `/impersonate` URL, or the page
   opened with no opener, does nothing.

3. **Redeem (portal host).** A small endpoint validates the grant (unexpired, unused —
   enforced by an atomic `UPDATE … SET used_at=now WHERE used_at IS NULL`, not by the read),
   re-checks the target is still a valid portal user, then starts the session **on this
   host** using the auth library's own `createSession(userId, …, { impersonatedBy })` and
   its own `setSessionCookie`. No hand-rolled session or cookie crypto. The cookie is
   host-only for the portal, so isolation is intact.

4. **Show it and get out.** `impersonatedBy` on the session drives a persistent banner
   ("Previewing as X — Exit"). Exit signs the impersonated session out and closes the tab
   the console opened. Audit both the grant and the redeem.

## Why each piece earns its place

- **Hash-only storage + single-use + short TTL** make the grant table useless to steal and
  a leaked token useless to replay.
- **postMessage over URL** removes the copyable-credential class of bug entirely.
- **Library primitives, not hand-rolled cookies** keep you clear of inventing auth crypto —
  the one thing never worth writing yourself.
- **Your permission + audit, not the library's admin role** keeps the authority where the
  rest of your authorization already lives.

The shape generalises to any "act as another user across an origin boundary": support
login-as, preview-as-customer, a desktop app opening a web session.
