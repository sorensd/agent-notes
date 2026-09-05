# Delivery rules for agent-built software

Practices that came out of building a production system with a coding agent. These are the
ones that changed outcomes, not the ones that sounded good.

---

## The closed-loop rule

**Every increment is a vertical slice: database → API → UI → tests, working end to end.**

No API-only increments. No frontend shells wired to nothing. When a change merges, a person
must be able to open the deployed app and *use* what was added.

A change is not mergeable unless it contains, together:

1. the migration, if the slice needs one
2. the endpoints, with validation and authorisation
3. the UI that exercises them, reachable from the navigation
4. audit events for every state change
5. tests covering the happy path **and** the permission boundary
6. a **demo path** in the description — the literal clicks that prove the loop

If it cannot be finished end to end, open a draft and stop.

**Why it matters more with an agent than without one.** An agent will happily produce a
beautiful API with no screen, or a screen wired to nothing, and both look like progress in
a diff. The closed-loop rule is what makes "done" checkable by a person rather than
inferable from a file listing.

*It gets violated quietly.* On one occasion the API and its tests shipped and the screen
never did — nobody noticed until the feature was needed months later. Check the loop, not
the ticket.

## Deploy small and often

Batching several finished slices before deploying produced the single most avoidable
failure of the project: the user testing live, seeing old behaviour, and reporting features
as broken that were written, tested and sitting in the working tree.

**If it is finished and green, ship it.** A deploy that lands one change is diagnosable. A
deploy that lands six is not.

## Plan before building, especially when the request is large

For anything with commercial consequences, write the design first and get it agreed. The
agent will otherwise start coding the first plausible interpretation, and the wrong shape is
expensive to unwind.

The corollary: when the plan says *don't build this yet*, don't build it. A request to "add
X to the scope" is a documentation task.

## Integrate, don't invent

Check the dependencies you already have, then the platform's built-ins, then the ecosystem.
Custom code is for your actual business logic.

Two honest exceptions, both worth stating out loud when you take them:

- **Bundle cost.** Pulling ~200 kB gzipped for four simple shapes is a specific technical
  reason to draw them yourself.
- **Contract fidelity.** When you must be byte-compatible with an existing system, use its
  format, not the standard one you would have chosen.

## Never invent crypto, auth or token signing

Ever. Use the library. If the library's model does not fit yours, bridge them deliberately
from one source of truth — do not hand-mint sessions or roll a signature scheme.

## Authorise on the server, always

Every protected route resolves permissions per request from the database. Never hardcode a
role check, never trust the client.

**Hiding a control is not disabling it.** An endpoint that answers is a way in. When a
capability is removed, remove the route — do not hide the button. When a whole surface is
restricted, refuse on every request to it, not in the UI.

Permissions as *data* — editable in an admin screen, effective on the next request with no
deploy — is worth the extra table. It is the difference between a five-minute change and a
release.

## Audit every state change

`event_type`, `entity_type`, `entity_id`, `actor`, `before`, `after`. Append-only.

Record the **whole row** before a delete, not a summary. That is what makes an undo feature
possible later without any new plumbing — the information was there all along.

Do not audit scratch state (drafts, chat threads, UI preferences). It buries the events that
matter.

## Deleting

Deletes should be **refused where something still points at the record**, with a reason a
person can act on:

- a supplier with orders against it → archive instead
- a product already ordered → mark inactive
- an order past draft → cancel it first

Those references carry what was quoted and promised; orphaning them makes the history
unexplainable afterwards.

And **replace the browser's `confirm()`.** It is untranslatable, unstyleable, blocks the
tab, and reads identically whether you are removing a thumbnail or deleting a supplier —
which trains people to dismiss it unread. Separate the description from the consequence, and
for the destructive cases ask for the record's name to be typed. That is not security; it is
a moment to notice which record is actually selected.

## Undo, if you build it

- Reverse **exactly one** recorded change
- **Never build SQL from audit data.** Table and column names come from an allow-list, never
  from the stored blob. An audit row is data; treating it as code turns an undo button into
  arbitrary SQL
- **Refuse when the world has moved on** — if anything has changed that record since,
  reversing would silently discard it
- An undo is itself a change, audited, and cannot itself be undone
- Restrict it to its own permission: re-creating a deleted record and overwriting current
  values are different authority from editing

## Commit and deploy as the human, and ask who that is

**Always ask for the author identity before the first commit. Never commit or deploy as
Claude, as an agent, or as an address you inferred from context.**

This went wrong in a way worth recording. An agent took the email from its own session
context and used it as the git author. That address happened to be registered to a
*different person's* GitHub account, so every commit in a new repository was publicly
attributed to a stranger.

The rules that follow from it:

- **Ask.** "What name and email should the commits use?" is one question and it costs
  nothing. An email visible in a tool's configuration is not necessarily the one the person
  commits under.
- **Match the existing repo.** `git log --format='%an <%ae>' | sort -u` tells you the
  identity already in use. Prefer it over anything you were told elsewhere.
- **Verify the attribution after pushing**, not just the local commit. GitHub matches the
  author *email* to an account; a local `user.name` of the right person means nothing if the
  email belongs to someone else:

  ```bash
  gh api repos/OWNER/REPO/commits --jq '.[] | {author: .commit.author.email, github: .author.login}'
  ```

- **Never add agent co-author trailers or agent-attributed commits** unless the person has
  explicitly asked for them.
- The same applies to deployments, releases and anything else that carries a name. It is
  the human's work and the human's account.

## Evidence over theory when debugging

The mistake worth naming: when a measurement disagrees with the person reporting the bug,
**the measurement is usually answering a different question.**

Server-side probes returned healthy 200s for hours while production was genuinely broken,
because the probe could not carry the state that caused it. The evidence that settled it was
the **browser's request headers**.

So: ask for the actual failing request early — headers, response body, status. A platform
error page and your application's error look identical to a user and are completely
different problems.

## Verify against the deployed thing

Unit tests on a local runtime are necessary and not sufficient. Layers they do not model —
edge caching, asset serving, challenges, WAF rules — are exactly where the surprising
failures live.

After deploying, check the real URL. Twice this caught a bug the whole test suite was happy
about.

Where behaviour depends on a layer the tests cannot model, **pin the invariant rather than
the behaviour**: assert the file does not exist, assert the route is absent.

## Write down what cost you time

Every non-obvious cause, in a document that travels: symptom → cause → fix. Include *why it
was hard to find*, because that is the part that saves the next person an afternoon.

Link it from wherever an agent starts reading, so it is hit before debugging rather than
after.
