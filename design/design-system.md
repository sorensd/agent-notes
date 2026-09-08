# Design system rules

Portable design-system guidance, with one project's tokens kept as a worked example.
The principles travel; the specific colours and numbers are illustrative.


**Binding for all UI work in this repository.**

The OMS will eventually contain an admin console, a vendor portal, a customer portal and a
tools hub. They must read as **one product**, not four tools that happen to share a
database. This document is how that stays true.

The mechanism is simple and non-negotiable:

> **One token layer → one component library → every surface.**
> Tokens live in [`src/client/theme.css`](../src/client/theme.css). Components come from
> shadcn/ui. **No screen defines its own colours, radii or spacing.**

---

## 1. Two things, kept separate

**The interaction design is inherited. The visual design is ours.**

This distinction is the whole point and it was got wrong twice before landing:

- First the UI ignored the legacy entirely and produced a generic dashboard.
- Then it copied the legacy's *visual* design outright — the dark card on black, the 26px
  radius, the exact fills — which is not what "use it as reference" meant either.

What carries over is the **UX**: the flows, thresholds, affordances and failure handling in
[`LEGACY_SYSTEM.md`](LEGACY_SYSTEM.md) §19 and §9. Those were earned over months of real
use and are not to be second-guessed — the 450 ms autosave debounce, the six-cell code that
submits itself, the 30-second resend, whole rows as click targets, errors on the field
shell, loading states that name the action.

What does **not** carry over is the styling. The OMS has its own visual language, built to
be better, using the brand's own colour and type as raw material rather than copying how
the legacy arranged them.

## 2. Where the brand colours come from

These values were recovered from the live the product stack, not invented:

| Token | Legacy source | Value |
|---|---|---|
| **RS Indigo** | `--rs-brand` | `#241A8F` → `oklch(33.3% 0.179 274.4)` |
| **RS Cyan** | `--accent` | `#58D6FF` → `oklch(82.2% 0.124 223)` |
| Dark ground | page background | `#050914` |
| Dark card | panel background | `#0D162B` |
| Typeface | `font-family` | **Inter** |

Everything else in the legacy CSS was default Tailwind slate (`#0f172a`, `#64748b`,
`#94a3b8`…). That default palette is a large part of why generated interfaces all look
alike, and we do not carry it forward. The OMS neutral ramp is custom, tinted toward the
brand's hue family (~265) at very low chroma.

### How the OMS uses them differently

| Legacy | OMS |
|---|---|
| A centred card floating on black | An editorial split: a deep indigo field against a quiet form column |
| Heavy borders and fills on every control | Rules and baselines; the content is the object, the line is a guide |
| Boxed input cells for the code | Six numerals on rules, with a caret and a lift on fill |
| Two competing tab boxes | One segmented control with a sliding indicator |
| Decorative nothing | A jersey-numeral motif at ~4.5% opacity — texture drawn from the product |

Same palette, same typeface, same interaction. Different composition.

---

## 2. Colour

### Roles, not shades

Never reach for a colour because it looks nice. Reach for the **role**:

| Token | Use for |
|---|---|
| `background` / `foreground` | The page and its default text |
| `card` | Any raised surface: panels, dialogs, popovers |
| `primary` | The single most important action on a screen. **One per view.** |
| `secondary` | Supporting actions |
| `muted` / `muted-foreground` | Table zebra, labels, secondary text, disabled states |
| `accent` | Highlight and focus only — never a large fill |
| `border` / `input` / `ring` | Structure and focus |
| `success` `warning` `destructive` | **State only.** Never decoration. |

### The two brand colours behave differently

**RS Indigo is the action colour.** At `oklch(33.3%)` it clears WCAG AA on white
comfortably, so it is safe for buttons, links and active navigation on light surfaces.

**RS Cyan is a highlight, not a text colour.** At `oklch(82.2%)` it fails contrast against
white — using it for body text or light-mode links is an accessibility bug. It belongs on
dark surfaces, focus rings, and small emphasis marks.

In dark mode the indigo lifts to `oklch(64% 0.17 272)`. Note it **keeps the same hue** —
brand identity is the hue, not the lightness. Never swap to a different hue for contrast.

### Semantic colour is information

A green badge means something succeeded. A red one means something failed. If colour is not
carrying information, it should not be there. Colour is also never the *only* signal —
pair it with an icon or text, because a meaningful share of users cannot distinguish red
from green.

---

## 3. Typography

**Inter**, with system fallbacks. One family across the whole system.

| Role | Size | Weight |
|---|---|---|
| Page title | `text-lg` | 600 |
| Section / card title | `text-xs` uppercase, tracked | 500 |
| Body & table cells | `text-sm` | 400 |
| Secondary / labels | `text-sm` muted | 400 |
| Metadata, code, IDs | `text-xs` mono | 400 |

`font-variant-numeric: tabular-nums` is set globally so quantities and money align in
columns. This matters constantly in an OMS — `Sewing 12/14` sits in a table next to a
hundred other numbers.

**No oversized headings.** This is operational software. If a heading is larger than
`text-lg`, something has gone wrong.

---

## 4. Space, density and shape

**Density is operational, not marketing.** Reference points: Linear, Stripe dashboard,
GitHub settings. A person uses this for hours to move real orders through real factories.

- Default control height **32px** (`h-8`).
- Card padding `px-4 py-3`. Table cells `px-4 py-2.5`.
- Spacing steps: `2 · 3 · 4 · 6 · 8`. Nothing else.

**One radius.** `--radius: 0.375rem`, with `sm` and `lg` derived from it. Five different
border radii in one interface is a reliable tell of generated design.

### Elevation

**Borders do the work. Shadows generally do not exist.** A `1px` border against a slightly
different surface is enough separation. Reserve shadow for genuinely floating layers —
dropdowns, dialogs, popovers — and keep it subtle.

---

## 5. Components

**Use shadcn/ui. Do not build primitives.**

`theme.css` deliberately uses shadcn's exact token names (`--background`, `--primary`,
`--muted-foreground`, `--ring`, …). This is the whole trick: **any shadcn component
dropped into this repository inherits the the product look with no restyling.** Adding a
dialog, a combobox or a data table costs one CLI command, and it will match.

| Need | Use |
|---|---|
| Tables, sorting, filtering | **TanStack Table** + shadcn table |
| Icons | **Lucide**, one family, `size-4` default |
| Forms & validation | shadcn form + **Zod** (the same schemas the API validates with) |
| Dialogs, menus, tooltips | shadcn (Radix underneath — accessibility included) |
| Charts, if ever needed | one charting library, chosen once |

Do **not** hand-build a modal system, toast system, dropdown, tooltip, form framework or
grid. `docs/instructions.md` §14 forbids it, and it is the fastest route to four
surfaces that look like four products.

---

## 6. Dark mode

Both themes are first-class — vendors update production from phones in poorly lit
warehouses; staff work long sessions.

Implemented as a `.dark` class on `<html>`, following the OS by default. Every token has a
dark value. **If you write a colour that only works in one theme, it is a bug.**

---

## 7. What this system does not look like

From `docs/instructions.md` §5 and §37, and enforced in review:

- ❌ Gradient hero sections, or gradients used because they look "modern"
- ❌ Glassmorphism, blur, floating blobs, decorative SVG waves
- ❌ Everything wrapped in a card; cards nested inside cards
- ❌ An icon on every button and every section heading
- ❌ Fabricated metrics or fake avatars to make a screen look populated
- ❌ Five border radii, excessive shadows, pill-shaped everything
- ❌ Oversized headings with vague marketing copy
- ❌ Hand-drawn SVG illustrations

**Show real data or show nothing.** An empty state that says "No vendors yet" is honest and
useful. A dashboard of invented numbers is neither.

---

## 8. Writing

Interface copy is plain and specific.

- Buttons say what happens: **Create vendor**, not *Submit*.
- Errors say what went wrong and what to do next.
- Empty states explain what would appear and how to get one.
- No exclamation marks. No "Oops!". No congratulating the user for routine work.
- Vendor-facing copy avoids internal jargon — vendors are not staff and never see internal
  pricing.

---

## 9. Accessibility

Non-negotiable, and mostly free because Radix does the hard part:

- Text contrast **≥ 4.5:1**; UI boundaries ≥ 3:1. This is why cyan is not a light-mode text colour.
- Every interactive element is keyboard reachable, with the single global focus ring.
- Icon-only buttons carry an accessible label.
- Colour is never the only carrier of meaning.
- Respect `prefers-reduced-motion`. Motion is functional — it shows where something came
  from — never ornamental.

---

## 10. Adding a screen: the checklist

- [ ] Every colour comes from a token. No hex values in the component.
- [ ] Components come from `src/client/components/ui/`, or were added via the shadcn CLI.
- [ ] Icons are Lucide, `size-4` unless there is a reason.
- [ ] Spacing uses the scale. Density matches the rest of the system.
- [ ] Works in light **and** dark.
- [ ] Keyboard navigable; focus visible.
- [ ] Real data or an honest empty state.
- [ ] Nothing from §7.

## Full width — no side margins
Operational apps fill the available width. Do NOT wrap page content in a centered max-width container
(`max-w-* mx-auto`) — it leaves empty left/right margins that read as a marketing site and is a common
generated-design tell. The page's own padding is the only gutter; the sidebar is the left edge, the
viewport the right. If one surface (a new portal) adds a `max-w-[Npx]` wrapper while the rest of the
app is full-bleed, the two stop reading as one product. Component-internal max-widths (a chat bubble at
`max-w-[85%]`, a search box) are fine — this is about the page container.
