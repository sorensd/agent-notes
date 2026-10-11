# Shopify: theme work with an agent, and the traps so far

Written from the Mother of Strays storefront refresh (Oct 2026). Scars first, then the
working method. Re-verify version-specific claims against the store you are on.

---

## Read the store before designing anything

A Shopify storefront leaks everything you need for a grounded concept without admin access:

- `https://<store>/products.json?limit=250` — every product with handles, variants, prices,
  `compare_at_price`, images, tags, `body_html`. Page with `&page=2`.
- `https://<store>/collections.json` and `/collections/<handle>/products.json` — the
  collection structure and membership.
- `Shopify.theme = {...}` inside the homepage HTML — theme name, schema version, role.
  (Mother of Strays: Dawn 15.2.0.) Match that exact Dawn tag as the base.
- Installed apps show in the HTML too: a `JudgemeStar` font-face meant reviews were already
  on Judge.me; no subscription app markup meant none was installed.

Do this before writing a line of design. It is what let the concept use real names, pack
sizes, prices and photos, and what surfaced the two findings that mattered most to the
client (below).

## Two things the catalogue told us that the brief did not

- **Compare-at prices were systematically 2–3× the selling price** on nearly every variant.
  Showing them would have been misleading. Rule: never render `compare_at_price` until the
  client confirms it is a genuine former price; gate it behind a theme setting default-off.
- **Most lead images were AI-generated** (filenames `ChatGPT_Image_*`, `Gemini_Generated_*`),
  but the real studio shots were still in the galleries behind them for 20 of 31 products.
  Count by filename, report the list, let the client reorder in admin. 16 products had only
  AI images and need a shoot. Filename is a heuristic, say so.

## CDN image sizing does not work on `/files/` paths

`?width=800` resizes images under `cdn/shop/products/…` but **not** images uploaded to
Content → Files (`cdn/shop/files/…`): the CDN returns the original bytes whatever you ask.
Neither `format=` nor the store-domain path helped. For a self-contained concept file we had
to resize locally (Pillow in a throwaway venv; the machine had no ImageMagick/ffmpeg).
Budget for that if you embed images.

## Concept before theme

The approved-concept page was one HTML file (Tailwind browser build + two Google Fonts), then
a fully inlined 5.5 MB build for sharing. Lessons:
- The Tailwind 4 browser build cannot `@apply` your own component classes, only utilities.
- Its preflight zeroes `<dialog>` margins; restore `margin:auto` or dialogs sit top-left.
- A horizontally scrolling row inside a grid item blew the page to 854 px on phones; grid
  and flex children need `min-width: 0`. Check `scrollWidth === clientWidth` at 375 px.
- Two literal U+FFFD characters inside Tailwind's JS blocked the artifact publisher; escape
  them as `�` when inlining.
- The client rejected a full-page colour background instantly. Colour goes on tiles,
  panels, badges and buttons; the page stays white or near-white. Ship a colourway picker
  with several options rather than arguing one.

## Theme build method (Dawn child, orchestrated)

- Base = the exact Dawn tag the store runs, committed untouched as the first commit, so
  every change is diffable and Dawn's cart, search, facets and checkout plumbing survive.
- `shopify theme check` on untouched Dawn 15.2.0 reports 16 offenses (3 errors, 13
  warnings) that are Dawn's own. Gate slices on "no new offenses", not on zero.
- One token layer (`assets/mos-tokens.css`) with colourways as `[data-colourway]` blocks and
  Dawn's `--color-*` scheme variables mapped onto it in `layout/theme.liquid`, so untouched
  Dawn sections also wear the palette.
- Shared files (`layout/theme.liquid`, `settings_schema.json`, tokens, components, `mos.js`)
  are orchestrator-only. Slice agents own `sections/mos-<name>.liquid`,
  `assets/section-mos-<name>.css`, `assets/mos-<name>.js` and their handoff.
- Contracts first: metafield namespace and keys, settings ids, collection handles, snippet
  signatures and JS globals are written into `docs/plans/01-contracts.md` before fan-out.
- No store access during the build is normal (clients add collaborators slowly). Agents
  verify with theme-check and Liquid review; the orchestrator verifies on a dev store later.
  Say this in every brief so nobody claims a visual check they could not do.
- Deploy = push to the theme repo after every integrated slice; the store pulls it through
  Online Store → Themes → Connect from GitHub, or `shopify theme push --unpublished` once a
  collaborator token exists.

## Commit identity, again

The concept repo's first commits were authored with an email taken from session context,
and GitHub attributed them to a stranger's account. The delivery-rules note already said
"ask, and match the repo"; it still happened. Use the `<id>+<login>@users.noreply.github.com`
address from the person's existing repos and verify with
`gh api repos/O/R/commits --jq '.[]|{email:.commit.author.email,login:.author.login}'`.
