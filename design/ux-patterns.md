# UX patterns that earned their place

Interaction patterns from building an operations product. Each one exists because the
obvious alternative was tried and was worse.

Companion to [design-system.md](design-system.md), which covers the visual layer. This one
is about behaviour.

---

## Inherit proven interaction, replace the visuals

When replacing an old system, the interaction design is usually the part that took months
of real use to get right. **The look is what is dated; the flow is what is earned.**

Separate the two explicitly and say which you are keeping. On this project the old sign-in
screen's *behaviour* — two channels, self-submitting code, resend timer, honeypot, live
formatting, two languages — was carried over untouched, while the *appearance* was rebuilt
from scratch.

Getting this backwards is the common failure: teams keep the old look out of caution and
redesign the flow out of boredom, which is precisely inverted.

## One-time codes

The details that make the difference between "fine" and "invisible":

- **Six separate boxes**, not one text field. It tells people the shape of what they are
  entering before they type.
- **Submit on the sixth digit.** Nobody should have to find a button after typing a code.
- **Autofill.** `autocomplete="one-time-code"` on iOS; the WebOTP API on Android, which
  requires a trailing `@host #code` line in the SMS body and is origin-bound.
- **A resend timer**, 30 seconds, visible and counting. Without it people press resend
  three times and burn three codes.
- **A honeypot field**, off-screen, `tabIndex={-1}`. Cheap and effective.
- **Paste the whole code** into the first box and have it distribute across all six.

If you remove a channel, **remove the endpoint, not the tab.** A hidden method still
answers, still costs money per attempt, and is a second way in on a channel nobody watches.

## Single-use links in email are not private

The one that catches everybody, in two forms: a magic sign-in link that is already spent
when the person clicks it, and an unsubscribe link that unsubscribes people who never
clicked.

**Cause.** Gmail, Outlook Safe Links, corporate mail scanners and crawlers all fetch links
to preview or scan them. If the URL performs the action on `GET`, whichever machine gets
there first performs it.

**The fix has two layers, and which you need depends on the action.**

**Put the token in the URL fragment.** `https://example.com/invite#token=…` — browsers
never transmit a fragment, so a scanner fetching that URL sends `GET /invite` and nothing
redeemable. This alone defeats every prefetcher that works by fetching a URL, which is
nearly all of them. Read it in the browser, then `history.replaceState` it away so it
cannot reach a screenshot, a shared URL or browser history.

**Then decide between a click and a challenge**, because the fragment does not stop an
agent that opens the link in a real browser and executes the page:

- **A destructive or irreversible action — unsubscribe, delete, decline — should require a
  click.** A confirmation page is the correct design there anyway, and the cost of a wrong
  automated action is high.
- **A sign-in link should not.** Somebody who has already clicked a link in their email
  should not be asked to click another. Run an invisible bot challenge instead: in managed
  mode most people see nothing, and it costs a human a moment of spinner while costing a
  bot the token.

**Whatever you do, the landing page must have no server-side side effects.** That property
is the whole defence. The moment a plain `GET` acts, you are back where you started.

State the residual risk rather than implying there is none: an agent driving a full
browser can still act, because at that point it is indistinguishable from the person. It
needs the token from the email to do it, so the failure is a stale link rather than a way
in.

## Bot challenges

- **Render lazily.** A challenge widget held on an idle tab is a challenge that can expire,
  or collide with another tab. Render it once the person has actually entered something.
- **Treat every token as spent** after use, success or failure, and reset the widget.
- **Refresh well inside the token's lifetime**, and re-check on `visibilitychange` —
  backgrounded tabs do not reliably run timers.
- **One widget per hostname.** See the Cloudflare notes for what sharing one does.

## Nothing is written until Save

The pattern that caused the most rework to get right, and the most obvious in hindsight.

**Bad:** "New product" immediately inserts a placeholder row and opens it for editing.
Abandoned forms leave junk records forever, and an id is burned before anyone decided the
thing should exist.

**Good:** "New product" opens a form at `/products/new`. The record and its id are both
produced by the save, which then lands on the created record.

The obvious objection — *what if they lose their work?* — is answered separately, and
better:

- **IndexedDB** holds the draft locally. Survives a refresh, a closed lid, no network.
- **A server copy**, written on a debounce, lets someone start on a laptop and finish on a
  tablet. Last write wins on a client-supplied revision.
- Show which state it is in ("saved on this device" / "synced to your account"), so nobody
  has to wonder.

Drafts are scratch space: keep them in their own table, scoped to the person, and do not
audit them. They have no effect on anything until saved.

## Unsaved-changes tracking, and the autofill problem

A global "you have unsaved changes" bar is worth building. The hard part is deciding what
counts as a change, because **browser autofill and programmatic population fire `input` and
`change` events exactly like a human does**. A naive listener marks the form dirty the
moment a password manager touches it.

Three guards, all necessary:

1. Nothing counts while data is still loading, or within a short settling window after it
   arrives — that is when programmatic population happens.
2. Nothing counts unless the event target is the **focused** element. Autofill writes to
   fields nobody focused.
3. The bar's own controls are excluded, or pressing Save re-dirties the form.

## Confirmations

Replace the browser's `confirm()`. It is untranslatable, unstyleable, blocks the tab, and
reads identically whether you are removing a thumbnail or deleting a supplier — which
trains people to dismiss it unread.

A good confirmation:

- separates **what this is** from **what else it affects**
- names the specific record
- for genuinely destructive actions, asks you to **type the record's name** — not security,
  since they already have permission, but a moment to notice which record is selected
- keeps the dialog open on failure and shows the reason, rather than closing and leaving a
  toast

## Overlays and stacking contexts

A `position: fixed` child of an element that creates a stacking context is positioned
against **that ancestor**, not the viewport. A `sticky` sidebar creates one. So does
`transform`, `filter`, `will-change`, and `opacity < 1`.

The symptom is a modal that appears behind page content, and the instinct is to raise
`z-index`. **That never fixes it.** Render overlays through a portal to `document.body`.

## Routes, not tab state

Every screen should be a real route. Refresh, back/forward, deep links and bookmarks all
work, and the URL always says where you are.

The same applies to tabs within a screen: put the active tab in the query string. It costs
one line and means a link can point at the Staff tab rather than at the page containing it.

Watch route ordering — `/products/new` must be declared before `/products/:id`, or "new" is
matched as an id.

## Tables for operations

- **Coloured pills** for status, assignment and dates. Down a long table, a late order and
  a delivered one must be distinguishable without reading.
- **Bulk actions** on selection. Assigning thirty rows one at a time is not a workflow.
- **Filters that answer real questions** — "needs a vendor", "in production" — not just a
  field-by-field filter builder.
- **Collapsible parent/child navigation**, so clicking a section opens its children rather
  than navigating away from them.
- Counts and money **right-aligned and tabular**, so columns of digits line up.

## Loading and failure

**Never replace the whole page with a spinner or an error.** Render the chrome — title,
filters, range picker — unconditionally, and degrade the panels. A slow query should look
slow, not look like an outage.

Use skeletons shaped like the real layout so nothing jumps when data lands.

Surface a failed region's error *in that region*, with what still works stated plainly.

## Honest empty states

Never invent metrics to fill a screen. An empty state should say what will appear there and
how to make it appear.

The same rule applies to a thin feature: showing a company name and an honest "nothing here
yet" beats fabricated figures, and it is the difference between a product people trust and
one they check twice.

## Telling people the app changed

- A **build tag** in the chrome, polled periodically, that offers a reload when it changes.
  Poll only while the tab is visible.
- **Release notes behind it**, written for the audience that reads them. Each change its own
  bullet, not a paragraph — a wall of text is a wall nobody reads.
- Pause polling on hidden tabs. There is no point asking for a version nobody is looking at.

## Density

Operational density, not marketing density. 32px controls, tight but not cramped rows,
generous hit targets on anything destructive.

Reference points worth studying: Linear, the Stripe dashboard, GitHub settings.

## Two audiences, two front doors

When a product serves both internal staff and external partners, give them **different
hosts and visibly different screens**. An outside company should never be unsure which
system they are in.

Share the *logic* — the sign-in form, the components, the tokens — so the hard-won
behaviour cannot fork. Vary the composition, the wording and the chrome.

And enforce the boundary on the server. Refusing in the UI is not refusing.
