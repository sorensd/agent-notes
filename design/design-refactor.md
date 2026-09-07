# De-slopping an existing UI: a one-pass refactor

Load this when the brief is *"this looks like AI slop, fix it"* on a codebase that already
exists. It is a procedure, not a style guide — the style guidance lives in
[ui-library-ecosystem.md](ui-library-ecosystem.md) and [design-system.md](design-system.md),
and this note tells you what to do with it, in what order, in one pass.

Written from doing it to a real app — a theme generator whose own interface was slop while
it lectured other people's interfaces about slop.

---

## 0. The rule that makes this a refactor and not a rewrite

**Inherit the behaviour, replace the surface.** The flows, the state handling and the data
wiring took real use to get right. What is dated is the paint. Getting this backwards —
keeping the look out of caution and redesigning the flow out of boredom — is the standard
failure (see [ux-patterns.md](ux-patterns.md)).

So: no route changes, no state-shape changes, no renamed props, unless the audit in §2
turns one up as an actual cause of the slop.

---

## 1. Name the use case first, and check the app against it

Before touching a pixel, write down which use case each screen is, from
[ui-library-ecosystem.md §7](ui-library-ecosystem.md). Then read the app against that row.

Most slop is a **use-case mismatch**, not bad taste: the landing-page aesthetic applied to
something that isn't a landing page. A tool that is really an operational console but was
built with hero gradients and 24px radii is not suffering from the wrong shade of indigo.

Write the mismatch down as a sentence — *"this is a console, and it is dressed as a
marketing page"* — because it predicts almost every fix that follows.

## 2. Audit: grep for the tells before you judge by eye

Slop has a small, greppable vocabulary. Run these over the source and count the hits; the
counts tell you the size of the job and give you a checklist that is done when it is empty.

```bash
grep -rn "bg-gradient\|from-indigo\|via-purple\|to-pink" src/          # gradient habit
grep -rn "backdrop-blur\|glass\|bg-white/\|bg-black/" src/             # arbitrary glass
grep -rn "rounded-2xl\|rounded-3xl" src/                               # radius inflation
grep -rn "animate-\|transition-all\|hover:scale" src/                  # motion everywhere
grep -rn "shadow-2xl\|shadow-xl\|drop-shadow" src/                     # elevation inflation
grep -rn "confetti\|sparkle\|blob\|mesh" src/                          # celebration junk
grep -rEn "#[0-9a-fA-F]{6}" src/components/                            # hex outside tokens
grep -rn "text-transparent" src/                                       # gradient text
```

Then the ones grep cannot find, which need eyes:

- **Fabricated numbers.** Any metric with no source behind it. The worst kind of slop
  because it survives every visual review.
- **Missing states.** Empty, loading, error, partial permission. Generated UI ships the
  happy path and nothing else — this is reliably the thinnest part of the app.
- **Missing focus rings.** Tab through the whole app once. If you lose the cursor, that is
  a defect, not a polish item.
- **Card-on-card.** A bordered card inside a bordered card inside a panel.
- **Two of anything.** Two Button components, two icon families, two spacing scales.

## 3. Fix in this order

The order matters: each step removes work from the next one. Doing them in reverse means
restyling components you were about to delete.

1. **Tokens first, one layer.** Define the palette and scale in one place — CSS custom
   properties on `:root`. Everything downstream reads from it. Do this before touching a
   component or you will hand-edit the same colour forty times.
2. **Delete the decoration.** Animated backdrops, mesh gradients, gradient text, confetti,
   floating blobs, decorative SVG. Deletion, not restyling. This is usually the single
   biggest visual improvement and it is pure subtraction.
3. **Collapse the accent.** Slop uses colour as texture. Pick **one** accent and let
   everything else be a neutral ramp. Colour should mean something: accent = the primary
   action and the current selection; red/amber/green = state. Nothing else is coloured.
4. **Normalise the primitives.** One button component with variants, one input, one panel,
   one option-row. Codify them as classes or components in the token layer. Every ad-hoc
   `className` soup at a call site becomes one of these.
5. **Re-density to the use case.** A console at low density is just as wrong as a landing
   page at high density. Tighten or loosen padding, type scale and radius as a set — they
   move together.
6. **Cut motion to the budget.** Delete `transition-all` and `hover:scale`. Keep
   transitions on state changes. Add the `prefers-reduced-motion` block once, globally.
7. **Build the missing states.** The empty, loading, error and permission states found in
   §2. This is the step most likely to be skipped and most likely to be noticed.
8. **Fix focus and keyboard.** A real `:focus-visible` style in the base layer. Tab the app
   again and confirm you can see where you are the whole way round.

## 4. The mechanical sweep

Steps 1–4 are largely a find-and-replace across the component tree, and doing it by hand is
how you miss half of it. Write the mapping as a table — old class string → new token class
— and run it as a script over every component file at once. It is faster, complete, and the
table doubles as the record of what the design system now is.

Two cautions:

- **Order the substitutions longest-first.** `bg-slate-900 border border-slate-800` must be
  replaced before `bg-slate-900`, or the first rule never matches.
- **The sweep handles the vocabulary, not the structure.** Card-on-card, missing states and
  density are hand work. Do not expect a clean grep to mean a clean app.

## 5. A trick worth stealing: let the content supply the accent

For a tool whose subject is itself visual — a design tool, a chart builder, a theme editor —
make the chrome monochrome and set its single accent from *the thing the user is working
on*, at runtime:

```js
root.style.setProperty('--sd-accent', activeTheme.colors.primary);
```

The tool then has no brand colour of its own to compete with the work, and the interface
reads as an instrument rather than a product with opinions. Pair it with a computed
readable foreground so the accent can be any colour the user picks.

## 6. Prove it, then say what you proved

Do not report a visual refactor as done on the strength of a passing build.

- Run the app and **screenshot every screen you touched**, light and dark.
- Re-run the §2 greps and show the counts at zero.
- Tab through one full screen.
- State plainly what you verified by looking and what you only changed in code.

"The build passes" is not "it looks right", and on this kind of work only the second one is
the deliverable.

---

## The one-paragraph version

Name the use case; the mismatch between it and the current design explains most of the
slop. Grep for the vocabulary — gradients, glass, oversized radii, `transition-all`,
`shadow-2xl`, loose hex — and count. Then: tokens, delete decoration, one accent, unify
primitives, set density to the use case, cut motion to its budget, build the missing states,
fix focus. Sweep the vocabulary mechanically, do the structure by hand. Screenshot the
result before calling it done.
