# The React UI library ecosystem, in layers

Companion to [framework-selection.md](framework-selection.md) (which picks one library per
capability for a project) and [generating-ui.md](generating-ui.md) (which is about getting a
design that doesn't look generated). This note is the map underneath both: what the
libraries actually are, which layer each one occupies, and how to combine them without
producing AI slop.

> **Provenance.** Unlike the platform notes, this is an ecosystem survey rather than a list
> of scars — the layer model and the anti-slop rules are ours and have held up; the
> per-library judgements are a snapshot of the 2026 landscape and should be re-checked
> against each project's docs and release activity before you commit. Treat the ratings as
> direction, not as verified fact.

---

## 1. The central idea: you are not picking *one* library

The most common mistake is flattening the ecosystem into a single ranking and picking the
winner. These libraries do not compete — most of them stack. A production UI is a pipeline:

```
        behaviour  →  design system  →  tokens  →  styling  →  motion  →  effects
```

Which maps to five layers:

| Layer | What it provides | Examples |
|---|---|---|
| **1 — Behavioural foundation** | Accessibility, keyboard, focus management, composability. No visuals. | Radix UI, Base UI, Ark UI, React Aria, Headless UI, Ariakit |
| **2 — Design system / component source** | Styled components you own the source of; tokens; breadth. | shadcn/ui, Park UI, Origin UI, ReUI, Kibo UI |
| **3 — Motion & visual enhancement** | Transitions, micro-interaction, spectacle. | Motion (Framer), Motion Primitives, Magic UI, Aceternity UI, Cult UI |
| **4 — Discovery / registries** | Somewhere to find a block and adapt it. | 21st.dev, shadcn registries, block libraries |
| **5 — Complete application systems** | Everything included, one opinion, one theme model. | MUI, Mantine, Chakra, HeroUI, Ant Design |

**Layer 5 is an alternative to layers 1–3, not an addition to them.** Everything else
composes. The decision that matters is: do you own your design system (1+2+3) or do you
adopt someone else's (5)?

```
                         UI ECOSYSTEM
                              │
              ┌───────────────┴────────────────┐
        FOUNDATION                        VISUAL LAYER
              │                                │
       ┌──────┼──────┐                  ┌──────┼──────────┐
     Radix  Base    Ark               Magic  Aceternity  Motion
             UI     UI                 UI                Primitives
       └──────┼──────┘
       HEADLESS PRIMITIVES
              ▼
         shadcn/ui  ──────►  Park UI · ReUI · Origin UI
              ▼
        APPLICATION UI  ──►  YOUR DESIGN SYSTEM
```

---

## 2. Layer 1 — behavioural foundations

Judge these on accessibility, keyboard/focus behaviour, composability, coverage, styling
freedom, animation compatibility, RSC compatibility, and maintenance activity. Not on looks
— they have none, and that is the point.

| Library | React | Other frameworks | Reach for it when | Watch for |
|---|---|---|---|---|
| **Radix UI** | ✅ | — | The proven default; the largest body of prior art and the base most Layer-2 systems assume | Component set is stable rather than growing |
| **Base UI** | ✅ | — | New high-end design system; from people behind Radix, MUI and Floating UI; unstyled, pairs with Tailwind + Motion | Younger; check coverage for the specific primitives you need |
| **Ark UI** | ✅ | Vue, Solid | You want Radix philosophy without React lock-in; state-machine driven | Smaller React-specific ecosystem than Radix |
| **React Aria** | ✅ | — | Accessibility is the product requirement: editors, Figma-likes, keyboard-first enterprise tools | Behaviour toolkit, not a component library — more assembly |
| **Headless UI** | ✅ | Vue | Tailwind-centric app that needs a few accessible primitives and nothing more | Deliberately small: less coverage and composability |
| **Ariakit** | ✅ | — | Complex accessible widgets outside the common set | Less mainstream; more of your own patterns to maintain |

**Default:** Radix if you want the largest ecosystem, Base UI if you are starting fresh and
want the modern successor, Ark UI if any non-React surface is plausible, React Aria if
accessibility is a contractual requirement rather than a goal.

## 3. Layer 2 — copy/paste systems (you own the source)

The important idea behind shadcn/ui is not the visual style — it is the distribution model.
You do not install a black box; the component source lands in your repo and becomes yours.
That is what makes a starting point turn into a design system instead of a ceiling.

| System | Built on | Best for | Weakness |
|---|---|---|---|
| **shadcn/ui** | Radix + Tailwind | The default starting point; source ownership, CSS-variable theming | Ubiquitous defaults — ship it unchanged and it reads as generated |
| **Park UI** | Ark UI | A real design-system layer: opinionated tokens and visual language over primitives | More opinion to unlearn if you disagree with it |
| **ReUI** | Radix *or* Base UI variants | Large SaaS: very broad registry (1,000+ components/effects claimed), animation included | Breadth over coherence — curate, don't import wholesale |
| **Origin UI** | Radix/Tailwind | Settings, forms, tables, filters, admin — restrained "Linear × Vercel × Stripe" feel | Not where you go for spectacle |
| **Kibo UI** | Radix/Tailwind | Application-level components for data-heavy products, AI platforms, dev tools | Newer, smaller community |

Park UI is the interesting one architecturally: `Ark UI → Park UI → your tokens → your app`
turns the question from "which pretty component do I copy?" into "what is the foundation of
my product's design system?"

## 4. Layer 3 — motion and visual effect

| Library | Character | Use it for | Do not use it as |
|---|---|---|---|
| **Motion Primitives** | Restrained, purposeful | Page/route transitions, dialogs, list and layout transitions, hover states | — add this to most stacks |
| **Magic UI** | "This product feels expensive" | AI products, SaaS marketing, pricing, animated dashboards | Your primitive layer |
| **Aceternity UI** | Maximum spectacle: glow, 3D, spotlight, scroll effects | Landing and hero sections, launch pages | The foundation of a complex SaaS app |
| **Cult UI** | Experimental, dark-mode, AI/dev-tool aesthetic | Inspiration and selected components | An accessibility foundation |

**The governing rule:** *don't make every component animated; make the important
transitions excellent.* Motion belongs on state changes the user needs to follow — a row
appearing, a panel opening, a route changing. Everything else is noise with a frame budget.

## 5. Layers 4 and 5

**Registries (21st.dev, shadcn registries, block libraries)** are for discovery, not
dependency. Useful to see what a pattern looks like when someone has already solved it;
adapt what you take into your own primitives rather than accumulating strangers' components.

**Complete systems** solve a different problem — they trade visual differentiation for
speed and completeness:

| System | Choose it when |
|---|---|
| **Mantine** | You need to ship a serious app: hooks, forms, modals, notifications, dates, tables, app utilities |
| **Chakra** | Developer ergonomics and a predictable theme model matter more than the current aesthetic |
| **HeroUI** | You want behaviour *and* a visual system out of the box, without weeks on primitives |
| **MUI** | Enterprise: complex forms, data grids, admin, standardisation, huge ecosystem |
| **Ant Design** | Same territory as MUI, different opinion; heavy visual identity |

If the goal is "this should not look like everything else", none of these is the starting
point — their whole value is that they already look like themselves.

---

## 6. What "futuristic" means, and what it doesn't

This is the part that prevents AI slop. The word "futuristic" is where briefs go wrong,
because two very different things answer to it.

**It means:** sophisticated motion; responsive micro-interactions; fluid transitions;
layered surfaces and real depth; subtle gradients; tasteful glow; a properly designed dark
mode; interactive data visualisation; command interfaces (⌘K); progressive disclosure;
intelligent empty states; rich hover and focus states; keyboard-first workflows; excellent
loading and skeleton states that match the real structure; optimistic UI; animated layout
changes.

**It does not mean:** gradients everywhere; unnecessary 3D; animation on every element;
poor contrast in service of a look; distracting effects; inaccessible interactions;
decorative motion with no UX purpose.

**A visually impressive component is not automatically good UX.** Keep the two axes
separate when evaluating anything from Layer 3 — score *visual spectacle* and *useful
interaction design* independently, and require `prefers-reduced-motion` support before
anything ships.

The slop failure mode in practice is: default shadcn tokens + a hero gradient + animation on
everything + no empty states + no focus rings. Every one of those is a decision not made.

---

## 7. Generation is use-case aware — name the surface first

The single biggest determinant of whether generated UI looks right is not the library or
the prompt wording. It is **which kind of surface you are generating**. A landing page and
an operations console are opposite products: what makes one excellent makes the other
unusable. An agent asked for "a modern UI" with no archetype named will default to the
landing-page aesthetic — big hero, huge type, gradient, generous whitespace, motion — and
apply it to a dashboard. That is the most common single source of slop.

**Name the archetype in the first line of the brief**, then inherit its column below.

| Surface | Optimise for | Density | Motion | Layer-3 effects | Signature components | Slop signal |
|---|---|---|---|---|---|---|
| **Landing / marketing** | Persuasion, first paint, SEO | Low — whitespace is the design | Scroll-linked, generous | **Yes** — this is where Aceternity/Magic UI belong | Hero, feature grid, logos, testimonial, pricing, FAQ, CTA | Generic hero + gradient blob + no real copy |
| **Pricing / conversion** | Comparison clarity, one obvious action | Medium | Minimal — plan toggle only | Restrained | Plan cards, feature matrix, toggle, FAQ | Three identical cards with a fake "most popular" badge |
| **Dashboard / analytics** | Scanning, comparison, drill-down | **High** | Only on data change | Almost none | KPI row, chart grid, filter bar, date range, drill-down table | Four meaningless KPI cards; decorative sparklines; invented numbers |
| **Operational console / admin** | Throughput on repeated tasks | **Highest** | Transitions only | None | Dense table, bulk select, row actions, detail drawer, saved views, keyboard nav | Card grids where a table belongs; pagination without sort/filter |
| **E-commerce — browse** | Findability, comparison, trust | Medium | Hover/quick-view only | Light | Product grid, faceted filters, sort, badges, quick view, wishlist | Filters that don't reflect stock; identical placeholder products |
| **E-commerce — product** | Confidence to buy | Medium | Gallery only | Light | Gallery, variant selector, stock/delivery, price + promo, reviews, sticky add-to-cart | Variant picker with no availability or price-delta feedback |
| **Checkout** | Completion; nothing else | Low | **None** | **None** | Step indicator, address/payment forms, inline validation, order summary, error states | Animation, upsells, or anything that can lose entered data |
| **Data entry / forms** | Correctness, recoverability | Medium | None | None | Sectioned form, inline validation, save/discard, unsaved-changes guard, autosave state | Validation only on submit; no dirty tracking (see ux-patterns.md) |
| **Settings / account** | Findability, safe destructive actions | Medium | None | None | Nav sections, toggles, confirm-by-typing, audit trail | One giant page; delete with a plain "Are you sure?" |
| **Docs / content** | Reading, navigation, search | Low | None | None | Sidebar tree, TOC, code blocks with copy, search, version switcher | Marketing polish applied to reference material |
| **Chat / AI product** | Legibility of an unfolding response | Medium | **Streaming matters** | Selective | Message list, streaming/token state, citations, tool-call disclosure, stop/regenerate, empty state with prompts | A blank box with no affordances and no visible model state |
| **Onboarding / wizard** | One decision per screen, visible progress | Low | Step transitions | Light | Progress, one-question steps, back without loss, skip, summary | A five-field form pretending to be a wizard |
| **Auth** | Trust and speed | Low | None | None | Provider buttons, OTP input, error states, rate-limit messaging | Ornamented sign-in; OTP without paste/autofill handling |

Three rules fall out of this table:

- **Density and motion trade off against each other.** The denser the surface, the less
  motion it tolerates. A console gets transitions; a landing page gets scroll effects.
- **Layer 3 belongs to marketing surfaces.** Aceternity/Magic UI on a checkout or a console
  is a defect, not a flourish. Product surfaces get Motion Primitives at most.
- **Every archetype has its own required states**, and this is where generated UI is
  usually thinnest: empty, loading (skeletons matching real structure), error, partial
  permission, and — for commerce — out-of-stock. Ask for them explicitly or you will not
  get them.

**In the brief**, this means the first line is *"This is a [archetype] for [product]"*, the
functional requirements come from that row's signature components, the demo data is
realistic for that domain (see generating-ui.md §2), and the stack line inherits the motion
and effects policy above. Generate variants **within one archetype** — comparing a landing
page against a dashboard tells you nothing.

## 8. Stacks that work

| Product | Stack |
|---|---|
| **AI SaaS** | Base UI or Ark UI + shadcn + Magic UI + selected Aceternity + Motion Primitives |
| **Developer tool** | Base UI + Park UI + ReUI + Motion Primitives — information density over effects |
| **Premium SaaS** | Base UI + Park UI + Origin UI + Magic UI |
| **Futuristic landing page** | shadcn + Aceternity + Magic UI + Motion Primitives |
| **Enterprise / operational** | Base UI or Radix + your own design system, with MUI or Mantine where they genuinely save time |
| **Operational console (ours)** | Radix via shadcn/ui + Tailwind tokens + CSS transitions first — see framework-selection.md §3 |

A defensible new-product default:

```
Recommended stack
─────────────────
Foundation:        Base UI (or Radix, for ecosystem weight)
Design system:     Park UI or shadcn/ui — source you own
Styling:           Tailwind + CSS variables (one token layer)
Motion:            Motion + Motion Primitives
Visual effects:    Magic UI / Aceternity — landing surfaces only
Components:        Origin UI / ReUI, curated not bulk-imported
Icons:             one family, everywhere (Lucide)
Charts:            Recharts or visx
Data tables:       TanStack Table
Forms:             React Hook Form + Zod
Command interface: cmdk
```

That gives a stable UX foundation while letting the product look like nothing else built on
the same primitives.

---

## 9. Re-running this evaluation

When you next need to choose, don't ask for a ranked list of component libraries — that is
what produces a flat, useless answer. Ask for the ecosystem **in layers**, and for each
candidate require:

1. **What problem does it actually solve, and which layer is it in?** Foundation or
   enhancement?
2. **Does it own or vendor the component source?** What does it depend on — Radix, Base UI,
   Ark UI, something else?
3. **Can it coexist with the others, and be adopted selectively?**
4. **Accessibility of the resulting components** — keyboard, focus, screen reader — scored
   separately from looks.
5. **Styling freedom and how much visual opinion it imposes.**
6. **Animation compatibility and reduced-motion support.**
7. **SSR / server-component compatibility.**
8. **Bundle and runtime cost; tree-shaking; CSS approach.**
9. **What happens at 100+ components?** Maintenance load is the real long-run cost.
10. **Maintenance activity and ecosystem size**, from repos and releases — not marketing copy.

Score with qualitative bands (excellent / very good / good / moderate / weak). Refuse
invented precise numbers. Keep verified fact, community sentiment and recommendation
visibly separate, and say plainly where the information may have gone stale.
