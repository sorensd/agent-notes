# Generating UI: give the design a render loop

A hard lesson from one day of work: six hand-written dashboard designs, all rejected
("AI slop", "2015", "just bad"). The one the owner approved came from the same model
family running our prompt in a harness that could **render and screenshot its own
output**, producing several variants for a human to pick from.

The difference was not taste. It was eyes.

---

## 1. Don't hand-emit visual design blind

A coding agent in a terminal usually cannot see what it ships: the in-app browser may not
be signed into the artifact host; local files render as dead snapshots. Writing HTML/CSS
under those conditions is mixing audio with the speakers off. Hand-cloning a polished
template from its markup lands in the uncanny valley — structurally right, polish wrong —
which a non-designer reads as "bad".

**Rule:** for anything whose acceptance criterion is *how it looks*, generate it where it
can be rendered and compared. Do not iterate on taste from a terminal.

## 2. The brief does the heavy lifting

The winning "single prompt" was mostly a brief built with the owner over several rounds.
A generation brief that works:

- **Use case, first line** — landing page, pricing, dashboard, operational console,
  e-commerce browse/product, checkout, forms, settings, docs, chat, onboarding, auth.
  Everything below inherits from it: density, motion budget, required states, which
  components must exist. Unnamed, a generator defaults to the landing-page aesthetic and
  applies it to whatever you asked for. See
  [ui-library-ecosystem.md §7](ui-library-ecosystem.md) for the table, and
  [coding-agent-instructions.md](../engineering/coding-agent-instructions.md) §6 for the
  underlying rule.
- **Context** — what the product is, who uses it, the domain vocabulary and lifecycle.
- **Functional requirements** — every panel/component that must exist and what it must do
  (sortable tables, bulk select, popover notifications, save/discard forms, skeletons that
  match structure, honest empty states, visible hover/press/focus).
- **Demo data, verbatim** — real-looking rows, KPIs, series, issues, notifications. Forbid
  invented numbers. Same data across every variant makes them comparable.
- **References by name** — a handful of products that define the calibre ("study the feel,
  don't copy one"), plus the qualities wanted (modern, agentic, information-dense).
- **No style prescription** — no colours, no fonts. Let the generator choose; that is the
  point of generating several. Ask for light and dark.
- **Stack line** at the end (e.g. React + TypeScript + Tailwind + shadcn/ui + lucide) so
  the output drops into the real codebase.

## 3. Variants, then a human picks

Ask for multiple variants (the reference project shipped four behind `?v=`), rendered and
screenshotted — all for the *same* use case, so they are actually comparable. The owner picks by looking, not by reading a description. Park the runner-up
as a variant; don't delete it.

## 4. Porting the winner: a design system is tokens AND chrome

When integrating the chosen design into the real app, the failure mode is subtle: port the
widgets but "keep the app's existing tokens and shell" — and the result looks exactly like
the old app. Users see palette and chrome first, widgets second.

- **Replace, don't merge.** The base tokens (background, foreground, primary, border,
  ring, radius, font…) must equal the reference *verbatim*, for light and dark. Diff them
  mechanically.
- **Port the chrome** — sidebar, header, theme toggle — not only the content area.
- **Then** wire real data underneath. Real routes, real permissions, real counts; honest
  placeholders where the backend has no figure; never a fabricated metric.
- **Keep every feature reachable.** A new sidebar must carry every screen the old one did
  (and its children); a page without a nav entry is a feature that vanished.
- **One primitive set.** Two Button components means two products. Consolidate.

## 5. Never declare "identical" without evidence

Sign in for real (locally the OTP email lands in a file; use the CAPTCHA vendor's
always-pass test key temporarily and restore it), capture screenshots in light and dark,
and compare against the reference captures. Report what was verified visually and what was
only code-diffed. "Tests pass" is not "looks right".

For the other case — an interface that already exists and needs the slop taken out rather
than a new design generated — see [design-refactor.md](design-refactor.md).

**Division of labour that works:** a render-capable generator designs the skin from a
strong brief; the coding agent ports it into the live system, wires it to real data, and
proves it with screenshots.
